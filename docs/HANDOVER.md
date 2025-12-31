# ListHouze Platform - Stakeholder Handover Document

**Version:** 1.0  
**Date:** December 31, 2024  
**Prepared by:** Technical Program Lead  
**Status:** Ready for Stakeholder Review

---

## Executive Summary

### What is ListHouze?

ListHouze is a comprehensive real estate platform serving Australia and New Zealand markets. It enables:
- **Property Listings**: Sale and rental property management with full lifecycle support
- **Agent Directory**: Verified agent profiles with reviews, ratings, and contact management
- **Agency Management**: Multi-tenant organization support with team, listing, and subscription management
- **Mortgage Brokers**: Dedicated broker directory with lead management and verification
- **Buyer/Renter Portal**: Property search, saved searches, inquiries, and application submission

### What is Live/Working Today

| Feature | Status | Notes |
|---------|--------|-------|
| User Authentication | ✅ Live | Email + OAuth (Google) |
| Role-Based Access Control | ✅ Live | 9 roles with RLS enforcement |
| Property Listings (CRUD) | ✅ Live | Full create/edit/publish flow |
| Agent Directory | ✅ Live | Public profiles, search, filters |
| Agency Owner Dashboard | ✅ Live | Team, listings, analytics |
| Mortgage Broker Directory | ✅ Live | Profiles, leads, verification |
| Customer Dashboard | ✅ Live | Saved properties, inquiries |
| Admin Portal | ✅ Live | User/property/agent management |
| Inquiry System | ✅ Live | Email notifications via Resend |
| Billing Integration | ⚠️ Partial | Stripe checkout ready, webhook testing needed |
| Blog & Help Center | ✅ Live | Database-connected content |
| Maps Integration | ✅ Live | Google Maps with property pins |

### What is Blocked/Unfinished

| Item | Priority | Blocker |
|------|----------|---------|
| Stripe Webhook Production Setup | P0 | Needs webhook secret + endpoint verification |
| Email Domain Verification | P0 | Resend domain DNS setup required |
| VaultRE Full Sync | P1 | API credentials + webhook endpoint needed |
| SEO Metadata | P1 | Dynamic meta tags partially implemented |
| Performance Indexes | P1 | Some slow queries identified |

### Launch Readiness Score: **78/100**

**Rationale:**
- ✅ Core functionality complete and functional
- ✅ RLS policies comprehensive with no critical gaps
- ✅ Authentication and role management solid
- ⚠️ Billing webhooks need production testing
- ⚠️ Email sending needs domain verification
- ⚠️ Some TypeScript warnings in build

---

## Module Status Table

| Module | Status | Owner | Risk | Notes |
|--------|--------|-------|------|-------|
| **Auth & Roles** | ✅ Done | Platform | Low | 9 app_role types, secure RLS |
| **User Profiles** | ✅ Done | Platform | Low | Profile photos, contact info |
| **Agents & Directory** | ✅ Done | Platform | Low | Verification, reviews, slugs |
| **Agency Management** | ✅ Done | Platform | Low | Multi-tenant, team invites |
| **Mortgage Managers** | ✅ Done | Platform | Medium | Verification workflow complete |
| **Property Listings** | ✅ Done | Platform | Low | Full CRUD, images, agents |
| **Property Search** | ✅ Done | Platform | Medium | Filters, map view, pagination |
| **Enquiries & Leads** | ✅ Done | Platform | Low | Email notifications, tracking |
| **Billing & Subscriptions** | ⚠️ WIP | Platform | High | Checkout works, webhooks need testing |
| **Admin Tooling** | ✅ Done | Platform | Low | Comprehensive admin portal |
| **Notifications/Emails** | ⚠️ WIP | Platform | Medium | Resend integrated, domain pending |
| **Media/Storage** | ✅ Done | Platform | Low | 8 buckets with RLS policies |
| **SEO/Vanity URLs** | ⚠️ WIP | Platform | Medium | Agent slugs done, property slugs pending |
| **Theme & UX** | ✅ Done | Platform | Low | Dark/light mode, responsive |
| **Blog & Help** | ✅ Done | Platform | Low | 12 content tables, full CMS |

---

## Detailed Implementation Status

### Completed Features

#### Authentication & Authorization
- **Files**: `src/hooks/useAuth.ts`, `src/components/auth/AuthGuard.tsx`
- **DB Tables**: `user_roles`, `profiles`
- **Features**:
  - Email/password authentication
  - Google OAuth integration
  - Role-based route guards
  - Session management with auto-refresh
  - Password reset flow

#### Property Management
- **Files**: `src/pages/agent/CreateListing.tsx`, `src/pages/agent/EditListing.tsx`
- **DB Tables**: `properties`, `property_images`, `property_agents`
- **Features**:
  - Full CRUD for listings
  - Multi-image upload to Supabase Storage
  - Agent assignment
  - Status workflow (draft → active → sold/leased)
  - Address autocomplete via Google Maps

#### Agent Directory
- **Files**: `src/pages/Agents.tsx`, `src/pages/AgentProfile.tsx`
- **DB Tables**: `agents`, `agents_public` (view), `agent_reviews`
- **Features**:
  - Public directory with filters
  - Vanity URL slugs (e.g., `/agents/john_smith`)
  - Review/rating system
  - Contact reveal with rate limiting

#### Agency Owner Portal
- **Files**: `src/pages/owner/*`
- **DB Tables**: `organizations`, `organization_members`, `team_invites`
- **Features**:
  - Team management with invites
  - Listing management across team
  - Analytics dashboard
  - Integration settings (VaultRE)

#### Mortgage Broker Module
- **Files**: `src/pages/mortgage/*`
- **DB Tables**: `mortgage_profiles`, `mortgage_leads`, `mortgage_verifications`
- **Features**:
  - Broker profiles with verification
  - Lead management with status tracking
  - Enquiry usage limits per tier
  - Service area filtering

#### Admin Portal
- **Files**: `src/pages/admin/*`
- **DB Tables**: `admin_activity_logs`, `admin_agency_reviews`
- **Features**:
  - User management
  - Property approval workflow
  - Agent verification
  - Activity logging
  - Deleted items archive

### Work in Progress

#### Billing & Subscriptions
- **Status**: 70% complete
- **What's Done**:
  - Plan definitions in database
  - Stripe checkout session creation
  - Customer portal link
- **What's Missing**:
  - Webhook handler production testing
  - Entitlement enforcement (fn_can_publish_property exists but needs testing)
  - Subscription status sync

#### VaultRE Integration
- **Status**: 60% complete
- **What's Done**:
  - Sync edge function structure
  - Property/agent mapping logic
- **What's Missing**:
  - Production API credentials
  - Webhook endpoint registration
  - Error handling refinement

---

## Known Issues & Bugs

### P0 - Must Fix Before Launch

| Issue | Repro Steps | Root Cause | Fix Recommendation |
|-------|-------------|------------|-------------------|
| Stripe webhook not receiving events | 1. Complete checkout 2. Check logs | Webhook secret not configured | Add `STRIPE_WEBHOOK_SECRET` and verify endpoint in Stripe dashboard |
| Emails may fail silently | 1. Submit inquiry 2. Check Resend dashboard | Domain not verified in Resend | Complete DNS verification for `listhouze.co.nz` |

### P1 - Strongly Recommended

| Issue | Repro Steps | Root Cause | Fix Recommendation |
|-------|-------------|------------|-------------------|
| Agent/VaultREAgent type conflicts | Build project | Interface extends row with optional fields | Fixed in this PR - omit conflicting fields |
| Slow property search on large datasets | Load 1000+ properties | Missing indexes on lat/lng/suburb | Add composite index on search columns |
| Bundle size warnings | `npm run build` | Large dependencies not code-split | Lazy load heavy routes (admin, maps) |

### P2 - Post-Launch

| Issue | Description | Priority |
|-------|-------------|----------|
| React hook warnings | Some useEffect dependencies | Low |
| Mobile drawer animation jank | Minor UX issue on some devices | Low |

---

## Recommended Next Steps (Top 10)

| # | Task | Effort | Why |
|---|------|--------|-----|
| 1 | Configure Stripe webhook in production | S | Required for billing to work |
| 2 | Verify Resend email domain | S | Required for reliable email delivery |
| 3 | Add missing performance indexes | M | Prevent slow queries at scale |
| 4 | Complete VaultRE integration | L | Import existing agency data |
| 5 | Add E2E tests for critical flows | L | Ensure stability |
| 6 | Implement property vanity URLs | M | SEO improvement |
| 7 | Add OpenGraph meta tags | S | Social sharing |
| 8 | Setup error tracking (Sentry) | M | Production observability |
| 9 | Configure CDN for images | M | Performance improvement |
| 10 | Load testing | M | Validate scale readiness |

---

## Launch Plan

### Minimum Launch Scope
1. ✅ Authentication working
2. ✅ Property listing and search functional
3. ✅ Agent directory accessible
4. ⏳ Stripe webhooks verified
5. ⏳ Email delivery confirmed

### Rollout Plan

**Week 1: Soft Launch**
- Internal team testing
- 5-10 beta agencies onboarded
- Monitor error logs and performance

**Week 2: Limited Public**
- Open registration (with manual verification)
- Marketing to 50 agencies
- Daily monitoring

**Week 3+: Full Public**
- Remove registration gates
- Scale marketing
- 24/7 monitoring first week

### Monitoring & Rollback

**Monitoring Checklist:**
- [ ] Supabase dashboard for DB metrics
- [ ] Edge function logs for errors
- [ ] Stripe dashboard for payment issues
- [ ] Resend dashboard for email delivery

**Rollback Steps:**
1. If critical DB issue: Restore from Supabase backup
2. If edge function failure: Deploy previous version
3. If frontend broken: Revert to last known good commit

---

## Environment Variables Required

```env
# Supabase (set automatically in Lovable)
SUPABASE_URL=
SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# Stripe
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=

# Email (Resend)
RESEND_API_KEY=

# Google Maps
GOOGLE_MAPS_API_KEY=
VITE_GOOGLE_MAPS_KEY=

# External APIs
ABR_GUID=  # Australian Business Register
NZBN_API_KEY=  # NZ Business Number lookup
ADDY_API_KEY=  # NZ address autocomplete

# Optional
OPENAI_API_KEY=  # AI description generator
```

---

## Contact & Resources

- **Supabase Project**: Check Lovable Cloud dashboard
- **Stripe Dashboard**: https://dashboard.stripe.com
- **Resend Dashboard**: https://resend.com/dashboard
- **Repository**: Current Lovable project

---

*Document generated: December 31, 2024*
