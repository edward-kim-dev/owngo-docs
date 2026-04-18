# Wiki SCHEMA

이 문서는 OwnGoDocs 저장소의 `wiki/` 운영 규칙을 정의합니다. Claude Code(LLM)와 사람이 모두 이 규칙을 준수합니다.

## 0. 목적

- **일관성**: 모든 위키 페이지가 동일한 포맷·구조를 따름.
- **추적성**: 모든 주장은 출처와 유형(Claim Type)을 명시.
- **재현성**: 사람이나 다른 LLM이 같은 규칙으로 위키를 이어갈 수 있음.
- **신뢰성**: 추측과 검증된 사실을 섞지 않음. 결락은 숨기지 않음.

## 1. 6-Layer 구조

| Layer | 디렉토리 | 역할 |
|-------|----------|------|
| 1 | `sources/` | `docs/` 원본 1:1 요약. 원본 경로·날짜 명시. |
| 2 | `topics/` | 도메인 주제별 지식 정리 (개념, 용어, 원칙). |
| 3 | `registry/` | 반복 등장하는 핵심 엔티티 레지스트리 (프로젝트, 시스템, 기관, 인물, 제품 등). |
| 4 | `decisions/` | 의사결정 기록 (ADR 스타일). 맥락-결정-결과 구조. |
| 5 | `synthesis/` | 종합 분석, 비교, 현황 오버뷰, 타임라인. |
| 6 | `infra/` | 위키 자체의 운영 규칙, 컨벤션, 자동화 훅. |

### 루트 파일

- `SCHEMA.md` — 이 문서 (위키 운영 규칙).
- `index.md` — 전체 페이지 카탈로그 (LLM이 자동 갱신).
- `log.md` — 시간순 활동 기록 (append-only).
- `review-queue.md` — 설계 변경 검토 큐.

## 2. 페이지 포맷

### 2-1. YAML Frontmatter

모든 wiki 페이지는 아래 frontmatter로 시작합니다.

```markdown
---
title: 페이지 제목
type: source | topic | registry | decision | synthesis | infra
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources:
  - docs/경로/파일명.pdf
tags: [태그1, 태그2]
related:
  - wiki/경로/관련페이지.md
---
```

필드 설명:
- `title`: 페이지 제목 (본문 첫 줄의 `# 제목`과 일치 권장).
- `type`: 6-Layer 중 하나. 오타·다른 값 금지.
- `created` / `updated`: ISO 8601 날짜. `updated`는 내용 변경 시마다 갱신.
- `sources`: 근거가 되는 `docs/` 하위 경로. 없으면 빈 리스트 `[]`.
- `tags`: 검색·분류용 태그. 소문자-하이픈 표기 (예: `data-governance`, `release-notes`).
- `related`: 교차 참조 대상 wiki 페이지 경로.

### 2-2. 파일 네이밍

- 소문자 + 하이픈: `owngo-platform-overview.md`
- 공백·한글 파일명 금지 (링크 깨짐 방지). 한글 제목은 frontmatter `title`에서 표기.
- `decisions/`는 ID 프리픽스 필수: `d2026-001-license-choice.md`.

## 3. 4종 Claim Type

모든 위키 페이지의 주장(claim)은 Obsidian callout으로 유형을 표시합니다. 유형을 표시하지 않은 주장은 작성 금지.

### 3-1. `[!source]` — 출처 인용

원문 직접 인용. 파라프레이징 금지. 출처와 인용 범위(페이지·섹션) 명시.

```markdown
> [!source] docs/planning/roadmap-2026.pdf, p.3
> "OwnGo Platform은 2026년 Q2까지 OSS 1.0 릴리스를 목표로 한다."
```

### 3-2. `[!analysis]` — 출처 기반 추론

하나 이상의 출처를 근거로 한 LLM·작성자의 추론. 추론 근거 반드시 기술.

```markdown
> [!analysis]
> roadmap-2026.pdf p.3의 "OSS 1.0" 목표와 release-notes-0.9.md의 진척도를 비교하면,
> 현재 계획 대비 약 2주 지연 가능성이 있음.
```

### 3-3. `[!unverified]` — 미검증 주장

출처가 없거나 권위 있는 출처를 찾지 못한 진술. 검증 필요 상태로 명시.

```markdown
> [!unverified]
> OwnGo Platform 내부 용어로 "Pod"와 "Unit"이 혼용되는 것으로 보이나,
> 정의 문서를 확인하지 못함. 검증 필요.
```

### 3-4. `[!gap]` — 명시적 결락

답이 없다는 사실 자체를 기록. 추측으로 채우지 않음.

```markdown
> [!gap]
> 현재 docs/에는 OwnGo의 인증·인가 모델을 설명하는 문서가 존재하지 않음.
> 관련 문서 확보 필요.
```

## 4. 레이어별 작성 규칙

### 4-1. `sources/`
- `docs/` 파일당 1개 페이지.
- 파일명: 원본 파일명을 기반으로 한 슬러그 + `.md`.
- 필수 내용: 원본 경로, 작성일(원본 기준), 작성자(파악 가능하면), 문서 목적, 핵심 내용 요약, 주요 인용.
- 원본 수정 금지. 요약은 위키에만 기록.

### 4-2. `topics/`
- 도메인 개념 단위 (예: `devops-integration`, `oss-licensing`, `observability-model`).
- 여러 source에서 추출한 지식을 주제별로 통합.
- 개념 정의, 관련 용어, OwnGo 문맥에서의 의미, 참조 sources 목록.

### 4-3. `registry/`
- 엔티티당 1개 페이지. 하위 카테고리 필요 시 `registry/projects/`, `registry/systems/`, `registry/people/` 등으로 분리.
- 필수 필드: 정식 명칭, 약어/별명, 분류, 현재 상태, 관련 문서 링크, 최신 업데이트 일자.
- 상태 변경 시 `log.md` 및 `review-queue.md` 등록.

### 4-4. `decisions/`
- 제목 형식: `[D{YYYY}-{NNN}] 결정 요약` (예: `D2026-001 내부 전용 라이선스 채택`).
- 파일명: `d2026-001-license-choice.md` (소문자-하이픈).
- 본문 구조 필수:
  1. **Context (맥락)** — 결정이 필요한 배경과 제약.
  2. **Decision (결정)** — 선택한 방향.
  3. **Consequences (결과)** — 긍정·부정·트레이드오프.
  4. **Alternatives (대안)** — 고려했으나 기각한 옵션과 이유.
  5. **Status** — `proposed | accepted | superseded | deprecated`.
- 생성·수정 시 `review-queue.md` 필수 등록.

### 4-5. `synthesis/`
- 복수 source·topic·registry를 횡단하는 분석.
- 단일 문서 요약은 `sources/`로 보냄.
- 종류 예시: 현황 오버뷰, 타임라인, 비교표, 갭 분석.

### 4-6. `infra/`
- 위키 자체 운영 규정 (네이밍 컨벤션, 태그 체계, 자동화 훅, 린트 규칙 등).
- SCHEMA.md와 중복되는 원칙은 SCHEMA.md를 단일 출처(Single Source of Truth)로 두고, infra/는 운영상의 세부 절차를 담음.

## 5. 작업 플로우

### 5-1. 표준 5단계

1. **합의 큐 선(先)확인**: 모든 위키 변경 작업 시작 전에 [review-queue.md](review-queue.md)의 **Pending Consensus** 섹션을 먼저 조회. 대기 항목이 있으면 작업을 잠시 중단하고, 각 팀원에게 "아래 합의 대기 항목부터 검토해 주세요"라고 안내한 뒤 그 결과가 반영된 후 본 작업을 재개.
2. **작업 시작 전**: `wiki/` 탐색 (특히 `index.md`, 관련 `topics/`·`registry/`, `related` 링크) → 맥락 확인.
3. **충돌 감지**: 제안된 추가/수정/삭제가 기존 위키 내용과 **의미적 충돌**을 일으키는지 점검 (§6-3 참조). 충돌 시 곧바로 적용하지 않고 [review-queue.md](review-queue.md)의 **Pending Consensus**에 등록.
4. **새 문서 인제스트 / 질의 응답**: 충돌이 없으면 `sources/` 생성, 관련 `topics/`·`registry/` 업데이트 또는 Claim Type을 명시한 답변 제공.
5. **작업 완료 후**: `index.md` 갱신, `log.md`에 활동 기록 append.

### 5-2. index.md 갱신 원칙

- 페이지 생성·삭제·이름 변경 시 **즉시** 반영.
- 레이어별 섹션에 `- [제목](경로)` 1줄로 등재.
- 태그·검색어 인덱스는 별도 섹션 또는 `infra/` 페이지로 분리 가능.

### 5-3. log.md append 원칙

- 시간순(오름차순) append-only.
- 1건당 형식: `- YYYY-MM-DD HH:MM — [행위] 간단한 설명 ([영향받은 페이지 링크])`.
- 수정·삭제 금지. 잘못 기록한 경우 새 항목으로 정정.

## 6. Review Queue 규칙

Review Queue는 **두 종류의 트랙**을 가집니다.

- **Pending Review (1인 리뷰 트랙)**: 구조적으로 영향력이 큰 변경. 작성자 외 1명 동의로 반영.
- **Pending Consensus (3인 합의 트랙)**: 기존 위키 내용과 의미적으로 충돌하는 변경. 참여자 **3인 전원 합의**로 반영.

### 6-1. Pending Review — 필수 등록 대상 (1인 리뷰)

다음 변경 시 `review-queue.md`의 **Pending Review**에 항목을 추가합니다:
- `decisions/` 페이지 생성 또는 의미 있는 수정.
- `SCHEMA.md` 수정.
- `infra/` 변경.
- `registry/` 엔티티의 상태 변경 (예: `active` → `deprecated`).

**프로세스**
- 작성자 외 **1명** 리뷰 → 합의 시 반영.
- 24시간 내 리뷰어가 나타나지 않으면 **셀프 리뷰 24시간 경과 후 자동 승인** 규칙 적용 (비상 운용용, 남용 금지).
- 리뷰 완료 항목은 **Resolved** 섹션으로 이동, 1개월 유지 후 삭제.

### 6-2. Pending Consensus — 충돌 시 3인 합의

제안된 변경이 기존 위키 내용과 **의미적 충돌**을 일으키는 경우, `review-queue.md`의 **Pending Consensus**에 항목을 추가한 뒤 작업을 중단합니다. 적용은 **3인 전원 합의** 후에만 가능합니다.

**프로세스**
- 충돌 내용을 §6-3의 템플릿대로 기록 (기존 주장 vs. 제안 주장, 근거 출처).
- 3인 모두 `approve` 표시 → 반영.
- 1인이라도 `reject` 또는 `needs-discussion` → 항목을 `needs-discussion`으로 전환하고 대안 제안까지 보류.
- **자동 승인 규칙 없음**: 3인 합의 트랙은 타임아웃 자동 승인을 적용하지 않음. 누락된 의견이 있으면 반영을 강제하지 않고 대기.
- 합의 완료 항목은 **Resolved** 섹션으로 이동, 1개월 유지 후 삭제.

### 6-3. 의미적 충돌 감지

**충돌로 간주하는 사례**
1. 동일 엔티티·개념에 대해 기존 `[!source]` 또는 `[!analysis]` 주장을 **부정·대체**하는 경우.
2. `topics/`의 정의·용어를 변경하여 기존 사용처와 **해석이 달라지는** 경우.
3. `registry/`의 정식 명칭, 약어, 분류를 변경하여 식별이 달라지는 경우.
4. 이전 `decisions/`의 결정을 뒤집거나 전제를 무력화하는 경우 (이 경우 새 decision을 생성하고 이전 항목을 `superseded` 처리, 동시에 Consensus 등록).
5. 삭제·이동이 다른 페이지의 `related`·`sources` 링크를 깨뜨리는 경우.

**충돌이 아닌 사례 (일반 리뷰 또는 바로 반영)**
- 기존 주장을 **보완·확장**하거나 새 출처를 추가하는 경우.
- 오탈자·포맷 수정 등 의미 변화가 없는 편집.
- 아직 어디에서도 참조하지 않는 완전 신규 페이지 (단, `decisions/` 등 §6-1 대상은 1인 리뷰 트랙 적용).

**감지 절차**
1. 대상 페이지의 `title`, `tags`, `related`, frontmatter `sources`로 연관 페이지 후보 수집.
2. 동일 엔티티를 다루는 `registry/`·`topics/` 검색 (`grep`·`Glob`).
3. 후보 페이지들의 Claim 섹션과 제안 내용을 대조 → 위 5개 유형 중 어디에 해당하는지 판정.
4. 충돌 발견 시 §6-4 템플릿으로 **Pending Consensus**에 등록, 대상 페이지 상단에 `> [!unverified] 합의 대기 중 (review-queue.md 참조)` 배너 삽입(선택).
5. 자동 감지가 모호할 때는 보수적으로 Consensus에 등록 — 의심스러우면 합의가 기본값.

### 6-4. Pending Consensus 항목 템플릿

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

## 7. 핵심 원칙

1. **`docs/`는 불변** — LLM은 읽기만 함.
2. **`wiki/`는 LLM 소유** — LLM이 작성·유지, 사람이 읽고 검증.
3. **`index.md`는 항상 최신** — 페이지 생성/삭제 시 즉시 갱신.
4. **출처 추적 의무** — 모든 주장에 Claim Type 표시.
5. **모순은 숨기지 않음** — 문서 간 불일치 발견 시 `[!analysis]` 또는 `[!gap]`으로 명시적 기록.
6. **점진적 성장** — 빈 레이어도 구조로 유지, 필요 시 채움.

## 8. 린트 체크리스트

주기적 점검 항목:
- [ ] 모든 페이지에 frontmatter 존재, `type` 값 유효.
- [ ] 모든 주장에 Claim Type 표시.
- [ ] `sources/` 페이지의 원본 경로가 `docs/`에 실제로 존재.
- [ ] `index.md`와 실제 파일 목록 일치.
- [ ] 고아 페이지(참조되지 않는 페이지) 존재 여부 확인.
- [ ] `related` 링크가 유효한 경로를 가리키는지.
- [ ] `decisions/` 변경이 `review-queue.md`에 기록되어 있는지.
- [ ] `review-queue.md`의 **Pending Consensus** 항목 중 24시간 이상 방치된 건이 있는지 (담당자 알림 필요).
- [ ] 대상 페이지 상단의 `> [!unverified] 합의 대기 중` 배너가 이미 Resolved 처리된 항목에 남아 있지 않은지.
