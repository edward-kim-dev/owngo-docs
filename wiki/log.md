# Wiki Activity Log

OwnGoDocs 위키의 활동 기록입니다. **Append-only** — 이미 기록된 항목은 수정·삭제하지 않습니다.

형식: `- YYYY-MM-DD HH:MM — [행위] 설명 ([영향 페이지 링크])`

행위 분류: `init`, `create`, `update`, `delete`, `rename`, `move`, `review`, `lint`.

---

- 2026-04-18 10:52 — [init] OwnGoDocs 저장소 초기화. CLAUDE.md 및 wiki/ 6-Layer 구조 생성 ([SCHEMA.md](SCHEMA.md), [index.md](index.md), [review-queue.md](review-queue.md))
- 2026-04-18 13:40 — [update] Consensus 워크플로우 도입. SCHEMA.md §5-1·§6 개편(1인 리뷰/3인 합의 두 트랙), §6-3 의미적 충돌 감지·§6-4 Consensus 템플릿 추가. [review-queue.md](review-queue.md) 재구성, [CLAUDE.md](../CLAUDE.md) 워크플로우·작업 규칙 동기화. 린트 체크리스트 2항목 추가
- 2026-04-18 14:05 — [update] [CLAUDE.md](../CLAUDE.md)에 §3-4 보조 참조 소스(git log·Obsidian) 추가, Step 0에 최근 이력 점검 지침 추가. 저장소 루트에 [README.md](../README.md) 신규 작성 (팀원 온보딩·작업 가이드)
- 2026-04-18 14:20 — [create] 저장소 루트에 .gitignore 신규 생성. Obsidian(`.obsidian/`, `.trash/`) · 크레덴셜(`.env`, `*.pem` 등) · OS 메타데이터(.DS_Store, Thumbs.db) · 편집기 임시파일 · Office 잠금파일(`~$*`, `.~lock.*#`) 제외. 원본 문서(PDF/DOCX/XLSX)는 추적 유지
