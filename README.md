# Shenron config package

Personal Shenron agent team and slash commands, packaged for `shenron package install`.

## Layout

```
plugin/
├── shenron-package.yaml   # manifest (name, version, skills)
├── shenron.yaml           # pivot — agents, commands, permissions
└── README.md
```

## Install (local, for testing)

```bash
shenron package install ./plugin
shenron package list
shenron package diff shenron-config
shenron package push shenron-config --allow-permissions
```

The `ask` agent declares `bash: allow` and `orchestrator` declares broad task permissions. The first `push` will print the full grant list and require `--allow-permissions`; the approval is stored at `~/.shenron/packages/shenron-config/permissions.json` (revision + sha256 of grants).

## Required skills

The manifest declares 20 skills under `skills.required`. They must be present in `~/.agents/skills/<name>/SKILL.md` or `push` will fail with `ErrPackageSkills` listing the missing ones. Install is a no-op until they exist locally.

## Release to Git

The store rejects branches and non-HTTPS remotes; only tag/SHA installs are accepted.

```bash
git init
git add .
git commit -m "feat: initial shenron-config v0.1.0"
git tag v0.1.0
git remote add origin git@github.com:<you>/shenron-config.git
git push --tags

# On the target machine
shenron package install https://github.com/<you>/shenron-config.git --ref v0.1.0
shenron package push shenron-config --allow-permissions
```

## Update

```bash
# edit files
git commit -am "feat: bump to v0.2.0"
git tag v0.2.0 && git push --tags

# on the target machine
shenron package update shenron-config --ref v0.2.0
```
