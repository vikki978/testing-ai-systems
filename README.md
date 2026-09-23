# Learning Roadmap: Testing AI Systems & Agents

A practical roadmap for experienced SDETs (10+ years in software testing & test automation) who want to learn quality engineering for AI systems, LLM applications, and agents.

---

## Who this is for

- Strong background in functional, automation, API, and e2e testing
- Comfortable with CI, flaky tests, observability, and quality gates
- Want to move into **LLM app QA**, **agent reliability**, or **AI safety/red team**

## Core mindset shift

| Classical testing | AI / agent testing |
|-------------------|--------------------|
| Assert exact output | Assert quality bands, policies, invariants, risk thresholds |
| Deterministic systems | Stochastic systems (sampling, temperature, model drift) |
| Pass/fail per run | Pass/fail against baselines + statistical confidence |
| Logs & screenshots | Traces, trajectories, eval reports, cost/latency budgets |

---

## Phase 0 — Orient (1 week)

**Goal:** Know what you’re testing and what “pass” means.

### Learn
- Types of AI products: LLM apps, RAG, agents, multimodal, fine-tuned models, ML classifiers
- Deterministic vs stochastic behavior
- Product vs model vs system testing boundaries
- Offline eval vs online / production monitoring

### Deliverable
- One-pager: how you would test a ChatGPT-like product vs RAG vs a multi-agent workflow

---

## Phase 1 — LLM Application Testing Foundations (2–3 weeks)

**Goal:** Test a single LLM call / chat feature like a product.

### Concepts
- Prompt contracts (inputs, tools, formats, refusals)
- Hallucination, grounding, instruction-following
- Jailbreak / prompt injection (basic red teaming)
- Latency, cost, and token usage as quality attributes
- Temperature, sampling, and strategies for non-determinism

### Test types

| Type | What you assert |
|------|-----------------|
| Functional | Schema, required fields, tool-call shape |
| Behavioral | Follows instructions, stays in role |
| Safety | Refuses disallowed content |
| Regression | Doesn’t get worse vs golden set |
| Non-functional | p95 latency, cost per request |

### Hands-on
- Build 20–50 golden prompts with expected **properties** (not exact strings)
- Score with: exact match (where valid), rubrics, LLM-as-judge (carefully), reference answers
- Add retry / flaky handling for stochastic outputs (majority vote, tolerance)

### Skills that transfer from classic SDET work
- Page objects → prompt / fixture factories
- Contract tests → JSON schema / tool-call schema checks
- Snapshot testing → semantic snapshots / embedding similarity

---

## Phase 2 — Evaluation Science (3–4 weeks)

**Goal:** Measure quality systematically instead of guessing.

### Learn
- Datasets: golden, adversarial, edge, production-sampled
- Metrics: task success rate, precision/recall (task-specific), exact match, F1, embedding similarity; use BLEU/ROUGE sparingly
- LLM-as-judge: bias, calibration, pairwise vs absolute scoring
- Human eval loops and basic inter-rater agreement
- A/B and shadow testing for model / prompt changes
- Statistical significance for noisy metrics (sample size, confidence intervals)

### Practice project
- Pick an open task (summarization, extraction, Q&A)
- Create an eval harness: dataset → run → score → report → CI gate
- Define pass criteria, e.g. task success ≥ 90%, grounding failures ≤ 2%, p95 latency < X

### Tools to explore (pick 1–2)
Promptfoo, DeepEval, RAGAS, LangSmith, Braintrust, TruLens, Helicone, Phoenix (OpenInference)

> Tip: Keep Playwright / Pytest / Jest — wrap evals as test suites with soft thresholds.

---

## Phase 3 — RAG System Testing (2–3 weeks)

**Goal:** Test retrieval + generation as a pipeline.

### Failure modes
- Retrieval miss / wrong chunk
- Context stuffing / irrelevant docs
- Citation lies (answer not supported by sources)
- Chunking / embedding regressions
- Freshness and access-control leaks in retrieved docs

### Test layers
1. **Retrieval unit** — recall@k, MRR, gold doc in top-k
2. **Context assembly** — size limits, ordering, dedupe
3. **Grounded generation** — answer supported by context; correct citations
4. **E2E** — user question → correct answer + evidence

### Hands-on
- Build or use a tiny RAG app
- Create Q → gold-doc → gold-answer triples
- Separate “retrieval failed” vs “generation failed” in reports

---

## Phase 4 — Agent Testing (4–6 weeks)

**Goal:** Test multi-step, tool-using, stateful systems.

### Concepts
- Planning vs acting vs reflecting loops
- Tool / function calling contracts
- Memory (short-term, long-term) and state machines
- Multi-agent handoffs and orchestration
- Autonomy levels and blast radius

### Failure modes
- Infinite loops / thrashing tools
- Wrong tool / wrong args
- Partial success then silent skip
- Goal drift / over-refusal / under-refusal
- Cascading errors across agents
- Unsafe tool use (delete, email, spend money)

### Testing strategy

```text
Unit:        tool adapters, parsers, prompts, policies
Contract:    tool schemas, auth scopes, idempotency
Trajectory:  step traces match expected plan patterns
Simulation:  fake tools + scripted environments
E2E:         real tools in sandbox with budgets
Adversarial: injection via docs, tools, memory, user
Online:      success rate, human takeover, cost, safety incidents
