# github-workflow

GitHub collaboration workflow skills for AI-assisted development.

This repository provides two variants of the same `git-flow` workflow:

- `codex/git-flow/`: Codex skill, invoked as `$git-flow`.
- `claude/git-flow/`: Claude Code custom skill, invoked as `/git-flow` or used automatically when relevant.

The skill helps an AI agent handle the collaboration parts of software work:

- start work from the right branch
- identify parallel-work and merge risks
- keep task branches separate from protected branches
- commit and push scoped changes
- write understandable PRs
- show PR and deployment URLs
- check merge readiness
- avoid automatic production/main deployment

The skill does not replace project-specific rules. Each agent must read the target repository's own guidance first, such as `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `README.md`, PR templates, and collaboration docs.

## Suggested Use

Start work:

```text
$git-flow
작업 시작할게. 운영 목록 필터 수정할거야.
```

Create a PR:

```text
$git-flow
다 했어. 커밋하고 PR 올려줘.
```

Check merge/release readiness:

```text
$git-flow
PR 확인하고 반영해줘.
```

For Claude Code, use `/git-flow` instead of `$git-flow`.

## Install

Codex:

```bash
mkdir -p ~/.codex/skills
cp -R codex/git-flow ~/.codex/skills/git-flow
```

Claude Code:

```bash
mkdir -p ~/.claude/skills
cp -R claude/git-flow ~/.claude/skills/git-flow
```
