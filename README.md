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
