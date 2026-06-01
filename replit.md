# Solace — Recovery & Wellness App

A full-stack AI-powered recovery and wellness app with Finch + Discord + Duolingo vibes. Solace gives users an AI companion to talk to, gamified wellness quests, journaling with AI responses, achievements, social rooms, and a personal dashboard.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080 → served at `/api`)
- `pnpm --filter @workspace/wellness-app run dev` — run the frontend (Vite, served at `/`)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL`, `SESSION_SECRET`, `AI_INTEGRATIONS_ANTHROPIC_BASE_URL`, `AI_INTEGRATIONS_ANTHROPIC_API_KEY`

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React + Vite, Tailwind CSS, shadcn/ui, Framer Motion, wouter routing
- API: Express 5
- DB: PostgreSQL + Drizzle ORM
- AI: Anthropic Claude (`claude-haiku-4-5`) via Replit AI Integrations
- Auth: `express-session` + `bcryptjs` — cookie-based session auth
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec at `lib/api-spec/openapi.yaml`)
- Build: esbuild (CJS bundle)

## Where things live

- `lib/api-spec/openapi.yaml` — source-of-truth OpenAPI contract
- `lib/api-client-react/src/generated/api.ts` — auto-generated React Query hooks + fetchers
- `lib/api-client-react/src/custom-fetch.ts` — fetch wrapper (credentials: "include" for session cookies)
- `lib/db/src/schema/` — Drizzle ORM schemas: users, quests, journal, achievements, companion, rooms
- `artifacts/api-server/src/routes/auth.ts` — register/login/logout/session + `formatUser()` helper
- `artifacts/api-server/src/lib/levels.ts` — 10-tier level system (Seedling → Luminous, non-linear XP)
- `artifacts/api-server/src/lib/auth-middleware.ts` — `requireAuth` middleware
- `artifacts/api-server/src/routes/` — Express route handlers (auth, users, quests, journal, achievements, companion, rooms, dashboard)
- `artifacts/wellness-app/src/pages/` — React pages: auth, home, companion, quests, quest-flow, journal, journal-new, achievements, rooms, room-chat, profile
- `artifacts/wellness-app/src/lib/auth-context.tsx` — AuthProvider + useAuth hook
- `artifacts/wellness-app/src/components/layout.tsx` — sidebar + mobile nav (shows real user avatar + level)

## Architecture decisions

- **Multi-account auth**: Cookie-based session auth via `express-session` + `bcryptjs`. `req.session.userId` used in all protected routes. `SESSION_SECRET` env var required.
- **Level system**: 10 named tiers (Seedling, Sprout, Explorer, Wanderer, Grower, Seeker, Nurturer, Bloom, Radiant, Luminous) with non-linear XP thresholds (0, 200, 500, 900, 1400, 2000, 2700, 3500, 4500, 6000). `getLevelInfo(xp)` returns `{level, levelName, levelEmoji, xpThisLevel, xpNeeded, xpToNextLevel, isMaxLevel}`.
- **formatUser()**: Exported from `routes/auth.ts`. Use it everywhere a full User object is returned (users, dashboard). Includes all level fields.
- **AI companion**: Uses `claude-haiku-4-5` with 5 modes (calm, motivation, crisis, late_night, focus) — each has distinct system prompt personality. Falls back to static warm messages if AI env vars are missing.
- **Contract-first**: OpenAPI spec is the single source of truth. Always run codegen after changing the spec.
- **Anthropic SDK**: `@anthropic-ai/sdk` must be in `artifacts/api-server/package.json` `dependencies` (not just root), because esbuild bundles it.
- **Fetch credentials**: `customFetch` in `lib/api-client-react/src/custom-fetch.ts` sends `credentials: "include"` — required for session cookie auth with React Query.

## Product

- **Auth** — Register/login with username + password + avatar emoji picker. Session persists 30 days.
- **Dashboard** — XP level bar showing level name (e.g. "🌱 Seedling"), companion greeting, daily quest cards, breathing reset button
- **AI Companion** — chat with Anthropic-powered companion in 5 distinct emotional modes (calm/motivation/crisis/late_night/focus)
- **Quests** — 8 wellness quests across mindfulness, outdoors, social, creativity, reflection, movement categories; multi-step quest flow; XP rewards on completion
- **Journal** — mood-tagged entries with AI responses from Claude; create + delete entries
- **Achievements** — 15 achievements across emotional, social, growth, fun categories; unlocked by completing quests / journal entries
- **Social Rooms** — 6 community rooms with real-time polling chat; messages use real user name + avatar
- **Profile** — editable display name, bio, avatar emoji grid (24 options); XP/level stats with named tier; skills breakdown; achievement grid

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- `@anthropic-ai/sdk` must be in api-server's own `dependencies` — esbuild bundles it and won't find it at the workspace root.
- Do NOT run `pnpm dev` at workspace root — use `restart_workflow` instead.
- `customFetch` must have `credentials: "include"` or session cookies won't be sent by React Query.
- `formatUser()` is exported from `routes/auth.ts` — import from there for consistent User shape.
- `xpThisLevel` and `xpNeeded` come from `getLevelInfo()` — don't compute level bar as `xp % 200` anymore.
- All protected routes use `requireAuth` middleware from `lib/auth-middleware.ts`.

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
