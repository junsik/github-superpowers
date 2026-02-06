---
name: using-git-worktrees
description: Use when starting feature work that needs isolation from current workspace or before executing implementation plans - creates isolated git worktrees with smart directory selection and safety verification
---

# Using Git Worktrees

## Overview

Git worktrees create isolated workspaces sharing the same repository, allowing work on multiple branches simultaneously without switching.

**Core principle:** Systematic directory selection + safety verification = reliable isolation.

**Announce at start:** "using-git-worktrees 스킬을 사용하여 격리된 작업 공간을 설정합니다."

## ⚠️ CRITICAL: 서브에이전트와 비호환

**서브에이전트(Agent Teams)는 worktree로 이동하지 않습니다.**

- 서브에이전트는 부모 에이전트의 워킹 디렉토리를 공유
- worktree로 cd 후 서브에이전트 호출 시, 서브에이전트는 원래 디렉토리에서 작업
- **결과**: 예상과 다른 브랜치에서 변경사항 발생

**사용 가이드:**
- ✅ **수동 구현**: worktree + executing-plans (서브에이전트 없이)
- ❌ **서브에이전트**: subagent-driven-development (worktree 없이 현재 브랜치에서)

## Directory Selection Process

Follow this priority order:

### 1. Check Existing Directories

```bash
# Check in priority order
ls -d ~/.worktrees 2>/dev/null   # Preferred (global, home)
ls -d .worktrees 2>/dev/null     # Alternative (project-local, hidden)
ls -d worktrees 2>/dev/null      # Alternative (project-local)
```

**If found:** Use that directory. Priority: `~/.worktrees` > `.worktrees` > `worktrees`

### 2. Check CLAUDE.md

```bash
grep -i "worktree.*director" CLAUDE.md 2>/dev/null
```

**If preference specified:** Use it without asking.

### 3. Ask User

If no directory exists and no CLAUDE.md preference:

```
No worktree directory found. Where should I create worktrees?

1. ~/.worktrees/<project-name>/ (global, home directory)
2. .worktrees/ (project-local, hidden)

Which would you prefer?
```

## Safety Verification

### For Project-Local Directories (.worktrees or worktrees)

**MUST verify directory is ignored before creating worktree:**

```bash
# Check if directory is ignored (respects local, global, and system gitignore)
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**If NOT ignored:**

Fix broken things immediately:
1. Add appropriate line to .gitignore
2. Commit the change
3. Proceed with worktree creation

**Why critical:** Prevents accidentally committing worktree contents to repository.

### For Global Directory (~/.worktrees)

No .gitignore verification needed - outside project entirely.

## Creation Steps

### 1. Detect Project Name

```bash
project=$(basename "$(git rev-parse --show-toplevel)")
```

### 2. Create Worktree

```bash
# Determine full path
case $LOCATION in
  ~/.worktrees)
    path="~/.worktrees/$project/$BRANCH_NAME"
    ;;
  .worktrees|worktrees)
    path="$LOCATION/$BRANCH_NAME"
    ;;
esac

# Create worktree with new branch
git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

### 3. Run Project Setup

Auto-detect and run appropriate setup:

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

### 4. Verify Clean Baseline

Run tests to ensure worktree starts clean:

```bash
# Examples - use project-appropriate command
npm test
cargo test
pytest
go test ./...
```

**If tests fail:** Report failures, ask whether to proceed or investigate.

**If tests pass:** Report ready.

### 5. Report Location

```
Worktree ready at <full-path>
Tests passing (<N> tests, 0 failures)
Ready to implement <feature-name>
```

## Quick Reference

| Situation | Action |
|-----------|--------|
| `~/.worktrees/` exists | Use it (no gitignore needed) |
| `.worktrees/` exists | Use it (verify ignored) |
| `worktrees/` exists | Use it (verify ignored) |
| Multiple exist | Priority: ~/.worktrees > .worktrees > worktrees |
| Neither exists | Check CLAUDE.md → Ask user |
| Directory not ignored | Add to .gitignore + commit |
| Tests fail during baseline | Report failures + ask |
| No package.json/Cargo.toml | Skip dependency install |

## Common Mistakes

### Skipping ignore verification

- **Problem:** Worktree contents get tracked, pollute git status
- **Fix:** Always use `git check-ignore` before creating project-local worktree

### Assuming directory location

- **Problem:** Creates inconsistency, violates project conventions
- **Fix:** Follow priority: existing > CLAUDE.md > ask

### Proceeding with failing tests

- **Problem:** Can't distinguish new bugs from pre-existing issues
- **Fix:** Report failures, get explicit permission to proceed

### Hardcoding setup commands

- **Problem:** Breaks on projects using different tools
- **Fix:** Auto-detect from project files (package.json, etc.)

## Example Workflow

```
You: using-git-worktrees 스킬을 사용하여 격리된 작업 공간을 설정합니다.

[Check .worktrees/ - exists]
[Verify ignored - git check-ignore confirms .worktrees/ is ignored]
[Create worktree: git worktree add .worktrees/auth -b feature/auth]
[Run npm install]
[Run npm test - 47 passing]

Worktree ready at /Users/jesse/myproject/.worktrees/auth
Tests passing (47 tests, 0 failures)
Ready to implement auth feature
```

## Red Flags

**Never:**
- Create worktree without verifying it's ignored (project-local)
- Skip baseline test verification
- Proceed with failing tests without asking
- Assume directory location when ambiguous
- Skip CLAUDE.md check

**Always:**
- Follow directory priority: existing > CLAUDE.md > ask
- Verify directory is ignored for project-local
- Auto-detect and run project setup
- Verify clean test baseline

## Integration

**Called by:**
- **brainstorming** - 설계 승인 후 구현 시작 전 (서브에이전트 없이 구현 시)
- **executing-plans** - Task 실행 전 (서브에이전트 없이 수동 구현 시)
- Any skill needing isolated workspace

**NOT compatible with:**
- **subagent-driven-development** - 서브에이전트는 worktree로 이동 안 됨

## 관련 스킬

- **executing-plans**: worktree에서 impl.md 실행
- **brainstorming**: 설계 후 worktree 생성
- **creating-prs**: 작업 완료 후 PR 생성
