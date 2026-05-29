# CLAUDE.md — OnePagerResearch

Context for Claude AI agents working on this project. Keep this file under 200 lines.
Update it whenever you discover something non-obvious.

---

## Quick Commands

```bash
pnpm install          # Install all deps (root workspace)
pnpm dev              # Start client (5173) + server (3001) concurrently
pnpm dev:client       # Vite frontend only
pnpm dev:server       # tsx watch server only
pnpm build            # tsc + vite build + server build
pnpm test             # Run all tests (Vitest)
pnpm test:watch       # Watch mode
pnpm test:cov         # Coverage report (target: 80% on server/services/)
pnpm lint             # ESLint
pnpm lint:fix         # ESLint --fix
pnpm typecheck        # tsc --noEmit (both client + server tsconfigs)
```

> **ALWAYS run `pnpm test` first** before making any changes. Never start from a broken baseline.

---

## Directory Map

```
src/                  Frontend (React + Vite)
  components/         Reusable UI components — SummaryCard, InputForm, HistoryPanel
  hooks/              Custom hooks — useSummary (main state), useHistory (localStorage)
  lib/                API client (wraps fetch), formatters, constants
  pages/              Route-level pages — HomePage is the only page in v0.1
  types/              Shared TypeScript types — import from here, not inline

server/               Backend (Express + Node)
  routes/             Thin route handlers — validate input, call service, return response
  services/           All business logic lives here:
                        summarizer.ts    — GPT-4o call + schema enforcement
                        urlScraper.ts    — Playwright scraping
                        pdfParser.ts     — pdf-parse wrapper
                        arxivIngester.ts — arXiv URL detection + PDF fetch
  middleware/         rateLimiter.ts, validate.ts (Zod), errorHandler.ts

tests/
  unit/               One file per service/component/hook
  integration/        Supertest against real Express app (OpenAI mocked)
  e2e/                Playwright browser tests

docs/spec.md          THE source of truth for what to build — read before implementing
docs/adr/             Architecture decisions — create a new ADR before big choices
```

---

## Workflow for Every Task

1. Read the current TODO.md phase — identify the next unchecked task
2. Run `pnpm test` — confirm baseline is green
3. Write failing tests first (red phase)
4. Implement until tests pass (green phase)
5. Run `pnpm lint && pnpm typecheck` — fix everything before committing
6. `git diff` — review your own changes
7. Commit with a message like `feat(summarizer): implement GPT-4o structured output`
8. Update this file if you learned something non-obvious

---

## Non-Obvious Conventions

### TypeScript
- Strict mode is on. No `any`, no `ts-ignore`, no `!` non-null assertions without a comment.
- All types live in `src/types/` — never inline interface definitions in component files.
- Use `zod` for runtime validation of API inputs AND for inferring TypeScript types from schemas.

### OpenAI Calls
- Always use **structured outputs** (`response_format: { type: "json_schema", ... }`), not `response_format: { type: "json_object" }`. Structured outputs enforce the schema server-side.
- The system prompt lives in `server/services/prompts.ts` — never inline prompts in route handlers.
- Mock OpenAI in all tests — never make real API calls in the test suite.
- `OPENAI_API_KEY` is **only** read in `server/` — it must never appear in any `src/` file.

### React State
- `useSummary` owns the loading/error/result state machine. Components do not fetch directly.
- History is managed by `useHistory` (localStorage). Max 20 items — oldest is dropped.
- No Redux or Zustand in v0.1 — React state + hooks is sufficient.

### Testing
- Use `vi.mock('openai')` to mock the OpenAI client in unit/integration tests.
- Fixtures for test PDFs and HTML are in `tests/fixtures/`.
- Supertest tests import the Express `app` directly — do not start a real server.

### Error Handling
- All Express errors flow through `server/middleware/errorHandler.ts`.
- Services throw typed errors (e.g., `class ScrapingError extends Error`).
- Client shows user-friendly messages — never expose raw error messages from OpenAI.

### Environment
- `.env` is gitignored. Use `.env.example` as the template.
- In tests, use `dotenv` only if needed — prefer mocking environment-dependent services.

---

## Key Files to Read First

1. `docs/spec.md` — feature spec and output schema
2. `src/types/summary.ts` — the `SummaryResult` interface (once created)
3. `server/services/summarizer.ts` — the core GPT-4o integration (once created)
4. `TODO.md` — current phase and next task

---

## Known Gotchas

- Playwright must be installed separately: `pnpm exec playwright install chromium`
- arXiv PDF URLs redirect — follow redirects with `axios` not `fetch` (undici redirect handling is inconsistent in Node 20)
- `pdf-parse` has a known issue with encrypted PDFs — catch `Error: bad XRef entry` and surface a user-friendly message
- GPT-4o structured outputs require the schema to have `additionalProperties: false` at every nested level
