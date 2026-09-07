# ai-brain-plugin

개인 지식 볼트(Obsidian Markdown)를 운영하는 Claude Code 플러그인. 스킬 1 · 에이전트 4 · 커맨드 3 · MCP 서버 연결.

**이 리포에는 지식 데이터가 없습니다.** 용어 카드·경험 로그·런북 같은 볼트 내용은 개인 기록이라 여기에 올리지 않습니다. 이 리포가 담는 것은 *절차를 실행하는 방법*뿐이고, *절차 자체*는 볼트의 런북에 있습니다.

## 무엇이 필요한가 (전제조건)

이 플러그인 단독으로는 동작하지 않습니다. 세 가지가 먼저 있어야 합니다.

| # | 필요한 것 | 확인 방법 |
|---|-----------|-----------|
| 1 | **Python 3.10+** | `py --version` |
| 2 | **AI-Brain 볼트** — `_mcp/server.py`를 포함한 Markdown 볼트 폴더 | 탐색기에서 `<볼트>\_mcp\server.py`가 보임 |
| 3 | **MCP 서버 설치** — 볼트에서 `_mcp\install.bat` 1회 실행. venv와 인덱스가 `%LOCALAPPDATA%\ai-brain-mcp`에 생깁니다(볼트 폴더 밖에 두는 이유: 클라우드 동기화 폴더에 수천 개 패키지 파일이 들어가지 않게) | `[OK] venv … index …` 출력 |

## 설치

**1) 볼트 위치를 환경변수로 알려줍니다** (PC당 1회, 관리자 권한 불필요)

```bat
setx AI_BRAIN_VAULT "G:\...\AI-Brain"
```

새 터미널을 열어야 반영됩니다. 이 값과 `%LOCALAPPDATA%` 두 가지가 PC마다 달라지는 전부이고, `.mcp.json`이 둘 다 `${변수}`로 참조하므로 **설정 파일을 PC마다 고칠 필요가 없습니다.**

**2) 플러그인을 설치합니다**

```
/plugin marketplace add <owner>/ai-brain-plugin
/plugin install ai-brain@ai-brain-plugin
```

**3) 확인**

```
/brain-promote
```

승격 사다리 3단의 후보가 출력되면 성공입니다. 도구가 안 보이면 Claude Code를 재시작하세요 — MCP 서버는 기동 시점의 코드를 물고 있습니다.

## Claude Desktop은 별도입니다

플러그인은 Claude Code 기능입니다. Claude Desktop에서 같은 MCP 서버를 쓰려면 볼트에서 `_mcp\install.bat`이 만든 `claude_desktop_config.generated.json`의 `"ai-brain"` 블록을 `%APPDATA%\Claude\claude_desktop_config.json`에 직접 병합해야 합니다. Desktop 설정은 동기화 폴더 밖이라 PC끼리 충돌하지 않습니다.

## 구성

```
.claude-plugin/plugin.json       플러그인 매니페스트
.claude-plugin/marketplace.json  이 리포 자체가 마켓플레이스
.mcp.json                        MCP 서버 연결 — 경로는 전부 ${변수}
hooks/hooks.json                 세션 시작 시 운영 규칙 주입
skills/notion-mirror-sync/       Notion 미러 증분 동기화 (얇은 스킬)
agents/                          읽기 1 · 추가 1 · 제안 2
commands/                        /brain-weekly · /brain-sync · /brain-promote
```

## 설계 원칙 — 스킬은 절차를 복사하지 않는다

스킬과 에이전트 본문에는 절차 전문이 없습니다. 실행할 때마다 MCP 리소스로 원본을 읽습니다:

```
brain://runbook/<런북 id>
```

사본은 반드시 원본과 어긋나기 때문입니다. 같은 실수를 Notion 미러에서 이미 겪었고, 그 드리프트가 유령 중복 행을 만들었습니다. 그래서 리소스 헤더의 `version`이 스킬의 `version`보다 높으면 **리소스 쪽을 따릅니다.**

워크스페이스 고유 값(Notion DB id, 격리 페이지 id, Drive 폴더 id)도 같은 이유로 이 리포에 없습니다. 배포물에 박으면 다른 워크스페이스에서 엉뚱한 페이지를 가리킵니다 — 전부 볼트의 런북에서 읽습니다.

## 권한 등급

에이전트는 **'제안'을 넘지 않습니다.** 읽기 ⊂ 추가 ⊂ 제안이고, 그 위는 어떤 에이전트에게도 열리지 않습니다.

| 에이전트 | 등급 | 할 수 있는 것 | 할 수 없는 것 |
|----------|------|---------------|---------------|
| `brain-reviewer` | 읽기 | 조회·통계·승격 후보 보고 | 지식을 바꾸는 모든 쓰기 |
| `brain-syncer` | 추가 | Notion 새 행 생성, 실행 기록 | 이동·삭제·본문 편집 |
| `brain-librarian` | 제안 | draft 카드 초안, 관계 추가, Inbox 분류 | verified 승격, 덮어쓰기 |
| `brain-verifier` | 제안 | 중복·손상 탐지, 격리 페이지로 이동 | 영구 삭제 |

어떤 등급으로도 열리지 않는 것: `verified` 승격 · 런북 신규 작성 · 기존 md 덮어쓰기·삭제 · Notion 페이지 영구 삭제 · 클라우드 휴지통 이동. 전부 사람이 합니다.

실행 기록(`brain_run_log`)은 예외적으로 '읽기' 등급에도 열립니다. 지식을 바꾸지 않고 자기가 무엇을 했는지만 남기기 때문이고, 읽기 전용 에이전트가 자기 실행을 기록하지 못하면 "실제로 돌았는가"를 사람이 확인할 방법이 사라집니다.

## 버전 규칙

스킬 버전 = 원본 런북 버전. 플러그인 자체는 semver로 릴리스합니다.

## 라이선스

MIT. 자세한 내용은 [LICENSE](LICENSE).
