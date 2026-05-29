# OnePagerResearch — Task Breakdown

## How to Use This File

Work one task at a time. For each task:
1. **Write tests FIRST** — red phase (tests must fail before you implement)
2. **Implement** until tests pass — green phase
3. **Review diff** manually (`git diff`)
4. **Commit** with a descriptive message
5. **Update** `CLAUDE.md` or `AGENTS.md` if you discovered something new (compound loop)
6. **Check the box** and move to the next task

Never start a new phase until the current phase is fully green and committed.

---

## Phase 0: Foundation ⬜

- [ ] `pnpm init` + workspace setup (client in `src/`, server in `server/`)
- [ ] Install and configure TypeScript (`tsconfig.json` for client + server)
- [ ] Install and configure ESLint + Prettier
- [ ] Install and configure Vitest + Supertest
- [ ] Write first smoke test: `server/health` endpoint returns `{ status: "ok" }`
- [ ] Set up Vite for React frontend
- [ ] Set up Tailwind CSS
- [ ] Create `.env.example` with `OPENAI_API_KEY`, `PORT`, `NODE_ENV`
- [ ] GitHub Actions CI: lint + typecheck + test on push/PR
- [ ] **Review** all AI config files (`CLAUDE.md`, `AGENTS.md`, `copilot-instructions.md`) — update if needed
- [ ] **Evidence gate**: CI passes, smoke test green ✅

---

## Phase 1: Core Summarization Engine ⬜

- [ ] Define `SummaryRequest` and `SummaryResult` types in `src/types/summary.ts`
- [ ] Write failing tests for `server/services/summarizer.ts` — plain text input → structured output
- [ ] Implement `summarizer.ts`: call GPT-4o with structured output schema
- [ ] Define the one-pager schema (tldr, keyFindings[], methodology, implications, limitations, furtherReading[])
- [ ] Write failing tests for prompt engineering — verify all 6 sections are populated
- [ ] Add token counting / truncation for large inputs (context window guard)
- [ ] Write edge case tests: empty input, non-English text, very short input (<50 words)
- [ ] Manual test: run `curl` against `/api/summarize` with a real topic
- [ ] **Evidence gate**: all tests green, manual curl output committed to `docs/manual-test-1.md` ✅

---

## Phase 2: Input Ingestion ⬜

- [ ] Write failing tests for `server/services/urlScraper.ts` — URL → clean text
- [ ] Implement URL scraper using Playwright (headless), strip boilerplate
- [ ] Write failing tests for `server/services/pdfParser.ts` — PDF buffer → text
- [ ] Implement PDF parser using `pdf-parse`
- [ ] Write failing tests for arXiv URL detection — auto-fetch abstract + body
- [ ] Implement arXiv ingestion (parse `arxiv.org/abs/` links → fetch PDF)
- [ ] Add input validation middleware: max file size (5MB), allowed MIME types
- [ ] Write integration tests for `/api/ingest` endpoint (URL + PDF + raw text)
- [ ] **Evidence gate**: all ingestion tests green, integration test suite passing ✅

---

## Phase 3: React Frontend ⬜

- [ ] Write component tests for `InputForm` — accepts topic text, URL, file upload
- [ ] Implement `InputForm` component
- [ ] Write component tests for `SummaryCard` — renders all 6 sections
- [ ] Implement `SummaryCard` component with clean one-page layout
- [ ] Write hook tests for `useSummary` — loading, error, and success states
- [ ] Implement `useSummary` hook (calls `/api/summarize`, manages state)
- [ ] Implement `HomePage` wiring `InputForm` → `useSummary` → `SummaryCard`
- [ ] Add copy-to-clipboard and export-to-PDF buttons
- [ ] Add loading skeleton and error states
- [ ] **Evidence gate**: component tests green, screenshot of rendered summary committed ✅

---

## Phase 4: History & Export ⬜

- [ ] Decide storage: localStorage (MVP) vs. server-side DB (v2) — document in ADR
- [ ] Write tests for `useHistory` hook — save, list, delete summaries
- [ ] Implement `useHistory` with localStorage persistence
- [ ] Implement `HistoryPanel` component (sidebar list of past summaries)
- [ ] Write tests for PDF export: `exportToPDF(summary)` produces valid PDF buffer
- [ ] Implement PDF export using `@react-pdf/renderer`
- [ ] Write tests for Markdown export
- [ ] Implement Markdown export
- [ ] **Evidence gate**: all history + export tests green ✅

---

## Phase 5: Polish & Harden ⬜

- [ ] Add rate limiting middleware (`express-rate-limit`): 20 req/min per IP
- [ ] Add request validation with Zod on all endpoints
- [ ] Add OpenAI error handling: retry on 429, surface 4xx/5xx clearly to client
- [ ] Write E2E test: full flow — type topic → click summarize → summary renders
- [ ] Lighthouse audit: target 90+ performance, 100 accessibility
- [ ] Add `robots.txt` and `sitemap.xml` stubs
- [ ] Review bundle size — code-split heavy deps (pdf, playwright)
- [ ] **Evidence gate**: E2E test green, Lighthouse scores committed ✅

---

## Phase 6: Ship ⬜

- [ ] Finalize `README.md` with real screenshots
- [ ] Write deployment guide (`docs/deploy.md`) for Railway / Render / Vercel
- [ ] Set up production environment variables in hosting platform
- [ ] Tag `v0.1.0` release
- [ ] Create GitHub release with changelog
- [ ] **Evidence gate**: production URL live, smoke test against prod URL ✅

---

## Parking Lot 🅿️

> Ideas not in current scope — revisit after v0.1.0

- [ ] Batch mode: upload multiple papers at once
- [ ] Comparison mode: two papers side-by-side
- [ ] Citation graph visualization
- [ ] Chrome extension for one-click summarization
- [ ] Slack / Discord bot integration
- [ ] Custom output templates (executive brief, academic review, investor memo)
- [ ] Team sharing / collaboration features

---

## Lessons Learned 📝

> Update this as you build. Each entry prevents the same mistake in future sessions.

- (none yet)
