---
name: git-flow
description: Git/GitHub collaboration workflow for AI-assisted development. Use when the user asks to start work, pull/update, create or switch branches, commit, push, open or update a PR, check merge readiness, merge, verify deployment, or says Korean phrases like "작업 시작", "다 했어", "올려줘", "PR 올려줘", "반영해줘", or "머지해줘". Helps identify parallel-work risk, prepare understandable PRs, and avoid unsafe main/production changes.
---

# Git Flow

## Overview

Use this skill to guide Git/GitHub collaboration across repositories. The goal is not code-quality review; it is to keep branch, commit, PR, merge, and deployment handoffs clear enough that non-expert users can collaborate safely with AI agents.

Prefer the repository's own rules over this generic workflow. Before acting, read relevant local guidance such as `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `README.md`, `.github/pull_request_template.md`, and docs under `docs/` when they exist.

## Phase Detection

Infer the phase from the user's request.

- Start phase: "작업 시작", "작업할게", "만들거야", "수정할거야", "삭제할거야", "start work", "create branch", "pull latest".
- PR phase: "다 했어", "올려줘", "커밋해줘", "push", "PR 올려줘", "open a PR".
- Release phase: "반영해줘", "머지해줘", "merge", "배포 확인", "check deployment".

At the start of the response, state the inferred phase in Korean. If the request spans multiple phases, run them in order only when safe.

## Universal Rules

- Do not work directly on protected integration or production branches such as `main`, `master`, `dev`, `develop`, or release branches unless the repo explicitly allows it.
- Do not overwrite unrelated local changes. Stage only files that belong to the current task.
- Do not automatically deploy or merge to production/main. Treat main/production release as human-approved unless the repo explicitly authorizes it.
- If GitHub authentication, `gh`, or API access is unavailable, explain the limitation and provide the exact URL or command the user can run manually.
- After creating or updating a PR, always show the PR URL. If PR creation fails, show the compare/PR creation URL and a ready-to-paste title/body.
- After a merge or deployment check, always show the PR URL and any deployment, preview, or site URL that was checked or needs human checking.
- Do not turn normal parallel branch work into a blocker. Classify merge risk and recommend coordination only when needed.

## Start Phase

1. Read repository guidance.
2. Inspect Git state:

```bash
git status --short
git branch --show-current
git remote -v
git fetch --all --prune
```

3. Identify the base branch from repo guidance or common defaults: `dev`, `develop`, then `main`.
4. If currently on an integration/production branch, create or switch to a task branch before editing.
5. Pull or rebase from the selected base branch when safe and no unrelated local work would be disturbed.
6. Look for parallel-work risk using available evidence:

- local branch name and diff
- open PRs when GitHub access is available
- changed files in related PRs
- repo docs naming common/shared files
- the user's described screen, feature, data contract, permission, or deployment scope

Classify risk by merge/coordination impact, not by whether files overlap:

- Normal parallel: proceed on a separate branch.
- Coordination needed: proceed, but note likely PR order or scope split.
- Semantic conflict risk: align on data contract, status logic, auth/permission, DB schema, or shared behavior before continuing.
- Blocked: stop for human decision when production data, migrations, destructive changes, or protected release actions are involved.

Output:

```text
GitHub 협업 단계: 작업 시작

진행 판단:
- <별도 브랜치로 진행 가능 | PR 순서 조정 필요 | 범위 합의 필요 | 보류 권장>

병렬 작업 판단:
- 수준: <정상 병렬 | 조정 필요 | 의미 충돌 위험 | 차단>
- 근거:
- 겹칠 수 있는 사람/PR/브랜치:
- 겹칠 수 있는 화면/기능:
- 겹칠 수 있는 파일:
- 권장 대응:

작업 요약:
- 기준 브랜치:
- 작업 브랜치:
- 작업 범위:
- 주의할 영향: DB / 실제 데이터 / 권한 / 배포

다음 진행:
- <AI가 이어서 할 일 또는 사용자가 확인할 일>
```

## PR Phase

1. Inspect current branch, diff, and status.
2. Confirm the branch is not a protected integration/production branch.
3. Review whether changes are within the requested scope.
4. Run relevant repo checks. Prefer documented commands. Common examples:

```bash
npm run lint
npm run typecheck
npm run build
```

5. For UI changes, open or run the app when practical and verify the changed screen.
6. Commit only task-related files. If the final diff clearly contains unrelated task types, split commits by meaning; otherwise keep one clear commit.
7. Push the branch.
8. Create or update a PR to the repo's integration branch, usually `dev`, `develop`, or `main` depending on repo rules.

Use this PR body shape unless the repo template says otherwise:

```md
## Summary
- 

## Changes
- 

## How to Verify
- 

## Data / DB / Permission / Deployment Impact
- DB:
- Real data:
- Permissions:
- Deployment:

## Checks
- 

## Not Checked
- 

## Risks
- 

## Merge Conditions
- 
```

Output:

```text
GitHub 협업 단계: PR 생성

- 현재 브랜치:
- 커밋:
- 푸시 상태:
- PR 대상 브랜치:
- PR URL:
- 변경 요약:
- 실행한 검증:
- 미검증:
- 남은 리스크:
- 리뷰어가 확인할 것:
```

If PR creation cannot be completed:

````text
GitHub 협업 단계: PR 생성 준비

PR을 직접 생성하지 못한 이유:
- 

PR 생성 URL:
- 

PR 제목:
- 

PR 본문:
```md
...
```

직접 확인할 것:
- 
````

## Release Phase

Do not assume release means production/main deployment. Determine the target from the user's words and repo rules. Default to checking merge readiness for the PR's target branch.

1. Locate the PR URL or branch.
2. Confirm target branch.
3. Check merge conflicts.
4. Check CI/status checks.
5. Check unresolved review comments or open decisions when available.
6. Confirm data, DB, permission, or deployment-impact reviews are complete when relevant.
7. Merge only when the user asked for it, repo rules allow it, required checks pass, and the target is not production/main unless explicitly approved.
8. After merge, check deployment, preview, or site URL when available.

Output:

```text
GitHub 협업 단계: 반영 확인

- PR URL:
- 대상 브랜치:
- 충돌 여부:
- CI/check 상태:
- 리뷰/미해결 코멘트:
- 머지 가능 여부:
- 머지 결과:
- 배포 상태:
- 확인한 URL:
- 확인한 동작:
- 미확인/남은 리스크:
```

## Commit Guidance

Do not interrupt the user based only on elapsed time, file count, or diff size. Suggest an intermediate commit only when the user asks or when the work clearly crosses a boundary such as docs to code, UI to DB schema, refactor to behavior change, or one completed feature to a different feature.

At PR phase, inspect the final diff and choose one or more commits based on meaning.
