# ListHouze Launch Readiness Checklist

**Version:** 1.0  
**Last Updated:** December 31, 2024  
**Target Launch Date:** TBD

---

## Launch Readiness Score: 78/100

| Category | Score | Max | Notes |
|----------|-------|-----|-------|
| Core Functionality | 25 | 25 | All features working |
| Security | 18 | 20 | RLS complete, minor improvements needed |
| Billing | 10 | 15 | Webhook testing required |
| Email | 8 | 10 | Domain verification needed |
| Performance | 10 | 15 | Indexes to add |
| Observability | 7 | 15 | Basic logging, need error tracking |

---

## P0 - Must Fix Before Launch

### 1. Stripe Webhook Configuration
- [ ] Add webhook endpoint in Stripe Dashboard
  - URL: `https://<project>.supabase.co/functions/v1/billing-webhook`
  - Events: `checkout.session.completed`, `customer.subscription.*`, `invoice.*`
- [ ] Copy webhook signing secret
- [ ] Add `STRIPE_WEBHOOK_SECRET` to Lovable Cloud secrets
- [ ] Test with Stripe CLI: `stripe trigger checkout.session.completed`
- [ ] Verify subscription records created in database

**Owner:** Platform Team  
**Effort:** 2-4 hours

### 2. Email Domain Verification
- [ ] Login to Resend dashboard
- [ ] Add domain `listhouze.co.nz`
- [ ] Copy DNS records (SPF, DKIM, DMARC)
- [ ] Add records to domain DNS provider
- [ ] Wait for verification (up to 24 hours)
- [ ] Test email delivery with real inquiry

**Owner:** Platform Team  
**Effort:** 1-2 hours (+ DNS propagation)

### 3. Fix Build Errors (COMPLETED)
- [x] Duplicate imports in App.tsx
- [x] TypeScript icon casting in Help.tsx
- [x] Agent interface type conflicts

---

## P1 - Strongly Recommended Before Launch

### 4. Database Performance Indexes
- [ ] Add geospatial index for property search
  ```sql
  CREATE INDEX idx_properties_geo ON properties (latitude, longitude) 
  WHERE status = 'active';
  ```
- [ ] Add suburb search index
  ```sql
  CREATE INDEX idx_properties_suburb ON properties (LOWER(suburb));
  ```
- [ ] Add agent service areas GIN index
  ```sql
  CREATE INDEX idx_agents_areas ON agents USING GIN (service_areas);
  ```

**Owner:** Platform Team  
**Effort:** 1 hour

### 5. Error Tracking Setup
- [ ] Add Sentry or similar error tracking
- [ ] Configure source maps for production
- [ ] Set up alerts for error thresholds
- [ ] Create on-call rotation

**Owner:** Platform Team  
**Effort:** 4-6 hours

### 6. Production Environment Check
- [ ] Verify all environment variables set
- [ ] Test all edge functions in production
- [ ] Verify storage bucket permissions
- [ ] Check rate limiting configuration
- [ ] Review CORS settings

**Owner:** Platform Team  
**Effort:** 2-3 hours

### 7. SEO Essentials
- [ ] Add robots.txt (exists, verify content)
- [ ] Add sitemap.xml (exists, verify pages)
- [ ] Add OpenGraph meta tags to key pages
- [ ] Verify Google Analytics/Tag Manager
- [ ] Submit sitemap to Google Search Console

**Owner:** Platform Team  
**Effort:** 3-4 hours

---

## P2 - Post-Launch Improvements

### 8. VaultRE Integration
- [ ] Obtain VaultRE API credentials
- [ ] Configure webhook endpoint
- [ ] Test with staging data
- [ ] Perform initial sync
- [ ] Set up monitoring for sync failures

### 9. Advanced Monitoring
- [ ] Set up Supabase database alerts
- [ ] Configure edge function metrics
- [ ] Create performance dashboard
- [ ] Set up uptime monitoring

### 10. Performance Optimization
- [ ] Implement lazy loading for heavy routes
- [ ] Add image optimization (WebP)
- [ ] Configure CDN caching rules
- [ ] Analyze and reduce bundle size

### 11. E2E Testing
- [ ] Set up Playwright or Cypress
- [ ] Create tests for critical flows:
  - User signup → onboarding
  - Property listing → publish
  - Inquiry submission → email
  - Checkout → subscription creation

### 12. Backup & Recovery
- [ ] Document Supabase backup policy
- [ ] Test database restore procedure
- [ ] Create runbook for incident response

---

## Go-Live Checklist

### Day Before Launch

- [ ] Final code freeze
- [ ] Run full test suite
- [ ] Verify all P0 items complete
- [ ] Brief support team
- [ ] Prepare rollback plan
- [ ] Notify stakeholders

### Launch Day

- [ ] Deploy to production
- [ ] Smoke test all critical flows
- [ ] Monitor error rates
- [ ] Check email delivery
- [ ] Verify payment processing
- [ ] Enable public access

### Post-Launch (24 hours)

- [ ] Review error logs
- [ ] Check performance metrics
- [ ] Monitor user signups
- [ ] Respond to support tickets
- [ ] Document any issues found

### Post-Launch (Week 1)

- [ ] Daily error log review
- [ ] Weekly performance review
- [ ] Gather user feedback
- [ ] Prioritize bug fixes
- [ ] Plan first update release

---

## Rollback Plan

### If Critical Issue Found

1. **Frontend Issue**
   - Revert to previous commit
   - Lovable will auto-deploy

2. **Database Issue**
   - Restore from Supabase backup
   - Backups are automatic (point-in-time recovery)

3. **Edge Function Issue**
   - Deploy previous version: `supabase functions deploy <name> --version <prev>`

4. **Integration Issue** (Stripe/Resend)
   - Disable affected feature via environment variable
   - Display maintenance message

### Emergency Contacts

| Role | Contact | Escalation |
|------|---------|------------|
| Primary | TBD | Immediate |
| Secondary | TBD | 15 min |
| Supabase Support | support@supabase.io | Critical only |

---

## Success Metrics

### Launch Day Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Error Rate | < 1% | Supabase logs |
| Page Load Time | < 3s | Lighthouse |
| Uptime | 100% | Monitoring |
| Email Delivery | > 95% | Resend dashboard |

### Week 1 Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| User Signups | 50+ | Database count |
| Properties Listed | 20+ | Database count |
| Inquiries Sent | 10+ | inquiries table |
| Zero Critical Bugs | 0 | Issue tracker |

---

## Dependencies & Blockers

### External Dependencies

| Dependency | Status | Risk |
|------------|--------|------|
| Supabase | ✅ Active | Low |
| Stripe | ✅ Active | Medium (webhook) |
| Resend | ✅ Active | Medium (domain) |
| Google Maps | ✅ Active | Low |

### Internal Blockers

| Blocker | Owner | ETA |
|---------|-------|-----|
| Stripe webhook secret | Platform | Before launch |
| Domain DNS access | Marketing | Before launch |

---

*Document generated: December 31, 2024*
