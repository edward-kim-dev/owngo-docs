# Review Queue

위키 변경 검토 큐입니다. **두 트랙**으로 나뉩니다. 상세 규칙은 [SCHEMA.md §6](SCHEMA.md#6-review-queue-규칙) 참조.

## 트랙 구분

| 트랙 | 대상 | 승인 조건 | 자동 승인 |
|------|------|-----------|-----------|
| **Pending Review** (1인 리뷰) | `decisions/`, `SCHEMA.md`, `infra/`, `registry/` 상태 변경 | 작성자 외 1인 동의 | 24시간 경과 시 셀프 승인 |
| **Pending Consensus** (3인 합의) | 기존 위키와 **의미적 충돌**이 있는 추가·수정·삭제 | 3인 전원 approve | 없음 — 누락 시 대기 |

## 위키 변경 전 필수 점검

1. 이 파일의 **Pending Consensus** 섹션을 먼저 조회.
2. 대기 항목이 있으면, 각 팀원에게 해당 항목 검토를 요청한 뒤 결과가 반영된 후 본 작업을 재개.
3. 제안하려는 변경이 §6-3의 충돌 사례에 해당하면 Consensus에 먼저 등록.

## 항목 템플릿

### Pending Review 템플릿

```markdown
### [{YYYY-MM-DD}] 변경 제목

- **작성자**: 이름
- **대상**: 영향받는 페이지 링크
- **요약**: 변경 내용 한 줄 설명
- **상태**: pending | reviewing | approved | rejected
- **리뷰어**: 이름 (또는 "셀프 리뷰 자동 승인")
- **리뷰 메모**:
- **반영 일시**:
```

### Pending Consensus 템플릿

```markdown
### [{YYYY-MM-DD}] 충돌: {한 줄 요약}

- **작성자**: 이름
- **대상 페이지**: wiki/경로/페이지.md
- **충돌 유형**: redefine-entity | reverse-decision | break-links | change-status | other
- **기존 주장**:
  - 출처: wiki/… 또는 docs/…
  - 내용: (인용 또는 요약)
- **제안 주장**:
  - 근거: (출처 링크)
  - 내용: (인용 또는 요약)
- **영향 범위**: 링크로 영향받는 페이지 목록
- **상태**: pending-consensus | needs-discussion | approved | rejected
- **투표**:
  - 사용자 A: pending | approve | reject | needs-discussion
  - 사용자 B: pending | approve | reject | needs-discussion
  - 사용자 C: pending | approve | reject | needs-discussion
- **논의 메모**:
- **반영 일시**:
```

---

## Pending Consensus (3인 전원 합의 필요)

_(현재 합의 대기 중인 항목이 없습니다.)_

## Pending Review (1인 리뷰)

_(현재 검토 대기 중인 항목이 없습니다.)_

## Reviewing (검토 진행 중)

_(현재 검토 진행 중인 항목이 없습니다.)_

## Resolved (해결됨 — 최근 1개월 유지)

### [2026-04-18] SCHEMA.md 개편 — Consensus 워크플로우 도입

- **작성자**: Claude (LLM)
- **대상**: [SCHEMA.md](SCHEMA.md) §5-1·§6·§8, [CLAUDE.md](../CLAUDE.md), [review-queue.md](review-queue.md)
- **요약**: 기존 위키와 의미적 충돌이 있는 변경은 3인 전원 합의 후 반영. 위키 쓰기 전 Consensus 큐 선(先)확인 절차 추가. Review Queue를 1인 리뷰/3인 합의 두 트랙으로 분리
- **상태**: approved
- **리뷰어**: 사용자 지시로 초기 구성 중 즉시 반영 (셀프 승인)
- **반영 일시**: 2026-04-18 13:40
