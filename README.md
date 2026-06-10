# Deep Research, Showcased — From a Persuasive Talk to a Verified Investment View

A worked, end-to-end example of the **`/deep-research` multi-agent harness** (Claude Code): how to turn a vague but compelling talk into a **fact-checked, valuation-aware investment view** — with the method's own limitations honestly labeled.

The source is Dan Dreyfus's critical-minerals / commodity-supercycle presentation (All-In). Two `/deep-research` runs verified the thesis, *corrected* the source, priced the opportunities, and — just as importantly — **refused to certify the data it couldn't corroborate.**

> **This repo is a methodology demonstration, NOT financial advice.** See [`LICENSE`](LICENSE). Do your own due diligence.

**▶ Live walkthrough (GitHub Pages):** **https://az9713.github.io/deep-research-demo/** — the `deep-research-harness.js` engine, rendered line-by-line in the browser.

**Last updated:** 2026-06-10

---

## Start here

| If you want to… | Read |
|---|---|
| **Understand how it was built** (the point of the repo) | **[DEVELOPMENT_JOURNEY.md](DEVELOPMENT_JOURNEY.md)** |
| **Understand how `/deep-research` works** | [deep_research_explained.md](deep_research_explained.md) |
| **Understand the engine, line by line** | [deep_research_harness_explained.md](deep_research_harness_explained.md) + walkthrough ([live](https://az9713.github.io/deep-research-demo/deep_research_harness_walkthrough.html) · [source](deep_research_harness_walkthrough.html)) |
| **See the investment output** | [investment_opportunity_analysis.md](investment_opportunity_analysis.md) → [investment_valuation_pass2.md](investment_valuation_pass2.md) |
| **Reuse the prompt on your own topic** | [investment_opportunity_prompt.md](investment_opportunity_prompt.md) |

---

## The documents

| # | Document | What it is | Read it for |
|---|---|---|---|
| 0 | **[DEVELOPMENT_JOURNEY.md](DEVELOPMENT_JOURNEY.md)** | The build process, phase by phase | How the brief was engineered, how the workflow was chosen, what each run taught |
| 1 | **[investment_opportunity_prompt.md](investment_opportunity_prompt.md)** | The engineered research brief | The `scenario → bottleneck → beneficiary → instrument` design + adversarial kill-risks |
| 2 | **[investment_opportunity_analysis.md](investment_opportunity_analysis.md)** | **Run 1** — thesis & opportunities | Bottleneck map, claims audit, 5-tier matrix (~25 names), top-5 ranked, what-to-avoid |
| 3 | **[investment_valuation_pass2.md](investment_valuation_pass2.md)** | **Run 2** — valuation & entry | Valuation matrix, per-name entry framework, best-entry ranking, priced-for-perfection callout |
| 4 | **[deep_research_explained.md](deep_research_explained.md)** | Method write-up | Phases, agent math, both runs documented, when-to-use guide |
| 5 | **[deep-research-harness.js](deep-research-harness.js)** | The actual engine (~350 lines JS) | The real dynamic-workflow script behind every run |
| 6 | **[deep_research_harness_explained.md](deep_research_harness_explained.md)** | Engine explainer | How Claude Code invokes it; how `args` flows in and through |
| 7 | **[deep_research_harness_walkthrough.html](deep_research_harness_walkthrough.html)** ([live](https://az9713.github.io/deep-research-demo/deep_research_harness_walkthrough.html)) | Line-by-line walkthrough (open in browser) | Full copyable source + an annotation beside nearly every line |

**Source material:** Dan Dreyfus — ["The Future of Critical Minerals" (All-In)](https://www.youtube.com/watch?v=xTO1aQ_m44I) *(the talk's transcript was used locally during research but is **not redistributed** in this repo)*

---

## What `/deep-research` is (in one diagram)

```
YouTube talk → transcript  ──►  [1] engineered prompt
                                     │
                    /deep-research    ▼
        ┌──────────── Run 1: "Is the thesis true?" ───────────┐
        │  5 phases · 109 agents · 27 sources · 123 claims     │
        │  → 25/25 verified  →  [2] analysis (opportunities)   │
        └──────────────────────────────┬──────────────────────┘
                                        ▼
        ┌──────────── Run 2: "Is the price right?" ───────────┐
        │  5 phases · 109 agents · 27 sources · 123 claims     │
        │  → 22/25 (3 refuted, 5 names uncovered)              │
        │  → [3] valuation (entry levels)                      │
        └──────────────────────────────┬──────────────────────┘
                                        ▼
                          [4] deep_research_explained
```

Each run = **Scope → Search → Fetch → Verify → Synthesize**. The agent budget breaks down as `1 + 5 + 27 + 75 + 1 = 109`, with **~69% of agents (75 of 109) spent *trying to disprove* the findings** before any reach the report.

---

## Headline findings

**Run 1 — thesis (25/25 claims verified):**
- ✅ Copper, rare-earth-processing, grid-equipment, and uranium bottlenecks are **real and primary-sourced**.
- ⚠️ Two of Dreyfus's headline numbers **don't verify** (the "50,000 t/GW" copper figure; the silver "3 years left").
- 🔁 **"Silver = solar" is backwards** — PV silver demand is *falling* (−19%); the deficit is investment-led.
- 🚨 **Highest-value catch:** China's rare-earth controls are **SUSPENDED until ~Nov 2026** — a fact the talk never mentioned.

**Run 2 — valuation (the margin-of-safety check):**
- The **best-thesis names are the worst-value entries.** PWR (~95x P/E), GEV (~70x EV/EBITDA), ETN (~60% above 10-yr avg) are **priced for perfection**; **FCX** (~8x) and **FSLR** offer real margin of safety.
- The engine **refused to certify** 3 conflicting price claims and left **5 of 10 names unranked** — failing loudly instead of fabricating.

**Best risk-adjusted entries (verified names):** FCX → FSLR → ETN → GEV → PWR.

---

## The method lesson

`/deep-research` is a **truth filter** — only as good as how stably "true" the data is.

| Question type | Example | Result |
|---|---|---|
| **Durable facts** (use it) | forecasts, laws, filings, fundamentals | Run 1: **25/25 verified** |
| **Live values** (don't) | today's price, current P/E | Run 2: **3 refuted, 5 names uncovered** |

Use it when *being wrong is expensive* and *the truth is spread across many sources that disagree*. For live single-source numbers, look them up directly and feed them in.

---

## Repository layout

```
.
├── README.md                          ← you are here
├── DEVELOPMENT_JOURNEY.md             ← how it was built (phase by phase)
├── deep_research_explained.md         ← the method, demystified
├── investment_opportunity_prompt.md   ← the engineered brief
├── investment_opportunity_analysis.md ← Run 1: thesis & opportunities
├── investment_valuation_pass2.md      ← Run 2: valuation & entry
├── deep-research-harness.js           ← the engine: the actual /deep-research workflow script
├── deep_research_harness_explained.md ← how the engine is invoked + how args flows in
├── deep_research_harness_walkthrough.html ← line-by-line annotated source (open in browser)
├── LICENSE                            ← MIT (original work in this repo)
└── .gitignore
```

*The source talk's transcript, raw page captures, and the session transcript are git-ignored (third-party / scratch / personal). The source is the [linked YouTube video](https://www.youtube.com/watch?v=xTO1aQ_m44I).*

---

## Reproduce it yourself

1. Provide your own source(s) — a transcript, PDF, article, or URL.
2. Adapt [`investment_opportunity_prompt.md`](investment_opportunity_prompt.md) to your domain.
3. In Claude Code, run `/deep-research` pointed at the brief (pin your universe/horizon/constraints).
4. Chain a second pass for whatever Run 1 leaves open (here: valuation).
5. Trust the "verified" label *because* the engine refuses to certify what it can't corroborate.

See [DEVELOPMENT_JOURNEY.md](DEVELOPMENT_JOURNEY.md) for the full reusable pattern.

---

## License & sources

Original work (prompt, analyses, write-ups, docs) is **MIT-licensed** — see [`LICENSE`](LICENSE). The underlying source is a **third-party talk** — Dan Dreyfus, ["The Future of Critical Minerals" (All-In)](https://www.youtube.com/watch?v=xTO1aQ_m44I) — whose transcript is **not redistributed in this repository**. **Nothing here is financial advice.**
