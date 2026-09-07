---
description: 주간 정리 — 상태 점검, Inbox 분류, 미러 검증, 승격 후보 보고를 순서대로 돌린다
---

AI-Brain 주간 정리를 실행한다. 순서와 담당은 아래를 따른다.

1. **상태 점검** — `brain-reviewer` 에이전트로 `brain_stats` + `brain_promotion_candidates`를 받아 현재 상태와 승격 후보를 정리한다.
2. **Inbox 분류** — Inbox가 1건 이상이면 `brain-librarian` 에이전트로 분류하고 카드 초안(draft)을 만든다. 원본은 삭제하지 않는다.
3. **미러 검증** — `brain-verifier` 에이전트로 Notion COUNT/DISTINCT와 볼트 용어 수가 일치하는지 확인한다. 미반영이 있으면 `brain-syncer`로 동기화한 뒤 다시 검증한다.
4. **보고** — 사람이 결정할 것만 추려서 제시한다: verified 승격 후보, 런북 승격 후보, 영구 삭제 대기, 플래그 정리 대상.
5. **기록** — `brain_log`로 주간 로그 1건, 볼트 `MEMORY.md` §4에 1행.

원본 삭제는 하지 않는다. `verified` 승격·런북 작성·영구 삭제는 사람이 결정한다.
