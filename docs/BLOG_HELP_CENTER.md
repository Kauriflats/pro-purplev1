# Listhouze Blog & Help Center Documentation

## Overview

This document describes the Blog, Help Center, Roadmap, and Release Notes system for Listhouze.

**Domain:** https://www.listhouze.com

## Technology Stack

| Component | Technology |
|-----------|------------|
| Framework | React + Vite + TypeScript |
| Styling | Tailwind CSS |
| UI Components | Radix UI + shadcn/ui |
| Animation | Framer Motion |
| Routing | React Router DOM 6.30.1 |
| Backend | Supabase |
| Database | PostgreSQL |

---

## Database Schema

All tables use the `help_site_` prefix for namespace separation.

### Tables

1. **help_site_blog_categories** - Blog post categories
2. **help_site_blog_tags** - Tags for blog posts
3. **help_site_blogs** - Blog post content
4. **help_site_blog_tag_relations** - Junction table for blog-tag relationships
5. **help_site_help_categories** - Help center categories
6. **help_site_help_articles** - Help center articles
7. **help_site_faqs** - Frequently asked questions
8. **help_site_roadmap_items** - Product roadmap items
9. **help_site_releases** - Release notes
10. **help_site_release_features** - Features per release
11. **help_site_subscribers** - Newsletter subscribers
12. **help_site_announcements** - Site announcements

### RLS Policies

- **Public Read**: All published content is publicly accessible
- **Admin Write**: Only super_admin users can create/edit/delete content
- **Public Insert**: Anyone can subscribe to the newsletter

---

## Routes

| Path | Component | Description |
|------|-----------|-------------|
| `/blog` | Blog.tsx | Blog listing with search and categories |
| `/blog/:slug` | BlogPost.tsx | Individual blog post |
| `/help` | Help.tsx | Help center index |
| `/help/category/:slug` | HelpCategory.tsx | Help category with articles |
| `/help/article/:slug` | HelpArticle.tsx | Individual help article |
| `/roadmap` | Roadmap.tsx | Product roadmap (Kanban view) |
| `/releases` | Releases.tsx | Release notes index |
| `/releases/:slug` | ReleaseDetail.tsx | Individual release details |

---

## React Hooks

| Hook | Purpose |
|------|---------|
| `useBlogPosts` | Fetch published blog posts |
| `useBlogPost` | Fetch single blog post by slug |
| `useBlogCategories` | Fetch blog categories |
| `useRelatedPosts` | Fetch related blog posts |
| `useHelpCategories` | Fetch help categories |
| `useHelpArticles` | Fetch help articles |
| `useHelpArticle` | Fetch single help article |
| `useFAQs` | Fetch FAQs |
| `useArticleFeedback` | Submit article feedback |
| `useRoadmapItems` | Fetch roadmap items |
| `useReleases` | Fetch releases |
| `useRelease` | Fetch single release |
| `useSubscribe` | Newsletter subscription |

---

## Components

### Reusable Components

| Component | Path | Description |
|-----------|------|-------------|
| `BlogCard` | `src/components/blog/BlogCard.tsx` | Blog post card (default, featured, compact) |
| `ShareButtons` | `src/components/blog/ShareButtons.tsx` | Social share buttons |
| `NewsletterSignup` | `src/components/blog/NewsletterSignup.tsx` | Newsletter form |
| `CTABanner` | `src/components/blog/CTABanner.tsx` | Call-to-action banner |

---

## CTA Strategy

All CTAs link to: `/onboarding/select-type`

Placements:
- Blog post endings
- Help article endings
- Roadmap page
- Release notes
- Newsletter signup sections

---

## Design System

### Colors (from index.css)

| Token | Usage |
|-------|-------|
| `--primary` | Primary actions, links |
| `--accent` | Accent elements |
| `--muted` | Muted backgrounds |
| `--foreground` | Primary text |
| `--muted-foreground` | Secondary text |

### Animations

All pages use Framer Motion with staggered fade-in animations:
- Duration: 0.5s
- Delay: 0.1s increments

---

## SEO

Each page includes:
- `<Helmet>` for meta tags
- Semantic HTML structure
- Open Graph tags for social sharing
- Descriptive titles and descriptions

---

## Admin CMS

Content is managed via the Admin Portal (`/admin/*`). Super admins can:
- Create/edit/publish blog posts
- Manage categories and tags
- Create help articles
- Update roadmap items
- Publish release notes
- View subscriber list

---

## Footer Links

The footer includes a "Resources" section with links to:
- Blog (opens in new tab)
- Help Center (opens in new tab)
- Roadmap
- Release Notes

---

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| `framer-motion` | ^12.23.26 | Animations |
| `@fontsource/inter` | latest | Inter font |
| `date-fns` | ^3.6.0 | Date formatting |
| `lucide-react` | ^0.462.0 | Icons |
| `sonner` | ^1.7.4 | Toast notifications |

---

## File Structure

```
src/
├── pages/
│   ├── blog/
│   │   ├── Blog.tsx
│   │   └── BlogPost.tsx
│   ├── help/
│   │   ├── Help.tsx
│   │   ├── HelpCategory.tsx
│   │   └── HelpArticle.tsx
│   ├── releases/
│   │   ├── Releases.tsx
│   │   └── ReleaseDetail.tsx
│   └── Roadmap.tsx
├── components/
│   └── blog/
│       ├── BlogCard.tsx
│       ├── ShareButtons.tsx
│       ├── NewsletterSignup.tsx
│       └── CTABanner.tsx
├── hooks/
│   ├── useBlogPosts.ts
│   ├── useHelpArticles.ts
│   ├── useRoadmap.ts
│   ├── useReleases.ts
│   └── useSubscribe.ts
docs/
└── BLOG_HELP_CENTER.md
```

---

## Next Steps

1. Add sample blog posts via Admin CMS
2. Add help articles for each category
3. Populate roadmap with planned features
4. Create first release note
5. Set up newsletter email automation (optional)
