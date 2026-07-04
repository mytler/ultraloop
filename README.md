# ultraloop

**A task isn't done when the code is written — it's done when the result is verified and actually works.**

`ultraloop` is an [agent skill](https://agentskills.io) that overrides an agent's default "code = done" reflex. It right-sizes its process to the task, recons your repo before asking anything, and never stops at "looks correct" — it either verifies the result or ships the code with an honest, one-step verification plan.

[![Add with skills.sh](https://skills.sh/b/mytler/ultraloop)](https://skills.sh/mytler/ultraloop)
![Agent Skill](https://img.shields.io/badge/type-agent%20skill-6E56CF)
![Claude Code](https://img.shields.io/badge/Claude%20Code-compatible-D97757)
![Codex](https://img.shields.io/badge/Codex-compatible-111111)
![Cursor](https://img.shields.io/badge/Cursor-compatible-1E90FF)
![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)

---

## Install

### The easy way — `npx skills` (Claude Code, Codex, Cursor, OpenCode & 60+ agents)

```bash
# install into the current project
npx skills add mytler/ultraloop

# or globally, for a specific agent
npx skills add mytler/ultraloop -g -a claude-code
npx skills add mytler/ultraloop -g -a codex
```

This uses the open [skills.sh](https://skills.sh) ecosystem, which wires the skill into whichever
supported agent you pick. See all flags with `npx skills add --help`.

### Manual — Claude Code

Copy the skill folder into your skills directory:

```bash
# personal (all projects)
git clone https://github.com/mytler/ultraloop
cp -r ultraloop/skills/ultraloop ~/.claude/skills/

# or per-project
cp -r ultraloop/skills/ultraloop .claude/skills/
```

Then trigger it with `/ultraloop <task>` or just mention **"ultraloop"** in your request.

### Manual — Codex / other agents

The skill is a plain Markdown file (`skills/ultraloop/SKILL.md`) with YAML frontmatter, following
the open Agent Skills spec — so any agent can use it. For **Codex**, add it to your `AGENTS.md`
(or reference the file) so it's loaded as project instructions. For any other agent, point the
agent at `SKILL.md` or paste its contents into your system/context.

---

## What it does

`ultraloop` runs a short triage first, then adapts:

- **Phase 0 — triage.** Light task or heavy? Process intensity is proportional to the cost of an error and the size of the task. A one-file function and a 50-agent workflow do **not** get the same ceremony.
- **Light tasks** (a snippet, no repo/CI): write the solution immediately on sensible defaults, state the assumptions, and still verify what you can. No interrogation, no ceremony.
- **Heavy tasks** (a feature in a real project):
  1. **Recon the repo first** — answer your own questions from the code (framework, package manager, DB, test runner, CI, existing patterns) instead of asking what's already in `package.json`. Includes a cheap build-consistency check (lock ↔ `package.json`, scripts exist).
  2. **Ask only the genuinely unresolvable** product/policy questions; everything else becomes a stated assumption. Use constraint-aware defaults instead of blocking.
  3. **Definition of Done** grounded in the real stack, then autonomous execution: change → tests → build → lint → live check → fix, until every item is green.
  4. **Deliver, always.** If the environment can't close the verify loop (no network/DB/git), still ship the full code + tests plus an exact run plan and expected results — tagged honestly as `written, not yet run`, never faked as green.
  5. **Finish** with a clean PR that passes CI/CD (deferred if it's not a git repo).

The one rule it never breaks: **separate "verified" from "produced."** It never claims something was verified when it wasn't — and it never withholds working code just because it couldn't run the loop here.

## Skill contents

```
skills/ultraloop/
├── SKILL.md                        # the skill (triggers, two-mode workflow, rules)
└── references/
    └── verification-recipes.md     # how to verify per stack (web/API/CLI/lib) and per CI
```

## License

MIT — see [LICENSE](LICENSE).
