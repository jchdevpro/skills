# Cursor skills

Public collection of personal [Cursor Agent Skills](https://cursor.com/docs). Each folder is one skill (`SKILL.md` plus optional extras).

## Install

Copy a skill into `~/.cursor/skills/` (Cursor) or `~/.pi/agent/skills/` (pi):

```bash
git clone git@github.com:jchdevpro/skills.git
cp -R skills/design-system-theming ~/.cursor/skills/
cp -R skills/pr-gate skills/pr-iterate ~/.pi/agent/skills/
```

Or clone this repo as `~/.cursor/skills` / `~/.pi/agent/skills` if you want every skill from here and nothing else.

## Skills

| Skill | Use when |
| --- | --- |
| [design-system-theming](design-system-theming/) | Design-system themes and tokens. Prefer `--padding-*`, `--margin-*`, `--border-width-*`, and `--gap-*` over `--space-*`. |
| [pr-gate](pr-gate/) | Score an open PR 1–5, post graded review comments, APPROVE or REQUEST_CHANGES, apply `human-needed` when blast radius warrants it. |
| [pr-iterate](pr-iterate/) | Drive a PR to healthy — satisfaction 5/5 and green CI — by healing gate findings and closing out review threads. |
