---
name: notion-mirror-sync
description: 볼트(Obsidian md)의 용어·런북이 Notion의 Terms/Runbooks DB에 빠져 있을 때 메타데이터와 Drive 링크만 증분 동기화한다. 트리거 — "Notion 동기화해줘", "볼트랑 Notion 맞춰줘", "Notion 미러 갱신", "sync notion mirror", "update notion terms db". Notion 본문 편집이나 역방향(Notion→볼트) 반영은 이 스킬이 아니다.
version: 1.3
---

# Notion 미러 동기화

볼트가 원본, Notion은 **메타데이터 + Drive 링크만** 담는 검색용 미러다. 본문은 복사하지 않는다.

## 먼저 할 일 — 절차 전문 읽기

이 파일에는 절차가 없다. **실행 전에 반드시** MCP 리소스를 읽는다:

```
brain://runbook/RB_Notion_미러_동기화
```

리소스가 돌려주는 헤더의 `version`이 이 스킬의 `version`(1.3)보다 높으면, 런북이 개정된 것이므로 **리소스 쪽을 따른다**. 절차를 이 파일에 옮겨 적지 않는다 — 사본은 반드시 원본과 어긋난다(실제로 Notion 미러에서 겪은 실패 F·H2가 그 사례다).

## 요약 절차 (5줄 — 위 리소스를 못 읽을 때만)

1. `brain_list(types=["term"])`와 Notion `SELECT group_concat("Name")`로 양쪽 목록을 **매번 새로** 조회한다.
2. 볼트의 `_mcp/notion_sync.py`로 정규화(공백 제거·NFC·소문자) 비교해 미반영 목록을 **프로그램으로** 계산한다.
3. 생성 직전 `SELECT "Name" ... WHERE "Name" IN (...)`로 전부 아직 없는지 실시간 재확인한다.
4. Drive `10_Terms` 폴더에서 파일 ID를 OR 배치로 조회한다(폴더 id는 런북 §3 4단계에 있다 — 배포물에 박지 않는다).
5. `notion-create-pages`로 ≤20건씩 생성하고 COUNT/DISTINCT로 검증한다.

## 자주 겪는 실패 (상세는 리소스 §6)

| 증상 | 조치 |
|------|------|
| `Invalid number value for property Source Logs` | Runbooks의 Source Logs는 **숫자** — 로그 건수를 넣는다 |
| `Invalid multi_select value for "Tools"` | 옵션에 없는 값 — `update-data-source`로 옵션을 먼저 추가 |
| Drive 검색 0건 | 동기화 대기 후 재검색. `title contains`에는 파일명 앞부분만 |
| 같은 Name 2행 | 병행 세션과의 경합 — 나중 행을 격리 페이지로 이동 |
| 미반영 목록이 실제와 다름 | 수기 대조를 한 것이다. 2·3단계를 프로그램으로 다시 계산 |
| 한글 낱자가 미세하게 깨짐 | One-liner는 정정하지 않는다(원본은 볼트). **Name이 깨졌으면** 그 행을 격리 이동 — 안 그러면 다음 라운드에 중복이 생긴다 |

## 안전 규칙

- Notion MCP에는 페이지 삭제 API가 **없다**. 잘못 만든 행은 격리 페이지로 **이동**까지만 하고(페이지 id는 런북 §6-G), 완전 삭제는 사람이 한다.
- 볼트 md는 읽기 전용이다. 이 스킬은 볼트에 쓰지 않는다.
- Notion에서 본문을 편집하지 않는다(단방향).

## 완료 판정

- Terms `COUNT(*)` = `brain_stats`의 용어 수, 그리고 `COUNT(*)` = `COUNT(DISTINCT "Name")`
- 끝나면 실행 결과를 로그로 남긴다.
