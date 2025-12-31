# ListHouze System Architecture

**Version:** 1.0  
**Last Updated:** December 31, 2024

---

## High-Level Architecture

```mermaid
graph TB
    subgraph "Client Layer"
        Browser[Web Browser]
        Mobile[Mobile Browser]
    end

    subgraph "Frontend - Lovable/Vite"
        React[React 18 SPA]
        Router[React Router 6]
        TanStack[TanStack Query]
        Theme[Theme Provider]
    end

    subgraph "Backend - Supabase"
        Auth[Supabase Auth]
        DB[(PostgreSQL)]
        Storage[Storage Buckets]
        Edge[Edge Functions]
        Realtime[Realtime Subscriptions]
    end

    subgraph "External Services"
        Stripe[Stripe Payments]
        Resend[Resend Email]
        Google[Google Maps API]
        VaultRE[VaultRE CRM]
        ABR[ABR/NZBN APIs]
    end

    Browser --> React
    Mobile --> React
    React --> Router
    React --> TanStack
    React --> Theme
    TanStack --> Auth
    TanStack --> DB
    TanStack --> Storage
    React --> Edge
    React --> Realtime
    Edge --> Stripe
    Edge --> Resend
    Edge --> VaultRE
    Edge --> ABR
    React --> Google
```

---

## Technology Stack

| Layer | Technology | Version |
|-------|------------|---------|
| Frontend | React | 18.3.1 |
| Build Tool | Vite | Latest |
| Styling | Tailwind CSS | 3.x |
| UI Components | shadcn/ui + Radix | Latest |
| State Management | TanStack Query | 5.83.0 |
| Routing | React Router | 6.30.1 |
| Animation | Framer Motion | 12.23.26 |
| Forms | React Hook Form + Zod | 7.61.1 / 3.25.76 |
| Backend | Supabase | 2.89.0 |
| Database | PostgreSQL | 14.1 |
| Edge Functions | Deno | Latest |
| Payments | Stripe | 14.21.0 |
| Email | Resend | API v1 |
| Maps | Google Maps | 3.58.1 |

---

## Data Flow Diagrams

### 1. User Signup → Profile Creation → Role Gating

```mermaid
sequenceDiagram
    participant U as User
    participant R as React App
    participant A as Supabase Auth
    participant T as Trigger: handle_new_user
    participant DB as PostgreSQL
    participant RLS as RLS Policies

    U->>R: Click Sign Up
    R->>A: signUp(email, password)
    A->>A: Create auth.users record
    A->>T: AFTER INSERT trigger
    T->>DB: INSERT INTO profiles
    T->>DB: Check user_roles
    Note over T,DB: If no role, insert 'user' role
    A-->>R: Return session
    R->>R: Redirect to onboarding
    U->>R: Select account type
    R->>DB: Call account-type-request function
    DB->>DB: Create request record
    Note over DB: Admin approves request
    DB->>DB: Grant role via user_roles
    R->>RLS: Future requests use new role
```

### 2. Listing Creation → Publish → Public Display

```mermaid
sequenceDiagram
    participant A as Agent
    participant R as React App
    participant DB as PostgreSQL
    participant S as Storage
    participant F as fn_can_publish

    A->>R: Create Listing Form
    R->>DB: INSERT properties (status: draft)
    DB-->>R: Return property_id
    
    A->>R: Upload Images
    R->>S: Upload to property-images bucket
    S-->>R: Return URLs
    R->>DB: INSERT property_images
    
    A->>R: Click Publish
    R->>DB: UPDATE status = 'active'
    DB->>F: BEFORE UPDATE trigger
    F->>F: Check subscription/limits
    alt Allowed
        F-->>DB: Return NEW
        DB-->>R: Success
    else Blocked
        F-->>DB: RAISE EXCEPTION
        DB-->>R: Error with hint
    end
    
    Note over R: Public listing now visible
```

### 3. Inquiry Flow → Email → Dashboard

```mermaid
sequenceDiagram
    participant V as Visitor
    participant R as React App
    participant E as Edge: send-inquiry-email
    participant RS as Resend API
    participant DB as PostgreSQL
    participant AG as Agent

    V->>R: Submit Inquiry Form
    R->>E: POST /send-inquiry-email
    E->>DB: Resolve recipient (property → agent)
    E->>RS: Send to agent
    E->>RS: Send confirmation to visitor
    E->>DB: INSERT INTO inquiries
    E-->>R: Success response
    R->>V: Show success toast
    
    Note over AG: Agent checks dashboard
    AG->>R: View Agent Dashboard
    R->>DB: SELECT FROM inquiries WHERE agent_id
    DB-->>R: Return inquiries
    R->>AG: Display inquiry list
    AG->>R: Mark as responded
    R->>DB: UPDATE inquiry status
```

---

## Deployment Architecture

```mermaid
graph LR
    subgraph "Lovable Platform"
        Build[Build System]
        Preview[Preview Environment]
        Prod[Production CDN]
    end

    subgraph "Supabase Cloud"
        Auth[Auth Service]
        PG[(PostgreSQL)]
        Storage[Object Storage]
        Functions[Edge Functions]
    end

    subgraph "DNS/CDN"
        Domain[listhouze.com]
        CF[Cloudflare CDN]
    end

    Build --> Preview
    Build --> Prod
    Prod --> CF
    CF --> Domain
    Domain --> Auth
    Domain --> Functions
    Functions --> PG
    Functions --> Storage
```

### Environment Separation

| Environment | Purpose | URL |
|-------------|---------|-----|
| Preview | Each PR gets a preview | `*.lovable.app` |
| Production | Live site | `listhouze.com` |

### Supabase Project
- Single project with production data
- Edge functions deployed via Lovable Cloud
- Migrations tracked in `supabase/migrations/`

---

## Security Architecture

### Authentication Flow

```mermaid
flowchart TD
    A[User Request] --> B{Has JWT?}
    B -->|No| C[Redirect to /auth]
    B -->|Yes| D{Valid Token?}
    D -->|No| C
    D -->|Yes| E{RLS Check}
    E -->|Pass| F[Return Data]
    E -->|Fail| G[403 Forbidden]
```

### Security Boundaries

1. **Frontend**: Public routes vs AuthGuard protected
2. **Database**: RLS policies per table per operation
3. **Edge Functions**: JWT validation + service role escalation
4. **Storage**: Bucket policies with user/org isolation

### Threat Model Highlights

| Threat | Mitigation |
|--------|------------|
| SQL Injection | Supabase SDK parameterized queries |
| XSS | React auto-escaping + CSP headers |
| CSRF | SameSite cookies + CORS headers |
| Privilege Escalation | RLS + security definer functions |
| Data Leakage | Column-level RLS + views |
| Rate Limiting | Edge function limits + DB rate checks |

---

## Key Components

### Frontend Structure

```
src/
├── pages/           # Route components (80+ pages)
│   ├── admin/       # Super admin portal
│   ├── agent/       # Agent dashboard
│   ├── owner/       # Agency owner dashboard
│   ├── customer/    # Buyer/renter dashboard
│   ├── seller/      # Private lister dashboard
│   ├── mortgage/    # Broker portal
│   └── ...          # Public pages
├── components/      # Reusable UI (30+ directories)
├── hooks/           # Custom hooks (38 files)
├── integrations/    # Supabase client + types
└── lib/             # Utilities
```

### Edge Functions

| Function | Purpose |
|----------|---------|
| `billing-start-checkout` | Create Stripe checkout session |
| `billing-webhook` | Handle Stripe events |
| `send-inquiry-email` | Route inquiries to recipients |
| `get-agent-contact` | Rate-limited contact reveal |
| `vaultre-sync` | Sync properties from VaultRE |
| `abn-lookup` | Validate AU business numbers |
| `nzbn-lookup` | Validate NZ business numbers |

### Storage Buckets

| Bucket | Public | Purpose |
|--------|--------|---------|
| `property-images` | Yes | Listing photos |
| `agent-photos` | Yes | Agent headshots |
| `avatars` | Yes | User profile photos |
| `organization-assets` | Yes | Agency logos/banners |
| `sold-property-images` | Yes | Historical sold photos |
| `broker-verification` | No | ID documents |
| `mortgage-lead-docs` | No | Lead attachments |

---

## Performance Considerations

### Database Indexes

Current indexes on frequently queried columns:
- `properties`: status, listing_category, organization_id
- `agents`: is_active, verification_status, slug
- `inquiries`: agent_id, property_id, status

### Recommended Additions

```sql
-- Geospatial search
CREATE INDEX idx_properties_geo ON properties (latitude, longitude);

-- Text search
CREATE INDEX idx_properties_suburb ON properties (suburb);
CREATE INDEX idx_agents_service_areas ON agents USING GIN (service_areas);
```

### Caching Strategy

- TanStack Query: 60s staleTime, 5min gcTime
- Static assets: CDN cached
- Database: Supabase connection pooling

---

## Scalability Notes

### Current Limits
- Supabase Free/Pro tier limits apply
- Edge function cold starts ~100-500ms
- Realtime connections limited by plan

### Scaling Path
1. **Database**: Upgrade Supabase plan for more connections
2. **Storage**: CDN for images already in place
3. **Edge Functions**: Scale automatically
4. **Search**: Consider Supabase Full-Text Search or Algolia

---

*Document generated: December 31, 2024*
