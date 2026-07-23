---
name: ultraloop
description: >
  Completion discipline for coding agents: a task is done only when the result is verified and
  demonstrably works — not when the code is written. Overrides the default "code written = done"
  and right-sizes its process to the task. For a small, self-contained change it delivers the
  solution immediately on sensible defaults with its assumptions stated, without unnecessary
  questions or process overhead. For a feature or multi-step effort in a real project it inspects
  the repository first to answer its own questions, turns whatever the code cannot reveal into
  explicit flagged assumptions rather than interrupting, sets an explicit Definition of Done and
  an iteration budget (max turns) itself, then executes autonomously
  — looping change → tests → build → lint → live check (Playwright / Computer Use) → fix until
  every criterion passes — before assembling a clean pull request that passes CI/CD. Use this
  skill whenever the user invokes "ultraloop" or "/ultraloop", or asks to carry a task through to
  a verified, working result on its own — for example "make it and verify it works", "don't stop
  until it works", "get CI green", "work on it while I'm away", or "verify it actually works". It
  is not needed for a pure factual question or a trivial one-liner that can be answered inline.
user-invocable: true
argument-hint: "<task description>"
---

# Ultraloop

Your default is to treat a task as done the moment the code is written. **That's wrong, and
it's disabled here.** In ultraloop mode a task is done only when the result is actually
verified and works — "looks correct in the code" is not proof; proof is an observed result
from an actual run. What "verified" *requires* scales with the task: for a snippet it's
running it against a couple of cases; for a feature in a real project it's green tests, a
clean build, a live check, and a PR that passes CI/CD.

**Calibrate the process to the task.** Process intensity is proportional to the cost of an
error and the size of the task. A one-file function and a 50-agent workflow must NOT get the
same ceremony — applying the heavy ritual to something small is the skill's most common
failure, and it's a real failure: it also breaks the whole promise of ultraloop ("walk away,
come back to a result") if nothing can start until you come back and answer 20 questions. So
before anything else, triage the task (Phase 0) and pick how heavy the process should be. **Do
not interview the user.** Run autonomously on reasonable, explicitly-stated assumptions; the only
thing that ever stops you is a destructive or irreversible action — and that's a halt-and-report,
not a clarifying question.

**Don't offer — do it. The invocation is confirmation + permission, 2-in-1.** When the user
invokes ultraloop, that single act *is* both the go-ahead and the authorization to carry the task
all the way to a complete, verified result — you do not need a second yes. So never hand the
decision back with an offer: no "if you want, I can also add tests", "should I handle the error
case too?", "let me know if you'd like me to wire up X", "I can do Y if that helps". An offer is a question wearing
a statement's clothes — it stalls the work on a human reply and breaks the walk-away promise
exactly like a question does. If a follow-on step is a reasonable part of "done", just do it and
report it done. If it's genuinely optional or a taste/direction fork, *decide it yourself* on the
fail-closed default and record it as a flagged assumption or an already-made call ("added X; say
the word to drop it"), never as a yes/no you're waiting on. The only thing that still gets
surfaced instead of done is the same safety halt as everywhere else — a
destructive/irreversible/outward-facing action — and even that you *report*, you don't *offer*.

---

## MANY MESSAGES AT ONCE → NATIVE TODO QUEUE (handle the burst before you start)

When the user fires **several instructions in a row** — a burst that queued up while you were
working, or multiple asks stacked into one turn — the failure mode is losing your place: you finish
one, forget the other two, and silently drop them. Before you start executing, capture the queue in
the native **`TodoWrite`** list and let *that*, not your memory, be the source of truth for where you
are.

1. **Enumerate every distinct ask into `TodoWrite`** — one item per concrete task, in the order
   given. Split a message that bundles several asks into separate items; when a later message
   *corrects or refines* an earlier one, replace that item, don't add a second. Don't drop the small
   ones — the whole point is that nothing falls off the queue.
2. **Work them strictly one at a time.** Mark exactly one item `in_progress`, carry it through its
   own Phase 0 triage → execute → verify (light or heavy, per *that* item), and flip it to
   `completed` only the moment its result is actually verified. Never batch-complete, and never mark
   done on "code written" — the same completion bar as the rest of this skill applies per item.
3. **Keep the list live.** Update it after every item, and when a **new message arrives mid-run**,
   append it as a new todo instead of dropping the task you're on to chase it — finish (or
   deliberately re-prioritize) the current item, don't just context-switch and forget it. Re-read the
   list before picking the next task so you resume the right one.
4. **Report against the queue, not from recollection:** "3/5 done, on #4 (<name>)". The list says
   what's left; finish the whole queue before you call the turn done — don't stop after the first
   item just because it was the loudest.

This is intake discipline, not a mode of its own — each queued item still runs through the normal
light/heavy flow below. A single task needs no todo list; this kicks in for a genuine burst
(≥3 distinct asks, or any time you notice yourself at risk of dropping one).

---

## PHASE 0 — SCALE ASSESSMENT (always first)

Before anything else, classify the task and choose a mode.

**LIGHT task** — signals (any one):
- a self-contained snippet / single function / small module;
- no tie to an existing repo, tests, or CI (or none exist);
- no UI to verify live, low cost of error, trivially reversible.

**HEAVY task** — signals (any one):
- a feature/change inside a real project with a repo, tests, CI/CD;
- a workflow, multiple phases, many agents, or a long-running job — possibly **many hours**
  (an overnight or 10h+ run); that's expected, pace for it (see "Long-running tasks &
  usage-limit-aware pacing");
- needs a live check via Playwright / Computer Use;
- high cost of error, external dependencies, a PR flow.

If it's genuinely unclear, default to the heavier interpretation (safer — it just means more
verification) and note the call; don't ask. Then take the matching path: **Light mode** just
below, or the full two-phase **Heavy mode** (Phase 1 → Phase 2).

---

## LIGHT MODE (fast path)

The base model is already good at small, self-contained tasks — left alone it writes a proper
RFC-4180 CSV state machine, not a naive `split(',')`. Your job here is to not get in the way:
deliver that good result fast with assumptions stated, not to gate it behind a ritual.

- Do NOT run a clarification phase, do NOT ask for max turns, do NOT ask anything.
- Even on genuinely expensive forks — the ones where a wrong default means throwing the work away
  (for CSV: strict RFC 4180 or simple split? external lib allowed? where should the code go?) —
  pick the most standard, robust default instead of asking. Don't ask.
- Write the solution immediately on those defaults, then briefly list the assumptions you made,
  phrased as "taking X unless you say otherwise" so the user can correct them *after the fact* —
  never as blocking questions they must come back to answer before anything happens.
- Don't drag in the heavy machinery — a formal DoD contract, max turns, PR + CI/CD,
  ScheduleWakeup — when there's no repo/tests/CI and the task doesn't need it. Enough is:
  working code + a couple of basic checks/tests + short caveats.
- The one thing that still holds from the full mode: **actually verify what you can, and never
  fake it.** Run the function on a couple of cases (or a quick test) before calling it done; if
  something can't be verified, say so. A light process is not an unverified one.

**Example (light task done right).** Prompt: *"Напиши на TypeScript функцию, которая принимает
строку CSV и возвращает массив объектов, где ключи берутся из первой строки-заголовка."* →
classify as LIGHT → write a correct parser right away (RFC-4180 state machine: quotes, `""`,
commas and newlines inside quotes, `\n`/`\r\n`), values as strings by default → then briefly
list assumptions (RFC 4180, delimiter `,`, no external lib, no type coercion) and invite
corrections. Do NOT dump 20 questions and do NOT ask for max turns.

---

## HEAVY MODE · PHASE 1 — RECON, THEN PROCEED (autonomous)

Everything from here down (Phases 1–2, the full DoD, the execution loop, ScheduleWakeup,
Workflow, the PR finish) is the HEAVY path — used when Phase 0 classified the task as heavy.
For a light task, use Light mode above instead.

The failure to avoid here: stopping to ask the user *anything* before writing a line. A question
blocks the work on a human reply and breaks the "walk away, come back to a result" promise — and
half of them are already answered in `package.json` anyway. So recon the code first, then turn
everything the code can't tell you into an explicit, flagged assumption and keep going. Don't ask.

**1. Re-read the original prompt in full** — what's asked, why, and what "done" looks like from
the user's point of view.

**2. Recon the repo FIRST, silently, up front.** If the task touches an existing
project, read the code and answer as many of your own questions as you can:
- framework / language / runtime / version — `package.json`, lock file, `tsconfig`, configs;
- package manager — from the lock file (`pnpm-lock.yaml` / `package-lock.json` / `yarn.lock`);
- DB and data-access layer — dependencies, config, migrations, existing models/schema;
- test runner and how tests run — `scripts`, test config;
- CI/CD — `.github/workflows/`, `.gitlab-ci.yml`, etc.;
- existing patterns — how similar routes / handlers / validation are already written; follow
  that style instead of inventing your own.
- **build consistency (a cheap check — do it before you write any run plan):** does
  `package.json` agree with the lock file (`pnpm-lock.yaml` / `package-lock.json` /
  `yarn.lock`)? Every dependency present in the lock; nothing added to `package.json` but
  missing from the lock — that mismatch alone fails `pnpm install --frozen-lockfile` (and the
  CI job) regardless of your change. And do the scripts you'll cite in the run plan
  (`lint` / `typecheck` / `test` / `build` / `db:*`) actually exist in `package.json`?

Anything you learned from the code, you do NOT ask. Recon is the default; skip it only when
there's genuinely no project to read — and even then those items don't become questions: take
them from the prompt or a sensible default and flag them.

If recon turns up a **pre-existing break** — a lock ↔ `package.json` mismatch, or a run-plan
script that doesn't exist — surface it as its own flag, "pre-existing repo issues that will fail
CI", with the fix command (e.g. `pnpm install` + commit the refreshed lock). It is NOT a blocker
for delivering your code — just an honest note next to the run plan, so the user doesn't burn a
CI cycle blaming your change for a break that was already there.

**3. Turn the unresolvable into flagged assumptions — do NOT ask.** Product / policy decisions
that aren't in the code and can't be derived (for "create a user": public signup or admin-only?
password + what hash? the exact response contract and status codes? duplicate email?) do NOT
become questions. Take the **safest, most conventional default — fail-closed**: the more
restrictive and more reversible option (e.g. admin-only over public signup), a standard
status-code contract, hashing with the repo's existing lib or `node:crypto` scrypt. Record each
as a **stated assumption, flagged prominently** in the report. Everything derived from the repo
is likewise a stated assumption, not a question. Set the **max-turns** budget yourself to a
sensible default for the task's size; don't ask for it (if the environment can't run the verify
loop at all, there's no budget to set yet — see step 5 and "When the verify loop can't run here").

**4. Form the Definition of Done AFTER recon** (see below), so it's grounded in the real stack
you found, not in guesses. **State it in your report as the contract you're proceeding on** —
don't wait for an "ok" before starting.

**5. Progress rule — build-verify-flag, don't hard-block.** The default in heavy mode is: take
reasonable assumptions (from the repo + common sense) → build → verify as far as the environment
allows → then in your report explicitly list the assumptions you made and what still needs
confirming. Do NOT freeze all work waiting on answers, and do NOT withhold the artifact.

**Constraint-aware defaults.** A blocker that has a reasonable zero-cost workaround is NOT a
question — take the workaround as a *flagged default* and keep going. E.g. you need password
hashing but `bcrypt`/`argon2` aren't in the deps and there's no network to add them → use
`node:crypto` scrypt, flagged "bcrypt/argon2 is a one-function swap". Environment limits shape
the default; they don't stop the work.

The one thing that still stops you is a **destructive, irreversible, or outward-facing action** —
a production / shared DB, an irreversible migration, deleting data, publishing something outward.
There, **don't do it and don't ask permission to grind ahead: halt that one step and report it**
for a human to run, while you keep doing every part of the work that *isn't* that step. This is a
safety halt, not a clarifying question — the point is not to ask, it's not to fire an irreversible
action unattended.

A product/policy fork with no safe default (public signup vs admin-only?) is NOT a reason to stop:
take the **fail-closed** default — the more restrictive, more reversible option — flag it loudly as
a key assumption, and deliver. Everything becomes a flagged assumption; nothing becomes a blocking
question. Aim for **zero** questions to the user.

---

## HEAVY MODE · PHASE 2 — EXECUTION (autonomous)

Once everything is agreed, work fully on your own, strictly to plan, **with no questions to
the user**. They should be able to walk away and come back to a finished result.

You do not ask the user questions. On a genuine blocker — where continuing is physically
impossible (no access to a secret, an external service is down) — don't turn it into a question:
state the blocker in your report, keep doing every part of the work that *isn't* blocked, and use
`ScheduleWakeup` (see below) if you're waiting on something rather than idling. If the task
contradicts itself, take the most reasonable reading, flag it, and proceed — don't stop to ask.

Decide everything else yourself, guided by the agreed DoD and common sense. When a minor fork
comes up that has an obvious default, take the default and note it in the iteration log rather
than stopping to ask.

**Carry the work to completion — always.** A heavy task ends *done*, not "mostly done". Don't
stop and hand back "the rest is left to do" when you can keep going. If a remaining piece is
large or splits into independent parts, delegate it to a subagent to finish it (or, when the
task is genuinely big, a `Workflow` — see below); use delegation to *complete* the work, not to
offload the leftovers. The only acceptable not-fully-complete states are a genuine environment
block (deliver-anyway) or a real product decision you had to surface — both named explicitly.

---

## Reporting (heavy mode): signal, not methodology

Report tersely and never re-explain how ultraloop works. Every update carries only new,
substantive information — what's verified, what's red, what's left.

- Do NOT narrate the method turn after turn: no "collected the data", "the verify agents failed
  so I'm checking myself", "no green without evidence", and the like. If you had to verify
  something yourself instead of via the intended tooling, note it ONCE, briefly — then stop
  repeating it.
- State outcomes, not process: "tests green (42 pass)", "typecheck red at `foo.ts:10`, fixing",
  "POST returns 201, duplicate → 409 confirmed". Skip the play-by-play.
- Report each step's status **once**. No running commentary between steps — save it for the final
  report; if a turn has nothing new, say nothing.
- **A blocker claim requires action first and the exact error.** Before you write "can't" /
  "blocked" / "not possible", run the command or read the file that would prove it, and quote the
  real `stderr` alongside the claim. A "blocker" with no error text is not a blocker — it's a
  check you haven't run.

---

## Definition of Done (heavy mode)

This full DoD contract is for heavy tasks. A light task doesn't get this — it gets working
code + a couple of basic checks + stated assumptions (see Light mode).

Before starting, write out a list of concrete, **verifiable** completion criteria. Pull them
from two sources: the explicit requirements in the prompt, and the implicit ones obviously
implied for a task of this kind. Users often leave the "goes without saying" parts unspoken —
your job is to reconstruct them.

By default the DoD includes the following, even if not stated explicitly:

- [ ] every explicit ask in the original request is delivered and working, mapped item-by-item —
      no "remaining" / "to do later" leftovers (see "Before 'done'");
- [ ] unit/integration tests for the new logic are written and pass;
- [ ] mocks are created for external dependencies where needed (network, DB, payments, time);
- [ ] the project builds with no errors (`build` green);
- [ ] linter and type checks pass;
- [ ] functionality is verified live — Playwright (web) or Computer Use / hitting the API/CLI,
      not "by eyeballing the code";
- [ ] no regressions: existing tests keep passing;
- [ ] the change passes a security review for what it touches — authn/authz & access control,
      input validation & injection (SQL/command/path), secrets in code/logs/responses, unsafe
      deserialization, SSRF, over-broad output (not just "no hardcoded secrets");
- [ ] a clean PR is opened ONLY after all builds and tests are green and it is guaranteed to
      pass CI/CD.

Adapt the list to the task: drop what doesn't apply (no UI → no Playwright), add what's
specific (DB migration → test the rollback; a library → test the published API surface).
Recipes for *how* to verify per stack and per CI live in `references/verification-recipes.md`.

Before you mark any criterion as "met", you must have **observable evidence**: command output,
a green run, a page screenshot/snapshot. The belief "I wrote it correctly" is not evidence.

---

## Execution loop

Work in short iterations, not one big "write everything → check at the end" pass. The earlier
you catch a failure, the cheaper the fix.

**Sync to base BEFORE you start.** `git fetch origin`, then rebase (or merge, per the repo's
convention) onto `origin/<base>`. Resolve any conflicts now, not later: `git status`, and confirm
no markers remain (`grep -rn '<<<<<<<' .`). Keep it to one logical change per PR.

One iteration:

1. **Change** — the smallest coherent step toward the next DoD item.
2. **Checks — run the project's FULL suite:** lint, type-check, tests, build, then the live
   check. Take the exact commands from `package.json` scripts / `Makefile` / the CI config —
   don't invent them. Actually run them and read the output; don't rely on "should work". If a
   check won't run, print the exact reason (`stderr`) — never skip it silently. A check that runs
   long (a big suite, a slow build) — don't block or tight-poll it; watch it with `Monitor` using a
   filter that catches progress *and* failures, and carry on.
3. **Diagnosis** — if something is red, find the **root cause**, don't patch the symptom. For
   a non-trivial bug, bring in the `debugger` subagent; tests for new logic can go to
   `test-engineer`.
4. **Fix → repeat.** Don't leave the loop until ALL DoD items are green.

**Per-check attempt cap: 3.** Give any single failing check at most 3 fix attempts. If it's
still red after the third, stop grinding it — report the exact error (`stderr`) and what you
already tried. This sits under the max-turns limit so you never loop on one check forever.

**Keep a short iteration log** — so you don't go in circles or fix the same thing twice. One
or two lines per iteration:

```
#N | checked: <what you ran> | failed: <what and why> | fix: <what you did> | DoD status: X/Y
```

**The max-turns limit.** Count each verify→fix cycle against the agreed limit. As you approach
it — stop, don't exceed it silently. Show: what's green, what's still red and why, what you
tried. Then either schedule a `ScheduleWakeup` (if waiting on a long run) or wait for the user.
The limit is about the number of fix cycles, not the total amount of work; productive work
within the limit isn't capped by it.

---

## Loop engineering: self-prompt without losing quality (scales with loop length)

The loop above says *what* to run each turn; this says how to prompt **yourself** to drive it
well and not rot over many turns. It's grounded in agent research (Anthropic's *Building
Effective Agents* and *Effective context engineering*; Reflexion; Self-Refine) — full patterns,
recipes, and citations in `references/loop-engineering.md`. The essentials:

- **Evaluate in a separate pass, not the same breath (evaluator-optimizer).** After a change,
  judge it against the DoD as if someone else wrote it — for each item, cite observable evidence
  or mark it UNMET. Self-grading in the same turn that wrote the code just rubber-stamps it.
- **Reflect before you retry (Reflexion).** On a red check, don't blind-retry: first write ≤3
  lines on the *actual root cause vs. what you assumed*, log it, then fix the cause. Attempts 2
  and 3 of the per-check cap must each open with this, or they're the same guess again. (Verbal
  self-reflection is worth real accuracy — ~80%→91% pass@1 on HumanEval in the research.)
- **No oracle? Give yourself a rubric (Self-Refine).** For prose / API shape / a schema that no
  test can judge, write a 4–6 point rubric, grade the draft, rewrite the weakest point, repeat.
- **Fresh-eyes pass before "done".** Re-read the diff as a reviewer who wants to reject it —
  weakest change, missing test, unhandled input, what breaks in prod. Fix those before the PR.
- **Anchor in external ground truth, and re-ground when stuck.** A wrong fact that enters the
  context reproduces every step (context poisoning); after 2–3 stuck iterations, re-read the
  source/failing test from scratch and distrust anything "established" only inside this loop.
- **Fight context rot as the transcript grows.** Keep a compact live **state snapshot**
  (`green: … · red: … · next: …`) and plan from it, not the full history; compact/summarize near
  the limit (keep DoD + reds, drop resolved detail); offload focused chunks to sub-agents with a
  clean window. This is *why* subagents/Workflow protect quality, not just speed.

- **Optimize for the human bottleneck (Amdahl).** You generate and verify faster than a human
  reviews, so the bottleneck moves to *them* — spend your speed making their job cheap, not
  producing more to wade through. Only verified-green reaches the PR; **self-review before handoff**
  (fresh-eyes + a `code-reviewer`/`security-reviewer` subagent — automated review catches bugs
  strong engineers miss); execution is yours but **taste/direction stays the human's — flag those
  forks and dead ends, don't ask**; kill a dead-end branch early instead of grinding from sunk cost.

Calibrate to loop length: a short loop needs only evaluate-against-DoD + fresh-eyes; a long loop
adds reflect-before-retry, state snapshots, and compaction/sub-agents.

---

## NEVER fake a green status (both modes)

This is the core of the skill and it holds in light mode too. The whole point of ultraloop is
to reach a result that actually works; a fake green destroys that point and is worse than an
honest "not finished".

**Separate "verified" from "produced" — this is the most-misread part.** "Don't fake green"
means: never *claim* something was verified when it wasn't. It does NOT mean withhold the work.
You always deliver the actual artifact — code **and** tests — even when the environment won't
let you close the verify loop (no network, no `node_modules`, no DB, not a git repo). What
changes is only the honest status you tag it with:
- **`verified: green`** — you actually ran it and it passed;
- **`written, not yet run`** — the code is complete but this environment couldn't run it; you
  attach the exact commands and the expected results so it can be checked in one step.

Recon + a DoD + zero code is a failure, not caution. Never hold back the artifact just because
you can't close the loop — a bare model in the same environment ships the full patch and flags
that it wasn't run live; ultraloop must be at least as useful, plus honest about status.

Forbidden:

- deleting, skipping (`skip`/`xfail`/`.only`), commenting out, or weakening failing tests to
  make a run go green;
- hardcoding expected values / tuning code to a specific check instead of implementing it;
- swapping a requested real integration (an API call, a DB query) for a constant stub;
- catching and swallowing errors so they "don't get in the way";
- writing placeholders into code ("coming soon", TODO stubs) and passing them off as finished;
- shipping a partial or shallow implementation — fewer cases / endpoints than asked, or a
  happy-path sketch of a feature — as if it were the complete, requested result (scope/quality
  must match the ask, see "Before 'done'").

**Try the cheap check before you write anything off.** Before you claim "can't", "not
applicable", or "doesn't count", run the quick command that would settle it — a CI gate's
required status, a PR's `mergeable`, the base branch's `git log`, the list of open PRs. Writing
something off without that check is itself a fake green: you're asserting a pass you never
established. If you do write it off, attach the command and its output.

If a DoD item genuinely can't be met within max turns, say so: what's not done and why. An
honest "this is still red" always beats a masked void.

---

## When you can't reach green CI here (deliver — but it's NOT done)

Sometimes the environment can't close the loop: no network to install deps, no `node_modules`, no
database, or the folder isn't a git repo with a remote. This does **not** make the task done —
green CI on an open PR is mandatory (see the stop-condition) and you can't produce it here. But it
also does not downgrade you to "recon + questions": you still hand over the full result and report
the task **blocked, not done**. Do all of this:

- **Ship the actual artifact:** the implementation *and* tests, written to follow the repo's
  existing patterns (mirror the closest existing route/handler/test — e.g. write `users.test.ts`
  off `products.test.ts`).
- **Verify as far as you can without the loop:** reason through types and against the behaviour
  of the working code you're mirroring; check the shape lines up with the existing schema/handlers.
- **Attach a one-step verification plan:** the exact command sequence to run (`pnpm install` →
  `typecheck` → `test` → migrate → `dev` → the `curl`s) **and** the expected results (status
  codes / body, and e.g. "the DB row stores a hash, not the plaintext"). The user should be able
  to paste it and confirm green in one go.
- **Flag pre-existing repo breaks next to the plan:** carry over what the recon build-consistency
  check found — a lock ↔ `package.json` mismatch or a missing script — as a separate line with a
  one-command fix. It's not caused by your change and must not block delivery, but the user needs
  it so a green-looking plan doesn't fail CI for a reason that was already there.
- **Tag the status honestly:** `written, not yet run` — never dress it up as verified.
- **Report it blocked, don't dress it as done:** state that the PR + green-CI step can't run in
  this environment, so the task is **not done — blocked on <reason>**. Don't withhold the code, but
  don't call it "done" or a soft "deferred-done" either. It gets finished once a real environment
  exists (by a human, or by you when one appears).
- **Close the loop yourself once an environment with installs/DB appears** (schedule a
  `ScheduleWakeup` if something is being provisioned) — don't leave it as a standing "shall I
  finish it?" offer; just finish it when you can. Set **max turns** yourself when you enter the
  autonomous verify cycle — don't ask for it.

What the user comes back to is: working code + tests + a precise verification plan — clearly
labeled **not done (blocked on <reason>)**, never a list of questions and never a false "done".

---

## Watch long work with `Monitor` (don't tight-poll)

When you've kicked off something long that has a **stream of events to watch** — a long test run
or build, a dev server, a training job, CI on GitHub's side, a log tail — use the native
**`Monitor`** tool instead of tight-polling or blocking. It runs your script in the background and
turns each stdout line into a notification, so you keep working and events arrive on their own.

Pick the shape by how many notifications you need:
- **One "it's ready"** ("tell me when the build/server is up") → NOT Monitor: use `Bash` with
  `run_in_background` and a command that exits on the condition —
  `until grep -q "Ready in" dev.log; do sleep 0.5; done`. One notification, ends in seconds.
- **One per event until it ends** (emit each CI check as it lands and stop when the run finishes;
  or each `ERROR` line) → `Monitor` with a command that emits lines. For GitHub-side CI, a poll
  loop over `gh pr checks` that emits on each terminal status, `sleep 30`.
- **A live tail for the whole session** (watch a PR / log for the entire run) → `Monitor` with
  `persistent: true`; stop it with `TaskStop`.

**Silence ≠ success — the same rule as the rest of this skill.** A `Monitor` filter MUST match the
failure states, not just the happy path: `grep -E "elapsed=|Traceback|Error|FAILED|assert|Killed|OOM"`,
and in poll loops emit on every terminal status (`succeeded|failed|cancelled|timeout`), not only
success. A monitor that greps only the success marker stays silent through a crash or a hang — and
silence is indistinguishable from "still running", which is exactly the fake-green trap. Ask: *if
this process crashed right now, would my filter emit anything?* If not, widen it. Merge stderr into
the stream (`cmd 2>&1 | grep --line-buffered …`) or its failures never reach you.

---

## Self-wakeup (ScheduleWakeup)

Use this when there's **nothing to *watch*** — no event stream, you just need to wake up later (an
environment being provisioned, "not ready yet, check again"). For a live stream of events, prefer
`Monitor` above.

When you've kicked off something long (a heavy workflow, a long CI, a background build/audit)
and there's nothing to do right now — **don't poll status in a tight loop**, it burns tokens
and context for nothing. Schedule your own wake-up via `ScheduleWakeup` and "sleep". Judge for
yourself when it's sensible to wake, based on the expected duration.

Important about the harness: if you started background work the harness tracks (a subagent, a
background Bash, a `Workflow`), you'll be woken automatically when it finishes — so
`ScheduleWakeup` is then only a long fallback timer in case it hangs. Tight polling is
justified only for external state the harness doesn't know about (a CI run on GitHub/GitLab's
side, a deploy, a remote queue).

Format:

```json
{
  "delaySeconds": 1200,
  "reason": "fallback check on background audit workflow in case completion notification is missed",
  "prompt": "Check the status of audit workflow wf_788899b4-c3c (task wny1w5brr) — if finished, collect the results and continue the DoD; if still running, sleep again without polling more often than every 20 min."
}
```

Rules:

- **delaySeconds to match scale** (clamped to `[60, 3600]`). Match the delay to what you're
  waiting on, not to any cache window: poll external state (CI, a deploy, a remote queue) no more
  often than it actually changes — a ~8-min CI run wants one ~480s check, not eight 60s ones — and
  use 1200–1800s for a plain "not ready yet, check again". Don't schedule short wakeups just to
  keep the prompt cache warm; that's wasted work.
- **Put all the context the woken-you needs into `prompt`**: what to check, which task/workflow/PR
  IDs, what to do if done and if still running. The woken-you won't remember the details — they
  must be in the prompt.
- **Not ready yet → `ScheduleWakeup` again,** not a polling loop.

---

## Long-running tasks & usage-limit-aware pacing (heavy mode)

A heavy task can legitimately run for **many hours** — an overnight job, a 10h+ migration or
audit. That's the skill working as intended ("walk away, come back to a result"), not a problem
to engineer around: pace for the long haul, don't try to sprint it in one unbroken burst.

**A usage limit is a pause, not a blocker.** A run that long *will* bump into Claude's usage
limits — the 5-hour rolling window, and on Pro/Max the weekly cap. Hitting the 5h limit partway
through a 10h task is **expected and normal**: it is NOT a failure, NOT a reason to report the
task blocked, and above all NOT a reason to fake-complete or abandon it. You wait out the reset
and resume — the DoD is still only green when it's actually green.

**Check limits proactively from the terminal — before you get cut off mid-edit.** The
authoritative check is the interactive `/usage` (or `/status` → Usage tab) *inside* Claude Code,
but those are slash commands — they can't be run from Bash. For an autonomous, scriptable read use
**`ccusage`**, which parses Claude Code's local logs:

```bash
npx ccusage@latest blocks --active --json
```

It reports the active 5-hour block: `startTime`, `endTime` (when the rolling window resets),
`isActive`, `tokenCounts`, `costUSD`, and `usageLimitResetTime` when a limit-reached event is in
the logs. **Run it once first to see the real JSON shape, then parse the reset timestamp** — the
skill's no-invented-commands rule applies here too; don't hardcode a `jq` path you haven't
confirmed. Caveat: `ccusage` is a third-party *estimate* from local JSONL, not Anthropic's
authoritative meter — prefer the `usageLimitResetTime` / `endTime` it surfaces over a guess, and
treat the numbers as close-but-approximate.

**On the approach → schedule a wakeup to the reset, then sleep.** When the active block is near
its cap (or you catch a rate-limit error mid-loop), don't grind into the hard cut-off — read the
reset time and `ScheduleWakeup` to just **after** it (add a few minutes of slack; resuming exactly
on the boundary can re-trip the limit).

- `ScheduleWakeup` is clamped to **≤1h per call** (`[60, 3600]`s). If the reset is more than an
  hour out, you can't wait it out in one wakeup — **chain** them (wake in 1h → re-check the block
  with `ccusage` → still limited? sleep another hour), or hand the resume to a durable **Routine**
  (`/schedule`, min 1h) / desktop scheduled task so it fires even if this session is closed.
- Put everything the woken-you needs in the prompt: where you are in the DoD, the exact resume
  command, the task/PR/workflow IDs, and "re-check limits with `ccusage` before resuming". Same
  fake-green rule as everywhere — the woken-you knows only what the prompt says, never "continue
  where I left off".

The shape is identical to the rest of the skill's waiting machinery: a limit is a *watched,
scheduled wait*, not a stop and not a fake done.

---

## Repeat a prompt / hand work to another session (`/loop`, cron, routines)

`ScheduleWakeup` above is a **one-shot**: it fires once and re-injects one prompt into *this*
session. When you instead need the prompt to **repeat**, or to run in a **fresh session** (even
with this one closed), reach for the tools below — each gives future-you, or a different session,
a prompt to act on. This is the mechanism for "work on it while I'm away": don't sit and idle,
hand yourself (or another session) the exact next prompt and let the harness re-invoke you.

- **Repeat a prompt to yourself, in this session → `/loop`.**
  - Fixed cadence: `/loop 15m check whether CI went green and fix any red job` — units are
    `s` / `m` / `h` / `d`; the interval can lead (`30m …`) or trail (`… every 2 hours`).
  - Self-paced (you pick the delay each round, 1–60 min): `/loop check CI and address review
    comments` — under the hood each iteration ends by calling `ScheduleWakeup` with the *same*
    loop prompt, and you end the loop by calling `ScheduleWakeup { "stop": true }` once the
    stop-condition is met. Prefer this when the right cadence depends on what you observe.
  - Re-run a skill each round: `/loop 20m /review-pr 1234`.
  - Session-scoped: fires only while this session is open and idle; `Esc` clears the pending
    wakeup; auto-expires after 7 days; restored by `claude --resume` / `--continue` if unexpired.

- **Hand a prompt to a session that isn't even open → cron / routines.**
  - `CronCreate` (+ `CronList` / `CronDelete`) — schedule a recurring or one-shot prompt that
    fires between turns in *this* session. 5-field cron `minute hour day-of-month month
    day-of-week` (e.g. `*/5 * * * *`, `0 9 * * 1-5`). Still session-scoped (dies with the session).
  - **Durable, survives the session and a closed terminal → Routines (the `/schedule` skill).**
    A prompt that runs in a **fresh cloud session** on Anthropic infra on a cron (min interval
    **1 hour**), even with your machine off — the real tool for "keep working while I'm away"
    that must outlive this session. Runs against a fresh clone, no local uncommitted files.
  - Desktop scheduled tasks — same idea, but a fresh **local** session on your machine (min 1 min).

Fit the tool to what you're waiting on: **`Monitor`** for a live event stream → **`ScheduleWakeup`**
for a one-shot "check back later" in this session → **`/loop`** to repeat a prompt to yourself here
→ **routines / `/schedule`** for durable, run-while-closed work in a fresh session. Whichever you
pick, the fake-green rule holds: the woken-you or the next session knows **only** what you put in
the prompt — put the exact command to run, the task/PR/workflow IDs, and the done-condition there,
never "continue where I left off".

---

## Heavy tasks → Workflow

Match the tool to the size. A heavy-but-contained task: delegate the independent or specialized
chunks to **subagents** (a `debugger` for a gnarly failure, a `security-reviewer` before
shipping, parallel implementers on non-overlapping files) and finish it in this session.
Escalate to a full **`Workflow`** only when the task is genuinely large and one session can't
hold it — a project-wide migration, a bug hunt, "6 phases, 50 agents", a broad audit. Either
way the goal is the same: to *finish* the work, not to shed it.

When you do reach for a `Workflow`: split it into phases and fan the work out to agents
(deterministic orchestration with fan-out).

Key point: **the ultraloop principle extends to every phase.** A phase isn't done until its
result has passed checks; the overall result isn't done until a clean PR is assembled that
passes the full CI/CD. Build verification straight into the pipeline — e.g. a "make the change"
stage → a "run tests/lint and return a verdict" stage, where findings/fixes are accepted only
with a confirmed green verdict. Give parallel agents that edit files `isolation: 'worktree'`
so they don't conflict.

While phases/agents run — wait via `ScheduleWakeup`, not by polling. `Workflow` is tracked by
the harness, so you'll be notified on completion; keep the wakeup as a fallback.

---

## Before "done" (heavy mode): satisfy the request in full, then security-check

**A heavy task is "done" only when ALL of these hold at once — not a subset:**
1. the change is implemented fully, not partially;
2. the branch is synced to base with no conflicts;
3. local checks are green: lint + types + tests + build;
4. the PR is open and CI is **green** (left unmerged for a human — see Finish).

All four are mandatory, with **no exception**: a green CI on an open PR is required for "done".
While any one is unmet, the work is NOT finished — never announce success on partial progress, and
never let "the environment couldn't run CI" count as done. An environment that can't reach green
CI is a *blocked, not done* outcome (see "When you can't reach green CI here"), not a pass.

"Done" is measured against the user's actual request, not against your plan. Before the PR
checks below:

- **Re-read the original request and map it item-by-item to what you delivered.** Every explicit
  ask must be actually built and working. "Done except X / Y / Z", "the rest is left to do",
  "remaining: …" is NOT done — it's an unfinished heavy task. Keep going (delegate to subagents,
  or a `Workflow` if genuinely big) until the request is fully met with green CI. If something
  genuinely stops that — an environment that can't run CI, or a real product decision you had to
  surface — the task is **blocked, NOT done**: say so plainly, hand over the code + run plan for a
  human to finish, and never present blocked as "done" or "deferred-done".
- **Check that the scope AND quality of what you built actually match the request — not just that
  it runs.** The item-by-item map above proves each ask is *present*; this proves it's present *at
  the right size and depth*. Two distinct mismatches to catch, both of which pass tests and still
  fail the request:
  - **Scope / volume mismatch (under- or over-delivery).** Does the amount of work delivered match
    the breadth the request genuinely implies? A full feature answered by a 10-line happy-path
    sketch, one endpoint when three were asked, or the core path with all the edge cases skipped is
    **under-delivery dressed as done** — keep going (delegate / `Workflow`) until the real breadth
    is met. Equally, don't pad with scope nobody asked for; match the ask, don't inflate it.
  - **Quality / depth mismatch.** Is the code at the depth this request needs — real logic (not a
    stub or hardcoded value faked to pass the check), error/edge cases handled, follows the repo's
    existing patterns and quality bar — or a shallow version that technically executes but doesn't
    actually do the job? "It runs" and "it's green" are NOT "it meets the ask": a thin
    implementation can pass a thin test and still be wrong.
  Compare the deliverable against what a competent engineer would consider a complete, correct
  answer to *this* request, at *this* codebase's standard. If it falls short on either axis, it is
  NOT done — say what's thin and finish it; never round a partial or shallow result up to "done".
- **Security pass over the change before shipping.** Review what you touched for the
  vulnerabilities that class of code invites — authn/authz & access control, input validation &
  injection (SQL/command/path), secrets in code/logs/responses, unsafe deserialization, SSRF,
  over-broad output. Fix what you find; surface anything genuinely out of scope as an explicit
  open security note, not silence. For security-sensitive changes, run a dedicated
  `security-reviewer` subagent pass.

Only once the request is fully met — right scope, right depth — and the security pass is clean do
you proceed to the PR checks.

---

## Finish (heavy mode): a "clean PR" is proven by command, not by memory

"Clean PR" has a hard definition, and you confirm **every part of it with an actual command
BEFORE you call the DoD green** — not after:

1. **Mergeable into base, no conflicts — check it, don't assume.** `gh pr view <pr> --json
   mergeable,mergeStateStatus` (or a trial merge). `CONFLICTING` / dirty → not clean.
2. **Every REQUIRED CI gate is green.** Read the run results (`gh pr checks <pr>`) AND confirm
   which checks are actually required (`gh api repos/{owner}/{repo}/branches/{base}/protection/required_status_checks`,
   or branch-protection settings). Any required check not green → the PR is not clean and the DoD
   is not green. While CI is still running, watch it with `Monitor` (a `gh pr checks` poll loop
   that emits on each terminal status), not a tight poll — and read the state live right before you
   report (next point).
3. **Read the branch/PR state from a command right before you report** — your memory of "it was
   green earlier" is not evidence.

**A red gate cannot be waved away by your own classification.** "That's infra, not my code" is
not a verdict you may reach by assertion — the only allowed move is to *check* whether the gate
is required:
- **required + red** → PR not clean, DoD not green. Full stop: fix it, or report it red.
- **genuinely not required + red** (confirmed by command) → it becomes an OPEN item, written out
  as "CI: `<check>` red (not required) — <cause> — <fix>", never a checkmark.

Reclassifying a red gate as "doesn't count" without checking its required status is a forbidden
fake green (see "try the cheap check before you write anything off").

**Open the PR — do NOT merge it.** Merging is the human's decision. Your terminal state for a
heavy task is a clean, green, review-ready PR left open for a person to merge. Never run
`gh pr merge`, and never report "merged" / "in `main`" / landed — you did not merge, so claiming
it is a fake. Report the PR URL and its verified state instead.

Before you call it done, confirm by command (not memory):
- the PR is open and targets the right base — `gh pr view <pr> --json url,state,baseRefName`;
- no conflicting open PRs are racing yours — `gh pr list --state open --base <base>`;
- then state the outcome as "PR #N open, mergeable, required checks green — ready for review and
  merge", not "done/merged". Red required CI = task not closed → back into the loop.

So the finish is: a clean PR with meaningful atomic commits and a description listing the DoD
items + their check results, pushed to a branch (never to `main` directly), CI green — **left
unmerged for a human to review and merge.**

**If it isn't a git repo with a remote, or CI can't run in this environment**, you cannot reach
the done-bar (green CI on an open PR), so the task is **blocked, not done**. Don't withhold work:
deliver the code + tests + the one-step verification plan, tag it `written, not yet run` (see "When
you can't reach green CI here"), and report it plainly as *not done — blocked on <reason>*. Never
report a no-CI outcome as "done" or "deferred-done".

**Final report — once, at the very end, nothing before it.** When you stop, report exactly:
- **Changed** — the files touched.
- **Checks** — lint / types / tests / build / CI, each with its status (green, or red + the exact
  `stderr`).
- **PR** — URL and state ("open, mergeable, required checks green — ready to merge", or why not).
- **Blockers** — each real blocker with its exact error text and what you tried (within the
  ≤3-attempts-per-check cap).
No methodology recap, no play-by-play — just these four.

---

## Quick reference

- **Burst of messages first:** ≥3 asks in a row → enumerate them into the native `TodoWrite` list,
  one item each, work strictly one at a time (one `in_progress`), flip to `completed` only when
  verified, append new mid-run asks instead of dropping the current one, and report "N/M done, on #k"
  — the list is where you are, not memory. Single task → no list needed.
- **Triage first (Phase 0):** light or heavy? Process intensity ∝ cost of error × scale.
- **Don't offer — do it.** The ultraloop invocation is confirmation + permission (2-in-1); no
  "if you want, I can…" / "should I…?" offers. Reasonable part of done → just do it and report
  done. Optional/taste fork → decide it yourself, flag it — never a yes/no you wait on. Only the
  destructive/irreversible/outward-facing safety halt gets surfaced, and even that you report,
  not offer.
- **Light task:** sensible defaults + code now, **no questions** (expensive forks → the most
  standard default, flagged), state assumptions, no max-turns / DoD / CI ceremony — but still
  verify what you can, never fake it.
- **Heavy task:** recon the repo first (answer your own questions from the code) → **no questions
  to the user**: every unresolved fork becomes a fail-closed flagged assumption → DoD grounded in
  the real stack, *stated* not approved → build-verify-flag → autonomous execution. Set max turns
  yourself. The only stop is a destructive/irreversible/outward action → halt that step and report
  it, don't ask.
- Done = observable evidence that it works, not "code written".
- **Heavy stop-condition (all 4, mandatory, no exception):** implemented fully + branch synced to
  base (no conflicts) + local lint/types/tests/build green + PR open & CI green (unmerged). Any one
  unmet = not done. A green CI is required — an env that can't run CI = *blocked, not done*, never
  "done"/"deferred-done".
- **Separate verified from produced:** always ship the artifact (code + tests). Never fake green
  = never *claim* verified when you didn't. Tag `verified: green` vs `written, not yet run`.
- Can't run the loop here (no net/deps/DB/git)? Deliver code + tests + exact run commands +
  expected results, tag `written, not yet run` — but report it **blocked, not done** (green CI is
  mandatory); never withhold the code, never call it done.
- Heavy loop: sync to base → change → run the FULL checks (lint/types/tests/build/live) using the
  repo's own commands → fix → repeat, ≤3 attempts per failing check, with an iteration log.
- **Loop engineering (self-prompt, don't rot):** evaluate against the DoD in a *separate* pass;
  reflect on root cause before retrying; rubric-critique when there's no test; fresh-eyes the diff
  before the PR; re-ground when stuck and keep a compact state snapshot as the transcript grows.
  Full patterns + citations in `references/loop-engineering.md`.
- A blocker claim needs the exact `stderr` and a command run first — not an assertion. Final
  report once, at the end: changed files / per-check status / PR state / blockers.
- **Report signal, not methodology:** what's verified / red / left; note self-verify once;
  nothing new → say nothing.
- Long run with events to watch (tests/build/CI/logs) → `Monitor` (streams events; the filter must
  catch failures, not just success). Nothing to stream, just wait → `ScheduleWakeup` (one-shot
  self-wakeup with a prompt). Repeat a prompt to yourself this session → `/loop <prompt>` (add an
  interval like `/loop 15m …` for fixed cadence; omit it to self-pace via `ScheduleWakeup`, end
  with `{stop:true}`). Durable, run-while-closed in a fresh session → routines (`/schedule`).
  Always put the exact check/IDs/done-condition in the prompt — the woken/next session knows only
  that. Never tight-poll. Heavy chunk → subagents; genuinely large → `Workflow`.
- **Long-running (overnight / 10h+) is normal for heavy tasks — pace for it.** Hitting Claude's
  usage limit mid-run is an expected *pause*, not a blocker and not a reason to fake done. Check
  limits from the terminal with `npx ccusage@latest blocks --active --json` (the authoritative
  `/usage` / `/status` is interactive-only, not scriptable); on the approach, `ScheduleWakeup` to
  just after the reset (`usageLimitResetTime` / `endTime`, + a few min slack) and sleep — chain 1h
  wakeups (clamp is ≤1h) or a Routine if the reset is >1h out. Then re-check and resume the DoD.
- **Before "done" (heavy):** every explicit ask delivered & working, item-by-item (no "remaining"
  leftovers) + **scope & quality actually match the request** (right breadth, not fewer
  cases/endpoints than asked; real depth, not a happy-path sketch that merely runs — "it's green"
  ≠ "it meets the ask") + a security pass over what you touched. Carry it to completion — delegate
  to subagents / a `Workflow` to *finish*, never hand back the rest.
- **Finish (heavy): open a clean PR, do NOT merge it** — leave the merge to a human. Prove it by
  command before "done": mergeable (no conflicts) + every REQUIRED CI gate green + state re-read
  live. Never wave away a red gate — check if it's required (required+red = not done;
  not-required+red = an open item, not a checkmark). Report "PR #N open, green, ready to merge",
  never "merged". Deferred if no git remote.
