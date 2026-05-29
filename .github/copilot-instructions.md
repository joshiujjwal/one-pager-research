# GitHub Copilot Instructions — OnePagerResearch

## Project Summary

AI-powered one-page research summary tool. Users submit a topic, URL, arXiv link, or PDF → the server calls GPT-4o with structured output enforcement → client renders a 6-section summary card.

**Stack:** TypeScript 5 · React 18 · Vite · Node.js 20 · Express · OpenAI GPT-4o · Tailwind CSS · Vitest · Playwright

---

## File Structure Rules

- Frontend code lives in `src/` — never import from `server/` here
- Backend code lives in `server/` — never import from `src/` here
- Shared types that BOTH sides need go in `src/types/` and are imported via path alias
- `OPENAI_API_KEY` is **only** used in `server/` — hard rule, no exceptions

---

## Coding Conventions

### TypeScript
- `"strict": true` — no `any`, no `ts-ignore`, no `!` without a comment
- `zod` for all runtime validation — infer TypeScript types from Zod schemas (`z.infer<>`)
- Named exports only from service files (no default exports)
- Path aliases: `@/` → `src/`, `@server/` → `server/`

### React Components
- Functional components with explicit return types
- Props interface named `<ComponentName>Props`
- Tailwind CSS only — no inline styles, no CSS modules
- `useSummary` hook owns all summarization state — components call the hook, don't fetch directly
- `useHistory` hook owns all localStorage history state

### Express Services
- Route handlers: validate with Zod → call service function → return JSON
- Services are pure functions (input → output), side-effect free where possible
- Throw typed custom errors (`ScrapingError`, `ParsingError`, `SummarizationError`)
- All errors caught by `server/middleware/errorHandler.ts`

### OpenAI Integration
- Use structured outputs (`response_format: { type: "json_schema" }`) — not `json_object`
- System prompt in `server/services/prompts.ts` — never inline prompts
- Mock OpenAI in all test files with `vi.mock('openai')`

---

## Testing Conventions

- **TDD always**: write failing test first, then implement
- Unit tests: `tests/unit/<service>.test.ts` or co-located with components
- Integration tests: `tests/integration/<feature>.api.test.ts` using Supertest
- E2E tests: `tests/e2e/<flow>.spec.ts` using Playwright
- Never call real external APIs in tests — mock everything

```typescript
// Correct pattern — mock before importing the module under test
vi.mock('openai', () => ({
  default: vi.fn().mockImplementation(() => ({
    chat: { completions: { create: vi.fn() } }
  }))
}));
```

---

## Do Not

- Do not refactor working code unless the user explicitly asks
- Do not remove existing tests — if a test is wrong, fix it, don't delete it
- Do not add new npm dependencies without checking if a built-in or existing dep covers it
- Do not use `console.log` in production code — use a logger
- Do not commit `.env` — only `.env.example`
- Do not expose `OPENAI_API_KEY` in any client-side code, ever
- Do not use `any` type — if you're unsure, use `unknown` and narrow it

---

## Key Interfaces

The core output type (all 6 sections mandatory):

```typescript
interface SummaryResult {
  id: string;
  tldr: string;                  // 2-sentence executive summary
  keyFindings: string[];         // 3–7 items
  methodology: string;           // 1–2 paragraphs
  implications: string;          // 1 paragraph
  limitations: string;           // 1 paragraph
  furtherReading: ReadingItem[]; // 3–5 items
  createdAt: string;
  input: SummaryInput;
}
```

See `src/types/summary.ts` for the full definition once created.
