# DeepEye — Research Agent Definitions for OpenCode

**OpenCode agent configurations for exhaustive multi-method web research:
1 orchestrator + 8 specialized subagents that together form a parallel
retrieval pipeline with authority scraping, Camoufox stealth extraction,
multilingual normalization, and structured synthesis.**

![OpenCode](https://img.shields.io/badge/OpenCode-agent%20definitions-blue)
![DeepSeek](https://img.shields.io/badge/model-DeepSeek%20V4%20Pro-4B6BFB)
![Agents](https://img.shields.io/badge/agents-9%20definitions-8A2BE2)
![License](https://img.shields.io/badge/license-MIT-brightgreen)

---

> **⚠️ These are agent definition files — configuration, not executable code.**
> They define prompts, tool permissions, model settings, and dispatch rules
> for OpenCode's agent system. They work when OpenCode is installed and
> configured with the required API keys (Exa, DeepSeek, PPQ AI, Kimi, ZAI MCP).
> The agents have been used for research tasks but have not been systematically
> tested across all query types.

---

## What It Is

DeepEye is a set of 9 OpenCode agent definitions that together configure a
maximum-depth research pipeline. The orchestrator (`deepeye.md`) classifies
queries, builds research plans, dispatches subagents in parallel, and
synthesizes results. Each subagent specializes in one retrieval method.

The key idea: never settle for search snippets. The orchestrator discovers
authoritative sources, dispatches subagents to scrape them deeply, resolves
conflicts, and produces a unified answer with a technique report showing
exactly how each piece of evidence was obtained.

---

## Agent Definitions

### Orchestrator

| Agent | Model | Size | Role |
|-------|-------|------|------|
| **deepeye.md** | DeepSeek V4 Pro | 932 lines | Query classification, research planning, parallel dispatch, evidence synthesis, self-audit |

### Subagents

| Agent | Size | Specialization |
|-------|------|---------------|
| **worker.md** | 335 lines | Recall expansion, authority source discovery, search diversity across multiple queries |
| **proxy.md** | 229 lines | Static page extraction from known URLs, JSON-LD/schema.org structured data detection |
| **scrapling.md** | 683 lines | Adaptive extraction from JS-rendered/dynamic pages, Camoufox stealth mode for anti-bot targets |
| **crawlee.md** | 479 lines | Multi-page traversal, pagination, bounded crawling, Camoufox stealth crawl for protected sites |
| **translator-normalizer.md** | 297 lines | Local-language-first retrieval, cross-language dedupe, entity alias mapping, translation provenance |
| **instagram.md** | 428 lines | Instagram profiles, posts, reels, hashtags, engagement data via Camoufox stealth browser |
| **maps.md** | 341 lines | Google Maps text search, nearby search, place details (hours, ratings, pricing, amenities) |
| **vision.md** | 363 lines | Image description, OCR, chart/diagram parsing, UI analysis, video analysis via kimi k2.6 + ZAI MCP |

**Total: 2,979 lines across 9 agent definitions.**

---

## How It Works (Intended Flow)

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

## Key Design Decisions

### Two-Phase Parallel Dispatch

Phase 1 (breadth: Exa + Worker + Translator + Maps) and Phase 2 (depth:
Scrapling + Crawlee + Proxy + Instagram + Vision) run as parallel batches.
The orchestrator performs a structured handoff between phases, passing
discovered authority URLs directly to extraction subagents.

### Camoufox Stealth Mode

Both `scrapling` and `crawlee` agent definitions include instructions for
Camoufox — a Firefox fork with C++ fingerprint injection. The agents are
configured to auto-escalate on 403/Cloudflare challenges and to accept
proactive stealth dispatch for known anti-bot domains.

### Local-Language-First

When a query has a local aspect (events in a city, municipal rules), the
orchestrator is configured to retrieve in the native language first, then
corroborate in English. `translator-normalizer` preserves provenance,
maps entity aliases across scripts, and deduplicates cross-language claims.

### Self-Audit

The orchestrator definition includes a 7-point self-audit that runs before
returning any answer: verifies mandatory subagents were dispatched, checks
authority source coverage, validates citations, resolves conflicts, and
attempts to fill evidence gaps before finalizing.

---

## Installation

Copy or symlink the agent definitions into OpenCode:

```bash
git clone https://github.com/lostsock1/deepeye.git
cd deepeye

# Copy all agents
cp agents/*.md ~/.config/opencode/agents/search/

# Or symlink for automatic updates
ln -s "$(pwd)/agents/"* ~/.config/opencode/agents/search/
```

### Prerequisites

These agents require OpenCode with the following configured:

- **Exa API** — for semantic web search (`exa_web_search_exa`, `exa_crawling_exa`)
- **DeepSeek API** — for the orchestrator LLM (`deepseek-v4-pro`)
- **PPQ AI API** — for Maps subagent (Google Maps data)
- **Kimi API** — for Vision subagent (native vision model k2.6)
- **ZAI MCP tools** — for Vision subagent (OCR, chart parsing, video analysis)
- **Camoufox** — for Scrapling/Crawlee/Instagram stealth extraction

Without these, some subagents will fail or degrade. The orchestrator is
designed to continue with partial results and report failures.

---

## Usage

From OpenCode, invoke the orchestrator:

```
@search/deepeye What are the best ramen restaurants in Tokyo with Michelin stars?
```

The orchestrator is configured to:
1. Classify as `targeted_research` with `local_aspect: true`
2. Dispatch Phase 1 (Exa + Worker + Translator + Maps) in parallel
3. Handoff discovered URLs to Phase 2 (Scrapling + Crawlee + Instagram)
4. Merge, deduplicate, resolve conflicts
5. Return structured answer with technique report

---

## Files

```
deepeye/
├── agents/
│   ├── deepeye.md                  # Orchestrator (932 lines)
│   ├── worker.md                   # Recall expansion + authority discovery (335 lines)
│   ├── proxy.md                    # Static page extraction + JSON-LD (229 lines)
│   ├── scrapling.md                # Adaptive scraping + stealth (683 lines)
│   ├── crawlee.md                  # Multi-page traversal (479 lines)
│   ├── translator-normalizer.md    # Multilingual normalization (297 lines)
│   ├── instagram.md                # Instagram extraction (428 lines)
│   ├── maps.md                     # Google Maps place search (341 lines)
│   └── vision.md                   # Visual analysis (363 lines)
└── README.md
```

---

## Limitations

- **No automated tests.** These are prompt/config files — there is no test
  suite verifying agent behavior across query types.
- **Dependency on external APIs.** The orchestrator dispatches to 8 subagents,
  each requiring specific API keys and tools. Without all of them configured,
  the full pipeline won't execute.
- **Model dependency.** The orchestrator is configured for DeepSeek V4 Pro.
  Behavior with other models has not been tested.
- **Instagram fragility.** Instagram extraction depends on Camoufox stealth
  and Instagram's DOM structure, which changes frequently.
- **No benchmark results.** Retrieval quality, answer accuracy, and latency
  have not been systematically measured.

---

## License

MIT
