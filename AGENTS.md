# AGENTS.md — OnePagerResearch

Configuration for OpenAI Codex and other AI coding agents.

---

## Setup

```bash
# 1. Install dependencies
pnpm install

# 2. Install Playwright browsers (required for URL scraping)
pnpm exec playwright install chromium

# 3. Configure environment
cp .env.example .env
# Set OPENAI_API_KEY in .env

# 4. Verify setup — all these must pass before making changes
pnpm typecheck
pnpm lint
pnpm test
```

---

## Project Context

**What this is:** An AI-powered one-page research summary tool. Users input a topic, URL, or PDF and get a structured 6-section summary via GPT-4o.

**The output schema** (6 mandatory sections) is defined in `docs/spec.md` → Section 2.

**Where to start:** Read `TODO.md` to find the current phase and next task.

---

## Code Style

### TypeScript
- Strict mode: `"strict": true` in all `tsconfig.json` files
- No `any`. No `ts-ignore`. Non-null assertions require an inline comment.
- Types go in `src/types/` (frontend) — never inline in component or service files
- Use `zod` for both runtime validation and TypeScript type inference: `z.infer<typeof MySchema>`
- Prefer `interface` over `type` for object shapes; `type` for unions/intersections

### React
- Functional components only — no class components
- Props interfaces named `<ComponentName>Props`
- Co-locate component test files: `SummaryCard.tsx` + `SummaryCard.test.tsx` in same dir
- Use `React.FC` sparingly — prefer explicit return type annotation
- No inline styles — Tailwind classes only

### Node / Express
- Route handlers are thin: validate → call service → return response
- All business logic in `server/services/` — routes import services, not vice versa
- Services are pure functions where possible (easier to test)
- Use `async/await` throughout — no raw `.then()` chains
- Always handle promise rejections in Express route handlers (wrap with `asyncHandler`)

### Naming
- Files: `camelCase.ts` for services/hooks/utils, `PascalCase.tsx` for components
- Functions: `camelCase`
- Types/Interfaces: `PascalCase`
- Constants: `UPPER_SNAKE_CASE`
- API routes: `kebab-case` (`/api/ingest/pdf`, not `/api/ingestPdf`)

### Imports
- Use path aliases: `@/` maps to `src/`, `@server/` maps to `server/`
- Group imports: external packages → internal modules → types
- No default exports from service files — use named exports

---

## Testing Rules

### Red/Green TDD — Strictly Enforced

1. **Write the failing test first.** Commit it as a separate commit if desired.
2. **Run `pnpm test` to confirm it fails** (for the right reason).
3. **Implement** until the test passes.
4. **Run the full suite** before committing.

### Test Patterns

```typescript
// Unit test for a service — always mock OpenAI
import { vi, describe, it, expect, beforeEach } from 'vitest';
vi.mock('openai');

// Integration test — use Supertest, never start a real server
import request from 'supertest';
import { app } from '../../server/app';

describe('POST /api/summarize', () => {
  it('returns 400 for empty content', async () => {
    const res = await request(app).post('/api/summarize').send({ type: 'text', content: '' });
    expect(res.status).toBe(400);
  });
});
```

### Coverage Target
- `server/services/`: 80% line coverage minimum
- `src/hooks/`: 80% line coverage minimum
- Run: `pnpm test:cov` — fails below threshold

### Never
- Make real OpenAI API calls in tests
- Start a real HTTP server in tests (use Supertest with the Express `app` directly)
- Skip tests to make coverage pass — fix the code instead

---

## PR Instructions

Every PR must include:

1. **Test evidence** — paste the output of `pnpm test` showing green before AND after
2. **Type + lint evidence** — `pnpm typecheck && pnpm lint` output
3. **What changed** — one paragraph explaining the change
4. **Why** — reference the TODO.md task being completed
5. **Evidence** — for API changes: a `curl` command and its output. For UI changes: a screenshot.

PR size: Keep PRs focused on one TODO item. If a PR touches >5 files, split it.

---

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `OPENAI_API_KEY` | Yes | OpenAI API key — server only, never in client |
| `PORT` | No | Server port (default: 3001) |
| `NODE_ENV` | No | `development` \| `production` \| `test` |
| `RATE_LIMIT_RPM` | No | Requests per minute per IP (default: 20) |

**Security rule:** `OPENAI_API_KEY` must never appear in any file under `src/`. If the linter finds it, fail the build.

---

## Architecture Decisions

Before making a significant architectural choice (new dependency, changed data model, alternative approach), create an ADR:

```bash
cp docs/adr/0001-template.md docs/adr/000N-your-decision.md
```

Fill it in before writing code. ADR decisions are permanent — they document why something was done, not just what.
