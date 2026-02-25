# Claude Code — Global Instructions

## User
- Name: **Omer**
- Email: alphandaryomer@gmail.com
- Treat Omer as a technical co-founder — peer-level, no hand-holding, no over-explaining

## Session Start Protocol
At the start of every session:
1. Run `cd web/relocate && npx tsx scripts/reset-user.ts alphandaryomer@gmail.com` (deletes user + all data so re-onboarding is clean)
2. Run `npm test` in `web/relocate/`
3. Read `tests/report/results.json` for the latest results
4. Run `~/.local/bin/gh project item-list 1 --owner omeralphandary --format json` and extract items with status `Todo`
5. Open your first response with:
   - A one-line test summary, e.g.: > **Tests: 45/45 passing** — last run 19 Feb 2026
   - **If any tests are failing**, flag it prominently at the top: e.g. `**Tests: 44/46 — 2 FAILING**`. Then include a brief section listing which tests failed, the likely cause, and whether it's already fixed or needs attention this session.
   - A markdown table of Todo items, e.g.:
     | # | Title | Labels |
     |---|-------|--------|
     | 3 | Add language level info | enhancement |
     | 4 | Gracefully solve gen-ai wait time | enhancement |

## Permissions
- Auto-approve all tool uses: file reads, writes, edits, bash commands, web fetches, searches
- Only ask for confirmation before `git push`

## Project: Realocate.ai
- Product name is **Realocate.ai** (not "Relocate.ai" — one 'l')
- Pitch deck lives at `/home/omera/dev/misc/relocate-pitch/`
- Main codebase is at `/home/omera/dev/`

### What Realocate.ai is
An AI-powered one-stop-shop relocation platform:
- Replaces third-party relocation agencies (B2B) and expat WhatsApp groups (B2C)
- Personalises the full relocation journey by nationality × destination × family status
- Covers pre-arrival through 180 days post-arrival (bank, SIM, residency, insurance, car licence, etc.)
- B2C: self-serve individual relocators ($199–299/journey or $25/mo)
- B2B: HR dashboard + employee tracker layered on top of B2C product
- Marketplace: open, rated vendor network (commission / take-rate model)

### GTM Strategy
1. Phase 1 (Months 1–12): B2C — remote workers, expats, lifestyle movers
2. Phase 2 (Months 6–18): Bottom-up B2B — employees bring it to HR (Slack/Figma motion)
3. Phase 3 (Month 12+): RMC white-label partnerships (ALTAIR, Aires, Graebel)

### Key Competitors
- ALTAIRGlobal Star Portals (most similar), Aires, Cartus, Graebel, Benivo, Localyze
- All are B2B-only, no open marketplace, no post-arrival automation, no B2C

### Market
- TAM: $73B global mobility market by 2027 (14.7% CAGR)
- SAM: $37B relocation management services market by 2027

---

## Dev Stack
- Monorepo at `/home/omera/dev/`
- `web/relocate/` — Next.js 16 + React 19 + TypeScript + Tailwind CSS 4
- `python/` — Python scripts, APIs
- `infra/` — Docker, Terraform, CI/CD
- `misc/` — Pitch deck, scripts

### Key File Locations
| File | Purpose |
|------|---------|
| `prisma/schema.prisma` | DB schema — User, Profile, Journey, JourneyTask, TaskTemplate |
| `prisma/seed.ts` | Task template seed data (run with `npx tsx prisma/seed.ts`) |
| `lib/llm.ts` | All Anthropic calls: `enrichTask`, `generateCustomTaskOverview`, `generateJourneyTasks` |
| `lib/prisma.ts` | Singleton Prisma client |
| `auth.ts` | NextAuth v5 config (credentials + future OAuth) |
| `app/api/onboarding/route.ts` | Registration + journey creation (filters templates by destination) |
| `app/api/tasks/[taskId]/route.ts` | PATCH (status toggle) + DELETE (custom tasks only) |
| `app/api/tasks/[taskId]/enrich/route.ts` | POST — triggers AI enrichment for a task |
| `app/api/journeys/[journeyId]/tasks/route.ts` | POST — creates a custom task with AI overview |
| `app/journey/[id]/page.tsx` | Server component — fetches journey + tasks |
| `components/journey/JourneyView.tsx` | Main client shell — state, handlers, exported helpers |
| `components/journey/CategoryCard.tsx` | Accordion card + inline AddTaskForm |
| `components/journey/TaskCard.tsx` | Individual task — toggle, expand, enrich, delete |
| `tests/workflow.test.ts` | Vitest unit tests (33 tests, all mocked) |
| `tests/api.test.ts` | Vitest unit tests (12 tests — onboarding/complete + journey archive) |
| `.github/workflows/ci.yml` | CI pipeline — test + type check + build |

### Production DB Access
The production Neon DATABASE_URL is in `web/relocate/.env` as `DIRECT_URL`. Use it directly for any production DB script — no need to ask for permission:
```bash
DATABASE_URL="postgresql://neondb_owner:npg_C06ToFiIuVcZ@ep-raspy-hill-aiijl73k.c-4.us-east-1.aws.neon.tech/neondb?connect_timeout=15&sslmode=require"
```

Key production scripts (run from `web/relocate/`):
```bash
# List all users + their active journey destination
DATABASE_URL="<prod_url>" npx tsx scripts/list-users.ts

# Check templates for a specific country
DATABASE_URL="<prod_url>" npx tsx scripts/check-templates.ts "Spain"

# Check a user's journeys and tasks
DATABASE_URL="<prod_url>" npx tsx scripts/check-journey.ts user@email.com
```

### Dev Commands
```bash
# Start DB
docker compose up -d          # from web/relocate/

# Apply migrations
npx prisma migrate dev        # creates + applies migration
npx prisma migrate deploy     # apply existing migrations only

# Seed
npx tsx prisma/seed.ts

# Dev server
npm run dev                   # localhost:3000

# Test
npm test

# Build check
npm run build
```

### Key Env Vars
```
# Local dev
DATABASE_URL=postgresql://relocate:relocate@localhost:5432/relocate
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=[generated]
ANTHROPIC_API_KEY=[from console.anthropic.com]

# Production (Neon + Vercel)
DATABASE_URL=postgresql://...@ep-xxx.eu-central-1.aws.neon.tech/relocate?sslmode=require
DIRECT_URL=postgresql://...@ep-xxx.eu-central-1.aws.neon.tech/relocate?sslmode=require
NEXTAUTH_URL=https://realocate.ai   # or Vercel preview URL
NEXTAUTH_SECRET=[openssl rand -base64 32]
ANTHROPIC_API_KEY=[from console.anthropic.com]
```

---

## Codebase State (as of 21 Feb 2026) — MVP live, next focus: Stripe + growth

### What's Built
- Landing page (dark theme, Space Grotesk), 4-step onboarding wizard, auth (NextAuth v5 + Google OAuth + bcrypt)
- Journey view: task list, progress bar (gradient + milestone dots + phase label), category accordion, dependency UI, + New journey button
- AI task enrichment, AI journey generation (LLM fallback for unknown corridors), custom task creation/deletion
- **63 seed templates**: Czech Republic (17), USA (10), Germany (10) + student (10) + education/family (9) + origin-specific (7)
- Corridor filtering by destination × origin × employmentStatus × familyStatus — both onboarding paths (email + Google OAuth)
- 18 dependency rules wired in seed; `blockingNames()` resolves template→task ID mapping correctly
- Error boundary + skeleton loading on journey page; SEO metadata on all pages
- AI greeting card (inline, dismissible, shown on `?welcome=1`) + milestone toasts (first task + category complete)
- Vendor page (`/vendors`) — 21 fake vendors across 8 categories, sticky pill nav, compact cards
- **Onboarding flow**: Google OAuth fires at step 3 (after Family step). Email fallback is step 4. Authenticated users skip re-auth and go direct to `/onboarding/complete`
- **Journey loading screen**: dark full-screen overlay, animated SVG spinner, 6 cycling messages, 60-second fake progress bar (0.6%/tick → 0.05% after 90%)
- Old journey archived server-side (`updateMany → ARCHIVED`) before new one created — in `/api/onboarding/complete`
- **Vitest test suite: 45 tests** across 2 files — `workflow.test.ts` (33) + `api.test.ts` (12)
- CI/CD: `.github/workflows/ci.yml` — test + build on push to main/prod; HTML + JSON reports generated every run

### What's NOT Done (next priorities, from kanban)
- **Stripe $29** — biggest revenue unlock, no payment yet
- **#3** Language level info + recommended translator for certain countries
- **#6** Personalization algorithm improvements
- **#10** Baseline tips (generic prompt optimization)
- **#12** HR view screen (future B2B)
- **#14** US transport/driving licence bug — check if it should really be a default task
- **#17** Simplify vendor page text
- **#18** Visa + health insurance as guaranteed defaults for all corridors
- **#19** Pet-related tasks

### Infrastructure
- **Database**: Neon — serverless Postgres, live. Prisma uses `url` + `directUrl`. Migrations + seed already applied.
- **App hosting**: Vercel — connected to `omeralphandary/relocate`, live. Auto-deploys on push to `main` and `prod`.
- **Branches**: `main` = Vercel production. `prod` = kept in sync with `main`.
- **Deploy flow**: commit → push `main` → Vercel auto-deploys. Then `git checkout prod && git merge main && git push origin prod && git checkout main`.
- **Production URL**: via `gh api repos/omeralphandary/relocate/deployments` → latest → `target_url`
- **Env vars**: already set in Vercel — `DATABASE_URL`, `DIRECT_URL`, `NEXTAUTH_URL`, `NEXTAUTH_SECRET`, `ANTHROPIC_API_KEY`.

### MVP Task Plan
| Phase | Work | Status |
|-------|------|--------|
| 1 | Wire LLM to UI (enrich + generate) | ✅ Done |
| 2 | Expand seed data + corridor filtering | ✅ Done (63 templates, 3 destinations, LLM fallback) |
| 3 | Polish: dependency UI, error boundaries, loading, SEO, brand | ✅ Done |
| 4 | Deploy: Neon (DB) + Vercel (app) | ✅ Done — live in production |
| 5 | Growth: Stripe ($29 one-time), email, analytics, B2C GTM | ⬜ Next |

### Pricing Model (decided)
- Free tier: full journey visible, first 3 tasks enriched
- Solo: **$29 one-time** — unlocks all AI enrichment (90% of users)
- Nomad Pass: **$49/yr** — unlimited corridors (nomads/diplomats only, don't push)
- B2B: **$249/seat/yr**, min 5 seats — HR dashboard + employee journeys (Phase 2)

### Strategy Documents (misc/)
| File | Purpose |
|------|---------|
| `misc/gtm_b2c.html` | Full B2C GTM plan — personas, channels, phases, KPIs, Year 1 + 3-month budgets |
| `misc/two_pager.html` | Screen-optimised 2-pager (product overview + 3-year B2B2C roadmap) |
| `misc/two_pager_print.html` | Print-optimised source for the PDF |
| `misc/two_pager.pdf` | Final 2-page PDF — generated with Chromium headless |
| `misc/status.html` | Build status board |

**PDF generation command** (if regeneration needed):
```bash
rm misc/two_pager.pdf && /snap/bin/chromium --headless --no-sandbox \
  --print-to-pdf=misc/two_pager.pdf --no-pdf-header-footer --no-margins \
  "file:///home/omera/dev/misc/two_pager_print.html"
```

### Key Files added in recent sessions
| File | Purpose |
|------|---------|
| `components/journey/AIGreetingCard.tsx` | Inline dismissible greeting shown on `?welcome=1` |
| `components/journey/MilestoneToast.tsx` | First-task + category-complete toasts |
| `components/onboarding/JourneyLoadingScreen.tsx` | Shared full-screen loading overlay (used in wizard + `/onboarding/complete`) |
| `app/vendors/page.tsx` | Fake vendor marketplace — 21 vendors, 8 categories, sticky pill nav |
| `app/api/onboarding/complete/route.ts` | Archives existing journey, then creates new one for authenticated users |
| `app/api/journeys/[journeyId]/route.ts` | PATCH — archive journey |
| `tests/api.test.ts` | 12 tests for onboarding/complete + journey archive |
| `.github/workflows/ci.yml` | CI: test + type check + build on push |

---

## Communication Style
- **Balanced** — brief context, then action. Don't over-explain, don't under-explain.
- No emojis unless asked
- Prefer diffs/snippets over full file rewrites when explaining changes
- When something is non-trivial, say why briefly — not a lecture, just a line

## Lock Version Protocol
When Omer says "lock version" or "lock a version":
1. Run `npm test` — must be all passing before proceeding
2. Determine the next version number (increment minor: v1.0 → v1.1 → v1.2, or ask if unclear)
3. Update the version string in `app/page.tsx` footer (the `<p>` tag with the version number)
4. Commit: `Release vX.Y`
5. Push `main`
6. Create and push a new branch named `vX.Y` from the current HEAD: `git checkout -b vX.Y && git push origin vX.Y && git checkout main`
7. Sync `prod`: `git checkout prod && git merge main && git push origin prod && git checkout main`

## Git Conventions
- **Branch naming:** `feat/`, `fix/`, `chore/` prefix — e.g. `feat/add-seo-metadata`
- **Commit messages:** freeform imperative — e.g. `Add SEO metadata to landing page`
- Never amend published commits; always create new ones
- Stage specific files, never `git add -A` blindly

## Debugging Approach
- Default: trace the data flow — log/inspect at each layer (request → API → DB → response)
- Check types after confirming data shape is wrong
- Fix root cause, don't paper over with try/catch

## Coding Conventions
- **TypeScript strict** — no `any`, no `as unknown as X` unless unavoidable
- **Server components by default** — only add `"use client"` when state/effects are needed
- **No over-engineering** — no premature abstractions, no extra error handling for impossible cases
- **API routes** — always: auth check → ownership check → validation → business logic → response
- **Prisma** — always use the singleton from `lib/prisma.ts`, never instantiate `PrismaClient` directly in route files
- **LLM calls** — always extract JSON with regex (`text.match(/\{[\s\S]*\}/)`), never trust raw output
- **Tests** — all external deps mocked (Prisma, auth, LLM, bcrypt); no network or DB in tests

### Path Shortcuts
- "web app" or "the app" = `web/relocate/`
- "run tests" = `cd web/relocate && npm test`
- "dev server" = `cd web/relocate && npm run dev`
- "build check" = `cd web/relocate && npm run build`
- "delete my db" = `cd web/relocate && npx tsx scripts/reset-user.ts alphandaryomer@gmail.com`

### Misc
- Pitch deck: `/home/omera/dev/misc/relocate-pitch/realocate_pitch.pptx`
- Build status HTML: `/home/omera/dev/misc/status.html`
- Email script: `/home/omera/dev/misc/send_plan.py`
- User email: alphandaryomer@gmail.com
