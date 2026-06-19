# github-workflow

AI가 GitHub 협업 흐름을 안전하게 따르도록 돕는 `github-workflow` 스킬 모음입니다.

이 저장소에는 같은 목적의 스킬이 두 가지 버전으로 들어 있습니다.

- `codex/github-workflow/`: Codex용 스킬. `$github-workflow`로 호출합니다.
- `claude/github-workflow/`: Claude Code용 스킬. `/github-workflow`로 호출합니다.

## 스킬 이름

스킬 이름은 저장소 이름과 동일하게 `github-workflow`로 통일합니다.

- Codex: `$github-workflow`
- Claude Code: `/github-workflow`

예전 초안에서 사용했던 `git-flow` 이름은 사용하지 않습니다. Git 명령 자체보다 GitHub 기반 협업 흐름을 다루는 스킬이기 때문입니다.

## 무엇을 돕는가

이 스킬은 코드 품질 리뷰용이 아닙니다. AI가 작업할 때 GitHub 협업 과정에서 자주 빠지는 단계를 챙기도록 돕습니다.

- 올바른 기준 브랜치에서 작업 시작
- 작업 브랜치 분리
- 병렬 작업과 머지 리스크 확인
- 작업 범위에 맞는 파일만 커밋
- 원격 브랜치로 push
- 타인이 이해할 수 있는 PR 작성
- PR URL, 생성 URL, 배포/preview URL 안내
- 머지 가능 여부 확인
- `main`/운영 배포 자동 진행 방지

## 사용 전 준비

스킬 파일만 있다고 해서 PR 생성이나 머지가 자동으로 되는 것은 아닙니다. AI가 실제로 GitHub 작업을 수행하려면 실행 환경에 GitHub 권한이 있어야 합니다.

필요한 준비:

- 대상 repo가 로컬에 clone되어 있어야 합니다.
- `git remote -v`에서 GitHub 원격 저장소가 보여야 합니다.
- push 권한이 있는 계정으로 Git 인증이 되어 있어야 합니다.
- PR 생성/조회/머지까지 AI가 하려면 `gh` CLI 로그인 또는 Codex/Claude의 GitHub 연동 권한이 필요합니다.
- 권한이 없으면 스킬은 자동 작업 대신 PR 생성 URL, PR 제목, PR 본문, 사용자가 직접 확인할 항목을 출력합니다.

확인 명령 예시:

```bash
git remote -v
git status --short --branch
gh auth status
```

`gh auth status`가 실패해도 스킬을 사용할 수는 있습니다. 다만 이 경우 AI가 PR을 직접 만들거나 머지하지 못할 수 있습니다.

## 사용 예시

작업 시작:

```text
$github-workflow
작업 시작할게. 운영 목록 필터 수정할거야.
```

PR 생성:

```text
$github-workflow
다 했어. 커밋하고 PR 올려줘.
```

반영 확인:

```text
$github-workflow
PR 확인하고 반영해줘.
```

Claude Code에서는 `$github-workflow` 대신 `/github-workflow`를 사용합니다.

## 설치

Codex:

```bash
mkdir -p ~/.codex/skills
cp -R codex/github-workflow ~/.codex/skills/github-workflow
```

Claude Code:

```bash
mkdir -p ~/.claude/skills
cp -R claude/github-workflow ~/.claude/skills/github-workflow
```

## 중요한 원칙

이 스킬은 각 프로젝트의 작업 규칙을 대체하지 않습니다. AI는 먼저 대상 저장소의 `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `README.md`, PR 템플릿, 협업 문서를 읽고 그 규칙을 우선해야 합니다.

`main` 또는 운영 배포는 자동으로 진행하지 않습니다. 스킬은 기본적으로 작업 브랜치, PR, 머지 가능성, 배포 확인을 돕는 역할입니다.
