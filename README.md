# Multi Search Engine (MSE)

**A four-layer methodology handbook for intent-driven multi-source web search — written for AI agents, usable by humans.**

Most "AI search" tools answer one question and stop. MSE is different: it is an
*operating procedure* an agent follows to answer questions that matter — where a
wrong source is worse than no answer.

> 中文简介：MSE 是一套给 AI Agent 用的「意图驱动多元搜索」方法论手册。它不提供搜索引擎，而是规定**怎么搜、搜到什么程度算够、证据怎么记账、结论怎么过检**。

---

## Why it exists

Single-source answers fail in three specific ways:

| Failure | What happens | MSE's answer |
|---|---|---|
| **False confidence** | One page agrees with you, so you stop | Evidence board with per-fact source tier + independent-source count |
| **Fake cross-validation** | 6 channels, all reprinting the same wire story | Source-category coverage check (official / news / UGC / academic / overseas index) |
| **Silent decay** | Channel returns HTTP 200, but the parser is broken | `ok_marker` / `bad_marker` probing + offline fixture regression |

## Architecture

```
Layer 0  Reconnaissance   — detect ambiguity, write a research brief
Layer 1  Decomposition    — break the question into sub-questions
Layer 2  Strategy         — construct queries (operators, echo, snowball, entity tracing)
Layer 3  Execution        — 45 channels, 4-level fallback chain, browser fallback for JS/auth walls
         ↓
         Evidence board   — fact / source / tier / independent count / retrieval status
         ↓
         3 quality gates  — coverage · citation spot-check · query log
         ↓
         Report           — generated ONLY from the evidence board
```

### Execution fallback chain (4 levels)

```
retry → echo search (find mirrors) → browser automation (JS render / login / captcha) → channel fallback
```

### Cost-aware sub-agent layer (optional)

Raw results are pre-processed by cheap/free sub-agents (classification, tagging,
dedup-summarisation) before entering the evidence board — with a **hard rule: enhancement
must never weaken evidence standards**. Three-tier fallback guarantees the pipeline never
stalls, ending with the main model doing the work itself.

## Files

| File | What it is |
|---|---|
| `Multi-Search-Engine_v3.1.0.md` | The handbook — the single entry point (methodology only) |
| `engines.yaml` | Machine-readable channel registry: 45 channels, routing groups, probe markers, status columns |

Channel URLs, statuses and fallback rules live **only** in `engines.yaml`, so the handbook
stays a pure methodology document and never goes stale on endpoint details.

## Quick start

MSE is not a package — it is a document an agent reads and follows. To use it:

1. Give `Multi-Search-Engine_v3.1.0.md` + `engines.yaml` to your agent as context.
2. Pick a mode: `quick` (3 channels) / `standard` (6–8) / `deep` (12–16 + academic/vertical).
3. The agent follows Layer 0–3, fills the evidence board, runs the three gates, and only
   then writes the report.

Most channels are plain HTTPS endpoints requiring **no API key**. Semantic search and
browser automation are optional extensions (see the appendix in the handbook).

## Design principles

- **Zero API key by default** — public endpoints first, paid/optional integrations clearly marked
- **Evidence over prose** — every factual claim carries source tier and independence count
- **Honest degradation** — a failed retrieval is recorded as *failed*, never silently summarised into a conclusion
- **Reproducible audits** — query log with fallback trajectory, so a wrong answer can be traced to a wrong step
- **Enhancement never weakens standards** — cost optimisation may not lower evidence quality

## Status

Current: **v3.1.0** (2026-10-07). The version history at the end of the handbook records
every change, including designs deliberately *not* adopted and why.

## License

MIT — see [LICENSE](LICENSE).

---

*Made by [KuMoo](https://github.com/kumoonet) · Methodology over tooling.*
