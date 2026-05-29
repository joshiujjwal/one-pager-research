# OnePagerResearch — Feature Specification

**Version:** 0.1.0-draft  
**Last updated:** 2025-01  
**Status:** Pre-implementation

---

## 1. Overview

### Problem Statement

Researchers, students, and professionals spend 30–90 minutes reading a paper or article before they can extract actionable insights. Most of that time is spent identifying structure, not learning content. OnePagerResearch eliminates that overhead by producing a structured, skimmable one-page summary in seconds.

### Solution

A web tool that accepts any of the following inputs:
- A plain-text topic or question (e.g., "attention is all you need transformer paper")
- A URL (news article, blog post, Wikipedia, arXiv abstract)
- An arXiv paper ID or link (auto-fetches and parses PDF)
- An uploaded PDF or plain text file

...and returns a structured one-pager with exactly six sections.

---

## 2. Output Schema

Every summary MUST contain all six sections. No section may be omitted, even for minimal input.

```typescript
interface SummaryResult {
  id: string;                  // UUID
  input: SummaryInput;         // What was submitted
  createdAt: string;           // ISO 8601

  tldr: string;                // 2-sentence executive summary
  keyFindings: string[];       // 3–7 bulleted findings
  methodology: string;         // How the research was done (1–2 paragraphs)
  implications: string;        // Why it matters and who should care (1 paragraph)
  limitations: string;         // What the work doesn't address (1 paragraph)
  furtherReading: ReadingItem[]; // 3–5 related works with title + URL
}

interface ReadingItem {
  title: string;
  url?: string;
  reason: string;  // One sentence: why this is relevant
}

type SummaryInput =
  | { type: 'text'; content: string }
  | { type: 'url'; url: string }
  | { type: 'pdf'; filename: string; size: number }
  | { type: 'arxiv'; id: string; url: string };
```

---

## 3. Functional Requirements

### Input Handling

- [ ] **FR-01** Accept plain text input (min 10 chars, max 50,000 chars)
- [ ] **FR-02** Accept a URL and scrape readable text (timeout: 15s, max 200KB text)
- [ ] **FR-03** Accept PDF file upload (max 5MB, MIME: `application/pdf`)
- [ ] **FR-04** Detect arXiv URLs (`arxiv.org/abs/*`, `arxiv.org/pdf/*`) and auto-fetch the full PDF
- [ ] **FR-05** Strip HTML boilerplate from scraped URLs (ads, navbars, footers)
- [ ] **FR-06** Validate and reject unsupported file types with a clear error message

### Summarization

- [ ] **FR-07** Generate all 6 summary sections for any valid input
- [ ] **FR-08** Use GPT-4o structured outputs (JSON schema enforcement — no regex parsing)
- [ ] **FR-09** Truncate inputs exceeding 100,000 tokens with a warning to the user
- [ ] **FR-10** Return summaries in under 10 seconds for inputs under 10,000 words
- [ ] **FR-11** Handle non-English inputs — detect language and summarize in English

### Frontend

- [ ] **FR-12** Single-page app with input form (text/URL/file upload tabs)
- [ ] **FR-13** Loading state with animated skeleton during generation
- [ ] **FR-14** Render summary as a styled one-pager card (print-friendly)
- [ ] **FR-15** Copy summary to clipboard as Markdown
- [ ] **FR-16** Export summary as PDF (client-side, using `@react-pdf/renderer`)
- [ ] **FR-17** History panel showing last 20 summaries (stored in localStorage)
- [ ] **FR-18** Click a history item to reload its summary without re-summarizing

### API

- [ ] **FR-19** `POST /api/summarize` — accepts `{ type, content }` JSON
- [ ] **FR-20** `POST /api/ingest/url` — accepts `{ url }`, returns extracted text
- [ ] **FR-21** `POST /api/ingest/pdf` — accepts multipart form, returns extracted text
- [ ] **FR-22** `GET /api/health` — returns `{ status: "ok", version: string }`

---

## 4. Non-Functional Requirements

- [ ] **NFR-01** P95 response time ≤ 10s for inputs under 5,000 words
- [ ] **NFR-02** Rate limit: 20 requests/minute per IP
- [ ] **NFR-03** All API inputs validated with Zod schemas before processing
- [ ] **NFR-04** No user data stored server-side (stateless API, localStorage only)
- [ ] **NFR-05** OPENAI_API_KEY never exposed to client bundle
- [ ] **NFR-06** TypeScript strict mode enabled (`"strict": true`)
- [ ] **NFR-07** 80%+ unit test coverage on `server/services/`
- [ ] **NFR-08** Lighthouse performance score ≥ 90, accessibility score = 100
- [ ] **NFR-09** Mobile-responsive layout (breakpoints: 375px, 768px, 1280px)

---

## 5. Data Model

### In-Memory / LocalStorage (MVP)

```typescript
// Stored in localStorage under key "opr_history"
interface HistoryStore {
  version: 1;
  summaries: SummaryResult[];  // most recent first, max 20
}
```

No server-side persistence in v0.1.0. See Parking Lot in TODO.md for v2 plans.

---

## 6. API Design

### `POST /api/summarize`

**Request:**
```json
{
  "type": "text" | "url" | "arxiv",
  "content": "string (text) | url string"
}
```

**Response (200):**
```json
{
  "id": "uuid",
  "tldr": "...",
  "keyFindings": ["...", "..."],
  "methodology": "...",
  "implications": "...",
  "limitations": "...",
  "furtherReading": [{ "title": "...", "url": "...", "reason": "..." }],
  "createdAt": "ISO8601",
  "input": { "type": "text", "content": "..." }
}
```

**Errors:**
| Code | Reason |
|------|--------|
| 400 | Invalid input (Zod validation failure) |
| 413 | Input exceeds size limit |
| 429 | Rate limit exceeded |
| 502 | OpenAI API error |

### `POST /api/ingest/pdf`

Multipart form with field `file`. Returns `{ text: string, wordCount: number }`.

### `POST /api/ingest/url`

`{ url: string }` → `{ text: string, title: string, wordCount: number }`.

---

## 7. Test Plan

### Unit Tests (`tests/unit/`)

| Test File | What It Covers |
|---|---|
| `summarizer.test.ts` | GPT-4o call, schema validation, token truncation |
| `urlScraper.test.ts` | HTML extraction, boilerplate stripping, timeout handling |
| `pdfParser.test.ts` | PDF text extraction, oversized file rejection |
| `arxivIngester.test.ts` | arXiv URL detection, PDF fetch, fallback to abstract |
| `InputForm.test.tsx` | Form rendering, tab switching, validation states |
| `SummaryCard.test.tsx` | All 6 sections render, empty state, loading state |
| `useSummary.test.ts` | Loading/error/success state machine |
| `useHistory.test.ts` | Save, retrieve, delete, max-20 cap |

### Integration Tests (`tests/integration/`)

| Test File | What It Covers |
|---|---|
| `summarize.api.test.ts` | Full `/api/summarize` round trip (mocked OpenAI) |
| `ingest.api.test.ts` | URL + PDF ingestion endpoints |
| `health.api.test.ts` | Health check endpoint |

### Edge Cases

- Input is a single word
- Input is 50,000+ characters
- URL returns 404 or times out
- PDF is password-protected
- PDF contains only scanned images (no text layer)
- OpenAI returns malformed JSON
- OpenAI returns 429 (rate limit)
- User submits two concurrent requests

---

## 8. Open Questions

1. **Caching**: Should identical URL inputs re-use a cached summary? (Saves tokens, risks stale data)
2. **Auth**: Do we need an API key gate to prevent abuse before rate limiting is sufficient?
3. **Language**: Summarize non-English papers in English only, or offer multilingual output?
4. **arXiv rate limits**: arXiv asks for 3s between requests — do we need a queue?
5. **PDF images**: If a paper is scan-only (no text layer), do we use GPT-4o Vision or fail gracefully?
