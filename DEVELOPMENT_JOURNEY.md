# Development Journey — How This Showcase Was Built

This document records the *process* behind the repository, not just the outputs. It exists because the most useful thing to learn from this example isn't "what stocks came up" — it's **how a vague, persuasive talk was turned into a verified, costed view using a multi-agent research harness, and how the workflow was chosen.**

Each phase below shows the decision, the reasoning, and what was produced.

---

## Phase 0 — The raw inputs

We started from a single source — Dan Dreyfus's ~13-minute talk, ["The Future of Critical Minerals" (All-In)](https://www.youtube.com/watch?v=xTO1aQ_m44I), arguing the US is entering a capital-intensive "critical-minerals supercycle." Working locally we used its transcript plus a prior GPT-generated summary. *(Neither is redistributed in this repo — refer to the linked video.)*

A persuasive narrative full of striking numbers ("we'll need as much copper in 18 years as in the last 10,000") — but **unverified**. The whole project is about closing that gap.

---

## Phase 1 — Engineering the research brief

**Decision:** Don't ask "find me critical-minerals stocks." Engineer a structured brief that forces rigor.

**Reasoning:** A naive prompt produces a hype list of obvious large-caps. The leverage is in encoding a *causal chain* and an *adversarial check*:
- `scenario → supply-chain pinch point → who benefits → specific instrument` — stops the model shortcutting to names everyone already owns.
- A mandatory **claims audit** (verify the headline numbers before relying on them).
- A mandatory **kill-risk per idea** — because Dreyfus's own warning was "don't buy the theme"; every thesis has a substitution / dumping / overbuild / permitting failure mode.

**Produced:** [`investment_opportunity_prompt.md`](investment_opportunity_prompt.md) — role, context, 6-step method, tiered output spec, constraints.

> **Lesson:** The prompt is where you encode the standard of proof. The harness only enforces what you ask it to.

---

## Phase 2 — Choosing the workflow (the part most people skip)

Before running anything, we worked out *which* tool fit. Three candidates were on the table — `/deep-research`, `/goal`, and `/loop` — and separating them turned out to be the key conceptual step.

| Tool | What it really is | Fit for this task |
|---|---|---|
| **`/deep-research`** | A **workhorse** — does research, verification, synthesis | ✅ Correct primary tool |
| **`/goal`** | A **control primitive** — repeats turns until a *condition* is met | ✋ Weak fit (report quality isn't a verifiable end-state) |
| **`/loop`** | A **control primitive** — repeats on an *interval* | ✋ Wrong for a one-off; right only for ongoing monitoring |

**A correction worth recording:** the first instinct was that `/goal` didn't exist (it isn't in the skills list). A lookup via the `claude-code-guide` agent proved that wrong — **`/goal` is a real built-in Claude Code command** (a continuous-execution controller). The mistake was a useful one: it forced the clean taxonomy above.

> **Lesson:** Skills are *workhorses* (what to do); `/goal` and `/loop` are *control primitives* (when to stop / how often to fire). You pick the workhorse for the task, then optionally wrap it in a control primitive. Don't confuse the layers.

**The invocation** pointed `/deep-research` at the brief file rather than pasting it, pinned the open parameters (universe, horizon, risk tolerance), and re-stated the two must-haves (verify the numbers, give every idea a kill-risk) directly in the command — because a harness weights its argument string more heavily than a referenced file.

---

## Phase 3 — Run 1: "Is the thesis true?"

**Ran `/deep-research`** on the brief.

| Metric | Value |
|---|---|
| Phases | 5 (Scope → Search → Fetch → Verify → Synthesize) |
| Agents | 109 |
| Sources / claims | 27 / 123 |
| Verified | **25 / 25 confirmed, 0 killed** |
| Duration | ~39 min |

**Outcome — a clean success that also *corrected the source*:**
- Confirmed the copper, rare-earth-processing, grid, and uranium bottlenecks from primary sources (Wood Mackenzie, MOFCOM legal analyses, SEC filings, UxC/IAEA).
- **Flagged two of Dreyfus's headline numbers as unverified/wrong** (the 50,000 t/GW copper figure; the silver "3 years left").
- **Reversed a claim:** PV silver demand is *falling* from substitution — the deficit is investment-led, not solar-driven.
- **Caught what the talk omitted:** China's rare-earth export controls are **suspended until ~Nov 2026** — the single highest-value finding.

**Produced:** [`investment_opportunity_analysis.md`](investment_opportunity_analysis.md).

---

## Phase 4 — Run 2: "Is the price right?"

Run 1 raised the obvious next question — the names with the best *story* might be the worst *value*. So we chained a second pass focused only on valuation and entry levels for the top 10.

| Metric | Value |
|---|---|
| Agents | 109 |
| Sources / claims | 27 / 123 |
| Verified | **22 confirmed, 3 REFUTED** |
| Coverage | **5 of 10 names** |
| Duration | ~26 min |

**Outcome — a *partial* result that taught the most:**
- The robust signal held: the best-thesis names (PWR ~95x P/E, GEV ~70x EV/EBITDA) are **priced for perfection**; FCX (~8x) and FSLR offer **real margin of safety**.
- But the engine **refuted 3 conflicting stock-price claims** and left **5 names unranked** — because live prices disagree across sources and go stale by the hour.

> **Lesson (the big one):** `/deep-research` is a **truth filter**, only as good as how stably "true" the data is. Durable facts (forecasts, laws, filings) → 25/25. Live values (prices, multiples) → it correctly **refuses to certify** what it can't corroborate. Failing loudly beats fabricating confidently.

**Produced:** [`investment_valuation_pass2.md`](investment_valuation_pass2.md).

---

## Phase 5 — Documentation & packaging

With both runs done, we wrote the method explainer, an index README, and made the folder publishable.

- [`deep_research_explained.md`](deep_research_explained.md) — phases, the agent math (`1 + 5 + 27 + 75 + 1 = 109`, with 69% of agents spent disproving findings), both runs, when-to-use guide.
- [`README.md`](README.md) — index, reading order, workflow diagram, headline findings.
- This journey doc, a `LICENSE`, a `.gitignore`, and a scrub of personal paths/info for GitHub.

---

## The reusable pattern

What this repo demonstrates, distilled to a template you can copy:

```
1. Start from an unverified narrative (a talk, a pitch, a report).
2. Engineer a brief that encodes the standard of proof
   (causal chain + claims audit + kill-risk per item).
3. Pick the right layer: workhorse skill (/deep-research) for the work;
   control primitive (/goal, /loop) only as a wrapper.
4. Run Pass 1 — verify the STORY. Let it correct the source.
5. Run Pass 2 — verify the ENTRY/PRICE/feasibility.
6. Trust the "verified" label precisely because the engine
   refuses to certify what it can't corroborate.
7. Document the process, not just the answer.
```

**Two passes, ~200 agents, ~8M tokens, ~65 minutes** took a persuasive talk to a fact-checked, valuation-aware investment view — with its own weaknesses honestly labeled. That honesty is the feature worth showcasing.
