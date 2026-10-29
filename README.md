# Factory skills

Two Claude Code skills that turn a session into the orchestrator of a software
factory, plus the thirteen agents they spawn.

- `small-factory` - interview, spec, build, four parallel reviews, approver, PR.
  For features with no new screen or flow.
- `big-factory` - the same loop with a Figma drawing and design review stage
  before the build. For new screens and flows.

## Install

Copy both folders into the `.claude/` folder of the project you want to run the
factory in (or into `~/.claude/` to have them in every project):

```
cp -R .claude/skills/small-factory .claude/skills/big-factory <project>/.claude/skills/
cp .claude/agents/*.md <project>/.claude/agents/
```

Then in Claude Code: `/small-factory <feature>` or `/big-factory <feature>`.

## Before you run it elsewhere

The skills and agents were written for the Buyerbase repo and read its
conventions: a `CLAUDE.md` with the validation commands, `docs/INDEX.md`, the
`main` branch, PRs via `gh`, and a set of design-guide skills the reviewers
load (`buyerbase-design-system`, `master-page-design-guide`,
`single-page-design-guide`, `settings-page-design-guide`). Replace those names
with your own project's rules and skills, or remove the lines, before the first
run. The big factory also needs the Figma MCP server connected.
