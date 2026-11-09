# ai-brain-plugin 위키

개인 지식 볼트(Obsidian Markdown, 이하 **볼트**)를 운영하는 Claude Code 플러그인이다. 스킬 1 · 에이전트 4 · 커맨드 3 · 세션 시작 훅 1 · MCP 서버 연결 1로 구성된다(`README.md`).

이 저장소를 이해하는 데 가장 중요한 한 가지: **이 저장소는 코드도 데이터도 아닌 '볼트의 파생물'이다.** 절차의 원본은 볼트의 런북에 있고, 실행 엔진(MCP 서버)도 볼트에 있다. 여기 있는 것은 그 둘을 Claude Code에 연결하는 얇은 껍데기다. 그래서 이 저장소만 받아서는 아무것도 동작하지 않는다.

## 무엇이 어디에 있나

| 경로 | 역할 |
|---|---|
| `.claude-plugin/plugin.json` | 플러그인 매니페스트 — 이름 `ai-brain`, 버전 `0.2.0` |
| `.claude-plugin/marketplace.json` | 이 저장소 자체가 마켓플레이스(`source: "./"`) |
| `.mcp.json` | MCP 서버 `ai-brain` 연결. 경로는 전부 `${AI_BRAIN_VAULT}`·`${LOCALAPPDATA}` 변수 |
| `hooks/hooks.json` | `SessionStart` 훅 — 볼트 운영 규칙 한 줄을 세션에 주입 |
| `skills/notion-mirror-sync/` | 유일한 스킬. 볼트 → Notion 미러 증분 동기화 |
| `agents/` | 에이전트 4종 — 등급 읽기 1 · 추가 1 · 제안 2 → [[agents-and-permissions]] |
| `commands/` | `/brain-weekly` · `/brain-sync` · `/brain-promote` → [[agents-and-permissions]] |
| `.githooks/pre-push` | push 전 게이트 6종 → [[release-and-gates]] |
| `CONTRIBUTING.md` | 브랜치·커밋·릴리스 규칙 → [[release-and-gates]] |

`.gitignore` 첫 블록이 볼트 폴더(`00_Inbox/`~`90_Index/`, `MEMORY.md` 등)를 전부 막는다. 볼트 안에서 실수로 이 저장소를 초기화해도 지식 데이터가 올라가지 않게 하려는 장치다.

## 실행 경로 — 무엇이 무엇을 부르나

```
Claude Code
  └─ 플러그인 로드 (.claude-plugin/plugin.json)
       ├─ .mcp.json ──► %LOCALAPPDATA%\ai-brain-mcp\.venv\python.exe
       │                  <볼트>\_mcp\server.py          ← 서버 코드는 볼트에 있다
       │                  인덱스 DB: %LOCALAPPDATA%\ai-brain-mcp\index.db
       ├─ hooks.json ──► SessionStart 에 운영 규칙 echo
       └─ skills / agents / commands
             └─ 실행 시 MCP 리소스 brain://runbook/<id> 를 읽어 절차 전문을 얻는다
```

**설계의 중심은 마지막 줄이다.** 스킬과 에이전트 본문에는 절차 전문이 없고, 실행할 때마다 `brain://runbook/<id>`에서 원본을 읽는다. 리소스 헤더의 `version`이 스킬의 `version`보다 높으면 리소스 쪽을 따른다(`skills/notion-mirror-sync/SKILL.md`). README는 그 이유를 이렇게 적는다 — 사본은 반드시 원본과 어긋나고, Notion 미러에서 그 드리프트가 유령 중복 행을 만든 적이 있다.

같은 이유로 **워크스페이스 고유 값(Notion DB id, 격리 페이지 id, Drive 폴더 id)도 이 저장소에 없다.** 배포물에 박으면 다른 사람의 워크스페이스에서 엉뚱한 페이지를 가리킨다. 전부 런타임에 런북에서 읽는다. 이 규칙은 pre-push 게이트 ⑤로도 강제되지만 **Notion id(소문자 hex)만 잡고 Drive 폴더 id는 못 잡는다** — 실측과 이유는 [[release-and-gates]].

## 설치 전제 (README 요약)

1. Python 3.10+
2. `_mcp/server.py`를 포함한 볼트 폴더
3. 볼트에서 `_mcp\install.bat` 1회 실행 — venv와 인덱스가 **볼트 밖** `%LOCALAPPDATA%\ai-brain-mcp`에 생긴다(동기화 폴더에 패키지 수천 개가 들어가지 않게)
4. `setx AI_BRAIN_VAULT "<볼트 경로>"` 후 새 터미널
5. `/plugin marketplace add <owner>/ai-brain-plugin` → `/plugin install ai-brain@ai-brain-plugin`
6. `/brain-promote`로 확인. 도구가 안 보이면 Claude Code 재시작 — MCP 서버는 기동 시점의 코드를 물고 있다

Claude Desktop은 플러그인을 쓰지 않는다. 볼트의 `claude_desktop_config.generated.json`에서 `ai-brain` 블록을 Desktop 설정에 직접 병합해야 한다(README).

## 바꾸기 전에 알아야 할 것

- **절차를 여기서 고치지 않는다.** 스킬·에이전트 동작이 틀렸으면 볼트의 런북을 고치고 다시 컴파일한다(`CONTRIBUTING.md` 제약 ②). 여기서 고치면 원본과 어긋나는 사본이 하나 더 생길 뿐이다.
- **`main`은 이미 배포 중이다.** `/plugin marketplace add`는 그 순간의 `main`을 읽는다. 깨진 매니페스트가 `main`에 오르면 그 시각에 설치하는 사람이 전부 깨진다(제약 ①).
- **Windows 전제다.** `.mcp.json`이 `${LOCALAPPDATA}`와 역슬래시 경로, `.venv\Scripts\python.exe`를 쓴다. macOS·Linux에서는 이 파일을 고치지 않으면 서버가 뜨지 않는다.
- **세션 시작 훅은 조건 없이 뜬다.** `hooks.json`의 matcher는 `startup|resume`뿐이라, 볼트와 무관한 프로젝트에서 Claude Code를 켜도 "MEMORY.md §1·§5를 먼저 읽어라" 규칙이 주입된다.

## 이력

커밋은 둘뿐이다(둘 다 2026-09-07, `main` 직선).

- `fed3323` 초기 배포본 — 17파일. 커밋 메시지가 두 원칙을 못 박았다: 볼트 데이터와 서버 코드는 포함하지 않는다(**D20**), 워크스페이스 고유 식별자는 런북에서 런타임에 읽는다(**D21**).
- `d29bdbb` pre-push 게이트에 ⑤(워크스페이스 id 검사) 추가 — 볼트 쪽 `_plugin/` 사본을 대조하다 Notion 격리 페이지 UUID와 Drive 폴더 id가 남은 것을 발견해서다. 커밋 본문: "gitleaks는 이걸 못 잡는다 — 시크릿이 아니라 남의 워크스페이스를 가리키는 주소라서, 유출이 아니라 오작동으로 나타난다."

`D20`·`D21` 같은 번호는 볼트의 결정 로그(`MEMORY.md`, 비공개)를 가리킨다. 이 저장소에는 그 파일이 없다.

## 페이지

- [[agents-and-permissions]] — 에이전트 4종의 권한 등급, 커맨드 3종이 그것을 어떻게 엮는지, 스킬 1종
- [[release-and-gates]] — 브랜치·커밋 규칙, pre-push 게이트 6종, 릴리스, 알려진 함정

## Backlog

| 영역 | 위치 | 미룬 이유 |
|---|---|---|
| MCP 서버 도구 명세 (`brain_*`) | 볼트 `_mcp/server.py` | 이 저장소 밖이다. 에이전트가 부르는 도구 이름만 [[agents-and-permissions]]에 있다 |
| 런북 본문 (`RB_Notion_미러_동기화` 등) | 볼트 `40_Runbooks/` | 비공개·저장소 밖. 스킬이 런타임에 읽는다 |
| 에이전트 권한 매트릭스 | 볼트 `50_Skills/에이전트_권한_매트릭스.md` | `commands/brain-promote.md`가 "비공개 — 이 리포에는 없다"고 명시 |
| `LICENSE` | `LICENSE` | MIT 표준 문구라 설명할 것이 없다 |
