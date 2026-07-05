---
name: ultraloop
description: >
  Completion discipline for coding agents: a task is done only when the result is verified and
  demonstrably works — not when the code is written. Overrides the default "code written = done"
  and right-sizes its process to the task. For a small, self-contained change it delivers the
  solution immediately on sensible defaults with its assumptions stated, without unnecessary
  questions or process overhead. For a feature or multi-step effort in a real project it inspects
  the repository first to answer its own questions, asks only what the code cannot reveal, agrees
  an explicit Definition of Done and an iteration budget (max turns), then executes autonomously
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
before anything else, triage the task (Phase 0) and pick how heavy the process should be. Ask
exactly as many questions as you need to avoid going down a clearly wrong path — no more.

---

## PHASE 0 — SCALE ASSESSMENT (always first, before any questions)

Before asking anything, classify the task and choose a mode.

**LIGHT task** — signals (any one):
- a self-contained snippet / single function / small module;
- no tie to an existing repo, tests, or CI (or none exist);
- no UI to verify live, low cost of error, trivially reversible.

**HEAVY task** — signals (any one):
- a feature/change inside a real project with a repo, tests, CI/CD;
- a workflow, multiple phases, many agents, long-running jobs;
- needs a live check via Playwright / Computer Use;
- high cost of error, external dependencies, a PR flow.

If it's genuinely unclear, ask ONE short question to settle the scale — don't unfold the full
interview before you've classified. Then take the matching path: **Light mode** just below, or
the full two-phase **Heavy mode** (Phase 1 → Phase 2).

---

## LIGHT MODE (fast path)

The base model is already good at small, self-contained tasks — left alone it writes a proper
RFC-4180 CSV state machine, not a naive `split(',')`. Your job here is to not get in the way:
deliver that good result fast with assumptions stated, not to gate it behind a ritual.

- Do NOT run a big clarification phase and do NOT ask for max turns.
- Ask at most 1–3 questions, and only about genuinely expensive forks — the ones where a wrong
  default means throwing the work away (for CSV: strict RFC 4180 or simple split? external lib
  allowed? where should the code go?). Everything else: take a sensible default, don't ask.
- Write the solution immediately on those defaults, then briefly list the assumptions you made
  and invite corrections. Phrase forks as "taking X unless you say otherwise", not as blocking
  questions the user must return to answer.
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

## HEAVY MODE · PHASE 1 — RECON, THEN CLARIFY (interactive)

Everything from here down (Phases 1–2, the full DoD, the execution loop, ScheduleWakeup,
Workflow, the PR finish) is the HEAVY path — used when Phase 0 classified the task as heavy.
For a light task, use Light mode above instead.

The failure to avoid here: dumping ~20 questions before writing a line when half of them are
answered by the repo itself. That asks the user for what's already in `package.json` and blocks
100% of the work on their reply — which breaks the "walk away, come back to a result" promise.
So recon the code first, then ask only what the code genuinely can't tell you.

**1. Re-read the original prompt in full** — what's asked, why, and what "done" looks like from
the user's point of view.

**2. Recon the repo FIRST, silently, before asking anything.** If the task touches an existing
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
there's genuinely no project to read (then those items become real questions).

If recon turns up a **pre-existing break** — a lock ↔ `package.json` mismatch, or a run-plan
script that doesn't exist — surface it as its own flag, "pre-existing repo issues that will fail
CI", with the fix command (e.g. `pnpm install` + commit the refreshed lock). It is NOT a blocker
for delivering your code — just an honest note next to the run plan, so the user doesn't burn a
CI cycle blaming your change for a break that was already there.

**3. Ask ONLY the genuinely unresolvable questions** — product / policy decisions that aren't in
the code and can't be safely defaulted. (For "create a user": public signup or admin-only? is
there a password, and what do we hash it with? the exact response contract and status codes?
what happens on a duplicate email?) Everything you derived from the repo or a sensible default
is NOT a question — record it as a **stated assumption** instead. Ask the **max turns** budget
in this same short block ONLY if you're actually about to enter the autonomous verify loop in
this environment; if the environment can't run the loop (no install / DB / network — see step 5
and "When the verify loop can't run here"), skip the max-turns ask, since there's no loop to
bound yet.

**4. Form the Definition of Done AFTER recon** (see below), so it's grounded in the real stack
you found, not in guesses. Show it and get an "ok" on the parts that are genuine decisions.

**5. Progress rule — build-verify-flag, don't hard-block.** The default in heavy mode is: take
reasonable assumptions (from the repo + common sense) → build → verify as far as the environment
allows → then in your report explicitly list the assumptions you made and what still needs
confirming. Do NOT freeze all work waiting on answers, and do NOT withhold the artifact.

**Constraint-aware defaults.** A blocker that has a reasonable zero-cost workaround is NOT a
question — take the workaround as a *flagged default* and keep going. E.g. you need password
hashing but `bcrypt`/`argon2` aren't in the deps and there's no network to add them → use
`node:crypto` scrypt, flagged "bcrypt/argon2 is a one-function swap". Environment limits shape
the default; they don't stop the work.

Hard-block (stop and ask *before* starting) ONLY when it's a genuine product/policy fork with
no safe default AND a high cost of getting it wrong — e.g. "is this endpoint public signup or
admin-only?" when the code doesn't say. Also stop before a destructive or irreversible action: a
production / shared DB, an irreversible migration, deleting data, publishing something outward.

In every other case, move on assumptions and deliver. Aim for at most ONE genuine blocking
question; everything else becomes a flagged assumption. If the user is around, asking a policy
question is fine — but keep producing the code that isn't blocked rather than idling.

---

## HEAVY MODE · PHASE 2 — EXECUTION (autonomous)

Once everything is agreed, work fully on your own, strictly to plan, **with no questions to
the user**. They should be able to walk away and come back to a finished result.

A question is allowed only on a genuine blocker — where continuing is physically impossible
without an answer (no access to a secret, the task contradicts itself, an external service is
down). In that case: state the blocker and use `ScheduleWakeup` (see below) so you don't idle
in wait and don't burn context on empty checks.

Decide everything else yourself, guided by the agreed DoD and common sense. When a minor fork
comes up that has an obvious default, take the default and note it in the iteration log rather
than stopping to ask.

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
- If a turn has nothing new to say, say nothing. Silence beats a paragraph that restates the plan.

---

## Definition of Done (heavy mode)

This full DoD contract is for heavy tasks. A light task doesn't get this — it gets working
code + a couple of basic checks + stated assumptions (see Light mode).

Before starting, write out a list of concrete, **verifiable** completion criteria. Pull them
from two sources: the explicit requirements in the prompt, and the implicit ones obviously
implied for a task of this kind. Users often leave the "goes without saying" parts unspoken —
your job is to reconstruct them.

By default the DoD includes the following, even if not stated explicitly:

- [ ] unit/integration tests for the new logic are written and pass;
- [ ] mocks are created for external dependencies where needed (network, DB, payments, time);
- [ ] the project builds with no errors (`build` green);
- [ ] linter and type checks pass;
- [ ] functionality is verified live — Playwright (web) or Computer Use / hitting the API/CLI,
      not "by eyeballing the code";
- [ ] no regressions: existing tests keep passing;
- [ ] no secrets in code/logs/frontend; server-side validation isn't bypassable (see the
      user's global security rules);
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

One iteration:

1. **Change** — the smallest coherent step toward the next DoD item.
2. **Checks** — run the relevant ones: tests → build → lint/types → live check. Actually run
   them, read the output, don't rely on "should work".
3. **Diagnosis** — if something is red, find the **root cause**, don't patch the symptom. For
   a non-trivial bug, bring in the `debugger` subagent; tests for new logic can go to
   `test-engineer`.
4. **Fix → repeat.** Don't leave the loop until ALL DoD items are green.

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
- writing placeholders into code ("coming soon", TODO stubs) and passing them off as finished.

**Try the cheap check before you write anything off.** Before you claim "can't", "not
applicable", or "doesn't count", run the quick command that would settle it — a CI gate's
required status, a PR's `mergeable`, the base branch's `git log`, the list of open PRs. Writing
something off without that check is itself a fake green: you're asserting a pass you never
established. If you do write it off, attach the command and its output.

If a DoD item genuinely can't be met within max turns, say so: what's not done and why. An
honest "this is still red" always beats a masked void.

---

## When the verify loop can't run here (deliver anyway)

Sometimes the environment simply can't close the loop: no network to install deps, no
`node_modules`, no database, or the folder isn't a git repo with a remote. That does not
downgrade you to "recon + questions". You still produce the full result — you just can't stamp
it `verified: green` yet. Do all of this:

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
- **Defer, don't block, on infra steps:** if it isn't a git repo with a remote, mark the "PR +
  green CI" step as *deferred until this is a repo with a remote* and move on — don't withhold
  code over it.
- **Offer to close the loop** when an environment with installs/DB appears (or via
  `ScheduleWakeup` if something is being provisioned) — and ask for **max turns** only then, at
  the point you actually enter the autonomous verify cycle.

The autonomous result the user comes back to is: working code + tests + a precise plan to verify
it — not a list of questions.

---

## Self-wakeup (ScheduleWakeup)

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

- **delaySeconds to match scale.** A short build — minutes; a heavy workflow / CI — tens of
  minutes. Mind the 5-minute cache TTL: sleeping <270s keeps the cache warm (for polling
  external CI), 1200–1800s is the default for a plain "not ready yet". Don't pick exactly 300s.
- **Put all the context the woken-you needs into `prompt`**: what to check, which task/workflow/PR
  IDs, what to do if done and if still running. The woken-you won't remember the details — they
  must be in the prompt.
- **Not ready yet → `ScheduleWakeup` again,** not a polling loop.

---

## Heavy tasks → Workflow

If the task is large (a project-wide migration, a bug hunt, "6 phases, 50 agents", a broad
audit) — don't try to dig through it all solo in one session. Split it into phases and fan the
work out to agents via `Workflow` (deterministic orchestration with fan-out).

Key point: **the ultraloop principle extends to every phase.** A phase isn't done until its
result has passed checks; the overall result isn't done until a clean PR is assembled that
passes the full CI/CD. Build verification straight into the pipeline — e.g. a "make the change"
stage → a "run tests/lint and return a verdict" stage, where findings/fixes are accepted only
with a confirmed green verdict. Give parallel agents that edit files `isolation: 'worktree'`
so they don't conflict.

While phases/agents run — wait via `ScheduleWakeup`, not by polling. `Workflow` is tracked by
the harness, so you'll be notified on completion; keep the wakeup as a fallback.

---

## Finish (heavy mode): a "clean PR" is proven by command, not by memory

"Clean PR" has a hard definition, and you confirm **every part of it with an actual command
BEFORE you call the DoD green** — not after:

1. **Mergeable into base, no conflicts — check it, don't assume.** `gh pr view <pr> --json
   mergeable,mergeStateStatus` (or a trial merge). `CONFLICTING` / dirty → not clean.
2. **Every REQUIRED CI gate is green.** Read the run results (`gh pr checks <pr>`) AND confirm
   which checks are actually required (`gh api repos/{owner}/{repo}/branches/{base}/protection/required_status_checks`,
   or branch-protection settings). Any required check not green → the PR is not clean and the DoD
   is not green.
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

**Never report "merged / in `main` / done" from an assumption** about how a merge or squash
resolved. Before you say it landed:
- verify what actually reached the base branch — `git fetch && git log origin/<base>` for your
  commits / the PR;
- confirm no conflicting open PRs are racing yours — `gh pr list --state open --base <base>`.
Run both BEFORE the word "done", not after.

Then, and only then: a clean PR with meaningful atomic commits, a description listing the DoD
items and their check results, and `gh pr merge --auto` after green CI/CD (don't push to `main`
directly). Red required CI = task not closed → back into the loop.

**If it isn't a git repo with a remote, or CI can't run in this environment**, don't block:
deliver the code + tests + the one-step verification plan, tag it `written, not yet run`, and
mark the PR + green-CI step as *deferred until this is a repo with a remote*. Close out at the
highest level the environment supports — and state plainly which level that is.

---

## Quick reference

- **Triage first (Phase 0):** light or heavy? Process intensity ∝ cost of error × scale.
- **Light task:** sensible defaults + code now, ≤1–3 questions on expensive forks only, state
  assumptions, no max-turns / DoD / CI ceremony — but still verify what you can, never fake it.
- **Heavy task:** recon the repo first (answer your own questions from the code) → ask at most
  ONE genuine policy blocker (everything else = flagged assumption; constraint-aware defaults, no
  blocking on defaultable forks) → DoD grounded in the real stack → build-verify-flag → autonomous
  execution. Ask max turns only when you actually enter the verify loop.
- Done = observable evidence that it works, not "code written".
- **Separate verified from produced:** always ship the artifact (code + tests). Never fake green
  = never *claim* verified when you didn't. Tag `verified: green` vs `written, not yet run`.
- Can't run the loop here (no net/deps/DB/git)? Deliver code + tests + exact run commands +
  expected results, tag `written, not yet run`, defer PR/CI — never withhold the code.
- Heavy loop: change → tests/build/lint/live check → fix → repeat, with an iteration log.
- **Report signal, not methodology:** what's verified / red / left; note self-verify once;
  nothing new → say nothing.
- Long run → `ScheduleWakeup`, not polling. Large task → `Workflow` split into phases.
- **Finish (heavy) is proven by command before "done":** mergeable (no conflicts) + every
  REQUIRED CI gate green + state re-read live. Never wave away a red gate — check if it's
  required (required+red = not done; not-required+red = an open item, not a checkmark). Verify
  what actually landed + no conflicting open PRs before saying "merged". Deferred if no git remote.
