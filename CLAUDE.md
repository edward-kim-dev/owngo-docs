# CLAUDE.md

이 파일은 Claude Code (claude.ai/code)가 OwnGoDocs 저장소에서 작업할 때 사용하는 가이드입니다.

## 1. 프로젝트 개요

- **저장소 이름**: OwnGoDocs
- **목적**: OwnGo Platform 문서 관리
- **도메인**: 오픈 소스 개발 - 운영 통합 관리 플랫폼 (DevOps Integrated Management Platform)
- **문서 유형**: 다양함 (기획서, 아키텍처 문서, 회의록, 릴리스 노트, 사용자 가이드, API 스펙, 의사결정 기록 등)
- **팀 규모**: 3인
- **라이선스**: PROPRIETARY — 내부 전용

이 저장소는 **코드를 포함하지 않는 비즈니스·기술 문서 전용 저장소**입니다. OwnGo Platform 관련 산출물을 축적하고, LLM 기반 지식 베이스(wiki)를 통해 재사용성을 높이는 것을 목표로 합니다.

## 2. 디렉토리 구조

### 2-1. 3-Layer 분리

```
OwnGoDocs/
├── CLAUDE.md              # 이 파일 — Claude Code 작업 가이드
├── docs/                  # Layer 1: 원본 문서 (불변, LLM은 읽기만)
└── wiki/                  # Layer 2: LLM이 작성·유지하는 지식 베이스
    ├── SCHEMA.md          # Layer 3: 위키 운영 규칙
    ├── index.md           # 전체 페이지 카탈로그 (자동 갱신)
    ├── log.md             # 시간순 활동 기록 (append-only)
    ├── review-queue.md    # 설계 변경 검토 큐
    ├── sources/           # docs/ 원본 1:1 요약
    ├── topics/            # 도메인 주제별 지식
    ├── registry/          # 핵심 엔티티 레지스트리
    ├── decisions/         # 의사결정 기록 (ADR 스타일)
    ├── synthesis/         # 종합 분석, 비교, 타임라인
    └── infra/             # 위키 자체 운영 규칙
```

### 2-2. docs/ 현재 상태 (2026-04-18 기준)

- `docs/` 디렉토리는 **비어 있음** (.gitkeep만 존재).
- 원본 문서 추가 시 기존 폴더 구조를 존중하며, 추가 후 wiki/sources/에 요약 페이지를 생성합니다.
- 권장 하위 구성 (필요에 따라 점진적으로 생성):
  - `docs/planning/` — 기획서, 로드맵
  - `docs/architecture/` — 시스템 아키텍처, 설계 문서
  - `docs/meetings/` — 회의록
  - `docs/releases/` — 릴리스 노트
  - `docs/guides/` — 사용자·운영 가이드
  - `docs/specs/` — API·기능 스펙

### 2-3. wiki/ 역할

- **LLM 소유 영역**: Claude Code가 작성·유지·갱신하는 구조화된 지식 베이스.
- **사람이 읽는 영역**: 팀원(3인)이 wiki를 통해 프로젝트 맥락을 빠르게 흡수하고 검증.
- **6-Layer 구조**: sources → topics → registry → decisions → synthesis → infra.

상세 규칙은 [wiki/SCHEMA.md](wiki/SCHEMA.md) 참조.

## 3. LLM Wiki 시스템

### 3-1. 3-Layer 원칙 요약

| Layer | 위치 | 성격 | 수정 주체 |
|-------|------|------|-----------|
| 1 | `docs/` | 원본 문서, 불변 | 사람만 |
| 2 | `wiki/` | 파생 지식, 구조화 | LLM 작성·유지 |
| 3 | `wiki/SCHEMA.md` | 운영 규칙 | LLM (review-queue 경유) |

### 3-2. 워크플로우

**모든 워크플로우의 Step 0 — 합의 큐 선(先)확인 + 이력 점검**
- 위키에 쓰기 작업을 하기 전 반드시 [wiki/review-queue.md](wiki/review-queue.md)의 **Pending Consensus** 섹션을 조회.
- 대기 항목이 있으면 작업을 잠시 중단하고, 각 팀원에게 "아래 합의 대기 항목부터 검토해 주세요"라고 안내한 뒤, 그 결과가 반영된 후 본 작업을 재개.
- 대상 페이지의 최근 `git log`와 Obsidian 백링크를 함께 확인하여 최근 변경·연관 페이지를 놓치지 않도록 함 (§3-4 참조).

**Ingest (문서 인제스트)**
1. 사람이 `docs/` 하위 적절한 폴더에 원본 추가
2. 사용자가 "위키에 반영해" 요청
3. **충돌 감지**: 요약·업데이트 대상이 기존 위키와 의미적 충돌([SCHEMA.md §6-3](wiki/SCHEMA.md#6-3-의미적-충돌-감지))을 일으키는지 점검
   - 충돌 있음 → **Pending Consensus**에 등록 후 중단 (3인 합의 대기)
   - 충돌 없음 → 다음 단계 진행
4. LLM이 `wiki/sources/`에 요약 페이지 생성
5. 관련 `topics/`, `registry/` 페이지 업데이트 또는 신규 생성
6. `index.md`, `log.md` 갱신

**Query (질의)**
1. `wiki/`에서 검색 → 관련 페이지 확인
2. 출처 명시하며 답변 (Claim Type 필수)
3. 정보 부족 시 `[!gap]` 으로 명시
4. 답변에 위키 수정이 수반되면 Ingest의 Step 3(충돌 감지)부터 동일 적용

**Lint (정합성 점검)**
1. 프런트매터 누락·형식 오류 점검
2. 깨진 링크, 고아 페이지(어디서도 참조되지 않는 페이지) 탐지
3. `index.md`와 실제 파일 목록 대조
4. **Pending Consensus** 중 24시간 이상 방치된 건 알림, Resolved 항목의 잔여 배너 제거

### 3-3. SCHEMA 참조

위키 작성·수정 시 반드시 [wiki/SCHEMA.md](wiki/SCHEMA.md)의 규칙을 따릅니다:
- YAML frontmatter 포맷
- 4종 Claim Type (`[!source]`, `[!analysis]`, `[!unverified]`, `[!gap]`)
- 레이어별 작성 규칙
- Review Queue 규칙

### 3-4. 보조 참조 소스 — git log · Obsidian

`wiki/log.md`만으로는 "누가·언제·무엇을 얼마나" 바꿨는지 모두 추적하기 어렵습니다. 작업 시 아래 두 소스를 **함께** 참조합니다.

**Git 이력**
- `git log -- wiki/경로/페이지.md` — 해당 페이지의 변경 이력과 커밋 메시지.
- `git log --follow -p -- wiki/…` — 파일 이동·이름 변경을 추적.
- `git blame wiki/…` — 각 라인의 최종 작성자·시점.
- `git diff {이전커밋}..HEAD -- wiki/` — 특정 시점 대비 위키 변화 일괄 확인.
- 역할 분담: `wiki/log.md`는 **의미적 변경 요약**(영향 페이지·이유), `git log`는 **실제 diff와 서명**. 두 기록이 상충하면 `git log`가 사실, `wiki/log.md`가 해석.

**Obsidian Vault**
- `wiki/` 디렉토리는 **Obsidian vault**로 그대로 열립니다 (Obsidian callout·YAML frontmatter·relative link 모두 지원).
- 팀원은 Obsidian에서 다음 기능을 활용:
  - **Backlinks** — 페이지를 참조하는 모든 곳 역추적.
  - **Graph View** — 엔티티 간 연결을 시각화해 고아 페이지 발견.
  - **Tag pane** — `tags:` 기반 분류 탐색.
  - **Quick Switcher** / **Search** — 전역 검색.
- LLM은 파일 시스템 도구로 충분하지만, `related` 링크를 채울 때는 **Obsidian 백링크 관점**으로 생각하여 양방향 참조가 성립하는지 확인.
- Vault 설정 팁: `.obsidian/` 폴더는 커밋 대상에서 제외 (사용자별 설정 차이). `docs/`는 Obsidian 설정에서 제외하거나 읽기 전용으로 취급.

**언제 어떤 소스를 쓰는가**
| 상황 | 우선 소스 |
|------|-----------|
| 최근 의미 변경 요약 보기 | `wiki/log.md` |
| 특정 페이지의 모든 diff | `git log -p` |
| 삭제된 페이지 복구 | `git log --diff-filter=D` + `git show` |
| 연관 페이지 탐색 | Obsidian backlinks / graph |
| 태그·카테고리 훑기 | Obsidian tag pane |
| 충돌 감지 보조 | git `blame` + Obsidian backlinks 양방향 확인 |

## 4. 작업 규칙

1. **docs/ 파일은 읽기 전용** — LLM은 절대 수정·삭제하지 않음. 원본 훼손 금지.
2. **wiki 페이지 생성/수정 후 index.md 갱신 필수** — 카탈로그 최신성이 위키 신뢰도의 기반.
3. **활동 기록은 log.md에 append** — 수정이 아닌 추가만. 시간 역순이 아닌 **시간순(오름차순)**.
4. **Claim Type 4종 필수 사용** — 모든 주장은 `[!source]`, `[!analysis]`, `[!unverified]`, `[!gap]` 중 하나로 표시.
5. **민감 파일 커밋 금지** — `.env`, 크레덴셜, API 키, 개인정보 포함 파일은 절대 추가하지 않음.
6. **Review Queue 준수 (두 트랙)**
   - **Pending Review (1인 리뷰)**: `decisions/`, `SCHEMA.md`, `infra/`, `registry/` 상태 변경.
   - **Pending Consensus (3인 합의)**: 기존 위키와 의미적으로 충돌하는 추가·수정·삭제. 3인 전원 approve 전까지 반영 금지, 자동 승인 없음.
7. **작업 시작 전 Consensus 큐 조회** — 대기 항목이 있으면 팀원 검토를 먼저 안내하고, 결과가 반영된 후 본 작업 재개.
8. **출처 추적** — 모든 `[!source]` 는 `docs/` 경로와 작성일을 명시.

## 5. 네비게이션 팁

| 목적 | 경로 |
|------|------|
| 원본 문서 탐색 | `docs/` |
| 특정 문서 요약 찾기 | `wiki/sources/` |
| 개념·용어 정리 | `wiki/topics/` |
| 프로젝트·시스템·인물 목록 | `wiki/registry/` |
| 과거 의사결정 추적 | `wiki/decisions/` |
| 현황 오버뷰·타임라인 | `wiki/synthesis/` |
| 위키 운영 규칙 | [wiki/SCHEMA.md](wiki/SCHEMA.md) |
| 전체 페이지 목록 | [wiki/index.md](wiki/index.md) |
| 최근 활동 확인 | [wiki/log.md](wiki/log.md) |
| 검토 대기 항목 | [wiki/review-queue.md](wiki/review-queue.md) |

## 6. 자주 쓰는 작업 패턴

- **"오늘 추가한 문서 위키에 반영해"** → Ingest 워크플로우 실행
- **"OwnGo의 X 기능에 대해 알려줘"** → Query 워크플로우, 출처 명시 답변
- **"위키 일관성 점검해"** → Lint 워크플로우 실행
- **"X에 대한 결정 기록 만들어"** → `decisions/` 신규 페이지 + review-queue 등록
