# Technical Documentation Overview

## Architecture & Design

### System Architecture
- **Monolithic Next.js App:** Single deployment unit combining frontend and API routes for simplicity and unified auth
- **Database-First Design:** Prisma ORM with PostgreSQL as single source of truth
- **Role-Based Access Control:** Middleware enforces team and role scoping at API layer
- **Stateless API:** Each request is independently authorized and scoped

### Directory Structure

```
app/                     # Next.js App Router pages and API routes
├── api/                 # RESTful API endpoints
│   ├── athletes/        # Athlete CRUD and filtering
│   ├── academic-records/# Academic tracking
│   ├── health-records/  # Medical records
│   ├── events/          # Calendar events
│   ├── imports/         # Reviewed CSV import workflow
│   ├── analytics/       # Performance snapshots & projections
│   └── assistant/       # AI-powered contextual queries
├── (coach pages)        # Coach workspace: overview, athletes, academics, health, events, notes
├── (athlete pages)      # Athlete portal: dashboard, calendar, academics, medical, workouts
├── track-and-field/     # Sport-specific workflows: workouts, rankings, meet entries, journals
└── components/          # Shared React components

lib/                     # Business logic & shared services
├── auth.ts              # NextAuth configuration & JWT session handling
├── rbac.ts              # Role definitions and permission checks
├── access.ts            # Team/athlete scoping helpers
├── api.ts               # Shared API response & error handling
├── analytics.ts         # Snapshot computation & persistence
├── assistant.ts         # LLM context building & safe prompting
├── imports/             # Import job service layer
├── clientApi.ts         # Client-side API wrapper (React Query)
└── hooks/               # Custom React hooks

prisma/                  # Database schema & migrations
├── schema.prisma        # Entity definitions, relations, constraints
├── seed.ts              # Development data seeding
└── migrations/          # Historical schema changes

tests/                   # Vitest suite
├── apiRoutes.test.ts    # API endpoint tests
├── rbac.test.ts         # Permission tests
└── (feature tests)      # Isolated business logic tests
```

---

## Core Concepts

### Authentication & Authorization

**Authentication** (Who are you?)
- Email + password via NextAuth.js
- JWT tokens stored in httpOnly cookies
- Password hashing with bcrypt

**Authorization** (What can you do?)
- Three roles: Athletic Director (AD), Coach, Athlete
- AD: Full institution access
- Coach: Team-scoped read/write
- Athlete: Self-service access only
- Guards: `requireRole()`, `requireSession()`, `ensureTeamAccess()`, `ensureAthleteAccess()`

### Data Access & Scoping

**Team Scoping**
- Coaches always filtered to their assigned teams
- ADs see all teams
- Filters automatically injected by middleware
- No raw Prisma queries bypass scoping

**Import Workflow**
- Coaches/ADs upload athlete roster CSV
- Preview phase: parse, normalize, validate, match against existing athletes
- Review phase: flag duplicates, ambiguities, create candidates
- Apply phase: atomic write with all-or-nothing semantics
- Result reporting: clear summary of created, updated, skipped, and failed rows

### Analytics Snapshots

**Why Snapshots?**
- Real-time analytics are slow and expensive
- Snapshots decouple data access from computation
- Pre-computed at schedule or on-demand, persisted to DB
- API reads from snapshots, not recalculating each request

**What's Included**
- `WorkoutAnalyticsSnapshot`: training load, adherence, recovery trends
- `AthletePerformanceSnapshot`: current form, recent results, projection
- `EventPerformanceTrend`: historical performance at event type, trend direction

**When Recomputed**
- After athlete roster changes
- After workout/result imports
- On manual rebuild via `npm run analytics:rebuild`
- Optional scheduled recompute in production

### Safe LLM/Assistant

**Why "Safe"?**
- No athlete PII sent to LLM
- Context built server-side from pre-aggregated snapshots
- Deterministic prompt template prevents injection
- Fallback responses available when API unavailable
- Each query scoped to authenticated user's team access

**How It Works**
1. User asks question via `/api/assistant/query` (e.g., "Which athletes are at academic risk?")
2. Backend resolves user's team scope
3. Builds context: aggregate snapshots, academic standing, recent flags
4. Constructs safe prompt (no raw athlete data, only summaries)
5. Calls LLM if available; falls back to deterministic response if not
6. Returns text response scoped to user's view

**Questions It Answers**
- "Summarize [athlete]'s academic and health status"
- "Which athletes need to be reviewed before the game?"
- "Highlight athletes at risk of ineligibility"
- "Summarize academic trends by position"

---

## API Reference

### Authentication
- `POST /api/auth/signin` – Credentials login
- `POST /api/auth/signout` – Destroy session
- `POST /api/auth/[...nextauth]` – NextAuth callback routes

### Athletes & Roster
- `GET /api/athletes?teamId=X&filters=...` – List athletes with filtering
- `GET /api/athletes/:id` – Athlete detail
- `POST /api/athletes` – Create athlete (coach/AD only)
- `PATCH /api/athletes/:id` – Update athlete
- `DELETE /api/athletes/:id` – Delete athlete (AD only)

### Academic Records
- `GET /api/academic-records?semester=Spring2025&standing=bad` – Filtered list
- `PATCH /api/academic-records/:id` – Update GPA, standing, compliance

### Health Records
- `GET /api/health-records?status=injured` – Filtered list
- `PATCH /api/health-records/:id` – Update injury status, rehab progress

### Events & Calendar
- `GET /api/events?teamId=X&start=...&end=...` – List events in date range
- `POST /api/events` – Create event
- `PATCH /api/events/:id` – Update event

### Analytics
- `GET /api/analytics/performance?athleteId=X` – Performance snapshot
- `GET /api/analytics/projections?teamId=X` – Team trend projections
- `GET /api/analytics/workouts?athleteId=X` – Workout analytics

### Imports
- `POST /api/imports` – Create import job, upload CSV, return preview
- `GET /api/imports/:jobId` – View import details
- `POST /api/imports/:jobId/apply` – Apply approved import
- `GET /api/imports?teamId=X&limit=10` – Import history

### Assistant
- `POST /api/assistant/query` – Ask contextual question about team/athlete
  - Body: `{ question: string, athleteId?: string, teamId?: string }`
  - Response: `{ answer: string, context: { ... } }`

---

## Testing

**Unit Tests** (Vitest)
- RBAC permission logic: `tests/rbac.test.ts`
- API filtering: `tests/apiFiltering.test.ts`
- Analytics computation: `tests/analyticsRoutes.test.ts`
- Assistant context building: `tests/assistant.test.ts`

**Running Tests**
```bash
npm test                # Watch mode
npm test -- --run       # Single run
npm test -- file.test.ts # Single file
```

---

## Common Development Tasks

### Add a New Athlete Field
1. Update `prisma/schema.prisma` (add field to `model Athlete`)
2. Run `npx prisma migrate dev --name add_field_name`
3. Update athlete CRUD in `app/api/athletes/route.ts`
4. Update validation in `lib/athleteValidation.ts`
5. Update UI in `app/athletes/` pages
6. Add test coverage

### Add a New API Filter
1. Update query param parsing in the route handler
2. Build Prisma `where` clause in `lib/api.ts`
3. Add tests in `tests/apiFiltering.test.ts`
4. Document in this file

### Rebuild Analytics
```bash
npm run analytics:rebuild
```
Recomputes all snapshots without data loss. Safe to run anytime.

### Seed Development Data
```bash
npx prisma db seed
```
Populates test athletes, teams, and records from `prisma/seed.ts`.

### Deploy to Production
```bash
npm run build
npm start
```
Or push to Vercel; deploys automatically on main branch.

---

## ML Export & Separation

The app includes export hooks at `scripts/exportForML.ts` that generate athlete datasets for downstream ML training pipelines. This separation ensures:
- App data remains normalized and accessible
- ML training workflows operate independently
- No real-time model inference latency in the app
- Analytics snapshots are deterministic and reliable

ML tooling lives in the separate `ml/` workspace.

---

## Debugging & Common Issues

### Permission Denied on API Call
- Check `lib/rbac.ts`: verify role has required permission
- Check `lib/access.ts`: verify athlete/team is scoped correctly
- Verify JWT session includes correct role and teamIds

### Analytics Snapshot Stale
- Run `npm run analytics:rebuild` to recompute
- Check `lib/analytics.ts` for snapshot trigger logic
- Verify scheduled recompute is running in production

### Import Preview Failed
- Check CSV column names match expected schema
- Verify team scoping in request
- Review `lib/imports/parsing.ts` for normalization rules

### LLM Assistant Returns Fallback
- Check `OPENAI_API_KEY` is set in `.env.local`
- Verify API key is not rate-limited
- Check assistant context building in `lib/assistant.ts`

---

## Performance & Scaling Considerations

- **Indexes:** Prisma schema includes indexes on `teamId`, `athleteId`, `semester` for common queries
- **Pagination:** List APIs support `take`/`skip` for cursor-based pagination
- **Caching:** React Query caches API responses on client
- **Analytics:** Snapshots avoid repeated computation; recompute on schedule, not per-request
- **Database:** Neon auto-scales; monitor connection pool and query latency

---

## Additional Resources

- Prisma Docs: https://www.prisma.io/docs/
- Next.js Docs: https://nextjs.org/docs
- NextAuth.js Docs: https://next-auth.js.org/
- PostgreSQL Docs: https://www.postgresql.org/docs/
- Neon Docs: https://neon.tech/docs/
