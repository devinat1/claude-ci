# Claude CI Reviewer — Design Spec

## Overview

A drop-in GitHub Actions integration that reviews every PR against four code quality principle sets, posts inline comments on violations, and leaves a summary comment with 1-5 ratings and personified feedback blurbs.

## File Structure

```
.github/
  workflows/
    claude-review.yml          # GitHub Actions workflow
.claude/
  principles/
    bob.md                     # Clean Code / SOLID (Robert C. Martin)
    clean.md                   # Clean code naming, functions, structure
    oop.md                     # Elegant Objects (Yegor Bugayenko)
    ddd.md                     # Domain-Driven Design
  review-prompt.md             # Review instructions + rating template
```

## Components

### 1. Workflow (`claude-review.yml`)

- **Trigger:** `pull_request` events (`opened`, `synchronize`)
- **Action:** `anthropics/claude-code-action@v1`
- **Auth:** `ANTHROPIC_API_KEY` repo secret
- **Prompt:** Instructs Claude to read `.claude/review-prompt.md` and all files in `.claude/principles/`
- **Allowed tools:** `mcp__github_inline_comment__create_inline_comment`, `Bash(gh pr comment:*)`, `Bash(gh pr diff:*)`, `Bash(gh pr view:*)`, `Read`, `Glob`

### 2. Principle Files (`.claude/principles/`)

Four standalone markdown files, each containing a single principle set:

| File | Philosophy | Voice |
|------|-----------|-------|
| `bob.md` | Clean Code, Clean Architecture, SOLID | Uncle Bob |
| `clean.md` | Clean code naming, functions, structure, SOLID, testing | The Clean Coder |
| `oop.md` | Elegant Objects | Yegor Bugayenko |
| `ddd.md` | DDD aggregates, bounded contexts, domain modeling | The Domain Modeler |

### 3. Review Prompt (`.claude/review-prompt.md`)

Instructions for Claude on how to conduct the review:

1. Read all principle files from `.claude/principles/`
2. Get the PR diff
3. Rate the code 1-5 against each principle set
4. Post inline comments on specific violations (citing principle + rationale)
5. Post a summary comment containing:
   - Ratings table (1-5 per principle set)
   - Personified blurbs (one sentence each, e.g., "Uncle Bob would say: ...")
   - Violation examples table (unclean version vs. clean version)

## Output Format

### Inline Comments
Posted on specific diff lines via `mcp__github_inline_comment__create_inline_comment`. Each cites the principle violated and a brief explanation.

### Summary Comment
Posted via `gh pr comment`. Structure:

```
## Code Review — Principle Ratings

| Principle | Rating | Philosophy |
|-----------|--------|------------|
| Bob Martin (Clean Code/SOLID) | X/5 | ... |
| Clean Code | X/5 | ... |
| Elegant Objects (OOP) | X/5 | ... |
| Domain-Driven Design | X/5 | ... |

## What The Masters Would Say

- **Uncle Bob** would say: "..."
- **The Clean Coder** would say: "..."
- **Yegor Bugayenko** would say: "..."
- **The Domain Modeler** would say: "..."

## Violations Found

| # | Principle | Violation | Unclean Version | Clean Version |
|---|-----------|-----------|-----------------|---------------|
| 1 | ... | ... | ... | ... |
```

## Integration

To add to any repo:
1. Copy `.github/workflows/claude-review.yml`
2. Copy `.claude/principles/` directory and `.claude/review-prompt.md`
3. Add `ANTHROPIC_API_KEY` as a repo secret
4. PRs will be reviewed automatically
