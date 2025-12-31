# ListHouze Edge Functions & Integrations

**Version:** 1.0  
**Last Updated:** December 31, 2024

---

## Edge Functions Overview

ListHouze uses **26 Supabase Edge Functions** deployed via Lovable Cloud:

```
supabase/functions/
├── _shared/              # Shared utilities
│   └── env.ts           # Environment validation
├── abn-lookup/          # AU business number lookup
├── account-type-*       # Role request handling (3 functions)
├── agent-approval/      # Agent verification
├── ai-assistant/        # AI description generator
├── billing-*            # Stripe integration (5 functions)
├── check-expired-listings/ # Cron job
├── get-agent-contact/   # Rate-limited contact reveal
├── get-maps-key/        # Secure Maps key delivery
├── google-maps-proxy/   # Server-side Maps API
├── inbound-api/         # External webhook receiver
├── linz-property-lookup/ # NZ property data
├── nzbn-lookup/         # NZ business number
├── outbound-webhook/    # Event notifications
├── rea-scraper/         # Property data import
├── send-*               # Email functions (4 functions)
└── vaultre-*            # VaultRE CRM sync (2 functions)
```

---

## Function Details

### Billing Functions

#### `billing-start-checkout`
Creates Stripe checkout sessions for subscriptions and one-time purchases.

```typescript
// Request
POST /billing-start-checkout
Authorization: Bearer <jwt>
{
  "planType": "agency_starter",
  "orgId": "uuid",
  "successUrl": "https://...",
  "cancelUrl": "https://..."
}

// Response
{
  "success": true,
  "sessionId": "cs_...",
  "url": "https://checkout.stripe.com/..."
}
```

**Dependencies**: STRIPE_SECRET_KEY, SUPABASE_URL, SUPABASE_SERVICE_ROLE_KEY

#### `billing-webhook`
Handles Stripe webhook events for subscription lifecycle.

**Events Handled**:
- `checkout.session.completed` → Create subscription record
- `customer.subscription.updated` → Update status
- `customer.subscription.deleted` → Mark cancelled
- `invoice.payment_failed` → Update to past_due

**Required Secret**: STRIPE_WEBHOOK_SECRET

#### `billing-check-eligibility`
Checks if user can publish based on plan limits.

#### `billing-confirm-payment`
Confirms one-time payment and updates purchase record.

#### `billing-customer-portal`
Creates Stripe customer portal session for subscription management.

---

### Email Functions

#### `send-inquiry-email`
Routes inquiries to appropriate recipients with dynamic resolution.

```typescript
// Request
POST /send-inquiry-email
{
  "inquiryType": "property" | "agent" | "appraisal" | "mortgage",
  "contactName": "John Doe",
  "contactEmail": "john@example.com",
  "contactPhone": "+64...",
  "propertyId": "uuid",
  "message": "..."
}

// Recipient Resolution
1. If agentEmail provided → use directly
2. If propertyId → lookup property_agents → agent email
3. If propertyId → lookup property owner → profile email
4. Fallback → platform support email
```

**Dependencies**: RESEND_API_KEY

#### `send-agent-inquiry`
Simplified inquiry directly to agent.

#### `send-open-home-email`
Confirmation emails for open home registrations.

#### `send-password-reset`
Custom password reset flow.

#### `send-workspace-invite`
Team invite emails with secure tokens.

---

### External API Functions

#### `abn-lookup`
Validates Australian Business Numbers via ABR API.

```typescript
// Request
POST /abn-lookup
{ "abn": "12345678901" }

// Response
{
  "valid": true,
  "entityName": "Company Pty Ltd",
  "entityType": "PRV",
  "gst": true
}
```

**Required Secret**: ABR_GUID

#### `nzbn-lookup`
Validates New Zealand Business Numbers.

```typescript
// Request
POST /nzbn-lookup
{ "nzbn": "9429000001234" }
```

**Required Secret**: NZBN_API_KEY

#### `linz-property-lookup`
Fetches NZ property data from LINZ.

#### `google-maps-proxy`
Server-side proxy for Maps API calls to protect API key.

#### `get-maps-key`
Securely delivers Maps API key to authenticated clients.

---

### VaultRE Integration

#### `vaultre-sync`
Full or incremental sync of properties and agents from VaultRE CRM.

```typescript
// Request (cron or manual trigger)
POST /vaultre-sync
Authorization: Bearer <service_role_key>
{
  "organizationId": "uuid",
  "syncType": "full" | "incremental"
}
```

**Flow**:
1. Fetch listings from VaultRE API
2. Map to ListHouze property schema
3. Upsert to properties table
4. Download and upload images
5. Sync agents similarly
6. Log to integration_sync_logs

#### `vaultre-webhook`
Receives real-time updates from VaultRE.

```typescript
// Incoming webhook
POST /vaultre-webhook
X-VaultRE-Signature: <hmac>
{
  "event": "listing.updated",
  "data": { ... }
}
```

---

### Account & Approval Functions

#### `account-type-request`
Submits request for role upgrade (e.g., user → agent).

#### `account-type-admin-review`
Admin approves/rejects role requests.

#### `account-type-cancel`
User cancels pending request.

#### `agent-approval`
Admin verifies agent profiles.

---

### Utility Functions

#### `ai-assistant`
Generates property descriptions using OpenAI.

```typescript
POST /ai-assistant
{
  "propertyType": "house",
  "bedrooms": 3,
  "features": ["pool", "garden"],
  "suburb": "Bondi"
}

// Response
{
  "description": "Stunning 3-bedroom home..."
}
```

**Required Secret**: OPENAI_API_KEY

#### `check-expired-listings`
Cron job to archive expired listings.

#### `inbound-api`
Generic webhook receiver for external systems.

#### `outbound-webhook`
Sends events to external endpoints.

---

## Third-Party Integrations

### Stripe Payments

| Feature | Status | Notes |
|---------|--------|-------|
| Checkout Sessions | ✅ Working | For subscriptions and one-time |
| Customer Portal | ✅ Working | Self-service billing |
| Webhooks | ⚠️ Needs Testing | Configure in Stripe dashboard |
| Plans | ✅ Configured | In `plans` table |

**Configuration Needed**:
1. Add webhook endpoint in Stripe dashboard
2. Copy webhook signing secret
3. Add `STRIPE_WEBHOOK_SECRET` to Lovable secrets

### Resend Email

| Feature | Status | Notes |
|---------|--------|-------|
| Transactional Email | ✅ Working | Inquiry notifications |
| Templates | ✅ Inline HTML | No external templates |
| Domain | ⚠️ Needs Setup | DNS verification required |

**Configuration Needed**:
1. Verify domain in Resend dashboard
2. Add DNS records (SPF, DKIM, DMARC)
3. Update `PLATFORM_DOMAIN` env var if needed

### Google Maps

| Feature | Status | Notes |
|---------|--------|-------|
| Places Autocomplete | ✅ Working | Address entry |
| Maps JavaScript | ✅ Working | Property maps |
| Geocoding | ✅ Working | lat/lng lookup |

**Secrets**:
- `GOOGLE_MAPS_API_KEY` (server-side)
- `VITE_GOOGLE_MAPS_KEY` (client-side)

### VaultRE CRM

| Feature | Status | Notes |
|---------|--------|-------|
| Listing Sync | ⚠️ Structure Ready | Needs API credentials |
| Agent Sync | ⚠️ Structure Ready | Needs API credentials |
| Webhooks | ⚠️ Structure Ready | Needs endpoint registration |

**Configuration Needed**:
1. Obtain VaultRE API key
2. Add `VAULTRE_API_KEY` secret
3. Register webhook URL in VaultRE
4. Test sync with staging data

### ABR / NZBN

| Service | Status | Notes |
|---------|--------|-------|
| ABR Lookup | ✅ Working | Australian business validation |
| NZBN Lookup | ✅ Working | NZ business validation |

---

## Environment Variables Summary

### Required for Core Function

| Variable | Used By | Status |
|----------|---------|--------|
| `SUPABASE_URL` | All functions | ✅ Auto-set |
| `SUPABASE_SERVICE_ROLE_KEY` | All functions | ✅ Auto-set |
| `STRIPE_SECRET_KEY` | billing-* | ✅ Configured |
| `RESEND_API_KEY` | send-* | ✅ Configured |
| `GOOGLE_MAPS_API_KEY` | maps functions | ✅ Configured |

### Required for Full Feature Set

| Variable | Used By | Status |
|----------|---------|--------|
| `STRIPE_WEBHOOK_SECRET` | billing-webhook | ⚠️ Needs setup |
| `OPENAI_API_KEY` | ai-assistant | ✅ Configured |
| `ABR_GUID` | abn-lookup | ✅ Configured |
| `NZBN_API_KEY` | nzbn-lookup | ✅ Configured |
| `ADDY_API_KEY` | NZ address | ✅ Configured |

### Optional / Future

| Variable | Used By | Status |
|----------|---------|--------|
| `VAULTRE_API_KEY` | vaultre-sync | ❌ Not set |
| `VAULTRE_WEBHOOK_SECRET` | vaultre-webhook | ❌ Not set |

---

## Deployment Notes

### Deploying Edge Functions

Edge functions are auto-deployed by Lovable Cloud when files in `supabase/functions/` are changed.

**Manual Deploy** (if needed):
```bash
supabase functions deploy <function-name>
```

### Testing Functions Locally

```bash
# Start local Supabase
supabase start

# Run function locally
supabase functions serve <function-name>
```

### Monitoring

- **Logs**: Supabase Dashboard → Edge Functions → Logs
- **Metrics**: Supabase Dashboard → Edge Functions → Metrics
- **Errors**: Check `error_logs` table for application-level errors

---

## Security Considerations

1. **CORS**: All functions use `*` origin - consider restricting in production
2. **Auth**: Most functions require JWT auth header
3. **Rate Limiting**: Implemented in `get-agent-contact` via DB log table
4. **Secrets**: Never log secret values, use env validation

---

*Document generated: December 31, 2024*
