# Headless WordPress + Next.js — Plan

## Goal

Use WordPress as the content back end (editors keep the familiar admin) and
serve the public site from a TypeScript Next.js front end.

```
Editors ──> WordPress (cms.example.com) ──GraphQL──> Next.js (example.com) ──> Visitors
                      │                                  ▲
                      └── save_post webhook ── revalidate┘
```

## Stack

| Layer      | Choice                                   |
| ---------- | ---------------------------------------- |
| CMS        | WordPress (Docker locally)               |
| API        | WPGraphQL                                |
| Fields     | ACF + WPGraphQL for ACF                  |
| SEO        | Yoast SEO + WPGraphQL Yoast              |
| Front end  | Next.js (App Router), TypeScript         |
| Hosting    | Vercel (FE), any PHP host (WP)           |

## Proposed structure

```
wordpress-FE-nextjs/
├── plan.md
├── docker-compose.yml      # WordPress + MySQL for local dev
├── wordpress/              # mu-plugins (webhook, headless redirect)
└── web/                    # Next.js app
    ├── app/
    │   ├── page.tsx                 # home
    │   ├── blog/page.tsx            # post list
    │   ├── blog/[slug]/page.tsx     # single post
    │   ├── [slug]/page.tsx          # WP pages
    │   ├── api/revalidate/route.ts  # webhook target
    │   ├── api/preview/route.ts     # draft mode entry
    │   └── sitemap.ts
    ├── lib/wp.ts                    # GraphQL fetch helper
    └── .env.local                   # WP_GRAPHQL_URL, REVALIDATE_SECRET, ...
```

## Phases

### Phase 1 — Local WordPress
- [ ] `docker-compose.yml` with `wordpress` + `mysql`
- [ ] Install WordPress, set permalinks to "Post name"
- [ ] Install WPGraphQL, ACF, WPGraphQL for ACF, Yoast, WPGraphQL Yoast
- [ ] Add sample posts, pages, categories, a menu
- [ ] Verify queries in GraphiQL (`/wp-admin` → GraphQL IDE)

### Phase 2 — Next.js skeleton
- [ ] `npx create-next-app@latest web --typescript`
- [ ] `lib/wp.ts` fetch helper with cache tags
- [ ] Home, blog list, single post, generic page routes
- [ ] `generateStaticParams` for posts/pages
- [ ] Sanitize WP HTML content (`sanitize-html`) before rendering
- [ ] Optional: GraphQL Codegen for typed queries

### Phase 3 — Content freshness
- [ ] `api/revalidate` route: verify secret, call `revalidateTag`
- [ ] WP mu-plugin: on `save_post` / `delete_post`, POST to the route
- [ ] Fallback time-based `revalidate` (e.g. 1 hour)

### Phase 4 — Previews
- [ ] WP application password or JWT for authenticated draft queries
- [ ] `api/preview` route enabling `draftMode()`
- [ ] Point WP "Preview" button at the Next.js preview URL

### Phase 5 — Site features
- [ ] Menus via `menuItems`
- [ ] SEO: `generateMetadata()` from Yoast fields, `sitemap.ts`, `robots.ts`
- [ ] Images: `next/image` with WP domain in `images.remotePatterns`
- [ ] 404 handling, pagination, category/tag archives
- [ ] Search (WP `search` arg, or external index later)

### Phase 6 — Deploy
- [ ] Host WordPress; restrict public access to admin + GraphQL
- [ ] Headless redirect: WP front end → Next.js site
- [ ] Deploy `web/` to Vercel with env vars
- [ ] Point webhook at production revalidate URL
- [ ] Smoke test: publish → page updates; preview works

## Decisions to make

- **Faust.js vs hand-rolled:** Faust.js gives previews/auth/template routing
  out of the box; hand-rolled gives full control. Default: hand-rolled.
- **REST vs GraphQL:** default GraphQL; REST needs no plugins.
- **Comments / forms:** WP comments via GraphQL mutations, or a third-party
  service.
- **Gutenberg blocks:** render raw HTML (simple) or parse blocks into React
  components (richer, more work).

## Risks

- Plugins that change the WP front end (page builders, shortcodes, forms) do
  not carry over automatically.
- Preview/auth setup is the most error-prone step.
- Raw HTML from WP must be sanitized if authors are not fully trusted.
