---
description: 볼트와 Notion 미러를 맞춘다 (동기화 후 자동 검증)
---

`brain-syncer` 에이전트로 Notion 미러를 동기화한다.

- 실행 전 `brain://runbook/RB_Notion_미러_동기화` 리소스로 절차 전문을 읽는다.
- diff는 볼트의 `_mcp/notion_sync.py`로 계산한다. 목록을 손으로 옮겨 적지 않는다.
- 생성 직전 `WHERE Name IN (...)`로 실시간 재확인한다.
- 끝나면 `COUNT = DISTINCT = 볼트 용어 수`를 확인하고, 어긋나면 `brain-verifier`로 넘긴다.

$ARGUMENTS 가 주어지면 그 범위(예: 런북만, 용어만)로 한정한다.
