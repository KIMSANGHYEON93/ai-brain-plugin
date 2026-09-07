# 기여·릴리스 규칙

이 리포는 **볼트의 파생물**이다. 원본은 Obsidian 볼트의 런북이고, 여기 있는 스킬·에이전트는 그것을 컴파일한 결과다. 그래서 일반 코드 리포와 두 가지가 다르다.

## 이 리포만의 제약 2가지

**① `main` 은 "배포 가능"이 아니라 "이미 배포 중"이다.**
`/plugin marketplace add <owner>/ai-brain-plugin` 은 그 순간의 `main` 을 읽는다. 깨진 매니페스트를 main 에 올리면 **그 시각에 설치하는 모든 사람이 깨진다.** 혼자 쓰더라도 main 에 직접 커밋하지 않는다.

**② 절차를 여기서 고치지 않는다.**
스킬·에이전트 본문에 절차를 적어 넣으면 볼트의 런북과 어긋난다. 고칠 일이 생기면 **볼트의 런북을 고치고 다시 컴파일**한다. 사본은 반드시 원본과 어긋난다 — Notion 미러에서 이미 두 번 겪었다.

## 브랜치

GitHub Flow. `main` + 짧은 수명의 작업 브랜치.

```
main
 ├── feat/agents-add-run-log
 ├── fix/marketplace-owner-field
 └── chore/bump-0.3.0
```

## 커밋 메시지

```
<type>(<scope>): <subject>

<body>

<footer>
```

**scope** 는 이 리포의 구조를 따른다: `agents` · `skills` · `commands` · `mcp` · `hooks` · `marketplace` · `docs`

**type** 은 표준(feat/fix/docs/refactor/chore/ci)을 쓰되, 아래 하나를 반드시 지킨다.

### 컴파일 출처 푸터 (이 리포 필수)

`skills/` 나 `agents/` 를 바꾸는 커밋은 **어느 런북 버전에서 나왔는지** 푸터에 남긴다. 이게 없으면 git 만 보고는 파생 관계를 확인할 수 없다.

```
feat(skills): notion-mirror-sync 실패 유형 H2 대응 추가

깨진 Name 행을 격리 페이지로 이동하는 분기를 요약 절차에 반영.
절차 전문은 여전히 brain://runbook 리소스에서 읽는다.

Compiled-From: RB_Notion_미러_동기화 v1.3
```

```
fix(marketplace): owner 필드를 실제 계정으로 교체

플레이스홀더가 남아 있어 마켓플레이스 등록이 실패했다.
```

## 최초 push

```bash
git init -b main
git config core.hooksPath .githooks          # ← 게이트 활성화. 잊으면 검사가 안 돈다
git add -A
git commit -m "chore: ai-brain 플러그인 초기 배포본

스킬 1 · 에이전트 4 · 커맨드 3 · MCP 서버 연결.
볼트 데이터와 서버 코드는 포함하지 않는다(D20).
워크스페이스 고유 식별자는 런북에서 런타임에 읽는다(D21)."

git remote add origin https://github.com/<owner>/ai-brain-plugin.git
git push -u origin main
```

## push 전 게이트

`.githooks/pre-push` 가 자동으로 막는다. 검사 항목:

| # | 검사 | 왜 |
|---|------|-----|
| 1 | `gitleaks git` — **커밋 이력** | 나중 커밋에서 지워도 이력에는 남는다. 공개 리포에서는 그게 유출이다 |
| 2 | `TODO-GITHUB-USERNAME` 잔존 | 남으면 마켓플레이스 등록이 깨진다 |
| 3 | 매니페스트 4종 JSON 유효성 | 하나만 깨져도 설치 전체 실패 |
| 4 | 규격 위치(`.mcp.json` 루트) | 위치가 틀리면 MCP 서버가 등록되지 않는다 |
| 5 | 태그 = `plugin.json` 버전 | 어긋나면 릴리스와 설치본이 다른 것을 가리킨다 |

**gitleaks 가 "0건"이라고 안전이 증명되는 것은 아니다.** 오탐을 줄인 만큼 놓치는 것이 있다(AWS 문서 예제키는 탐지 목록에서 제외된다). 게이트는 마지막 방어선이지 유일한 방어선이 아니다.

## 릴리스

플러그인 자체는 semver, 스킬 버전은 **원본 런북 버전**을 따른다(둘은 별개다).

```bash
# 1. plugin.json 의 version 을 올린다
# 2. 그 변경을 커밋한다
git commit -am "chore(marketplace): v0.3.0"
# 3. 태그는 커밋과 같은 지점에
git tag v0.3.0
git push origin main --tags     # pre-push 가 태그·버전 일치를 확인한다
```

## 되돌리기

공개 리포에 시크릿이 올라갔다면 **커밋을 지우는 것으로 끝나지 않는다.** 이미 노출된 것으로 간주하고 해당 자격증명을 폐기·재발급한다. 이력 재작성은 그다음이다.
