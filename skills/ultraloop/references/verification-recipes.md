# Verification recipes

How to turn each DoD item into an *actual run* with observable evidence, per stack and per CI.
The rule everywhere: run it, read the output, keep the evidence. Never mark an item done off a
belief that the code is correct.

First figure out the project's real commands — don't assume. Check `package.json` scripts,
`Makefile`, `justfile`, `pyproject.toml`, `Cargo.toml`, `go.mod`, the CI config, and the
README before inventing a command. Match the runner the repo already uses.

## Table of contents
- [Live checks by stack](#live-checks-by-stack)
- [Web UI via Playwright](#web-ui-via-playwright)
- [APIs / backend services](#apis--backend-services)
- [CLIs and binaries](#clis-and-binaries)
- [Libraries / packages](#libraries--packages)
- [Confirming CI passes](#confirming-ci-passes)
- [Evidence checklist](#evidence-checklist)

---

## Live checks by stack

"Live check" means exercising the running result the way a user or caller would — not reading
the diff. Pick the recipe that matches what you built.

### Web UI via Playwright

Use the Playwright MCP tools (`mcp__playwright__*`) to drive a real browser against the running
app. Load their schemas via `ToolSearch` first (query `select:mcp__playwright__browser_navigate,mcp__playwright__browser_snapshot,...`).

1. Start the dev/preview server (background Bash), wait until it's actually listening.
2. `browser_navigate` to the page under test.
3. `browser_snapshot` to read the accessibility tree — assert the expected elements/text exist.
4. Drive the flow: `browser_click`, `browser_type`, `browser_fill_form`, `browser_select_option`.
5. `browser_wait_for` on the expected post-action text/state, then snapshot again to confirm.
6. `browser_console_messages` and `browser_network_requests` to catch errors/failed calls the
   UI hides.
7. `browser_take_screenshot` as visual evidence of the working state.

Evidence = the post-action snapshot showing the expected state + a clean console. A page that
renders is not proof the flow works; drive the flow.

### APIs / backend services

1. Boot the service (background Bash), wait for the port / health endpoint.
2. Hit real endpoints with `curl` (or the repo's HTTP client): happy path **and** the edge
   cases from the DoD (bad input → correct 4xx, auth required → 401/403, not found → 404).
3. Assert on status code *and* body shape, not just "no crash".
4. If it touches a DB, verify the persisted state (query it back), don't trust the response alone.
5. Check the server log for swallowed errors / stack traces during the run.

Evidence = the request/response pairs (status + body) for happy path and each edge case.

### CLIs and binaries

1. Build/install the binary, then run it end-to-end on a real input.
2. Assert exit code (`echo $?`), stdout/stderr content, and any files it produced.
3. Cover the failure modes: missing args, bad flags, nonexistent input → correct nonzero exit
   and a useful message.

Evidence = the command line + its output + exit code for the main path and the error paths.

### Libraries / packages

1. Write and run tests against the **public API surface** (what consumers import), not internals.
2. Do a consumption smoke test: build the package, import it from a tiny throwaway script the
   way a downstream user would, and run it — this catches broken exports/build config that unit
   tests miss.
3. If it ships types, type-check the consumption script too.

Evidence = green tests + the smoke-script run output.

---

## Confirming CI passes

Local green ≠ CI green (different env, stricter flags, more checks). The DoD isn't met until CI
is confirmed green on the actual PR.

### GitHub Actions

```
gh pr checks <pr>           # watch the check runs
gh pr checks <pr> --watch   # block until they settle
gh run list --branch <b>    # find the run
gh run view <run-id> --log-failed   # read only the failed step's log
```

Flow: open the PR → wait for checks (poll the *external* CI, or sleep via `ScheduleWakeup` for
long runs) → if red, `gh run view --log-failed`, fix the root cause, push, re-check → repeat.
Only after all required checks are green: `gh pr merge --auto` (per the user's rules).

### GitLab CI

```
glab ci status              # pipeline status for the current branch/MR
glab ci view                # pipeline stages/jobs
glab ci trace <job>         # stream a job's log
```

Same flow: create the MR → watch the pipeline → read the failed job's trace → fix root cause →
push → re-check. Merge only after the pipeline is green.

### Other CI

Find how the repo reports status (a status badge, a webhook, a provider CLI). If there's no CLI,
check the provider's API or the PR/MR status checks. The principle is unchanged: get an
observed green from the real pipeline, don't declare it from a local run.

---

## Evidence checklist

For each DoD item, before marking it done, you should be able to point at one of:

- command output showing a pass (test/build/lint/typecheck exit 0 + summary);
- a Playwright post-action snapshot/screenshot + clean console;
- request/response pairs (status + body) for API paths incl. edge cases;
- a CLI run with exit code and output for main + error paths;
- a green run of the confirmed CI pipeline on the actual PR.

No evidence → the item is not done, regardless of how right the code looks.
