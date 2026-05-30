# OnePagerResearch

> 🚧 **Status: Early Development**

AI-powered tool that takes any topic, paper, URL, or document and generates a clean, structured one-page research summary — key findings, methodology, implications, and further reading — in seconds.

[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-blue)](https://www.typescriptlang.org/)
[![React](https://img.shields.io/badge/React-18-61dafb)](https://reactjs.org/)
[![Node.js](https://img.shields.io/badge/Node.js-20-green)](https://nodejs.org/)
[![OpenAI GPT-4o](https://img.shields.io/badge/OpenAI-GPT--4o-orange)](https://openai.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow)](LICENSE)

---

## What It Does

Paste a topic, arXiv link, PDF URL, or raw text → get a structured one-pager back in seconds:

| Section | Description |
|---|---|
| **TL;DR** | 2-sentence executive summary |
| **Key Findings** | Bulleted takeaways |
| **Methodology** | How the research was conducted |
| **Implications** | Why it matters, who should care |
| **Limitations** | What the work doesn't address |
| **Further Reading** | Related works and links |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 18 + TypeScript + Vite |
| Backend | Node.js + Express + TypeScript |
| AI | OpenAI GPT-4o (structured outputs) |
| Scraping | Playwright (URL ingestion) |
| PDF parsing | pdf-parse |
| Styling | Tailwind CSS |
| Testing | Vitest (unit) + Supertest (API) |
| CI | GitHub Actions |

---

## Getting Started

### Prerequisites

- Node.js 20+
- pnpm 9+ (`npm install -g pnpm`)
- OpenAI API key

### Install

```bash
git clone https://github.com/joshiujjwal/one-pager-research.git
cd one-pager-research
pnpm install
```

### Configure

```bash
cp .env.example .env
# Add your OPENAI_API_KEY to .env
```

### Develop

```bash
pnpm dev          # Start frontend (Vite) + backend (tsx watch) concurrently
pnpm dev:client   # Frontend only — http://localhost:5173
pnpm dev:server   # Backend only  — http://localhost:3001
```

### Test

```bash
pnpm test         # Run all tests (Vitest + Supertest)
pnpm test:watch   # Watch mode
pnpm test:cov     # Coverage report
```

### Build

```bash
pnpm build        # Type-check + build client + build server
pnpm start        # Run production build
```

### Lint

```bash
pnpm lint         # ESLint
pnpm lint:fix     # ESLint --fix
pnpm typecheck    # tsc --noEmit
```

---

## Project Structure

```
one-pager-research/
├── src/                          # React frontend
│   ├── components/               # UI components (SummaryCard, InputForm, etc.)
│   ├── hooks/                    # Custom React hooks (useSummary, useHistory)
│   ├── lib/                      # Shared utilities (api client, formatters)
│   ├── pages/                    # Route-level page components
│   └── types/                    # Shared TypeScript types
├── server/                       # Express backend
│   ├── routes/                   # API route handlers
│   ├── services/                 # Business logic (summarizer, scraper, parser)
│   └── middleware/               # Auth, rate-limit, validation middleware
├── tests/
│   ├── unit/                     # Unit tests (services, hooks, utils)
│   ├── integration/              # API integration tests (Supertest)
│   └── e2e/                      # End-to-end tests (Playwright)
├── docs/
│   ├── spec.md                   # Feature specification
│   └── adr/                      # Architecture Decision Records
├── .github/
│   ├── copilot-instructions.md   # GitHub Copilot context
│   └── workflows/                # CI/CD pipelines
├── CLAUDE.md                     # Context for Claude AI
├── AGENTS.md                     # Context for OpenAI Codex agents
├── TODO.md                       # Evidence-gated task phases
└── docs/spec.md                  # Full feature specification
```

---

## Contributing

1. **Read** `TODO.md` for the current phase and next task
2. **Run tests first** — never start from a broken baseline
3. **Write failing tests** before implementing anything (red phase)
4. **Implement** until tests pass (green phase)
5. **Review your diff** manually before committing
6. **Commit** with a descriptive message referencing the TODO item
7. **Update** `CLAUDE.md` / `AGENTS.md` if you learned something new about this project

PRs must include:
- Test output showing green before and after
- Description of what changed and why
- Screenshot / curl output as evidence for UI/API changes

---

## 🚀 Improvement Proposals

### First-Principles Analysis
- **The fundamental job-to-be-done is reducing the cognitive cost of evaluating whether a source is worth reading in full** — the one-pager format is a delivery mechanism, not the goal; optimising for "did the user click through to read more / cite this?" is a better north-star metric than generation speed.
- **LLM summarisation is lossy compression**: GPT-4o has no access to the actual experimental data, reproducibility details, or supplementary materials in a paper; the Methodology and Limitations sections it generates are reconstructed, not extracted, which can mislead readers who treat them as ground truth.
- **URL/PDF ingestion is the hardest reliability problem**, not the AI layer — paywalled journals, JavaScript-rendered pages, and DRM-protected PDFs will fail silently; the scraping layer needs explicit failure modes documented.
- **The structured output schema (TL;DR, Key Findings, etc.) is a UI assumption baked into the backend** — different disciplines (legal, clinical, engineering) need different section schemas; a schema-per-domain approach would dramatically widen the addressable market.

### Key Risks & Assumptions
- **Assumes GPT-4o context window is sufficient for all inputs** — long papers (100+ pages) or multi-document inputs will exceed the window; chunking strategies affect summary coherence and are not yet addressed in the README.
- **Playwright-based scraping will be fragile in production** — many academic sites (Nature, Springer, IEEE) actively block headless browsers; the system needs a fallback chain (direct fetch → Playwright → user-provided text).
- **Assumes users trust AI-generated summaries enough to act on them** — hallucination risk is high for quantitative claims (statistics, p-values, sample sizes); without inline citations pointing to source text, accuracy cannot be verified.
- **No mention of caching or cost controls** — repeated GPT-4o calls on the same URL are both expensive and slow; without a cache layer, the cost per active user is unbounded.

### Concrete Improvement Ideas
1. **Add per-claim source anchoring** — after generating each bullet in Key Findings, include the verbatim quote and page/section reference from the source; this transforms the tool from a trust-me summariser into a verifiable research assistant (highest trust and retention impact).
2. **Implement a response cache keyed on content hash** — cache summaries by SHA-256 of the raw document text; eliminates redundant LLM calls for the same paper and enables sharing without re-generation cost.
3. **Build a domain-specific schema selector** — offer 3–4 pre-built section schemas (academic paper, legal brief, news article, patent); let users define custom schemas; this unlocks B2B use-cases (law firms, consulting, pharma) without changing the core pipeline.
4. **Add a graceful degradation chain for ingestion failures** — implement ranked fallback: direct HTTP fetch → Playwright → Jina Reader API → prompt user to paste text; log failure reason per URL to guide future improvements.
5. **Surface a confidence / completeness indicator** — use the LLM to self-evaluate: "How much of the source document was accessible and processed?" Flag summaries generated from <50 % of the content as partial.
6. **Add export formats** — one-click export to Notion, Markdown, and PDF; research workflows live in note-taking tools, not web apps; meeting users where they work drives retention.
