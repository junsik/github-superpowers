# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

GitHub Superpowers is a Claude Code plugin (`.claude-plugin/`) that provides a complete development workflow integrated with GitHub Issues, Milestones, and Projects. All content is in Korean. The plugin is installed into target repositories and activated via a SessionStart hook.

## Repository Structure

- **skills/** — 24 skill modules, each with a `SKILL.md` (YAML frontmatter: `name`, `description`)
- **templates/** — GitHub issue/PR body templates (`design-issue.md`, `epic-issue.md`, `pull-request.md`)
- **hooks/** — SessionStart hook that injects `using-github-superpowers` skill content into every session
- **commands/** — CLI command help files (invoked as `/brainstorm`, `/milestone`, etc.)
- **.claude-plugin/** — Plugin manifest (`plugin.json`, `marketplace.json`)
- **docs/** — Blog posts per release

## Core Workflow (Skill Chain)

```
brainstorming → design.md → Design Issue
    → writing-plans (Plan Mode) → impl.md
    → creating-issues → Epic Issue
    → executing-plans (TDD per task) → verification → creating-prs → closing-issues
```

Each skill's `SKILL.md` contains bash snippets that are **reference scripts**, not executable files. Claude follows these as templates when running `gh` commands.

## Key Architectural Decisions

**SessionStart Hook** (`hooks/session-start.sh`): Reads `skills/using-github-superpowers/SKILL.md`, JSON-escapes it, and injects as `additionalContext` in the hook response. This is how the workflow rules are enforced in every session.

**Config file** (`.github/github-superpowers.json` in target repos): Stores GitHub Project ID, field IDs (Start Date, End Date, Priority, Issue Type), option IDs, current milestone. Created by `init-github-superpowers` skill. All `gh project item-edit` commands in skills reference this file.

**Plan files** (`.claude/github-superpowers/plans/` in target repos): Where `design.md` and `impl.md` are saved. Git-ignored via target repo's `.gitignore`.

**Agent Teams**: Two patterns exist — pipeline (`subagent-driven-development` with implementer → spec-reviewer → quality-reviewer) and parallel (`dispatching-parallel-agents`). Agent team skills include separate prompt files per agent role.

## Skill Conventions

- Directory: hyphenated lowercase (e.g., `test-driven-development`)
- Entry point: always `SKILL.md` with YAML frontmatter
- Supporting docs: `references/` subdirectory or sibling `.md` files
- Agent prompts: `*-prompt.md` files (e.g., `implementer-prompt.md`)
- Skill types: **Rigid** (TDD, debugging — follow exactly) vs **Flexible** (patterns — adapt to context)

## Editing Skills

When modifying skill bash snippets, maintain consistency across all issue-creation points:
- **brainstorming/SKILL.md** — Design Issue creation (label: `design`, Issue Type: `design`)
- **creating-issues/SKILL.md** — Epic creation (label: `epic,feat`, Issue Type: `feat`)
- **init-github-superpowers/SKILL.md** — Config schema definition (source of truth for field structure)

All three must stay in sync for Project field names, config JSON paths, and `gh project item-edit` patterns.

## Validation

This is a documentation-only plugin (no build, no tests, no linting). Validate changes by:
1. Reading modified `SKILL.md` files to verify bash snippet consistency
2. Checking `github-superpowers.json` schema references match across skills
3. Ensuring YAML frontmatter `name` matches directory name
