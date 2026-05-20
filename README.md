# DeepEye — Exhaustive Research Agent Suite for OpenCode

**A team of 9 specialized AI agents that execute maximum-depth web research:
multi-method retrieval, authority source scraping, Camoufox stealth
extraction, multilingual normalization, and structured synthesis — all
orchestrated from a single prompt.**

![OpenCode](https://img.shields.io/badge/OpenCode-agent%20suite-blue)
![DeepSeek](https://img.shields.io/badge/model-DeepSeek%20V4%20Pro-4B6BFB)
![Agents](https://img.shields.io/badge/agents-9%20specialized-8A2BE2)
![License](https://img.shields.io/badge/license-MIT-brightgreen)

---

## What It Is

DeepEye is a **research agent framework** for [OpenCode](https://github.com/opencode-ai/opencode).
Send it a question and it dispatches up to 8 subagents in parallel, each
specializing in a different retrieval method — semantic search, adaptive
scraping, multi-page crawling, Instagram extraction, Google Maps lookup,
visual analysis, and multilingual normalization — then merges, deduplicates,
and synthesizes the evidence into a structured, cited answer.

The orchestrator (DeepEye) never settles for search snippets. It discovers
authoritative sources, scrapes them deeply, resolves conflicts, and produces
a unified answer with a full technique report showing exactly how every piece
of evidence was obtained.

```
User question
     │
     ▼
┌─────────────────────────────────────────────────────────┐
│                   DeepEye Orchestrator                   │
│  • Classifies query (closed / targeted / open / scrape) │
│  • Detects local aspect + freshness requirements        │
│  • Builds research plan                                 │
│  • Dispatches Phase 1 + Phase 2 in parallel             │
│  • Synthesizes, dedupes, resolves conflicts             │
│  • Produces structured output + technique report        │
└──────┬──────────────────────────────────────────────────┘
       │
       │  Phase 1 (parallel)               Phase 2 (parallel)
       │  ┌──────────────┐                ┌──────────────────┐
       ├──┤ exa search   │                │ scrapling        │
       ├──┤ exa crawling │                │  ↳ Camoufox      │
       ├──┤ worker       │─── handoff ──▶│    stealth        │
       ├──┤ translator   │                │ crawlee          │
       ├──┤ maps         │                │  ↳ pagination    │
       │  └──────────────┘                │ proxy            │
       │                                  │ instagram        │
       │                                  │ vision            │
       │                                  └──────────────────┘
       │                                            │
       ▼                                            ▼
┌─────────────────────────────────────────────────────────┐
│                    Merge + Synthesize                    │
│  • Deduplicate across 9 sources                         │
│  • Resolve conflicts (authority sources win)            │
│  • Rank evidence: primary > authoritative > aggregator  │
│  • Generate technique report with per-method counts     │
│  • Propose intelligent follow-up questions              │
└─────────────────────────────────────────────────────────┘
```

---

## Agent Team

### Orchestrator

| Agent | Model | Role |
|-------|-------|------|
| **DeepEye** | DeepSeek V4 Pro | Classifies queries, builds research plans, dispatches subagents in parallel, merges evidence, synthesizes answers |

### Subagents (dispatched via Task tool)

| Agent | Specialization | Key Capability |
|-------|---------------|----------------|
| **Worker** | Recall expansion, authority discovery | Runs multiple search queries, flags authority candidates and database sources |
| **Proxy** | Static page extraction | Fetches known URLs, detects JSON-LD/schema.org structured data |
| **Scrapling** | Adaptive extraction | Handles JS-rendered pages, dynamic content, anti-bot targets; auto-escalates to Camoufox stealth on 403/Cloudflare/DataDome |
| **Crawlee** | Multi-page traversal | Pagination, bounded crawling, structured harvesting; Camoufox stealth crawl mode for protected multi-page targets |
| **Translator-Normalizer** | Multilingual normalization | Local-language-first retrieval, cross-language dedupe, entity alias mapping, translation provenance |
| **Instagram** | Social/visual evidence | Camoufox-based extraction of profiles, posts, reels, hashtags, engagement metrics |
| **Maps** | Place intelligence | Google Maps text search, nearby search, place details with structured fields (hours, ratings, pricing, amenities) |
| **Vision** | Visual analysis | Image description, OCR, chart/diagram parsing, UI analysis, error diagnosis, video analysis via kimi k2.6 + ZAI MCP |

---

## Key Capabilities

### Two-Phase Parallel Dispatch

Phase 1 (breadth) and Phase 2 (depth) run in parallel batches, not
sequentially. Each phase dispatches all subagents in a single message —
no serial Task calls. The orchestrator performs a structured handoff
between phases, passing discovered authority URLs directly to the
appropriate extraction subagent.

### Authority Source Discovery & Scraping

DeepEye doesn't stop at search snippets. After Phase 1 identifies
authoritative sources (official sites, primary databases, domain experts),
Phase 2 scrapes them deeply — extracting JSON-LD, internal APIs, structured
HTML, or rendered content. One authority-scraped page outweighs 50 snippets.

### Structured Database Retrieval

When the topic involves events, reviews, pricing, businesses, jobs, real
estate, or government data, DeepEye scrapes the authoritative databases
directly rather than relying on search engine summaries. Default targets
include Eventbrite, TripAdvisor, Yelp, Google Maps, Crunchbase, Zillow,
PubMed, and domain-specific platforms.

### Camoufox Stealth Mode

Both Scrapling and Crawlee include **Camoufox** — a Firefox fork with C++
fingerprint injection that is undetectable through JavaScript inspection.
Used for anti-bot-protected targets (LinkedIn, Amazon, Booking.com,
Ticketmaster, Instagram) with automatic escalation on 403/Cloudflare
challenges and proactive dispatch for known protected domains.

### Local-Language-First Research

When a query has a local aspect (events in a city, municipal rules,
transport), DeepEye retrieves in the native language first, then
corroborates in English. Translator-Normalizer preserves provenance,
maps entity aliases across scripts, and deduplicates cross-language claims.

### Self-Audit Before Final Answer

The orchestrator runs a 7-point self-audit before returning any answer:
verifies mandatory subagents were dispatched, checks authority source
coverage, validates citations, resolves conflicts, and fills evidence
gaps by dispatching missing tools before finalizing.

---

## Installation

Copy the agent files into your OpenCode agents directory:

```bash
git clone https://github.com/lostsock1/deepeye.git
cd deepeye

# Copy all agents to OpenCode config
cp agents/*.md ~/.config/opencode/agents/search/
```

Or symlink for automatic updates:

```bash
ln -s "$(pwd)/agents/"* ~/.config/opencode/agents/search/
```

### Prerequisites

- [OpenCode](https://github.com/opencode-ai/opencode) installed
- API keys configured:
  - **Exa** — for semantic web search (`exa_web_search_exa`, `exa_crawling_exa`)
  - **DeepSeek** — for the orchestrator LLM (`deepseek-v4-pro`)
  - **PPQ AI** — for Maps subagent (Google Maps data API)
  - **Kimi** — for Vision subagent (native vision model)
  - **ZAI MCP** — for Vision subagent (OCR, chart parsing, video analysis)

---

## Usage

Invoke DeepEye from OpenCode:

```
@search/deepeye What are the best ramen restaurants in Tokyo with Michelin
stars, and how do their prices compare?
```

DeepEye will:
1. Classify as `targeted_research` with `local_aspect: true` (Tokyo → Japanese-first)
2. Dispatch Phase 1: Exa search + Worker + Translator-Normalizer (Japanese queries) + Maps
3. Handoff discovered URLs to Phase 2: Scrapling (Tabelog, Michelin guide) + Crawlee (review pages) + Instagram (visual evidence)
4. Merge all evidence, deduplicate, resolve pricing conflicts
5. Return structured answer with technique report

### Example Output Structure

```json
{
  "status": "success",
  "query_type": "targeted_research",
  "research_plan": {
    "query_class": "targeted_research",
    "local_aspect": true,
    "target_locale": "Tokyo, Japan",
    "local_languages": ["ja"],
    "database_targets": ["Tabelog", "Michelin Guide", "Google Maps"],
    "authority_sources_identified": ["tabelog.com", "guide.michelin.com"]
  },
  "findings": [...],
  "technique_report": {
    "methods_used": [
      {"tool": "exa_web_search_exa", "technique": "semantic search", "results_count": 15},
      {"tool": "search/worker", "technique": "recall expansion", "results_count": 42},
      {"tool": "search/maps", "technique": "google maps text search", "results_count": 8},
      {"tool": "search/scrapling", "technique": "adaptive scraping", "results_count": 12},
      {"tool": "search/translator-normalizer", "technique": "multilingual normalization", "results_count": 18}
    ],
    "databases_scraped": ["Tabelog", "Michelin Guide"]
  },
  "follow_up_questions": [
    "Should I prioritize lunch vs dinner pricing?",
    "Are any of these temporarily closed or reservation-only?"
  ]
}
```

---

## Files

```
deepeye/
├── agents/
│   ├── deepeye.md                  # Orchestrator (primary agent)
│   ├── worker.md                   # Recall expansion + authority discovery
│   ├── proxy.md                    # Static page extraction + JSON-LD
│   ├── scrapling.md                # Adaptive scraping + Camoufox stealth
│   ├── crawlee.md                  # Multi-page traversal + pagination
│   ├── translator-normalizer.md    # Multilingual normalization
│   ├── instagram.md                # Instagram content extraction
│   ├── maps.md                     # Google Maps place intelligence
│   └── vision.md                   # Visual analysis (images, charts, video)
└── README.md
```

---

## Design Philosophy

**Quality over speed, always.** DeepEye will dispatch 8 subagents and wait
for all of them before synthesizing. It won't return a fast but shallow
answer. Every material claim must be grounded in retrieved evidence.

**Depth, not breadth of snippets.** One fully scraped authority page is
worth more than 50 search snippets. DeepEye prioritizes extraction depth
over result count.

**Local context matters.** A question about Tokyo restaurants is a
Japanese-language research problem first. DeepEye queries in the local
language, translates evidence, and preserves provenance.

**No silent failures.** Every method that was applicable but not used is
documented with a reason. Every blocked source is reported as an evidence
gap. Partial results are better than silent failure, but must be declared
as partial.

---

## License

MIT
