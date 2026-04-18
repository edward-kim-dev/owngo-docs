---
title: OwnGo 플랫폼 아키텍처 설계서 v1.0
type: source
created: 2026-04-18
updated: 2026-04-18
sources:
  - docs/architecture/owngo-platform-architecture-v1.0.md
tags: [architecture, platform, harness, connector, ontology, github-app, staging, mvp]
related:
  - wiki/registry/owngo-platform.md
---

# OwnGo 플랫폼 아키텍처 설계서 v1.0

## 원본 정보

- **경로**: [docs/architecture/owngo-platform-architecture-v1.0.md](../../docs/architecture/owngo-platform-architecture-v1.0.md)
- **작성일 (원본 기준)**: 2026-04-02
- **작성 주체**: OwnGo 팀 (딥리서치 기반 설계 산출물)
- **버전**: v1.0
- **문서 목적**: OwnGo Platform의 전체 아키텍처(시스템 구조·플로우·기술 스택·MVP 단계)를 합의 기준 문서로 확립

## 문서 구조

원본은 총 19개 섹션과 부록(핵심 설계 결정 요약)으로 구성된다.

| # | 섹션 | 핵심 주제 |
|---|------|-----------|
| 1 | 핵심 용어 정의 | 플랫폼/커넥터/하네스/온톨로지/스킬/Connector Vault/고객 리포 7종 용어 |
| 2 | 설계 원칙 | 고객 코드 소유, OSS 비종속 코어, 지식 분리, GitHub App 중심, 플랫폼 사이드 CI/CD, 임시 Staging |
| 3 | 시스템 아키텍처 개요 | 웹서버(Java) + AI 엔진(TS) + 빌드 서비스 + 하네스 + 커넥터 + GitHub App 구성도 |
| 4 | 고객 흐름 | 가입 → 서비스 생성 → 프로비저닝 → AI 커스터마이징 → Staging → Production 9단계 |
| 5 | OwnGo팀 흐름 | 새 OSS 온보딩 8단계 (조사 → 커넥터 → Vault → 하네스 → 스킬 → 테스트 → 템플릿 → 카탈로그) |
| 6 | GitHub App 관리 | 권한 5종, JWT 인증, Create+Push 방식, Direct Commit vs PR 모드, 업스트림 동기화 |
| 7 | 프로비저닝 파이프라인 | 리포 생성 → VM 프로비저닝 → 이미지 빌드 → 배포 → Health Check |
| 8 | CI/CD 및 배포 | 플랫폼 사이드 파이프라인, `.owngo/config.yaml` 계약, MCP 통합 |
| 9 | Staging 전략 | Ephemeral Staging(B4ms 공유, 3~8분 생성), 변경 유형별 Staging 필요 여부 표 |
| 10 | 커넥터 아키텍처 | `OSSConnector` 인터페이스, 디렉토리 구조, 8종 추상 Tool Registry |
| 11 | 지식 분리 | 4계층(Platform Core / Connector / Customer / Session), conventions.md는 Good/Bad 코드 쌍 |
| 12 | 하네스 3중 제어 | tools 배열 + Pre/Post Hook + Managed 잠금, 3-Tier 프로파일, SDK 제어 메시지 7종, 멀티에이전트 |
| 13 | 커스터마이징 → 소스 관리 | 변경 형상의 고객 리포 반영, 고객 리포 구조, "관리된 코어 수정" 5단계 게이트 |
| 14 | 멀티테넌트 인프라 | Azure B2s per Tenant (~5만원/월), 고객 수 구간별(1~5/5~15/15+) 운영 전략 |
| 15 | 온톨로지 | O-1~O-4 4단계 로드맵 (JSON → 선별 주입 → PostgreSQL+AGE → Neo4j), 정확도 추정 |
| 16 | 채팅 UX | Vue 3 + @ai-sdk/vue, 3단계 행동 표현(✅/⚠️/🚫), 실시간 상태 전달 |
| 17 | 보안 | Prompt Injection 3-Layer, 데이터 마스킹, 고객 소유권 보장(탈퇴 절차 포함) |
| 18 | 기술 스택 종합 | 프론트(Vue 3) + 백엔드(Java + TS) + 빌드 + Azure 인프라 + 온톨로지 스택 |
| 19 | MVP 단계 구분 | Phase 1~5 (각 3주/3주/3주/3주/2주), Phase 6+ 제외 항목 |
| 부록 | 핵심 설계 결정 요약 | 10개 결정 포인트와 선택 이유 |

## 핵심 인용

> [!source] docs/architecture/owngo-platform-architecture-v1.0.md §2 설계 원칙
> "오픈소스 코드는 고객의 GitHub 리포지토리에 존재한다. 고객은 언제든 OwnGo를 떠날 수 있다."

> [!source] docs/architecture/owngo-platform-architecture-v1.0.md §2 설계 원칙
> "플랫폼 코어 설계에 특정 OSS(ERPNext 등)가 등장하지 않는다. 모든 OSS 특화 로직은 커넥터로 격리한다."

> [!source] docs/architecture/owngo-platform-architecture-v1.0.md §12.2 3중 제어 계층
> "tools 배열 (하드) — 목록에 없는 도구는 호출 불가, 우회 가능성: 없음"

> [!source] docs/architecture/owngo-platform-architecture-v1.0.md §15.2 단계별 구축
> "온톨로지 '데이터'는 O-1부터 필수이고, '인프라'는 규모가 정당화될 때 진화한다."

> [!source] docs/architecture/owngo-platform-architecture-v1.0.md §19 MVP 단계 구분
> "Phase 1 — 하네스 프레임워크 + 첫 번째 커넥터 (3주) / Phase 2 — GitHub App + CI/CD + 배포 자동화 (3주) / Phase 3 — 백엔드 API + 승인 플로우 (3주) / Phase 4 — 프론트엔드 + 통합 검증 (3주) / Phase 5 — 고도화 (2주)"

## 요약 분석

> [!analysis]
> 이 문서는 "플랫폼 코어는 OSS 비종속, OSS 특화 로직은 커넥터로 격리"라는 단일 원칙을 19개 섹션에 걸쳐 일관되게 적용한다. 지식 분리(§11), 하네스(§12), 온톨로지(§15) 모두 동일 원칙의 다른 표현으로 읽힌다.

> [!analysis]
> MVP 일정(§19)은 총 14주(Phase 1~5)로 구성되며, Phase 1이 하네스 프레임워크 + 첫 커넥터를 동시에 다루는 구조다. 하네스는 플랫폼 코어, 커넥터는 플러그인이라는 분리를 전제하면 두 작업을 병렬로 진행하는 것이 합리적이다.

## 결락

> [!gap]
> 문서 본문에 "첫 번째 커넥터"의 대상 OSS가 명시되지 않았다(§10.2의 디렉토리 예시에는 ERPNext가 등장하나 §19 Phase 1의 "첫 번째 커넥터"는 추상적으로만 기술됨). 선정 근거와 일정 확정이 필요하다.

> [!gap]
> 비용 모델(고객당 월 5만원 VM + 공유 Staging 12만원)만 제시되며, 요금 정책·원가 계산·BEP 시나리오는 별도 문서로 분리되어 있지 않다.

> [!gap]
> 모니터링/관측성(로그 수집, 메트릭, 알림) 세부 설계가 부재하다. §3 개요도에는 "모니터링" 블록이 표기되어 있으나 본문 섹션이 없다.
