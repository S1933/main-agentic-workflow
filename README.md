# Shenron config package

Personal Shenron agent team and slash commands, packaged for `shenron package install`.

## Layout

The package files live at the repository root (there is no `plugin/` subdirectory):

```
.
├── shenron-package.yaml   # manifest (name, version, required skills)
├── shenron.yaml           # pivot — agents, commands, permissions
└── README.md
```

## Install (local, for testing)

```bash
shenron package install .
shenron package list
shenron package diff shenron-config
shenron package push shenron-config --allow-permissions
```

Because several agents declare `bash` permissions (`ask`, `verify`, `git`) and `orchestrator` declares broad task permissions, the first `push` prints the full grant list and requires `--allow-permissions`; the approval is stored at `~/.shenron/packages/shenron-config/permissions.json` (revision + sha256 of grants).

## Agents

| Agent | Mode | Role |
|-------|------|------|
| `ask` | primary | Read-only Q&A, exploration, PRD framing |
| `plan` | primary | Execution-ready implementation plans |
| `build` | primary | Implements approved plans, smallest safe diff |
| `debug` | primary | Root-cause diagnosis for hard bugs |
| `git` | primary | Git workflows, with destructive commands gated |
| `verify` | subagent | Runs build/lint/test and reports a hard pass count |
| `salameche` | subagent | Deep reviewer — correctness & edge cases |
| `carapuce` | subagent | Balanced reviewer — maintainability |
| `bulbizarre` | subagent | Fast reviewer — regressions |
| `orchestrator` | primary | Delivery: plan → git branch → build → verify → review → PR → git commit |

The three reviewers run on distinct models per runtime to decorrelate their errors and produce a meaningful consensus.

## Commands

`/build`, `/plan`, `/debug`, `/git`, `/to-prd`, `/sync-skills`, plus:

- `/review` — **read-only** parallel 3-reviewer diff review with consensus synthesis. Never edits code.
- `/review-fix` — review → apply fixes for blocking findings → re-verify → re-review (max 2 iterations).
- `/ship` — thin delegation to the `orchestrator` agent (single source of truth for the delivery algorithm).

## Required skills

The manifest declares **22 skills** under `skills.required`. They must be present at `~/.agents/skills/<name>/SKILL.md` or `push` fails with `ErrPackageSkills` listing the missing ones. Install is a no-op until they exist locally. Use `/sync-skills` to symlink them from `~/.agents/skills` into `~/.claude/skills`.

## Release to Git

The store rejects branches and non-HTTPS remotes; only tag/SHA installs are accepted.

```bash
git init
git add .
git commit -m "feat: shenron-config v0.2.0"
git tag v0.2.0
git remote add origin git@github.com:<you>/shenron-config.git
git push --tags

# On the target machine
shenron package install https://github.com/<you>/shenron-config.git --ref v0.2.0
shenron package push shenron-config --allow-permissions
```

## Update

```bash
# edit files
git commit -am "feat: bump to v0.3.0"
git tag v0.3.0 && git push --tags

# on the target machine
shenron package update shenron-config --ref v0.3.0
```

## Inspect generated output

Preview what each runtime adapter emits before pushing:

```bash
shenron doctor
shenron explain shenron-config --target codex
shenron explain shenron-config --target claude
shenron explain shenron-config --target opencode
```
