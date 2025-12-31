# ListHouze Known Issues & Bugs

**Version:** 1.0  
**Last Updated:** December 31, 2024

---

## Issue Severity Definitions

| Severity | Definition | Response Time |
|----------|------------|---------------|
| **P0** | Blocks launch or causes data loss | Immediate |
| **P1** | Major feature broken, workaround exists | 24-48 hours |
| **P2** | Minor issue, cosmetic, edge case | Next sprint |
| **P3** | Enhancement, nice-to-have | Backlog |

---

## P0 - Critical Issues

### ISSUE-001: Stripe Webhook Not Configured
**Status:** Open  
**Discovered:** December 31, 2024  
**Owner:** Platform Team

**Description:**  
Stripe webhook endpoint is not receiving events because the webhook secret is not configured in production environment.

**Impact:**  
- Subscriptions created via checkout are not recorded in database
- Billing status changes not reflected
- Users may pay but not receive access

**Reproduction Steps:**
1. Go to /pricing
2. Click subscribe on any plan
3. Complete Stripe checkout
4. Check `subscriptions` table - no new record
5. Check Supabase logs - no webhook invocation

**Root Cause:**  
`STRIPE_WEBHOOK_SECRET` environment variable not set in Lovable Cloud secrets.

**Fix Recommendation:**
1. Go to Stripe Dashboard → Developers → Webhooks
2. Add endpoint: `https://<project>.supabase.co/functions/v1/billing-webhook`
3. Select events: `checkout.session.completed`, `customer.subscription.*`, `invoice.*`
4. Copy signing secret
5. Add to Lovable Cloud secrets as `STRIPE_WEBHOOK_SECRET`

**Files Affected:**
- `supabase/functions/billing-webhook/index.ts`

---

### ISSUE-002: Email Domain Not Verified
**Status:** Open  
**Discovered:** December 31, 2024  
**Owner:** Platform Team

**Description:**  
Resend email domain `listhouze.co.nz` is not verified, causing emails to potentially fail or go to spam.

**Impact:**  
- Inquiry emails may not be delivered
- Password reset emails may fail
- Low deliverability rates

**Reproduction Steps:**
1. Submit an inquiry on any property
2. Check Resend dashboard for delivery status
3. May show "domain not verified" or emails bouncing

**Root Cause:**  
DNS records (SPF, DKIM, DMARC) not configured for domain.

**Fix Recommendation:**
1. Login to Resend dashboard
2. Go to Domains → Add Domain
3. Add `listhouze.co.nz`
4. Copy provided DNS records
5. Add to domain registrar DNS settings
6. Wait for verification (up to 24 hours)

**Files Affected:**
- `supabase/functions/send-inquiry-email/index.ts` (uses `PLATFORM_DOMAIN`)

---

## P1 - High Priority Issues

### ISSUE-003: TypeScript Interface Conflicts (FIXED)
**Status:** Resolved  
**Fixed:** December 31, 2024

**Description:**  
Agent and VaultREAgent interfaces incorrectly extended AgentRow, causing build errors.

**Fix Applied:**
```typescript
// Before
export interface Agent extends AgentRow { ... }

// After
export interface Agent extends Omit<AgentRow, 'title' | 'total_sales'> { ... }
```

**Files Fixed:**
- `src/hooks/useAgents.ts`
- `src/hooks/useVaultREAgents.ts`

---

### ISSUE-004: Duplicate Imports in App.tsx (FIXED)
**Status:** Resolved  
**Fixed:** December 31, 2024

**Description:**  
Blog and Help pages were imported twice, causing build failure.

**Files Fixed:**
- `src/App.tsx` - Removed duplicate import block at lines 161-169

---

### ISSUE-005: Icons Type Casting Error (FIXED)
**Status:** Resolved  
**Fixed:** December 31, 2024

**Description:**  
Lucide icons module couldn't be cast to `Record<string, LucideIcon>`.

**Fix Applied:**
```typescript
// Before
const IconComponent = (Icons as Record<string, LucideIcon>)[iconName];

// After
const IconComponent = (Icons as unknown as Record<string, LucideIcon>)[iconName];
```

**Files Fixed:**
- `src/pages/help/Help.tsx`

---

### ISSUE-006: Missing Performance Indexes
**Status:** Open  
**Discovered:** December 31, 2024  
**Owner:** Platform Team

**Description:**  
Property search queries may be slow at scale due to missing database indexes.

**Impact:**  
- Slow property search results (>2s with 1000+ properties)
- Slow agent directory filtering
- Poor user experience on public pages

**Root Cause:**  
Indexes not added for common query patterns.

**Fix Recommendation:**
```sql
-- Add via migration
CREATE INDEX idx_properties_geo ON properties (latitude, longitude) WHERE status = 'active';
CREATE INDEX idx_properties_suburb ON properties (LOWER(suburb));
CREATE INDEX idx_agents_areas ON agents USING GIN (service_areas);
CREATE INDEX idx_properties_created ON properties (created_at DESC);
```

---

### ISSUE-007: VaultRE Integration Not Connected
**Status:** Open  
**Discovered:** December 31, 2024  
**Owner:** Platform Team

**Description:**  
VaultRE CRM integration is structured but not configured with production credentials.

**Impact:**  
- Agencies cannot sync existing listings from VaultRE
- Manual data entry required

**Root Cause:**  
- `VAULTRE_API_KEY` not set
- Webhook endpoint not registered with VaultRE

**Fix Recommendation:**
1. Obtain API credentials from VaultRE
2. Add `VAULTRE_API_KEY` to secrets
3. Register webhook URL with VaultRE support
4. Test with staging agency data

---

## P2 - Medium Priority Issues

### ISSUE-008: Bundle Size Warnings
**Status:** Open  
**Discovered:** December 31, 2024

**Description:**  
Build shows warnings about large bundle chunks.

**Impact:**  
- Slightly slower initial page load
- Higher bandwidth usage

**Root Cause:**  
Heavy dependencies not code-split (maps, admin).

**Fix Recommendation:**
```typescript
// Lazy load heavy routes
const AdminPortal = lazy(() => import('./pages/admin/AdminPortal'));
const ExploreMap = lazy(() => import('./pages/ExploreMap'));
```

---

### ISSUE-009: React Hook Dependency Warnings
**Status:** Open  
**Discovered:** December 31, 2024

**Description:**  
Some useEffect hooks have missing or unnecessary dependencies.

**Impact:**  
- Console warnings in development
- Potential unexpected re-renders

**Fix Recommendation:**  
Run ESLint with `react-hooks/exhaustive-deps` rule and fix warnings.

---

### ISSUE-010: Mobile Drawer Animation Jank
**Status:** Open  
**Discovered:** December 31, 2024

**Description:**  
Mobile navigation drawer has slight animation stuttering on some devices.

**Impact:**  
- Minor UX issue
- Only affects mobile menu

**Files Affected:**
- `src/components/nav/MobileDrawer.tsx`

---

## P3 - Low Priority / Enhancements

### ISSUE-011: Property Vanity URLs Not Implemented
**Status:** Enhancement  

**Description:**  
Properties use UUID in URL instead of human-readable slugs.

**Current:** `/property/550e8400-e29b-41d4-a716-446655440000`  
**Desired:** `/property/3-bedroom-house-bondi-beach`

---

### ISSUE-012: OpenGraph Images Missing
**Status:** Enhancement  

**Description:**  
Dynamic OG images not generated for properties and agents.

---

### ISSUE-013: Full-Text Search Not Implemented
**Status:** Enhancement  

**Description:**  
Search uses LIKE queries instead of PostgreSQL full-text search.

---

## Issue Tracking Template

```markdown
### ISSUE-XXX: [Title]
**Status:** Open | In Progress | Resolved  
**Severity:** P0 | P1 | P2 | P3  
**Discovered:** YYYY-MM-DD  
**Owner:** [Team/Person]

**Description:**  
[What is the issue?]

**Impact:**  
[How does this affect users/business?]

**Reproduction Steps:**
1. Step 1
2. Step 2
3. Expected vs actual result

**Root Cause:**  
[Why is this happening?]

**Fix Recommendation:**
[How to fix it]

**Files Affected:**
- file1.ts
- file2.tsx
```

---

*Document generated: December 31, 2024*
