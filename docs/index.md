# ListHouze Documentation Index

**Version:** 1.0  
**Last Updated:** December 31, 2024

---

## Quick Links

| Document | Description |
|----------|-------------|
| [HANDOVER.md](./HANDOVER.md) | **Primary stakeholder handover** - Executive summary, module status, launch readiness |
| [ARCHITECTURE.md](./ARCHITECTURE.md) | System architecture, data flows, deployment |
| [DB_SCHEMA_AND_RLS.md](./DB_SCHEMA_AND_RLS.md) | Database tables, RLS policies, storage buckets |
| [EDGE_FUNCTIONS_AND_INTEGRATIONS.md](./EDGE_FUNCTIONS_AND_INTEGRATIONS.md) | Edge functions, Stripe, Resend, VaultRE |
| [LAUNCH_READINESS.md](./LAUNCH_READINESS.md) | Launch checklist, P0/P1 items, go-live plan |
| [CHANGELOG_SINCE_START.md](./CHANGELOG_SINCE_START.md) | Implementation history by module |
| [KNOWN_ISSUES_AND_BUGS.md](./KNOWN_ISSUES_AND_BUGS.md) | Current issues with severity and fixes |
| [PRODUCT_ROADMAP_WIP.md](./PRODUCT_ROADMAP_WIP.md) | WIP features, backlog, sequence |

---

## Document Map

```
docs/
├── index.md                          ← You are here
├── HANDOVER.md                       ← Start here for overview
├── ARCHITECTURE.md                   ← Technical architecture
├── DB_SCHEMA_AND_RLS.md             ← Database deep dive
├── EDGE_FUNCTIONS_AND_INTEGRATIONS.md ← Backend functions
├── LAUNCH_READINESS.md              ← Pre-launch checklist
├── CHANGELOG_SINCE_START.md         ← What was built
├── KNOWN_ISSUES_AND_BUGS.md         ← Current issues
├── PRODUCT_ROADMAP_WIP.md           ← What's next
└── BLOG_HELP_CENTER.md              ← Blog/Help implementation
```

---

## Reading Order

### For Stakeholders / Project Managers
1. Start with [HANDOVER.md](./HANDOVER.md) for the executive summary
2. Review [LAUNCH_READINESS.md](./LAUNCH_READINESS.md) for go-live status
3. Check [KNOWN_ISSUES_AND_BUGS.md](./KNOWN_ISSUES_AND_BUGS.md) for blockers

### For Developers
1. Start with [ARCHITECTURE.md](./ARCHITECTURE.md) for system understanding
2. Deep dive into [DB_SCHEMA_AND_RLS.md](./DB_SCHEMA_AND_RLS.md)
3. Review [EDGE_FUNCTIONS_AND_INTEGRATIONS.md](./EDGE_FUNCTIONS_AND_INTEGRATIONS.md)
4. Check [CHANGELOG_SINCE_START.md](./CHANGELOG_SINCE_START.md) for implementation details

### For DevOps / Launch Team
1. Review [LAUNCH_READINESS.md](./LAUNCH_READINESS.md)
2. Check [EDGE_FUNCTIONS_AND_INTEGRATIONS.md](./EDGE_FUNCTIONS_AND_INTEGRATIONS.md) for env vars
3. Address items in [KNOWN_ISSUES_AND_BUGS.md](./KNOWN_ISSUES_AND_BUGS.md)

---

## Related Files

### In Repository Root
- `README.md` - Project overview and setup
- `ENVIRONMENT_SETUP.md` - Environment configuration
- `DEPLOYMENT_CHECKLIST.md` - Deployment steps
- `PRODUCTION_READINESS_REPORT.md` - Production status

### Configuration Files
- `tailwind.config.ts` - Design system
- `supabase/config.toml` - Supabase configuration
- `vercel.json` - Vercel deployment config (if applicable)

---

## Maintaining Documentation

When making changes to the platform:

1. **New Feature** → Update `CHANGELOG_SINCE_START.md`
2. **New Issue** → Add to `KNOWN_ISSUES_AND_BUGS.md`
3. **Database Change** → Update `DB_SCHEMA_AND_RLS.md`
4. **New Edge Function** → Update `EDGE_FUNCTIONS_AND_INTEGRATIONS.md`
5. **Architecture Change** → Update `ARCHITECTURE.md`

---

*Documentation index generated: December 31, 2024*
