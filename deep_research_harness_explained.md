# `deep-research-harness.js`, Explained

The core component behind every `/deep-research` run is a single ~350-line JavaScript file: **`deep-research-harness.js`**. This document explains **(1) how Claude Code invokes it as a dynamic workflow** and **(2) how your question (`args`) flows into and through the script.**

For a near **line-by-line** walkthrough, open **[`deep_research_harness_walkthrough.html`](deep_research_harness_walkthrough.html)** in a browser — it shows the full source (copyable) with an annotation beside almost every line.

> **The one-sentence model:** the `.js` is the **engine** (the fixed 5-phase pipeline); your question is the **fuel** (`args`), injected at runtime. Same engine + different fuel = the two runs in this repo, whose script files were *byte-identical* because the task text was never in the script.

---

## Part 1 — How Claude Code invokes this in a dynamic workflow

### 1.1 What a "dynamic workflow" is

A **dynamic workflow** is a JavaScript program that *orchestrates subagents deterministically*. Instead of one model improvising tool calls, a script decides — with real loops, conditionals, and fan-out — exactly which agents run, in what order, and how their outputs combine. Claude Code runs this script in a sandboxed JS runtime and exposes a small set of orchestration primitives to it:

| Primitive | What it does |
|---|---|
| `agent(prompt, opts)` | Spawns one subagent; returns its text, or a **validated object** if `opts.schema` is given |
| `pipeline(items, ...stages)` | Runs each item through all stages **with no barrier** (item A can be in stage 2 while B is still in stage 1) |
| `parallel(thunks)` | Runs tasks concurrently and **waits for all** (a barrier) |
| `phase(title)` | Opens a progress group in the `/workflows` UI |
| `log(msg)` | Emits a progress line to the user |
| `args` | **The value you passed to `Workflow({args})`, verbatim** — this is the entry point for your question |
| `budget` | The token target for the run |

`deep-research-harness.js` uses `agent`, `pipeline`, `parallel`, `phase`, `log`, and `args`.

### 1.1a Design principles — why *these* primitives, and why so few?

The primitive set is deliberately tiny. The reasoning behind it:

**1. One irreducible unit of work.** `agent()` is the only primitive that *does* anything — it spawns a stochastic worker (an LLM subagent). Everything else exists to compose, feed, bound, or observe agent calls. Remove `agent()` and there is no workflow; remove anything else and you can still (more clumsily) compute. The set has exactly **one "verb" and a few "connectives."**

**2. Borrow the host language; don't reinvent it.** Notice what is *not* a primitive: there is no `sequence()`, `if()`, `loop()`, `map()`, `filter()`, or `variable()`. JavaScript already provides sequencing (`await`), branching (`if`), iteration (`for`/`while`), and data transforms (`map`/`filter`/`reduce`). Re-implementing those as primitives would duplicate the host language and bloat the API for nothing. The set adds *only what JS lacks*: spawning model agents and scheduling them concurrently. **This single decision is why the set is small.**

**3. Two concurrency operators, because there are exactly two join semantics.** Concurrent composition has one fundamental axis — do you *synchronize* (wait for everything) or *stream* (let each item flow)?
- `parallel()` = **barrier / fan-in**: run N things, wait for all. Needed whenever a later step requires the *complete* set — dedup across all sources, rank the full claim pool, synthesize from all findings.
- `pipeline()` = **no barrier / streaming dataflow**: each item runs through all stages independently (item A can be in stage 3 while B is still in stage 1). Needed when items are independent and you want throughput — wall-clock becomes the slowest *single chain*, not the sum of slowest-per-stage.

These two are duals; together they **span** the space of concurrent composition. More exotic patterns (work-stealing, partial barriers, races) are rare and can be *built* from these two plus host control flow — so they aren't primitives.

**4. Power from composition, not enumeration (the combinator philosophy).** Like Unix pipes or functional combinators, the design bets that a few orthogonal operations that compose cleanly beat a large catalogue of special-purpose ones. A bigger API is more to learn, more to misuse, more to keep coherent. Anything derivable from existing primitives + JS is left out on purpose: `retry` = a loop; `map-reduce` = `pipeline` then a final `parallel`/`agent`; `race` = compose + first-resolve.

**5. Separate concerns onto distinct planes** — each primitive owns one concern, with no overlap:

| Plane | Primitive(s) | Concern |
|---|---|---|
| Compute | `agent` (+ host control flow) | do the work |
| Concurrency | `parallel`, `pipeline` | compose work; correct join semantics; concurrency caps; error isolation; resume |
| Input | `args` | the parameter |
| Limits | `budget` | resource envelope |
| Observability | `phase`, `log` | progress to the human (pure side-effect) |
| Nesting | `workflow` | compose whole workflows |

**6. The unifying idea: deterministic control around stochastic work.** The script is deterministic JavaScript; the agents are non-deterministic LLMs. The primitives are precisely the **seam** between them. That is also why **schema validation lives on `agent()`** — it converts fuzzy model output into typed data the deterministic layer can branch on. The primitive set's whole job is to let deterministic code (a) spawn stochastic work, (b) compose it concurrently with correct join semantics, (c) turn its output back into typed data, and (d) observe and bound it — and to *borrow everything else from the host language*.

### 1.1b Is the set complete / exhaustive?

Not in a formal closed-algebra sense — you *can* add primitives (indeed `workflow` and `budget` already sit beyond the core three), and nothing forbids a future `race()` or `retry()`. The honest, useful claim is narrower: **it is a minimal sufficient basis under a stated philosophy.** The criteria that define "complete" here:

1. **Inclusion test — irreducibility.** A primitive earns its place only if it *cannot* be reconstructed from the others plus the host language. `agent` (can't fake an LLM call), `parallel`/`pipeline` (can't get the scheduler's concurrency cap, error→`null` isolation, progress, and resume from raw `Promise.all`), and `args`/`budget`/`phase`/`log` (each a distinct runtime capability) all pass.
2. **Exclusion test — derivability.** If a candidate is expressible by composing existing primitives + JS, it stays out (`sequence`, `if`, `loop`, `map`, `retry`, `map-reduce`, `race`). This is what stops the set from growing.
3. **No host-language duplication.** Sequencing / branching / iteration / data-transforms are JavaScript's job, not the harness's.
4. **Concurrency-join completeness.** The barrier (`parallel`) / no-barrier (`pipeline`) pair covers both ends of the synchronization axis, so *any* concurrent composition can be assembled.
5. **Computational adequacy.** `agent` + host control flow can express any DAG of agent calls (it is orchestration-complete); `parallel`/`pipeline` add *concurrency and scheduling correctness*, not raw expressive power. So the set is sufficient to express any orchestration, and minimal in that removing any member loses a capability not recoverable from the rest.

In short, the set is "complete" the way a **basis** is complete — small, orthogonal, spanning — not the way an exhaustive feature list is. **This harness is the proof by demonstration:** a non-trivial, branching, fault-tolerant, resumable, 109-agent pipeline expressed entirely in `agent` + `pipeline` + `parallel` + `phase`/`log` + `args`.

### 1.2 The `meta` block makes it a *registered* workflow

The file begins with `export const meta = { name: 'deep-research', … }` (lines 1–6). Three things matter:

- **`name: 'deep-research'`** registers the script under that id. Because it's registered, you invoke it **by name** — and *every* invocation runs **this exact script**. That is precisely why the two run files in this project were identical: same registered script, different `args`.
- **`description`** is shown in the permission dialog before the run starts.
- **`phases`** declares the five progress groups (`Scope, Search, Fetch, Verify, Synthesize`). These titles must match the `phase()` calls later so the live UI groups agents correctly.

`meta` must be a **pure literal** — no variables or function calls — because Claude Code reads it *before* executing the body.

### 1.3 The invocation path

When you type `/deep-research <question>`, the skill resolves to a single tool call:

```js
Workflow({ name: "deep-research", args: "<your question>" })
```

Claude Code then:
1. Looks up the registered script by `name`.
2. **Persists the script to disk** under the session directory as `deep-research-<runId>.js` (this is the file we copied into the repo) and assigns a **Run ID** (e.g. `wf_4bcb6ee1-d33`).
3. Runs the body in the sandbox, **injecting `args`** as a global.
4. Streams `phase()`/`log()` output to the `/workflows` UI.
5. Returns the script's final `return` value as the tool result.

Because the script is persisted per run, you can **resume** it: `Workflow({ scriptPath, resumeFromRunId })` replays completed `agent()` calls from cache and only re-runs changed/new ones.

### 1.4 The five phases map 1-to-1 onto the code

| Phase (UI) | Code | What runs |
|---|---|---|
| **Scope** | `phase("Scope")` (line 91) + 1 `agent` | Decompose the question into 5 angles |
| **Search** | `pipeline` stage 1 (lines 172–178) | 5 search agents, one per angle |
| **Fetch** | `pipeline` stage 2 (lines 199–222) | Dedup URLs, fetch sources, extract claims |
| **Verify** | `phase("Verify")` (line 247) + nested `parallel` | 3 adversarial voters per claim |
| **Synthesize** | `phase("Synthesize")` (line 290) + 1 `agent` | Merge, rank, cite, write the report |

---

## Part 2 — How `args` is passed in and used

### 2.1 Injection: `args` is a runtime global

You never see `args` declared in the file — Claude Code **injects it** into the script's scope, set to whatever you passed to `Workflow({args})`. Pass a string and you get a string; pass an object and you get an object.

### 2.2 Capture + guard (line 92–95)

```js
const QUESTION = (typeof args === "string" && args.trim()) || ""
if (!QUESTION) {
  return { error: "No research question provided. Pass it as args: Workflow({name: 'deep-research', args: '<question>'})." }
}
```

- Line 92 reads the global `args`, type-checks it's a non-empty string, and stores it as `QUESTION`.
- Lines 93–95 **fail fast** with a friendly error if nothing was passed — the script refuses to run blind.

### 2.3 Threading: `QUESTION` flows into every prompt

After capture, `QUESTION` is woven into **all five phases** so every subagent is anchored to *your* question:

| Where | Line(s) | How `QUESTION` is used |
|---|---|---|
| Scope prompt | 96–105 | "Decompose **this** question … " + QUESTION |
| Search prompt | 130 | `Research question: "<QUESTION>"` in every searcher |
| Fetch prompt | 139 | Extractors are told the question so they pull *relevant* claims |
| Verify prompt | 154 | Each adversarial voter sees the question to judge relevance |
| Synthesis prompt | 306 | The final report is written to answer **QUESTION** |

So a single argument fans out to ~100 agents, each receiving it in a role-appropriate prompt. **Nothing about the *topic* is hard-coded** — change `args` and the entire run re-aims.

### 2.4 Why both run files were identical (the proof)

Because the task lives in `args` (runtime) and not in the script (compile time):

- `diff` of the two run files → **identical**; SHA-256 → **identical** (`8179cadd…`).
- Searching the script for `copper`, `valuation`, `FCX`, etc. → **0 hits**.

The two runs differed *only* by the `args` string:
- **Run 1 (thesis):** *"Execute the analyst brief… verify the copper/silver figures… give every idea a kill-risk."*
- **Run 2 (valuation):** *"For these 10 names, find current valuation and entry levels…"*

Same engine, different fuel.

---

## The agent math (line 348)

The final `return` reports `agentCalls` with this exact formula:

```js
agentCalls: 1 + scope.angles.length + allSources.length + (voted.length * VOTES_PER_CLAIM) + 1
```

```
  1   Scope agent
+ 5   Search agents      (scope.angles.length)
+ 27  Fetch/extract      (allSources.length)
+ 75  Verify agents      (voted.length 25 × VOTES_PER_CLAIM 3)
+ 1   Synthesis agent
─────
 109  total   ← the number reported by both runs
```

**~69% of the agents (75 of 109) exist only to *try to disprove* the findings** — the verification phase. That ratio is the whole design philosophy: deep-research spends most of its compute refuting itself, which is why a surviving claim is trustworthy.

---

## Key constants you can tune (lines 12–15)

| Constant | Value | Effect |
|---|---|---|
| `VOTES_PER_CLAIM` | 3 | Adversarial voters per claim |
| `REFUTATIONS_REQUIRED` | 2 | Refuting votes needed to kill a claim (2-of-3) |
| `MAX_FETCH` | 15 | Cap on sources fetched (cost control) |
| `MAX_VERIFY_CLAIMS` | 25 | Cap on claims sent to the expensive verify phase |

---

## See also
- **[`deep_research_harness_walkthrough.html`](deep_research_harness_walkthrough.html)** — full source + line-by-line annotations
- **[`deep_research_explained.md`](deep_research_explained.md)** — the harness in action across this project's two runs
- **[`DEVELOPMENT_JOURNEY.md`](DEVELOPMENT_JOURNEY.md)** — how the whole project was built
