# OwnGoDocs

> OwnGo Platform 문서 관리 — 오픈 소스 개발·운영 통합 관리 플랫폼 (DevOps Integrated Management Platform)

OwnGo Platform과 관련된 모든 문서(기획, 설계, 회의록, 릴리스 노트, 가이드 등)를 한곳에 모으고, **LLM 기반 지식 베이스**로 재가공하여 3인 팀이 맥락을 빠르게 공유하기 위한 저장소입니다.

- **라이선스**: PROPRIETARY — 내부 전용
- **팀 규모**: 3인
- **코드 없음**: 이 저장소는 순수 문서 관리 저장소입니다.

---

## 1. 저장소 한눈에 보기

```
OwnGoDocs/
├── README.md              # 이 파일 — 팀원 온보딩·작업 가이드
├── CLAUDE.md              # Claude Code(LLM) 작업 지침
├── docs/                  # 원본 문서 (PDF/DOCX/XLSX 등, 불변)
└── wiki/                  # LLM 소유 지식 베이스 (Obsidian vault)
    ├── SCHEMA.md          # 위키 운영 규칙 (단일 출처)
    ├── index.md           # 전체 페이지 카탈로그
    ├── log.md             # 시간순 활동 기록 (append-only)
    ├── review-queue.md    # 변경 검토 큐 (1인 리뷰 / 3인 합의)
    ├── sources/           # docs/ 원본 1:1 요약
    ├── topics/            # 도메인 주제별 지식
    ├── registry/          # 핵심 엔티티 목록
    ├── decisions/         # 의사결정 기록 (ADR 스타일)
    ├── synthesis/         # 종합 분석·타임라인
    └── infra/             # 위키 자체 운영 규칙
```

## 2. 3-Layer 구조

| Layer | 위치 | 성격 | 수정 주체 |
|-------|------|------|-----------|
| 1 | `docs/` | 원본 문서, 불변 | 사람만 |
| 2 | `wiki/` | 파생 지식, 구조화 | LLM 작성·유지, 사람이 검증 |
| 3 | `wiki/SCHEMA.md` | 운영 규칙 | LLM (review-queue 경유) |

원본(`docs/`)과 요약·해석(`wiki/`)을 분리하여, 원본은 훼손되지 않고 요약은 자유롭게 갱신할 수 있게 합니다.

## 3. 빠른 시작

### 팀원(사람) 기준

1. 저장소를 클론합니다.
   ```bash
   git clone <repo-url> OwnGoDocs
   cd OwnGoDocs
   ```
2. `wiki/` 디렉토리를 **Obsidian vault**로 엽니다.
   - Obsidian → "Open folder as vault" → `OwnGoDocs/wiki/` 선택.
   - callout, backlinks, graph view, tag pane이 바로 동작합니다.
3. [wiki/index.md](wiki/index.md)부터 훑어 전체 페이지 카탈로그를 파악합니다.
4. [wiki/SCHEMA.md](wiki/SCHEMA.md)로 운영 규칙을 숙지합니다.

### LLM(Claude Code) 기준

1. [CLAUDE.md](CLAUDE.md)를 따릅니다 — 모든 작업 지침의 출발점.
2. 모든 쓰기 작업 전에 [wiki/review-queue.md](wiki/review-queue.md)의 **Pending Consensus** 섹션을 먼저 조회.
3. 변경이 기존 위키와 의미적으로 충돌하면 즉시 반영하지 말고 **Pending Consensus**에 등록.

## 4. 기본 워크플로우

### 4-1. 새 문서 인제스트 (Ingest)

```
사람: docs/ 하위에 원본 추가 → 커밋
사람: "위키에 반영해" 요청
LLM: 합의 큐 선확인 → 충돌 감지 →
     sources/ 페이지 생성 → 관련 topics/·registry/ 갱신 →
     index.md·log.md 갱신
```

### 4-2. 질의 (Query)

```
사람: "OwnGo의 X 기능은?" / "Y 결정의 맥락은?"
LLM: wiki/ 검색 → Claim Type 명시 답변
     - [!source] 원문 인용 + 출처
     - [!analysis] 추론 + 근거
     - [!unverified] 미검증
     - [!gap] 명시적 결락
```

### 4-3. 정합성 점검 (Lint)

```
사람: "위키 일관성 점검해"
LLM: frontmatter·링크·index.md·고아 페이지·Consensus 방치 점검 →
     문제 보고, 필요 시 수정 제안 (수정은 규칙에 따라 진행)
```

## 5. 검토·합의 규칙 (두 트랙)

| 트랙 | 대상 | 승인 조건 | 자동 승인 |
|------|------|-----------|-----------|
| **Pending Review** (1인 리뷰) | `decisions/`, `SCHEMA.md`, `infra/`, `registry/` 상태 변경 | 작성자 외 1인 동의 | 24시간 경과 시 셀프 승인 |
| **Pending Consensus** (3인 합의) | 기존 위키와 **의미적 충돌**이 있는 추가·수정·삭제 | 3인 전원 approve | 없음 — 누락 시 대기 |

**의미적 충돌의 예** — 기존 주장을 부정/대체, 용어 재정의, 레지스트리 정체성 변경, 이전 결정 뒤집기, 링크 파괴 등. 상세는 [wiki/SCHEMA.md §6-3](wiki/SCHEMA.md#6-3-의미적-충돌-감지).

## 6. Claim Type — 모든 주장에 표시

| 타입 | 용도 | 요건 |
|------|------|------|
| `[!source]` | 원문 인용 | 출처·페이지 명시, 파라프레이징 금지 |
| `[!analysis]` | 출처 기반 추론 | 근거 출처 필수 |
| `[!unverified]` | 권위 있는 출처 없음 | 검증 필요 표시 |
| `[!gap]` | 명시적 결락 | 추측으로 채우지 않음 |

예시:

```markdown
> [!source] docs/planning/roadmap-2026.pdf, p.3
> "OwnGo Platform은 2026년 Q2까지 OSS 1.0 릴리스를 목표로 한다."

> [!analysis]
> 위 로드맵과 현재 릴리스 노트를 대조하면 약 2주 지연 가능성이 있음.

> [!gap]
> 인증·인가 모델 설명 문서는 docs/에 존재하지 않음. 확보 필요.
```

## 7. 이력 추적 — 보조 참조

위키 `log.md`만으로는 상세한 변경 내역을 모두 담을 수 없습니다. 두 가지 보조 소스를 함께 사용합니다.

- **Git 이력**: `git log`, `git blame`, `git diff`로 실제 diff와 작성자·시점 추적.
- **Obsidian**: 백링크·그래프 뷰·태그 팬으로 연관 관계와 분류 탐색.

사용 구분 상세는 [CLAUDE.md §3-4](CLAUDE.md#3-4-보조-참조-소스--git-log--obsidian) 참조.

## 8. 자주 쓰는 작업 패턴

| 요청 문장 | 동작 |
|-----------|------|
| "오늘 추가한 문서 위키에 반영해" | Ingest 워크플로우 |
| "OwnGo의 X 기능에 대해 알려줘" | Query 워크플로우, 출처 명시 |
| "위키 일관성 점검해" | Lint 워크플로우 |
| "X에 대한 결정 기록 만들어" | `decisions/` 신규 페이지 + review-queue 등록 |
| "합의 대기 항목 보여줘" | `wiki/review-queue.md` Pending Consensus 조회 |
| "페이지 Y의 변경 이력" | `git log -p -- wiki/Y.md` + `wiki/log.md` 대조 |

## 9. 커밋·보안 원칙

- **docs/는 사람만 수정** — LLM은 읽기 전용.
- **민감 파일 커밋 금지** — `.env`, 크레덴셜, API 키, 개인정보 포함 파일은 절대 추가하지 않음.
- **Obsidian 사용자별 설정 제외** — `.obsidian/` 는 `.gitignore` 권장 (아직 미설정이면 필요 시 추가).
- **커밋 메시지** — 위키 의미 변화는 `wiki/log.md`에 요약, 커밋 메시지는 간결한 변경 사유 위주.

## 10. 주요 링크

- [CLAUDE.md](CLAUDE.md) — LLM 작업 지침
- [wiki/SCHEMA.md](wiki/SCHEMA.md) — 위키 운영 규칙 (단일 출처)
- [wiki/index.md](wiki/index.md) — 전체 페이지 카탈로그
- [wiki/log.md](wiki/log.md) — 시간순 활동 기록
- [wiki/review-queue.md](wiki/review-queue.md) — 변경 검토 큐

## 11. 팀원 역할 요약

- **모든 팀원**: `docs/`에 원본을 추가하고, LLM이 반영한 `wiki/` 결과를 검토. Consensus 대기 항목에 투표.
- **LLM (Claude Code)**: `wiki/` 작성·유지, 충돌 감지, 카탈로그·로그 갱신.
- **리뷰어 순환**: 3인 중 작성자를 제외한 나머지가 1인 리뷰 트랙의 리뷰어 역할을 교대로 수행 (운영 세부는 `wiki/infra/`에서 필요 시 정의).

## 12. 다음 단계

- `docs/` 하위에 실제 문서를 추가하면 위키가 성장하기 시작합니다.
- 반복 등장하는 프로젝트·시스템·인물은 `wiki/registry/`에 먼저 등록하면 이후 `sources/`·`topics/` 연결이 깔끔해집니다.
- 운영 세칙(네이밍, 태그 체계 등)은 `wiki/infra/`에 점진적으로 축적합니다.
