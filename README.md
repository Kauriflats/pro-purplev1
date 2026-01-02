
# List Houze - Real Estate Platform

A modern real estate platform built with React, TypeScript, and Supabase.

## Environment Variables

remium, mobile-first real-estate platform with Houzez-inspired UX, real-time MRI VaultRE sync, role-based portals (Admin, Agency Owner, Agent, Seller, Customer), Google Maps search, CRM, and SEO-first pages. Black & Silver brand. Fast. Accessible. Deploys on Vercel.

✨ Highlights

Houzez-style UI (not copied; rebuilt): crisp cards, filter drawer, list⇄map, sticky CTAs on mobile

Real-time VaultRE sync: properties & agents (images downloaded to our bucket — no hotlinking)

Full RBAC: admin, agency_owner, agent, seller, customer (visitor/buyer/renter)

Portals

Admin: Site Builder (edit every public section), Users & Roles, Sync, CRM, SEO, Visibility controls

Owner: agency/team, routing rules, reports

Agent: listings, open homes, leads/tasks, profile (override-safe)

Seller: property dashboard (open homes, enquiries, views—admin decides what’s visible)

Customer: saved properties, saved searches + alerts (immediate/daily/weekly)

Maps & Search: Google Places Autocomplete (AU) + Maps clustering + “search this area”

SEO + Perf: SSR/ISR or SPA best practices, JSON-LD, sitemaps, canonical; LCP < 2.5s target

Accessibility: WCAG-minded components, keyboard flow, strong contrast (Black/Silver)

🧱 Tech Stack

Frontend: React + TypeScript + Tailwind + shadcn/ui

Backend: Vercel Serverless Functions (/api/**)

DB/ORM: Postgres (Neon/Supabase) + Prisma

Auth: Supabase Auth (Google OAuth + Email/Password)

Storage: S3/R2 (or Supabase Storage) for media (VaultRE images downloaded & served locally)

Email: Resend/SMTP (invites, alerts, notifications)

Design target is Houzez-like look & flow only. We do not reuse or copy any theme code.

📦 Project Structure
/src
  /components
    /property        # PropertyCard, Gallery, Specs
    /search          # FilterDrawer, Chips, List↔Map toggle
    /auth            # AuthGuard, BottomAuthBar, forms
    /home            # HeroSearchSection, FeaturedProperties
    /layout          # ScrollToTop, page scaffolding
  /pages             # SPA pages (Home, Buy/Rent/Sold/Leased, Property, Agents, About, Contact, Auth)
  /admin             # Admin portal screens (Site Builder, Users, Properties, Sync, SEO, Logs)
  /agent             # Agent portal
  /owner             # Agency Owner portal
  /seller            # Seller portal
  /lib               # vaultre client, storage, email, search builders, visibility filters
  /db                # prisma client & seeds
/api
  /webhooks/vaultre.ts       # signature verify + upsert enqueue
  /cron/vaultre-sync.ts      # 15-min incremental sync
  /cron/alerts-daily.ts      # saved-search digests
  /cron/alerts-weekly.ts
  /admin/sync-*.ts           # manual sync triggers (role-guarded)
prisma/schema.prisma
vercel.json

🔐 Roles & Portals (at a glance)
Role	Portal Path	Can See / Do
Admin	/admin/**	Site Builder, Users/Roles, Properties, Agents, Sellers, CRM, Sync, SEO, Logs, Visibility Matrix
Owner	/owner/**	Agency profile, team, routing rules, reports
Agent	/agent/**	My listings, open homes, leads/tasks, profile
Seller	/seller/**	Property dashboard (open homes, enquiries, views) per admin visibility
Customer	/account/**	Saved properties, saved searches + alerts, enquiries & bookings

Visibility Matrix (admin-controlled): decide which metrics (open homes, enquiries, views, saves, days on market…) are visible to which roles, plus per-property overrides.

⚙️ Requirements

Node.js 18+

Postgres 14+ (Neon/Supabase recommended)

S3-compatible storage (R2 / Supabase Storage)

Vercel account (for hosting & crons)

Supabase project (Auth + optional DB)

Google Maps API key (Places + Maps JS)

Resend/SMTP credentials (emails)

VaultRE API Key (Integrator) + Access Token (Office Integrations → Third-Party Access)

🔧 Environment Variables

Create .env.local (dev) and set the same keys in Vercel Project Settings:

# Public (client needs these)
VITE_PUBLIC_SITE_URL=https://silverkeyrealty.com.au
VITE_SUPABASE_URL=...
VITE_SUPABASE_ANON_KEY=...
VITE_GOOGLE_MAPS_API_KEY=...

# Server (never expose to client)
DATABASE_URL=postgres://user:pass@host/db
S3_ENDPOINT=...
S3_REGION=ap-southeast-2
S3_BUCKET=skr-media
S3_ACCESS_KEY_ID=...
S3_SECRET_ACCESS_KEY=...

# VaultRE (MRI)
VAULTRE_ACCOUNT_ID=7040
VAULTRE_INTEGRATION_NAME="SilverKey Realty - Ajuba Tech"
VAULTRE_API_KEY=***           # server-only
VAULTRE_ACCESS_TOKEN=***      # server-only
VAULTRE_BASE_URL=https://ap-southeast-2.api.vaultre.com.au/api/v1.3
VAULTRE_WEBHOOK_TOLERANCE_MS=300000

# Email
RESEND_API_KEY=...
EMAIL_FROM="SilverKey Alerts <info@listhouze.com>"


VaultRE Compliance:
Always send X-Api-Key and Authorization: Bearer <token> headers; respect rate limits; do not hotlink images (download & serve locally); verify webhooks via HMAC-SHA512 (X-VaultRE-Signature, payload t.rawBody).

🚀 Quick Start (Local)
# 1) Install
npm i

# 2) Generate & migrate DB
npx prisma generate
npx prisma migrate dev --name init

# 3) (optional) Seed: creates an admin and sample content
npm run seed

# 4) Dev server (Vite)
npm run dev


Open http://listhouze.com

Default admin (if seeded): shown in terminal after npm run seed. Change immediately.

🧭 Core Scripts
npm run dev            # start local app
npm run build          # production build
npm run preview        # preview local prod build
npm run lint           # code linting
npm run test           # unit tests
npm run test:e2e       # Playwright E2E
npm run prisma:studio  # Prisma Studio
npm run seed           # seed demo data (safe)

🏗️ Deployment (Vercel)

vercel.json

{
  "version": 2,
  "builds": [
    { "src": "index.html", "use": "@vercel/static-build", "config": { "distDir": "dist" } },
    { "src": "api/**/*.[tj]s", "use": "@vercel/node" }
  ],
  "routes": [
    { "src": "/api/(.*)", "dest": "/api/$1" },
    { "src": "/(.*)", "dest": "/index.html" }
  ],
  "crons": [
    { "path": "/api/cron/vaultre-sync", "schedule": "*/15 * * * *" },
    { "path": "/api/cron/alerts-daily", "schedule": "0 8 * * *" },
    { "path": "/api/cron/alerts-weekly", "schedule": "0 8 * * MON" }
  ]
}


Vercel Settings

Framework preset: Vite (or Other)

Build command: npm run build

Output directory: dist

Add env vars from “Environment Variables”

Production domain: silverkeyrealty.com.au

🔄 VaultRE Integration (Overview)

Initial ingest: properties (sale/rent + sold/leased), agents (users)

Webhooks: /api/webhooks/vaultre

Verify X-VaultRE-Signature with HMAC-SHA512 (secret = API key)

Reject stale timestamps (VAULTRE_WEBHOOK_TOLERANCE_MS)

Polling fallback: /api/cron/vaultre-sync every 15 min (updatedSince watermark)

Images: download to S3_BUCKET; re-download only if modtime changed

Enquiries: public form → local Lead + POST to VaultRE (Add/Edit Enquiries)

Agent overrides: overrideMask ensures admin/agent edits persist across syncs

🧠 Site Builder (Admin → Control Every Public Section)

ComponentBlock model: key, variant, data, published

Blocks: home.hero, home.featuredListings, home.suburbs, home.featuredAgents, about.story, about.values, about.stats, about.team, global.header, global.footer, global.announcement

Features: live preview, publish/draft, drag-reorder, role visibility flags

📣 Saved Searches & Alerts

Favourites: /api/saved (POST/DELETE)

Saved Searches: /api/saved-searches (CRUD), frequency Off/Immediate/Daily/Weekly

Immediate: triggered on property upsert

Daily/Weekly: email digests via cron endpoints

🏡 Seller & Agent Metrics (Admin-controlled)

Visibility Matrix (global) & Per-Property overrides decide which roles can see:

Open homes (past/upcoming) + attendance (optional)

Enquiries count

Views, saves/shortlists

Days on market, price changes

Enforcement via filterByVisibility(role, rules, overrides) in all dashboard APIs.

🧭 UX Polishing Defaults

Scroll to top on navigation (<ScrollToTop />)

Sticky bottom Login / Sign-up bar for guests (Google + Email)

Error boundaries + branded 404/500

Skeleton loaders, accessible focus rings, ARIA labels

Consistent CTAs (Call / Enquire / Book) on property detail

🔒 Security & Privacy

Server-only secrets (VaultRE keys/tokens, DB, email)

Webhook signature + timestamp verification

Rate limit public POSTs; spam checks on forms

AuditLog for sensitive actions (role changes, visibility toggles, sync runs)

If Supabase DB: enable RLS; keep policies strict

🧪 Testing

Unit: utilities (formatting, search builder, visibility filter)

E2E (Playwright): RBAC gates, admin Site Builder, agent routing (unique slugs), seller dashboard visibility, saved searches & alerts, sync happy path

Lighthouse (mobile): Perf ≥ 85, SEO ≥ 95, CLS < 0.1

🛠️ Troubleshooting

VaultRE 401/403: check API key/token; scopes enabled (View Properties, View Contacts, View Enquiries, Add/Edit Enquiries, View Users)

429 rate limits: backoff; reduce concurrency; cache list pages

Images not showing: confirm S3 credentials, bucket CORS, public read or signed URLs

Google OAuth: add correct redirect URL in Supabase Auth → Google Provider

Map not loading: domain restrictions on Google key; enable Places + Maps JS

📄 License

Proprietary / Confidential — © AjubaTech. All rights reserved.
Distribution or reuse requires written permission.

🙌 Acknowledgements

MRI VaultRE (data source & API)

Google Maps Platform

📬 Contact

AjubaTech — smart real-estate software & integrations
✉️ info@ajubatech.com

We build fast, elegant software that converts.
