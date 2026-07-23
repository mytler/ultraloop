# Loop engineering: self-prompt without losing quality

The execution loop (change → run the full checks → fix → repeat) says *what* to do each turn. This file is *how to drive it well*: how to prompt **yourself** each iteration, and how to avoid the specific ways a long loop quietly rots. The core discipline of ultraloop — a result is done only when external evidence says so — is exactly what keeps a self-driven loop honest; these are the mechanics that make that hold over many turns.

Grounded in agent research: Anthropic's *Building Effective Agents* and *Effective context engineering for AI agents*; Reflexion (Shinn et al., 2023); Self-Refine (Madaan et al., 2023). Sources at the end.

Two halves: **Part A** — patterns for prompting yourself. **Part B** — the failure modes that erode quality over a long loop, and how to hold it.

---

## Part A — self-prompting patterns (how to prompt yourself each iteration)

Each pattern has a real, paste-ready self-prompt. Use the ones the loop needs; don't run all five on a two-line fix.

### 1. Evaluator-optimizer — the core loop
One pass generates; a **separate** pass evaluates against explicit criteria; the feedback drives the revision. Anthropic: *"one LLM call generates a response while another provides evaluation and feedback in a loop"* — worth it *"when we have clear evaluation criteria, and when iterative refinement provides measurable value."* You already have the criteria: the **DoD**. The catch: the evaluator must be a *separate* pass — a fresh re-read or a subagent — not the same breath that wrote the code, because self-grading in the same turn rubber-stamps its own work.

> **Self-prompt (evaluate):** "Evaluator pass — ignore that I wrote this. For each DoD item, cite the observable evidence it's met (command output, a green run, a screenshot) or mark it UNMET. 'I wrote it correctly' is not evidence."

### 2. Reflexion — reflect before you retry
On a red check, don't blind-retry the same fix. First write a short **verbal reflection** on the root cause, keep it in the iteration log, and use it as context for the next attempt. In the research, adding verbal self-reflection lifted HumanEval pass@1 from ~80% to ~91% — the reflection is what carries the lesson between attempts. This upgrades the existing per-check ≤3-attempt cap: attempts 2 and 3 must each *open* with a reflection, or they're just the same guess again.

> **Self-prompt (2nd/3rd attempt):** "Reflect first, ≤3 lines: what is the actual root cause, vs. what I assumed last attempt? What specifically will I do differently? Now fix the cause, not the symptom."

### 3. Self-Refine — when there's no oracle
For outputs no test or compiler can judge — prose, an API's shape, UX copy, a schema, a design — generate → critique against an explicit rubric → refine, 1–3 rounds. Self-Refine reported ~20% average gains from a single model acting as generator, critic, and refiner. Without a rubric the critique drifts into vibes.

> **Self-prompt:** "No test can judge this. Write a 4–6 point rubric for what 'good' means here, grade the draft against each point, rewrite the weakest point, repeat until no point is weak."

### 4. Self-consistency / vote — for high-stakes forks
When a decision is genuinely uncertain *and* expensive (an architecture choice, a tricky algorithm, a data model), generate 2–3 **independent** approaches and choose by merits/consensus rather than committing to the first thing you wrote. Use sparingly — it's for real forks, not every step (see Anthropic: add complexity *"only when it demonstrably improves outcomes"*).

### 5. Fresh-eyes / adversarial re-read — before "done"
Re-read your own diff as a hostile reviewer seeing it cold. This catches what the author is blind to right before the PR.

> **Self-prompt:** "Read this diff as a reviewer who wants to reject it. Weakest change? Missing test? Unhandled input/error path? The thing that breaks in prod? Fix those before opening the PR."

---

## Part B — don't lose quality: the long-loop failure modes

The through-line: **anchor every iteration in external ground truth, not in your own accumulating narrative.** And verification must *co-evolve* with the work — as the code gets more capable, a fixed check stops catching things, so strengthen the check (a real, independent evaluator), not only the code. A green you engineered is worse than an honest red.

Context rot is architectural, not a you-problem: as the transcript grows, every token attends to every other (n² attention), so recall degrades. It's preventable. Three named modes and their fixes:

### Context poisoning — a wrong "fact" reproduces every step
A hallucination or a bad assumption enters the context and then gets treated as established truth at every later turn, so the whole loop drifts off a false premise.
- **Fix:** when stuck ≥2–3 iterations, **re-ground** — re-read the primary source / failing test / spec from scratch, and distrust any "fact" that was only established earlier *inside this same loop*. Don't build on unverified in-loop claims.

### Context distraction — leaning on history instead of planning
As the transcript grows (this has been observed around ~100k tokens), the model starts relying on the bloated history instead of forming a fresh plan.
- **Fix:** keep a compact **state snapshot** and plan from it, not from the whole transcript. Make the iteration log a live running state, not a diary.

> **Self-prompt:** "State snapshot — green: […] · red: […] · next: […]. Plan the next step from this line, not from the full transcript."

### Context confusion / bloat — irrelevant material degrades output
Stale, resolved detail crowding the window drags quality down. Anthropic's three levers:
- **Compaction** — near the context limit, summarize and reinitialize: **keep** the DoD, the state snapshot, and open reds; **discard** resolved detail. (The art is what to keep — over-aggressive compaction drops a subtlety whose importance only shows up later.)
- **Structured note-taking** — write durable state to a `NOTES.md` / the iteration log and pull it back when needed; persistent state, minimal context cost.
- **Sub-agents** — hand a focused chunk to a subagent with a *clean* window (a `debugger` for one gnarly failure, a `security-reviewer` for the security pass). This is *why* ultraloop's reach for subagents/Workflow protects quality, not just speed.

### Error accumulation / drift — small errors compound over long horizons
Long loops destabilize; each unfixed wrong turn biases the next.
- **Fix:** the loop's existing bounds are drift control, not just budget — short iterations, the ≤3-attempts-per-check cap, and max-turns. When you hit a cap, **stop and re-ground**, don't push a drifting chain further.

### Reward hacking / fake green — optimizing the check, not the goal
Skipping/weakening tests, hardcoding expected values, premature "done". ultraloop already forbids this ("NEVER fake green"); the research reason it matters here: **no fixed verification stays valid as the generator gets stronger**, so the check must be real and independent, and must get sharper as the work does — not gamed.

---

## Optimize for the human bottleneck (Amdahl)

When you generate and verify faster than a human can review, the bottleneck moves *to the human*
— Amdahl's law: overall pace is capped by the part that didn't speed up. Anthropic has observed
exactly this: as more code moves through the org, **human code review became the new bottleneck**.
Closing the loop well means spending your speed on making the human's remaining job cheap, not on
producing more for them to wade through.

- **Cut the reviewer's load, not just your own time.** Only verified-green work reaches the PR;
  make the review cheap — item-by-item DoD→evidence mapping, atomic commits, a description of what
  changed and why. A pile of unverified code doesn't accelerate anything; it just relocates the
  work onto the slowest step.
- **Self-review before handoff.** Run a separate critic pass — fresh-eyes, plus a `code-reviewer`
  / `security-reviewer` subagent — *before* handing the diff to a human. Automated review catches
  bugs that even strong engineers miss: in Anthropic's own retrospective, an automated review of
  every change would have caught roughly a **third of the bugs behind past production incidents**.
  This is evaluator-optimizer pointed at unblocking the reviewer.
- **Execution is yours; taste and direction are the human's — surface, don't ask.** You carry the
  work to a verified result; *which* problem is worth solving, *which* results to trust, and *when*
  an approach is a dead end stay the human's call. Don't request permission (autonomy holds) — but
  **flag direction/taste forks and suspicious results prominently** in the report so a human can
  apply judgment cheaply. That is where human comparative advantage still sits.
- **Kill dead ends early.** Knowing an approach is a dead end is part of the skill. If after a
  re-ground + reflect a branch hasn't moved toward the DoD in a couple of iterations, it's a dead
  end: stop it, record what you tried and why it failed, switch approach or surface it — don't
  grind on from sunk cost (this is exactly what the ≤3-attempts cap and max-turns protect). And
  after you speed one step up, check whether the bottleneck *moved* — optimize the new slow step,
  not the one that's already fast.

## When to reach for which (calibrate to loop length)

Same calibration principle as the rest of ultraloop — match the machinery to the loop:

- **Short / light loop:** evaluator-optimizer against the DoD + a fresh-eyes pass before done. Skip the rest.
- **Long / heavy loop:** add reflect-before-retry on failing checks, a live state snapshot, and — as the transcript grows — compaction, note-taking, and offloading focused chunks to sub-agents.

## Sources

- Anthropic — *Building Effective Agents*: evaluator-optimizer, "add complexity only when it demonstrably improves outcomes", stopping conditions. https://www.anthropic.com/research/building-effective-agents
- Anthropic — *Effective context engineering for AI agents*: context rot (poisoning / distraction / confusion), compaction, structured note-taking, sub-agents. https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- Shinn et al. 2023 — *Reflexion: Language Agents with Verbal Reinforcement Learning* (verbal self-reflection; ~80%→91% pass@1 on HumanEval). https://openreview.net/pdf?id=vAElhFcKW6
- Madaan et al. 2023 — *Self-Refine: Iterative Refinement with Self-Feedback* (generate → critique → refine; ~20% average gain).
- Anthropic report on automating AI development ("closing the loop") — human code review as the Amdahl bottleneck; an automated review would have caught ~1/3 of the bugs behind past production incidents; research taste/judgment (which problem, which result to trust, when it's a dead end) as the remaining human advantage.
