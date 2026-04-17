# Inkwell Plugin Architecture

**Date:** 2026-04-17
**Author:** Kasra
**Status:** Draft — for Hadi review

---

## Part 1: Plugin Architecture Decision

### Recommendation: Option B — Plugin Directory (with config registration)

**Reject A (monolithic flags):** The current approach works for 5–8 features. It breaks when you have 40. Every fork carries dead code for verticals they don't use. DNU features like CE credits and data sovereignty have no place in a generic Inkwell config.

**Reject C (npm packages):** Correct long-term, wrong now. Publishing, versioning, and `npm install` friction slow down a small team shipping fast. The plugin surface isn't stable enough yet to freeze into packages.

**Reject D (Astro integrations):** Astro integrations are good for infra (analytics, sitemap, image optimization) but clunky for domain features. They require `astro.config.mjs` hooks, a separate package, and don't cover the Worker side at all. You'd still need a parallel plugin system for routes.

**Why B wins:**
- Plugins live in the repo — no publish step, no versioning friction
- Each plugin is self-contained: components, routes, MCP tools, migrations, config schema
- `inkwell.config.ts` is still the single entry point — it lists active plugins
- Easy to fork: clone, delete the plugins you don't need, done
- Works on Cloudflare Workers (dynamic module loading via static imports gated by config)
- Astro pages/components are imported at build time based on plugin presence

---

### Directory Structure

```
inkwell/
  inkwell.config.ts          ← registers plugins
  plugins/
    geo/
      plugin.ts              ← plugin manifest (name, version, config schema)
      components/            ← Astro + React components
        GeoMatrix.astro
        LocationDetect.tsx
      pages/                 ← Astro page templates
        [city]/[service].astro
      routes/                ← Hono route modules
        geo.ts
      migrations/            ← D1 SQL migrations
        001_geo_cities.sql
        002_geo_services.sql
      mcp/                   ← MCP tool definitions
        tools.ts
      types.ts               ← TypeScript types for this plugin
    topics/
      plugin.ts
      components/
        TrustScore.tsx
        ConsensusBar.tsx
        ExpertCarousel.tsx
      pages/
        topics/[slug].astro
      routes/
        topics.ts
      migrations/
        001_topics.sql
        002_trust_scores.sql
      mcp/
        tools.ts
    ghl/
      plugin.ts
      routes/
        ghl.ts
      mcp/
        tools.ts
    partner-dashboard/
      plugin.ts
      components/
        OnboardingWizard.tsx
        PartnerAnalytics.tsx
        DandanCopilot.tsx
      pages/
        dashboard/partner/
      routes/
        partner.ts
      migrations/
        001_partner_roles.sql
    publisher-pro/
      plugin.ts
      routes/
        publish.ts
      migrations/
        001_content_pipeline.sql
      mcp/
        tools.ts
    data-sovereignty/
      plugin.ts
      components/
        DataEarningsDashboard.tsx
      routes/
        sovereignty.ts
      migrations/
        001_consent.sql
        002_earnings.sql
  src/                       ← core (never grows for vertical features)
  workers/inkwell-api/src/   ← core Worker
```

---

### Plugin Manifest (`plugin.ts`)

Each plugin exports a manifest. This is the contract between plugin and core.

```typescript
// plugins/geo/plugin.ts
import type { InkwellPlugin } from '../../src/lib/plugin-types'

export const geoPlugin = {
  name: 'geo',
  version: '1.0.0',
  description: 'City × service geo matrix with location detection',

  // Config schema (Zod) — merged into inkwell.config.ts plugin block
  configSchema: z.object({
    cities: z.array(z.object({
      slug: z.string(),
      name: z.string(),
      lat: z.number(),
      lng: z.number(),
      region: z.string().optional(),
    })),
    services: z.array(z.object({
      slug: z.string(),
      name: z.string(),
    })),
    defaultRadius: z.number().default(50),
    locationDetect: z.boolean().default(true),
  }),

  // D1 migrations to run on first deploy
  migrations: ['./migrations/001_geo_cities.sql', './migrations/002_geo_services.sql'],

  // MCP tools this plugin adds
  mcpTools: () => import('./mcp/tools'),

  // Hono routes this plugin adds (mounted by core at /api/geo)
  routes: () => import('./routes/geo'),
} satisfies InkwellPlugin
```

---

### How `inkwell.config.ts` Enables Plugins

```typescript
// inkwell.config.ts
import { geoPlugin } from './plugins/geo/plugin'
import { topicsPlugin } from './plugins/topics/plugin'
import { ghlPlugin } from './plugins/ghl/plugin'

export const config = {
  name: 'DentalNearYou',
  domain: 'dentalnear.you',

  // ... core config unchanged ...

  plugins: [
    {
      plugin: geoPlugin,
      config: {
        cities: [
          { slug: 'toronto', name: 'Toronto', lat: 43.65, lng: -79.38, region: 'ON' },
          { slug: 'vancouver', name: 'Vancouver', lat: 49.28, lng: -123.12, region: 'BC' },
        ],
        services: [
          { slug: 'teeth-cleaning', name: 'Teeth Cleaning' },
          { slug: 'dental-implants', name: 'Dental Implants' },
        ],
        defaultRadius: 50,
        locationDetect: true,
      },
    },
    {
      plugin: topicsPlugin,
      config: {
        trustScores: true,
        expertCarousel: true,
        consensusBar: true,
        ceCredits: false,        // DNU-specific, off by default
      },
    },
    {
      plugin: ghlPlugin,
      config: {
        oauthCallbackPath: '/api/ghl/callback',
        pipelineId: 'leads',
      },
    },
  ],
} as const
```

---

### How the Worker Loads Plugin Routes

In `workers/inkwell-api/src/index.ts`, after all core routes are mounted:

```typescript
import { config } from '../../../inkwell.config'
import type { AppBindings } from './types'

// Core routes (always present)
app.route('/api/analytics', analyticsRoutes)
app.route('/api/content', contentRoutes)
// ...

// Plugin routes (loaded from config)
// Workers can't do runtime dynamic imports, so this is a static
// import-with-gate pattern: all plugin route modules are imported
// at bundle time, but only mounted if the plugin is configured.

import { geoRoutes } from '../../../plugins/geo/routes/geo'
import { topicsRoutes } from '../../../plugins/topics/routes/topics'
import { ghlRoutes } from '../../../plugins/ghl/routes/ghl'

const activePlugins = new Set(config.plugins.map(p => p.plugin.name))

if (activePlugins.has('geo'))     app.route('/api/geo', geoRoutes)
if (activePlugins.has('topics'))  app.route('/api/topics', topicsRoutes)
if (activePlugins.has('ghl'))     app.route('/api/ghl', ghlRoutes)
```

Workers bundle at build time — dead code elimination means inactive plugins
don't increase bundle size if tree-shaking catches the branches. For safety,
keep each plugin route module under 50KB. Total worker stays under 1MB.

---

### How Astro Loads Plugin Pages and Components

In `astro.config.mjs`, add a custom integration that reads the plugin list
and injects page routes:

```javascript
// astro.config.mjs
import { config } from './inkwell.config'
import { inkwellPlugins } from './src/lib/astro-plugins'

export default defineConfig({
  integrations: [
    react(),
    inkwellPlugins(config.plugins),  // auto-registers plugin pages
  ],
})
```

`inkwellPlugins` is a thin Astro integration:

```typescript
// src/lib/astro-plugins.ts
export function inkwellPlugins(plugins): AstroIntegration {
  return {
    name: 'inkwell-plugins',
    hooks: {
      'astro:config:setup': ({ injectRoute }) => {
        for (const { plugin } of plugins) {
          const pagesDir = `./plugins/${plugin.name}/pages`
          // Astro's injectRoute adds file-based routes from a directory
          // This is supported in Astro 4+ via integration hooks
          if (existsSync(pagesDir)) {
            injectRoute({ pattern: '/', entrypoint: pagesDir })
          }
        }
      },
    },
  }
}
```

For a small team, it's acceptable to add plugin pages directly under
`src/pages/` during development and move them to `plugins/` when the
feature stabilizes. Don't over-engineer the Astro integration until
you have 3+ plugins that need it.

---

### How Plugins Add MCP Tools

The MCP route (`workers/inkwell-api/src/routes/mcp.ts`) currently has a
hardcoded `TOOLS` array. Extend it to merge plugin tools:

```typescript
// In mcp.ts, before the TOOLS const:
import { config } from '../../../inkwell.config'
import { geoMcpTools } from '../../../plugins/geo/mcp/tools'
import { topicsMcpTools } from '../../../plugins/topics/mcp/tools'

const activePlugins = new Set(config.plugins.map(p => p.plugin.name))

const PLUGIN_TOOLS: ToolDef[] = [
  ...(activePlugins.has('geo')    ? geoMcpTools    : []),
  ...(activePlugins.has('topics') ? topicsMcpTools : []),
]

const ALL_TOOLS: ToolDef[] = [...CORE_TOOLS, ...PLUGIN_TOOLS]
```

Each plugin's `mcp/tools.ts` exports a `ToolDef[]` array following the
same schema as the existing core tools. No new abstraction needed.

---

## Part 2: DNU Extraction Plan

### Plugin 1: `topics`

**Priority: Must-have for launch** — this is the content moat.

**What it does:** Expert consensus scoring, trust signals, opinion carousels, and source citations on topic pages. The trust score is what separates Inkwell topic pages from generic content sites — it's an E-E-A-T signal baked into the schema.

**Extracts from DNU:**
- `components/expert/TrustScoreTooltip.tsx` → `plugins/topics/components/TrustScore.tsx`
- `components/expert/ConsensusBar.tsx` → `plugins/topics/components/ConsensusBar.tsx`
- `components/expert/ExpertCarousel.tsx` → `plugins/topics/components/ExpertCarousel.tsx`
- `components/expert/ExpertStoryViewer.tsx` → `plugins/topics/components/ExpertStoryViewer.tsx`
- `components/expert/ExpertRing.tsx` → `plugins/topics/components/ExpertRing.tsx`
- `migrations/027_trust_score_and_semantic_indexes.sql` → `plugins/topics/migrations/001_trust_scores.sql`
- `web/lib/schema/` (Person + AggregateRating builders) → `plugins/topics/lib/schema.ts`

**Generic vs vertical-specific:**
- Generic: trust score calculation, consensus bar, expert carousel shell, opinion cards, source citations
- DNU-specific: dentist-specific schema fields (DDS credentials, license numbers, CE credits). Strip these — keep the pattern, parameterize the credential type.

**Config schema:**
```typescript
{
  trustScores: boolean          // show consensus bar on topic pages
  expertCarousel: boolean       // show expert opinion carousel
  ceCredits: boolean            // credential/continuing-ed system (off by default)
  credentialType: string        // 'dental' | 'legal' | 'financial' | 'general'
  sourceCitations: boolean      // show external sources panel
}
```

**D1 tables:** `topics`, `experts`, `expert_opinions`, `trust_scores`, `sources`

**Supabase adapter note:** DNU's Supabase schema has 36 migrations. Topics-related tables need to be recreated as D1 migrations for Inkwell. For DNU itself, keep Supabase — the adapter question is deferred until a customer with existing Supabase data needs to fork Inkwell.

**MCP tools to add:** `get_topic`, `upsert_topic_expert`, `get_consensus_score`

**Migration plan:**
1. Copy components, strip dental-specific fields, make `credentialType` a prop
2. Write D1 migrations from DNU's `027_trust_score_and_semantic_indexes.sql`
3. Add API routes: `GET /api/topics/:slug`, `POST /api/topics/:slug/opinion`
4. Wire into existing Inkwell topic pages under `src/pages/topics/[slug].astro`

---

### Plugin 2: `geo`

**Priority: Must-have for any local-service vertical**

**What it does:** Generates city × service page matrix (225+ pages from 15 × 15 config). Haversine distance sorting for "near me" queries. IP-based location detection.

**Extracts from DNU:**
- `app/[lang]/[city]/[service]/page.tsx` — route pattern → `plugins/geo/pages/[city]/[service].astro`
- `mumega-cms/ARCHITECTURE.md` — city/service config structure
- Haversine implementation (wherever it lives in DNU utils)

**Generic vs vertical-specific:**
- Everything here is generic. The city list, service list, and distance logic are all config-driven. Zero DNU-specific code survives extraction.

**Config schema:**
```typescript
{
  cities: Array<{ slug, name, lat, lng, region? }>
  services: Array<{ slug, name, category? }>
  defaultRadius: number          // km for "near me" results
  locationDetect: boolean        // IP geolocation via CF headers
  matrixPageTemplate: string     // which Astro layout to use
  hreflang: boolean              // inject hreflang for multilingual matrices
}
```

**D1 tables:** `geo_cities`, `geo_services`, `geo_matrix_pages` (cache of generated page metadata)

**How page generation works:** At build time, Astro generates static pages for each city × service combination from config. The Worker handles the dynamic "sort by distance" at request time using `cf-ipcountry` + `cf-ipcity` headers — no external geolocation API needed.

**MCP tools to add:** `get_geo_matrix_status`, `upsert_city`, `upsert_service`

**Migration plan:**
1. Write `plugins/geo/pages/[city]/[service].astro` as a parameterized template
2. At build time, generate static params from `config.plugins.geo.cities × services`
3. Worker route `GET /api/geo/nearby` accepts `lat/lng` and returns sorted providers
4. For DNU migration: extract city/service lists from their hardcoded routes into the plugin config

---

### Plugin 3: `partner-dashboard`

**Priority: Must-have if selling B2B (partners/vendors pay to be listed)**

**What it does:** Role-based dashboard routing (patient / partner / admin), partner onboarding wizard, analytics per partner, AI copilot (Dandan-style).

**Extracts from DNU:**
- `app/dashboard/partner/onboarding/page.tsx` → `plugins/partner-dashboard/pages/dashboard/partner/onboarding.astro`
- `app/dashboard/partner/` (8 pages) → `plugins/partner-dashboard/pages/dashboard/partner/`
- `app/dashboard/admin/` (10 pages) — admin ops layer

**Generic vs vertical-specific:**
- Generic: onboarding wizard (step tracker is completely generic), analytics shell, role-based routing, AI copilot shell
- DNU-specific: license verification step, GHL connect step (that's the `ghl` plugin), dental insurance fields
- Strip dental specifics. The onboarding wizard becomes a configurable step list. Each step is a component.

**Config schema:**
```typescript
{
  roles: ('partner' | 'admin' | 'patient')[]   // which dashboard layers to enable
  onboarding: {
    steps: Array<{
      id: string
      label: string
      component: string    // component key, resolved by plugin
      required: boolean
    }>
  }
  aiCopilot: {
    enabled: boolean
    model: 'gemini' | 'claude' | 'gpt'
    systemPrompt: string
  }
}
```

**D1 tables:** `partner_profiles`, `partner_onboarding_state`, `partner_analytics_snapshots`

**Role-based access:** Cloudflare Access handles authentication (already in Inkwell via CF-Access-JWT-Assertion). The tenant middleware sets `cf_access_email`. The partner-dashboard plugin adds a `roles` table in D1 mapping email → role. The Worker middleware checks role before mounting dashboard routes.

**Migration plan:**
1. Port onboarding wizard first — it's self-contained and highest value
2. Port partner analytics page (already has a shell in Inkwell dashboard)
3. Add role middleware to Worker
4. AI copilot last — it needs a model binding (Workers AI or external API)

---

### Plugin 4: `ghl`

**Priority: Nice-to-have for launch, must-have for any sales-led vertical**

**What it does:** OAuth app connecting Inkwell leads to a GHL (GoHighLevel) CRM pipeline. Handles token exchange, refresh, lead push with retry queue.

**Extracts from DNU:**
- `ghl-connector/app.py` — this is a Python Flask app, not a Cloudflare Worker. Full rewrite needed.

**Generic vs vertical-specific:**
- 100% generic. GHL's API is the same regardless of vertical. The pipeline ID and stage names are config.

**Config schema:**
```typescript
{
  clientId: string        // from wrangler secret
  clientSecret: string    // from wrangler secret
  pipelineId: string
  defaultStageId: string
  leadFieldMapping: Record<string, string>   // inkwell field → GHL field
  retryAttempts: number   // default 3
}
```

**Plugin vs core:** Plugin. GHL is one CRM option — future plugins could add HubSpot, Pipedrive, Salesforce. Keep core CRM-agnostic.

**Migration plan:**
1. Rewrite `ghl-connector/app.py` as a Hono route module in TypeScript
2. OAuth flow: `GET /api/ghl/auth` → redirect, `GET /api/ghl/callback` → token exchange → store in KV
3. Lead push: `POST /api/ghl/leads` → GHL contact create API with retry
4. Add `connect_ghl` step to partner-dashboard onboarding wizard

**Note on DNU:** DNU's ghl-connector is a separate service running on Hetzner. Once this plugin exists, DNU can decommission that service and run everything on Workers.

---

### Plugin 5: `publisher-pro`

**Priority: Nice-to-have — Inkwell's current publisher works for simple sites**

**What it does:** Supabase → Strapi → Cloudflare KV/R2 pipeline. AI-generated content at scale. Handles 15,000+ pages (DNU's full build: 2 languages × 15 cities × 15 services + topics). Inkwell's current `npm run publish` (file-drop approach) tops out around 500 pages before it gets unwieldy.

**Extracts from DNU:**
- `mumega-cms/` (Strapi config + content types)
- `publisher-mcp` (20 tools: `publisher_upsert_tenant`, `publisher_upsert_site`, etc.)
- DNU's Supabase content tables (articles, guides, topic pages)

**Generic vs vertical-specific:**
- Generic: the pipeline architecture (source → transform → publish), the MCP tools interface, the KV/R2 storage layer
- DNU-specific: the Strapi content types (dental articles, dentist profiles). Strip these — plugin takes content type definitions as config.

**Config schema:**
```typescript
{
  source: 'supabase' | 'strapi' | 'files'
  supabase?: { url: string; anonKey: string; contentTable: string }
  contentTypes: Array<{ name: string; slugField: string; bodyField: string }>
  target: 'kv' | 'r2' | 'both'
  batchSize: number         // pages per run, default 100
  aiEnrich: boolean         // run AI enrichment pass before publish
}
```

**Migration plan:**
1. Extract the 20 publisher-mcp tools into `plugins/publisher-pro/mcp/tools.ts`
2. Write a Worker route that handles the Supabase → KV/R2 pipeline
3. Add a scheduled trigger for incremental rebuilds
4. Replace Inkwell's current `npm run ingest` with a call to this plugin when active

---

### Plugin 6: `data-sovereignty`

**Priority: Deferred — post-revenue**

**What it does:** Users opt-in to sell anonymized behavioral/health data. Experts earn from content contributions. Earnings dashboard. Consent management.

**Extracts from DNU:**
- `migrations/025_data_sovereignty_marketplace.sql`
- `components/expert/DataEarningsDashboard.tsx`

**Is it a plugin?** Yes, and it should stay isolated until there's a legal framework in place (PIPEDA, GDPR consent flows) and a data buyer. The concept is sound but shipping it prematurely creates liability. Build the consent layer (a thin plugin) first, wire the earnings later.

**Migration plan:** Deferred. Tag in backlog. Revisit when first paying data buyer exists.

---

## Part 3: Execution Order

The order below prioritizes what unblocks revenue and what's cheapest to extract.

**Sprint 1 — Foundation (this week)**
1. Create `plugins/` directory with `InkwellPlugin` type in `src/lib/plugin-types.ts`
2. Write `plugins/topics/` — port TrustScore + ConsensusBar components + D1 migrations
3. Wire topics plugin into existing `src/pages/topics/[slug].astro`
4. Add topics MCP tools (3 tools: `get_topic`, `upsert_topic_expert`, `get_consensus_score`)

**Sprint 2 — Geo (next week)**
5. Write `plugins/geo/` — city × service page matrix
6. Port Haversine distance sort from DNU
7. Add `GET /api/geo/nearby` Worker route
8. Test with a 3-city × 3-service matrix before scaling to 15×15

**Sprint 3 — Partner Dashboard**
9. Port onboarding wizard from DNU (`app/dashboard/partner/onboarding/`)
10. Add role middleware to Worker (email → role via D1)
11. Port partner analytics page
12. Add AI copilot shell (model-agnostic, Gemini by default)

**Sprint 4 — GHL + Publisher Pro**
13. Rewrite ghl-connector as a Hono plugin module
14. Add `connect_ghl` step to onboarding wizard
15. Extract publisher-mcp tools into `plugins/publisher-pro/`
16. Replace `npm run ingest` with publisher-pro when active

**Deferred**
- `data-sovereignty` — post-revenue
- npm packages (`@inkwell/plugin-*`) — when 3+ customers are forking and need versioned updates
- Astro integration refactor — when plugin page injection needs to be cleaner than manual `src/pages/` additions

---

## Notes

**On Supabase vs D1:** DNU runs Supabase because it needs pgvector and row-level security for multi-tenant patient data. Inkwell uses D1 for simplicity on edge. The plugins above write D1 migrations. For a DNU fork of Inkwell that needs Supabase, add a `db: 'supabase'` option to the plugin config and a thin adapter layer — but don't build this until a customer asks for it.

**On forking:** When a customer forks Inkwell for a non-dental vertical, they delete the plugins they don't need from `inkwell.config.ts` and optionally remove the `plugins/` subdirectories. Core stays untouched.

**On bundle size:** Each plugin adds routes to the Worker bundle. Monitor with `npx wrangler publish --dry-run` — if bundle exceeds 900KB, split the Worker (separate Worker per plugin domain is valid on Cloudflare).

**On DNU migration timeline:** DNU doesn't need to migrate to Inkwell. The extraction flows one way: DNU features become Inkwell plugins. DNU can consume Inkwell plugins as a library when they're stable, or stay on Next.js — that's a separate decision.
