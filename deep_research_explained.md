# Deep Research, Demystified — Two Real Runs Documented

*A plain-language record of the two `/deep-research` processes run on the Dan Dreyfus critical-minerals thesis, what each phase actually did, and what it teaches about when the tool works.*

**Date:** 2026-06-10

---

## 1. What `/deep-research` actually is

`/deep-research` is **not** a single AI answering from memory. It is a **multi-agent harness** — a script that spawns ~100 small specialized agents, runs them in a fixed 5-phase pipeline, and makes them *check each other's work* before anything reaches you. Its defining feature is **adversarial verification**: every factual claim must survive a 3-agent vote before it's allowed into the final report.

Think of it as a newsroom, not a columnist: reporters fan out (Search), pull documents (Fetch), fact-checkers try to kill each claim (Verify), and an editor assembles only what survives (Synthesize).

### The 5-phase pipeline (identical for both runs)

| Phase | What it does | Output |
|---|---|---|
| **1. Scope** | Decomposes your question into **5 distinct search angles** so coverage is broad, not redundant | 5 angles |
| **2. Search** | **5 parallel web-search agents**, one per angle, each finding ~6 sources | ~30 candidate URLs |
| **3. Fetch** | De-duplicates URLs, fetches the top sources, and **extracts falsifiable claims** from each | ~27 sources → ~123 claims |
| **4. Verify** | **3-vote adversarial check per claim** — agents *try to refute* it; a claim is killed if ≥2 of 3 vote against it | confirmed / killed |
| **5. Synthesize** | Merges duplicate claims, ranks by confidence, attaches citations, writes the report | final findings |

### Where the ~109 agents go (the math)

Both runs spawned **109 agents**. The breakdown maps cleanly onto the pipeline:

```
  1  Scope agent        (decompose the question)
+ 5  Search agents      (one per angle)
+ 27 Fetch/extract      (one per fetched source)
+ 75 Verify agents      (25 claims × 3 adversarial voters)
+ 1  Synthesis agent    (assemble + cite)
─────
 109 total
```

> The verification phase is **69% of all the compute** (75 of 109 agents). That is the whole point: deep-research spends most of its effort *trying to disprove its own findings*, not generating them.

---

## 2. Run 1 — "What should I invest in?" (the thesis pass)

**Question:** Uncover US investment opportunities grounded in the Dreyfus critical-minerals supercycle thesis; verify the headline copper/silver numbers; give every idea a kill-risk.

| Metric | Value |
|---|---|
| Run ID | `wf_4bcb6ee1-d33` |
| Phases | 5 |
| Agents | **109** |
| Sources fetched | 27 |
| Claims extracted | 123 |
| Claims verified | 25 |
| **Confirmed / Killed** | **25 / 0** |
| Findings after synthesis | 6 |
| Duration | **~39 minutes** |
| Tokens (subagents) | ~4.0 million |
| Tool calls | 643 |

**The 5 angles it chose:**
1. Copper supply-demand fundamentals + number verification
2. Copper miners / developers / ETFs (the investable set)
3. Silver PV supply deficit + silver names
4. Rare-earth processing/magnets + US government deals (the China chokepoint)
5. Grid T&D / transformers / EPC labor + energy enabling AI load

**What it found (all 6 findings high-confidence, 25/25 claims survived):**
- ✅ **Copper gap is real** — Wood Mackenzie: demand +24% to 42.7 Mtpa by 2035, ~$210B of new mines needed, 28% of supply uncommitted.
- ⚠️ **Dreyfus's specific copper number is wrong** — the "50,000 t/GW" data-center figure couldn't be verified; NVIDIA publicly corrected a related figure to ~200 t/GW.
- 🔁 **"Silver = solar" is backwards** — the deficit is real (6th straight year) but PV silver demand is *falling* −19% from substitution; the deficit is investment-led.
- ✅ **China chokepoint is processing, not ore** — verified via MOFCOM legal analyses — **but currently SUSPENDED until ~Nov 2026** (a catalyst the source never mentioned).
- ✅ **US government backstop is real** — DoD's $110/kg price floor + equity + 100% offtake for MP Materials, confirmed from SEC filings.
- ✅ **Grid demand is already booked, not forecast** — GE Vernova $2.4B data-center orders, Eaton orders +42%, from Q1 2026 SEC filings.

**Deliverable:** `investment_opportunity_analysis.md` — bottleneck map, claims audit, 5-tier opportunity matrix (~25 names), top-5 ranked, what-to-avoid.

---

## 3. Run 2 — "Are they cheap?" (the valuation pass)

**Question:** For the top 10 names, find current valuation and entry levels; answer the margin-of-safety question Run 1 raised ("already re-rated on the narrative").

| Metric | Value |
|---|---|
| Run ID | `wf_f99fc7ee-317` |
| Phases | 5 |
| Agents | **109** |
| Sources fetched | 27 |
| Claims extracted | 123 |
| Claims verified | 25 |
| **Confirmed / Killed** | **22 / 3** |
| Names fully verified | **5 of 10** |
| Findings after synthesis | 2 |
| Duration | **~26 minutes** |
| Tokens (subagents) | ~4.0 million |
| Tool calls | 622 |

**The 5 angles it chose:**
1. Grid/electrical valuation (GEV, ETN, HUBB, PWR)
2. Copper miner valuation (FCX, SCCO)
3. Rare-earth re-rated processing valuation (MP)
4. Precious streaming + uranium valuation (WPM, CCJ)
5. Solar policy-sensitive valuation (FSLR)

**What it found:**
- **The robust signal:** the Run-1 *thesis winners are the worst-value entries.* PWR (~95x P/E, +126%/yr), GEV (~70x EV/EBITDA), ETN (~60% above its 10-yr avg) are **priced for perfection**; FCX (~8x EV/EBITDA) and FSLR (modest multiple, growing earnings) offer **real margin of safety**.
- **3 claims REFUTED (0-3 votes):** GEV "$1,050 PT / 0% upside," ETN "$401.72 / 33.39 P/E," PWR "$679 / $104B cap" — killed because sources disagreed.
- **5 names couldn't be verified at all:** MP, CCJ, WPM, SCCO, HUBB — left *unranked* rather than guessed.

**Deliverable:** `investment_valuation_pass2.md` — valuation matrix, per-name entry framework, best-entry ranking, priced-for-perfection callout.

---

## 4. The two runs side by side

| | Run 1 — Thesis | Run 2 — Valuation |
|---|---|---|
| Question type | Durable facts (supply/demand, law, policy) | Live market data (prices, multiples) |
| Agents | 109 | 109 |
| Claims confirmed | **25 / 25** | **22 / 25** |
| Claims refuted | 0 | **3** |
| Coverage | Full | **5 of 10 names** |
| Duration | 39 min | 26 min |
| Verdict | **Clean success** | **Partial — honest failure on live data** |

### The single most important lesson

**Deep-research is a *truth filter*, and it is only as good as how "true" the underlying data is.**

- Run 1 asked about things that are **stably true** — a Wood Mackenzie forecast, a MOFCOM law, an SEC-filed backlog don't change between sources or by the hour. Result: **25/25 verified.**
- Run 2 asked about things that are **not stably true at any instant** — a stock's price and trailing P/E differ across aggregators and go stale within hours. Result: the 3-vote check **correctly refused to certify** conflicting numbers (3 refuted, half the names uncovered).

That second outcome is a **feature, not a bug.** A naive single-AI summary would have confidently printed one of the conflicting prices as fact. Deep-research **failed loudly instead of fabricating** — which is exactly what you want from a verification engine.

> **Rule of thumb:** use `/deep-research` for **durable, verifiable, multi-source facts** (fundamentals, regulations, filings, scientific claims, competitive landscapes). For **live, single-source, fast-moving values** (today's price, current odds, live inventory), use a direct lookup — the harness will distrust them by design.

---

## 5. How to use deep-research for meaningful work

The pattern that worked here generalizes. Deep-research earns its ~4M tokens when **being wrong is expensive** and **the truth is spread across many sources that don't agree.**

**Strong fits:**
- **Investment / due-diligence theses** — verify the claims a pitch rests on before committing capital (exactly this project).
- **Competitive & market landscapes** — who are the real players, what's verified vs. marketing.
- **Regulatory / legal posture** — what a law actually says and its current status (the China-controls *suspension* was the highest-value catch — the source never mentioned it).
- **Scientific / technical literature reviews** — separate established findings from single-study hype.
- **Fact-checking a document** — feed it a report and have it adversarially test the load-bearing claims.

**Poor fits (use a direct query instead):**
- Live prices, current scores, real-time inventory, anything that changes by the hour.
- Questions with one authoritative source (just read that source).
- Subjective/taste questions with no verifiable ground truth.

**How to get the most out of it:**
1. **Pre-specify the angles** — both runs included an explicit angle list, which made the Scope phase sharper than a bare question.
2. **Demand a kill-risk / disconfirmer per item** — forces the Verify phase to do real adversarial work.
3. **Tell it to flag what it *couldn't* verify** — that's how Run 2 surfaced the 5-name gap honestly instead of hiding it.
4. **Match the question to durable facts** — if you need a live number, get it live and feed it *in*, don't ask the harness to certify it.
5. **Chain passes** — Run 1 (is the thesis true?) → Run 2 (is the price right?) is a reusable two-step: verify the *story*, then verify the *entry*.

---

## 6. One-paragraph summary

Two `/deep-research` runs, 109 agents each, ~4M tokens and ~30 min apiece, took the Dreyfus critical-minerals talk from a persuasive narrative to a verified, costed investment view. Run 1 confirmed the supply-demand thesis (25/25 claims), *corrected* two of Dreyfus's own headline numbers, and caught a market-moving fact the source omitted (China's controls are suspended). Run 2 then showed the best-thesis names are the worst-value entries — and, just as usefully, **refused to certify** stock prices that its sources disagreed on. The takeaway: deep-research is a disprove-it-first engine that shines on durable facts and honestly fails on live data — which is precisely what makes its "verified" label worth trusting.

---

### Artifacts produced across this project
- `investment_opportunity_prompt.md` — the engineered research brief
- `investment_opportunity_analysis.md` — Run 1 output (thesis + opportunities)
- `investment_valuation_pass2.md` — Run 2 output (valuation + entry)
- `deep_research_explained.md` — this document
