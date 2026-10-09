# FIT X FREAK – Elite Performance & Luxury Fitness

SEO-first, feature-based **Next.js 16 (App Router) + React 19 + TypeScript + Tailwind CSS v4** site.
Every page is **statically pre-rendered** (SSG), so crawlers receive complete HTML with unique
`<title>`, description, canonical, Open Graph and JSON-LD per URL.

## Quick start

```bash
npm install
cp .env.example .env.local     # set NEXT_PUBLIC_SITE_URL + LEADS_WEBHOOK_URL
npm run dev                    # http://localhost:3000
npm run check                  # typecheck + lint + tests
npm run build && npm start     # production build
```

Requires Node ≥ 20.9 (see `.nvmrc`).

## Routes

| Route | Purpose | Schema.org |
| --- | --- | --- |
| `/` | Home hub (hero, why-us, teasers with internal links) | HealthClub + WebSite (site-wide) |
| `/equipment` | Full catalog | Breadcrumb |
| `/equipment/category/[category]` | Crawlable category filters (cardio, strength, functional, legs) | Breadcrumb |
| `/equipment/[slug]` | Equipment detail + specs + related | Breadcrumb |
| `/locations` | Branches + live opening status | Breadcrumb |
| `/locations/[slug]` | Branch detail, hours, on-site equipment | **ExerciseGym** + opening hours |
| `/trainers`, `/trainers/[slug]` | Trainer list / profile | **Person** |
| `/membership` | Plans & pricing | **OfferCatalog** |
| `/gallery`, `/success-stories`, `/about` | Content pages | Breadcrumb |
| `/contact` | Enquiry form (`?interest=&location=&note=` prefill) | Breadcrumb |
| `/api/leads` | Validated lead intake (POST) | – |
| `/og?title=` | Dynamic social card (PNG) | – |
| `/sitemap.xml`, `/robots.txt` | Generated from the data layer | – |

## Architecture

```
src/
├── app/                  # Routing only: thin pages that compose features + export metadata
├── features/             # One folder per product feature (vertical slices)
│   ├── equipment/        #   components/ · data.ts · queries.ts · types.ts
│   ├── locations/        #   + hours.ts (open-now logic, also feeds JSON-LD)
│   ├── trainers/ membership/ success-stories/ gallery/ about/ home/
│   └── contact/          #   LeadForm, zod schema (shared client + server), constants
├── components/
│   ├── layout/           # Header, Footer, FloatingActions
│   ├── ui/               # Button, Section, PageHeader, Modal, RemoteImage, Icon
│   └── seo/              # JsonLd, Breadcrumbs
├── config/               # site.ts (brand, contact, URL), navigation.ts, brand-assets.ts
├── lib/                  # seo.ts (buildMetadata), schema.ts (JSON-LD builders), leads.server.ts, utils.ts
└── styles/globals.css    # Tailwind v4 theme tokens
```

**Rules of the codebase**

- Pages never contain content or markup soup – they import from `features/*`.
- Content lives in `features/*/data.ts`; components read it through `queries.ts`.
  Swapping to a CMS / DB later means changing `queries.ts` only.
- All metadata goes through `lib/seo.ts#buildMetadata`; all JSON-LD through `lib/schema.ts`.
- Brand/contact details exist in exactly one place: `config/site.ts`.
- Only interactive pieces are Client Components (header, modal, gallery lightbox, form, countdown, open-now badge).

## What changed vs. the original single-file app

- **1,449-line `App.tsx` → feature modules**; one-page hash scrolling → real URLs.
- **SEO:** per-page title/description/canonical/OG/Twitter, JSON-LD (Org, ExerciseGym, Person, OfferCatalog, Breadcrumb), sitemap, robots, dynamic OG image, `lang`, semantic landmarks, a single `<h1>` per page, skip link, `generateStaticParams` + 404 for unknown slugs.
- **Performance:** `next/image` (AVIF/WebP, lazy, sized, LCP-priority hero instead of a CSS background), self-hosted fonts via Fontsource, Material Symbols icon font (≈400 KB) replaced by `lucide-react`.
- **Correctness fixes:** fake always-"Open Now" badge → real open/closed from structured hours + timezone; looping fake countdown → real date from config (auto-hides); footer listed a non-existent "Westside Arena" – now generated from location data; hard-coded "© 2024" → dynamic year; Marina Bay hours summary now matches its weekend hours; clickable `div`s → buttons/links.
- **Lead form:** zod validation (client + server), honeypot, `/api/leads` webhook delivery with loud failure if unconfigured.
- Removed unused deps (`@google/genai`, `express`, `dotenv`, `motion`, `vite` toolchain) and AI-Studio-only files.
- Security headers, ESLint, Prettier, Vitest, GitHub Actions CI.

## Before going live (checklist)

1. Set `NEXT_PUBLIC_SITE_URL` to the real domain.
2. Set `LEADS_WEBHOOK_URL` (form returns an error in production without it).
3. **Host the images yourself.** Images are still hot-linked from Google's `aida-public` CDN (allow-listed in `next.config.ts`). Download them into `public/images/` and update the `data.ts` files – ownership + long-term stability.
4. Replace placeholders: member-portal capacity (`liveCapacityPercent`), phone `+1-800-555-FREAK`, hero stats (25+/150+/1000+), challenge date, and add real postal codes + geo coordinates to locations for stronger local SEO.
5. Add a real favicon set / `apple-touch-icon` (currently a simple `app/icon.svg`).
6. Submit `/sitemap.xml` to Google Search Console; claim a Google Business Profile per branch.
7. Add rate limiting (e.g. Upstash) and a CSP header in front of `/api/leads` if you expect abuse.
