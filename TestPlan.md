# ListHouze Billing & Entitlements Test Plan

**Version:** 1.0
**Date:** 2025-12-26
**Author:** Senior Full-Stack Engineer + Supabase/Stripe Specialist

---

## Overview

This document provides comprehensive testing procedures for the Stripe billing integration, entitlements enforcement, and system reliability features implemented for ListHouze.

---

## Pre-Test Setup

### Required Environment Variables

Ensure the following secrets are configured in Supabase Edge Functions settings:

```bash
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_SERVICE_ROLE_KEY=eyJ...
SITE_URL=https://listhouze.com
```

### Stripe Dashboard Configuration

1. Log into Stripe Dashboard
2. Go to **Developers → Webhooks**
3. Add endpoint: `https://your-project.supabase.co/functions/v1/billing-webhook`
4. Select events (see Webhook Setup section below)

### Database Migration

Apply the entitlement enforcement migration:

```bash
# Ensure migration is applied
supabase db push

# Or manually apply
psql -f supabase/migrations/20251226124513_entitlement_enforcement.sql
```

---

## Test Suite

### 1. Stripe Environment Validation

**Objective:** Verify environment secrets are validated on Edge Function invocation

#### Test 1.1: Missing STRIPE_SECRET_KEY

**Steps:**
1. Temporarily remove `STRIPE_SECRET_KEY` from Supabase secrets
2. Call `/billing-start-checkout` Edge Function
3. Verify response is HTTP 500 with error:
   ```json
   {
     "error": "Configuration error",
     "message": "Missing required environment variable: STRIPE_SECRET_KEY",
     "hint": "Please ensure all required Stripe secrets are configured in Supabase Edge Function settings"
   }
   ```

**Expected Result:** ✅ Function fails loudly with clear error message

#### Test 1.2: Missing STRIPE_WEBHOOK_SECRET

**Steps:**
1. Temporarily remove `STRIPE_WEBHOOK_SECRET` from Supabase secrets
2. Trigger a webhook event from Stripe (or use Stripe CLI)
3. Verify webhook returns HTTP 500 with error about missing secret

**Expected Result:** ✅ Webhook fails and refuses to process events without signature verification

#### Test 1.3: All Secrets Present

**Steps:**
1. Ensure all secrets are configured
2. Call `/billing-start-checkout` with valid auth token
3. Verify checkout session is created successfully

**Expected Result:** ✅ Function executes normally

---

### 2. Stripe Webhook Event Coverage

**Objective:** Verify all 6 required webhook events are handled correctly

#### Test 2.1: checkout.session.completed (Subscription)

**Steps:**
1. Start a subscription checkout (e.g., `individual_renewal` plan)
2. Complete payment in Stripe test mode
3. Check `subscriptions` table for new row:
   - `subject_type` = 'user'
   - `subject_id` = user's UUID
   - `status` = 'active'
   - `stripe_subscription_id` populated
   - `stripe_customer_id` populated

**Expected Result:** ✅ Subscription record created in DB

#### Test 2.2: checkout.session.completed (One-Time Payment)

**Steps:**
1. Start checkout for one-time purchase (e.g., $99 sale listing)
2. Complete payment in Stripe test mode
3. Check `one_time_purchases` table for new row:
   - `user_id` populated
   - `purchase_type` = 'listing_add_on'
   - `status` = 'active'
   - `stripe_payment_intent_id` populated

**Expected Result:** ✅ Purchase record created in DB

#### Test 2.3: customer.subscription.created

**Steps:**
1. Use Stripe CLI to trigger event:
   ```bash
   stripe trigger customer.subscription.created
   ```
2. Ensure subscription metadata includes `user_id` and `plan_type`
3. Check `subscriptions` table for record

**Expected Result:** ✅ Subscription created (similar to checkout.session.completed)

#### Test 2.4: customer.subscription.updated

**Steps:**
1. Update an existing subscription in Stripe (e.g., change status to `past_due`)
2. Webhook should update subscription status in DB
3. Verify `subscriptions.status` matches Stripe status

**Expected Result:** ✅ Subscription status synced correctly

#### Test 2.5: customer.subscription.deleted

**Steps:**
1. Cancel a subscription in Stripe Dashboard
2. Webhook triggers and updates DB
3. Verify `subscriptions.status` = 'canceled'
4. Verify `canceled_at` timestamp is set

**Expected Result:** ✅ Subscription marked as canceled

#### Test 2.6: invoice.paid

**Steps:**
1. Trigger successful invoice payment (happens automatically on subscription renewal)
2. Webhook updates subscription period dates
3. Verify `current_period_start` and `current_period_end` match invoice periods

**Expected Result:** ✅ Subscription period updated

**Note:** Webhook also handles `invoice.payment_succeeded` as an alias for backwards compatibility.

#### Test 2.7: invoice.payment_failed

**Steps:**
1. Use Stripe test card that triggers payment failure (e.g., `4000 0000 0000 0341`)
2. Invoice payment fails
3. Webhook updates `subscriptions.status` = 'past_due'

**Expected Result:** ✅ Subscription marked as past_due

---

### 3. Customer Portal

**Objective:** Verify users can access Stripe Customer Portal to manage subscriptions

#### Test 3.1: Open Customer Portal

**Steps:**
1. Log in as a user with an active subscription
2. Navigate to `/settings/billing`
3. Click "Manage Subscription" button
4. Verify redirect to Stripe Customer Portal
5. Verify portal shows:
   - Current subscription details
   - Cancel subscription option
   - Update payment method option

**Expected Result:** ✅ Portal opens successfully and displays subscription

#### Test 3.2: Cancel Subscription via Portal

**Steps:**
1. In Customer Portal, cancel subscription
2. Confirm cancellation
3. Return to `/settings/billing`
4. Verify subscription shows as canceled

**Expected Result:** ✅ Cancellation reflected in app

#### Test 3.3: Manual Billing (No Portal Access)

**Steps:**
1. Create a manual subscription (set `is_manual` = true in DB)
2. Navigate to `/settings/billing`
3. Verify "Manage Subscription" button is hidden

**Expected Result:** ✅ Portal button hidden for manual subscriptions

---

### 4. Database Entitlement Enforcement

**Objective:** Verify triggers block unauthorized publish/renew operations at DB level

#### Test 4.1: Individual - First Sale Listing (FREE)

**Steps:**
1. Log in as individual user with 0 active sale listings
2. Create new property (listing_category = 'sale', status = 'draft')
3. Update status to 'active'
4. Verify update succeeds without error

**Expected Result:** ✅ First sale listing published for free

**DB Check:**
```sql
SELECT expires_at FROM properties WHERE id = '[listing_id]';
-- Should be ~2 months from now
```

#### Test 4.2: Individual - Second Sale Listing (REQUIRES PAYMENT)

**Steps:**
1. User already has 1 active sale listing
2. Create another sale listing (draft)
3. Attempt to update status to 'active' WITHOUT payment
4. Verify update is BLOCKED by trigger with error:
   ```
   Payment required for additional sale listings
   HINT: One-time payment of $99 required
   ```

**Expected Result:** ✅ Trigger blocks publish, raises exception

#### Test 4.3: Individual - Second Sale Listing WITH Payment

**Steps:**
1. Complete $99 one-time payment for the specific listing
2. Record created in `one_time_purchases` with `listing_id`
3. Retry update status to 'active'
4. Verify update SUCCEEDS

**Expected Result:** ✅ Publish allowed after payment

#### Test 4.4: Individual - Rental Listings (Up to 3 FREE)

**Steps:**
1. User has 0 active rental listings
2. Create and publish 3 rental properties
3. Verify all 3 publish successfully without payment

**Expected Result:** ✅ Up to 3 rentals allowed for free

#### Test 4.5: Individual - 5th Rental (REQUIRES LANDLORD PRO)

**Steps:**
1. User has 4 active rental listings
2. Create 5th rental listing
3. Attempt to publish WITHOUT Landlord Pro subscription
4. Verify blocked with error:
   ```
   Landlord Pro subscription required for 5+ rental listings
   HINT: Subscribe to Landlord Pro for $29/month
   ```

**Expected Result:** ✅ Trigger blocks publish

#### Test 4.6: Individual - 5th Rental WITH Landlord Pro

**Steps:**
1. Subscribe to `landlord_pro` plan
2. Retry publishing 5th rental listing
3. Verify update SUCCEEDS

**Expected Result:** ✅ Publish allowed with subscription

#### Test 4.7: Agency - No Subscription

**Steps:**
1. Create organization
2. Create property with `organization_id` set
3. Attempt to publish (status = 'active')
4. Verify blocked with error:
   ```
   Agency subscription required to publish listings
   HINT: Subscribe to Agency Starter or Agency Pro plan
   ```

**Expected Result:** ✅ Trigger blocks agency publish without subscription

#### Test 4.8: Agency - Within Listing Cap

**Steps:**
1. Subscribe to `agency_starter` (cap = 100 listings)
2. User has 50 active listings
3. Publish new listing
4. Verify update SUCCEEDS

**Expected Result:** ✅ Publish allowed within cap

#### Test 4.9: Agency - Listing Cap Reached

**Steps:**
1. Organization has 100 active listings (cap reached)
2. Attempt to publish 101st listing
3. Verify blocked with error:
   ```
   Listing cap reached (100 / 100)
   HINT: Upgrade your plan for more listings
   ```

**Expected Result:** ✅ Trigger blocks publish at cap

#### Test 4.10: Renewal - Sale Listing After Trial

**Steps:**
1. Create sale listing with `expires_at` in the past
2. Attempt to extend `expires_at` (renew) WITHOUT renewal subscription
3. Verify blocked with error:
   ```
   Renewal subscription required to keep listing active
   HINT: Subscribe to Individual Renewal for $15/month
   ```

**Expected Result:** ✅ Trigger blocks renewal without subscription

#### Test 4.11: Renewal - WITH Subscription

**Steps:**
1. Subscribe to `individual_renewal` plan
2. Retry extending `expires_at`
3. Verify update SUCCEEDS

**Expected Result:** ✅ Renewal allowed with subscription

---

### 5. Direct Database Bypass Test

**Objective:** Verify entitlement enforcement even when UI is bypassed

#### Test 5.1: Direct Supabase Client Update

**Steps:**
1. Use Supabase client directly (e.g., from browser console):
   ```javascript
   const { data, error } = await supabase
     .from('properties')
     .update({ status: 'active' })
     .eq('id', 'some-listing-id-without-entitlement')
   ```
2. Verify update is BLOCKED by trigger
3. Error should be in `error.message`

**Expected Result:** ✅ Trigger enforces entitlement even for direct DB calls

---

### 6. Error Boundary

**Objective:** Verify React Error Boundary catches runtime errors

#### Test 6.1: Trigger Runtime Error

**Steps:**
1. Temporarily introduce an error (e.g., undefined variable in a component)
2. Navigate to the page
3. Verify Error Boundary displays:
   - Friendly error message
   - "Reload Page" button
   - "Report Issue" button (opens mailto:support@listhouze.com)

**Expected Result:** ✅ Error Boundary catches error and shows fallback UI

#### Test 6.2: Reload After Error

**Steps:**
1. Click "Reload Page" button
2. Verify page reloads successfully

**Expected Result:** ✅ Page reloads

#### Test 6.3: Report Issue

**Steps:**
1. Click "Report Issue" button
2. Verify mailto link opens with pre-filled error details

**Expected Result:** ✅ Email client opens with error report

---

### 7. Toast Notification Consistency

**Objective:** Verify unified toast system works correctly

#### Test 7.1: Success Notification

**Steps:**
1. Trigger success action (e.g., save profile)
2. Use `notify.success('Profile saved')` in code
3. Verify Sonner toast appears with success styling

**Expected Result:** ✅ Success toast displayed

#### Test 7.2: Error Notification

**Steps:**
1. Trigger error (e.g., DB constraint violation)
2. Use `notifyDatabaseError(error, 'Failed to publish')` in code
3. Verify toast shows user-friendly error message

**Expected Result:** ✅ Error toast displayed with formatted message

#### Test 7.3: Database Error Formatting

**Steps:**
1. Trigger various DB errors:
   - Entitlement trigger error
   - Unique constraint violation (23505)
   - Foreign key violation (23503)
   - Permission error (42501)
2. Verify `formatDatabaseError()` converts to user-friendly messages

**Expected Result:** ✅ All errors formatted correctly

---

### 8. Expired Listing Cron Job

**Objective:** Verify cron job archives expired listings automatically

#### Test 8.1: Manual Cron Invocation

**Steps:**
1. Create a test listing with:
   - `status` = 'active'
   - `expires_at` = yesterday's date
2. Manually invoke Edge Function:
   ```bash
   curl -X POST https://your-project.supabase.co/functions/v1/check-expired-listings \
     -H "Authorization: Bearer SUPABASE_ANON_KEY"
   ```
3. Verify response:
   ```json
   {
     "success": true,
     "message": "Archived 1 expired listing(s)",
     "archived_count": 1,
     "archived_listings": [...]
   }
   ```
4. Verify listing status changed to 'archived'

**Expected Result:** ✅ Expired listing archived

#### Test 8.2: No Expired Listings

**Steps:**
1. Ensure no listings have expired
2. Invoke cron function
3. Verify response:
   ```json
   {
     "success": true,
     "message": "No expired listings found",
     "archived_count": 0
   }
   ```

**Expected Result:** ✅ Function completes with no errors

#### Test 8.3: Schedule Cron (Supabase Dashboard)

**Steps:**
1. Go to Supabase Dashboard → Edge Functions → Cron
2. Create new cron job:
   - Function: `check-expired-listings`
   - Schedule: `0 0 * * *` (daily at midnight UTC)
3. Wait for next execution or trigger manually
4. Verify cron runs successfully

**Expected Result:** ✅ Cron scheduled and runs automatically

---

## Test Execution Checklist

Use this checklist to track test execution:

### Phase 1: Stripe
- [ ] 1.1: Missing STRIPE_SECRET_KEY
- [ ] 1.2: Missing STRIPE_WEBHOOK_SECRET
- [ ] 1.3: All Secrets Present
- [ ] 2.1: checkout.session.completed (Subscription)
- [ ] 2.2: checkout.session.completed (One-Time)
- [ ] 2.3: customer.subscription.created
- [ ] 2.4: customer.subscription.updated
- [ ] 2.5: customer.subscription.deleted
- [ ] 2.6: invoice.paid
- [ ] 2.7: invoice.payment_failed
- [ ] 3.1: Open Customer Portal
- [ ] 3.2: Cancel Subscription via Portal
- [ ] 3.3: Manual Billing (No Portal Access)

### Phase 2: Entitlements
- [ ] 4.1: Individual - First Sale Listing (FREE)
- [ ] 4.2: Individual - Second Sale Listing (BLOCKED)
- [ ] 4.3: Individual - Second Sale Listing WITH Payment
- [ ] 4.4: Individual - Rental Listings (Up to 3 FREE)
- [ ] 4.5: Individual - 5th Rental (BLOCKED)
- [ ] 4.6: Individual - 5th Rental WITH Landlord Pro
- [ ] 4.7: Agency - No Subscription (BLOCKED)
- [ ] 4.8: Agency - Within Listing Cap
- [ ] 4.9: Agency - Listing Cap Reached (BLOCKED)
- [ ] 4.10: Renewal - WITHOUT Subscription (BLOCKED)
- [ ] 4.11: Renewal - WITH Subscription
- [ ] 5.1: Direct Database Bypass Test

### Phase 3: Reliability
- [ ] 6.1: Trigger Runtime Error
- [ ] 6.2: Reload After Error
- [ ] 6.3: Report Issue
- [ ] 7.1: Success Notification
- [ ] 7.2: Error Notification
- [ ] 7.3: Database Error Formatting
- [ ] 8.1: Manual Cron Invocation
- [ ] 8.2: No Expired Listings
- [ ] 8.3: Schedule Cron

---

## Troubleshooting

### Webhook Not Receiving Events

1. Check Stripe webhook endpoint URL is correct
2. Verify `STRIPE_WEBHOOK_SECRET` matches Stripe Dashboard
3. Check Supabase Edge Function logs for errors
4. Use Stripe CLI to test locally:
   ```bash
   stripe listen --forward-to localhost:54321/functions/v1/billing-webhook
   ```

### Trigger Errors Not Appearing

1. Check migration was applied: `SELECT * FROM pg_trigger WHERE tgname = 'trg_properties_publish_guard';`
2. Verify functions exist: `SELECT * FROM pg_proc WHERE proname LIKE 'fn_can_%';`
3. Check RLS is not preventing trigger from running
4. Ensure user is authenticated (triggers skip for service role)

### Cron Not Running

1. Verify cron schedule is correct (cron syntax)
2. Check Edge Function logs for errors
3. Test manual invocation first
4. Ensure Supabase project is not paused

---

## Success Criteria

All tests must pass with the following outcomes:

- ✅ No Stripe operations succeed without proper secrets
- ✅ All 6 webhook events create/update DB records correctly
- ✅ Customer Portal opens and allows subscription management
- ✅ Database triggers block ALL unauthorized publish/renew operations
- ✅ Direct DB updates are also blocked by triggers
- ✅ Error Boundary catches and displays errors gracefully
- ✅ Toast notifications are consistent across the app
- ✅ Expired listings are automatically archived by cron job

---

## Notes

- Run tests in Stripe **test mode** to avoid real charges
- Use test card numbers: https://stripe.com/docs/testing
- Always test trigger enforcement with direct DB calls to ensure bypass prevention
- Verify logs in Supabase Dashboard → Edge Functions for detailed debugging

---

**End of Test Plan**
