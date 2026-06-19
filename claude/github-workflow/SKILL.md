---
name: github-workflow
description: Claude Code에서 Git/GitHub 협업 흐름을 돕는 스킬. Use when the user asks to start work, pull/update, create or switch branches, commit, push, open or update a PR, check merge readiness, merge, verify deployment, or says Korean phrases like "작업 시작", "다 했어", "올려줘", "PR 올려줘", "반영해줘", or "머지해줘". 병렬 작업 리스크, PR 설명, merge readiness, PR/deployment URL 안내, unsafe main/production changes 방지를 돕는다.
---

# GitHub Workflow

## 개요

이 스킬은 Claude Code에서 `/github-workflow`로 호출하거나, 사용자의 요청이 Git/GitHub 협업 흐름에 해당할 때 사용한다. 목적은 코드 품질 리뷰가 아니라, 브랜치, 커밋, push, PR, 머지, 배포 확인 같은 협업 경계가 누락되지 않게 하는 것이다.

대상 저장소의 규칙을 항상 우선한다. 작업 전 `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`, `README.md`, `.github/pull_request_template.md`, `docs/` 아래 협업 문서가 있으면 먼저 읽는다.

## 단계 판단

사용자 요청을 보고 현재 단계를 판단한다.

- 작업 시작: `작업 시작`, `작업할게`, `만들거야`, `수정할거야`, `삭제할거야`, `start work`, `create branch`, `pull latest`.
- PR 생성: `다 했어`, `올려줘`, `커밋해줘`, `push`, `PR 올려줘`, `open a PR`.
- 반영 확인: `반영해줘`, `머지해줘`, `merge`, `배포 확인`, `check deployment`.

응답 첫 줄에 어떤 단계로 이해했는지 한국어로 밝힌다. 요청이 여러 단계를 포함하면 안전한 순서대로 진행한다.

## 공통 원칙

- 보호 브랜치나 통합 브랜치(`main`, `master`, `dev`, `develop`, release branch 등)에서 직접 작업하지 않는다. 단, 저장소 규칙이 명시적으로 허용하면 그 규칙을 따른다.
- 사용자나 다른 작업자의 무관한 변경을 되돌리거나 섞지 않는다. 현재 작업에 속한 파일만 stage한다.
- `main`/production 배포는 자동으로 진행하지 않는다. 명시적 승인과 저장소 규칙이 있을 때만 안내하거나 진행한다.
- GitHub 인증, `gh` CLI, GitHub 앱/API 접근 권한이 없으면 그 이유를 설명하고, 사용자가 직접 열 수 있는 URL이나 실행할 명령을 제공한다.
- PR을 만들거나 업데이트하면 항상 PR URL을 보여준다.
- PR 생성에 실패하면 compare/PR 생성 URL과 바로 붙여넣을 수 있는 PR 제목/본문을 보여준다.
- 머지나 배포 확인 후에는 항상 PR URL과 확인한 배포/preview/site URL을 보여준다.
- 브랜치를 나눠 하는 정상적인 병렬 작업을 차단 사유로 보지 않는다. 중요한 것은 파일 겹침 자체가 아니라 merge/의미 충돌 리스크다.

## 작업 시작 단계

1. 저장소의 작업 규칙을 읽는다.
2. Git 상태를 확인한다.

```bash
git status --short
git branch --show-current
git remote -v
git fetch --all --prune
```

3. 기준 브랜치를 확인한다. 저장소 문서가 없으면 일반적으로 `dev`, `develop`, `main` 순서로 추정한다.
4. 현재 브랜치가 통합/운영 브랜치라면 작업 브랜치를 만들거나 전환한다.
5. 무관한 로컬 변경을 건드리지 않는 범위에서 기준 브랜치 최신 상태를 반영한다.
6. 병렬 작업 리스크를 확인한다.

확인 근거:

- 현재 브랜치명과 local diff
- GitHub 접근이 가능할 때 open PR 목록
- 관련 PR의 변경 파일
- 저장소 문서에서 지정한 공통 파일/공통 영역
- 사용자가 말한 화면, 기능, 데이터 계약, 권한, 배포 범위

리스크는 파일 겹침 여부가 아니라 merge/조정 영향으로 분류한다.

- 정상 병렬: 별도 브랜치로 진행 가능.
- 조정 필요: 진행 가능하지만 PR 순서, 작업 범위, 담당 영역 표시 필요.
- 의미 충돌 위험: 데이터 계약, 상태값, 권한, DB schema, 공통 로직 기준을 먼저 맞춰야 함.
- 차단: 운영 데이터 삭제, migration, destructive change, production/main 반영처럼 사람 결정이 필요한 상태.

출력 형식:

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

## PR 생성 단계

1. 현재 브랜치, diff, status를 확인한다.
2. 현재 브랜치가 보호/통합/운영 브랜치가 아닌지 확인한다.
3. 변경 내용이 요청 범위 안에 있는지 확인한다.
4. 저장소 문서나 package script에 정의된 검증 명령을 우선 실행한다. 일반 예시는 아래와 같다.

```bash
npm run lint
npm run typecheck
npm run build
```

5. UI 변경이면 가능한 경우 앱이나 preview를 열어 바뀐 화면을 확인한다.
6. 현재 작업에 속한 파일만 커밋한다. 최종 diff가 명확히 다른 성격의 작업을 섞고 있으면 의미 단위로 커밋을 나눈다. 그렇지 않으면 명확한 하나의 커밋으로 충분하다.
7. 원격 브랜치로 push한다.
8. 저장소 규칙에 맞는 대상 브랜치로 PR을 생성하거나 업데이트한다.

저장소 PR 템플릿이 없으면 아래 구조를 사용한다.

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

출력 형식:

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

PR을 직접 생성하지 못하면 아래처럼 출력한다.

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

## 반영 확인 단계

`반영`이 곧 production/main 배포를 의미한다고 가정하지 않는다. 사용자 표현과 저장소 규칙을 기준으로 대상을 판단한다. 기본값은 PR 대상 브랜치에 대한 merge readiness 확인이다.

1. PR URL 또는 브랜치를 찾는다.
2. 대상 브랜치를 확인한다.
3. 충돌 여부를 확인한다.
4. CI/status check 상태를 확인한다.
5. 가능한 경우 unresolved review comment나 열린 결정을 확인한다.
6. 데이터, DB, 권한, 배포 영향이 있으면 필요한 확인이 끝났는지 확인한다.
7. 사용자가 명시적으로 요청했고 저장소 규칙이 허용하며 필수 조건이 통과했을 때만 merge한다.
8. `main`/production 대상이면 명시적 승인 없이는 merge/deploy하지 않는다.
9. merge 후 접근 가능한 배포, preview, site URL을 확인한다.

출력 형식:

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

## 커밋 기준

시간, 파일 수, diff 크기만으로 중간 커밋을 강요하지 않는다. 사용자가 요청했거나, 문서에서 코드로, UI에서 DB schema로, refactor에서 동작 변경으로 넘어가는 것처럼 작업 성격이 명확히 바뀔 때만 커밋 분리를 제안한다.

PR 생성 단계에서는 최종 diff를 보고 하나의 커밋으로 충분한지, 의미 단위로 나눌지 판단한다.
