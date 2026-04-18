---
title: AI ERP OwnGo 대회 제출 영상 통합 스크립트 v0.8
type: source
created: 2026-04-18
updated: 2026-04-18
sources:
  - "docs/competition/[DOC] AI ERP OwnGo 대회 제출 영상 통합 스크립트_v0.8.md"
tags: [competition, video-script, submission, owngo-team, pitch]
related:
  - wiki/sources/ai-champion-competition-2026-briefing.md
  - wiki/registry/owngo-platform.md
  - wiki/registry/ai-champion-competition-2026.md
---

# AI ERP OwnGo 대회 제출 영상 통합 스크립트 v0.8

## 원본 정보

- **경로**: [`docs/competition/[DOC] AI ERP OwnGo 대회 제출 영상 통합 스크립트_v0.8.md`](../../docs/competition/%5BDOC%5D%20AI%20ERP%20OwnGo%20대회%20제출%20영상%20통합%20스크립트_v0.8.md)
- **작성일 (원본 기준)**: 2026-04-17
- **작성 주체**: 서명원
- **버전**: v0.8
- **문서 목적**: 인공지능 챔피언 대회 제출용 3분 이내 영상의 나레이션·시각화·팀원 소개 스크립트 통합본

## 문서 구조

원본은 총 4개 블록, 3분 타임라인으로 구성된다.

| # | 블록 | 구간 | 핵심 주제 |
|---|------|------|-----------|
| 1 | 서론 | 00:00 ~ 00:30 | 문제 정의(관찰·판단의 늪) → 핵심 가치(실행력 전환) → 선언 + 팀 소개 |
| 2 | 팀원 소개 | 00:30 ~ 01:15 | 김광혁(AI/Backend), 선윤호(Frontend/UX), 서명원(Infra/DevOps & Architecture) 3인 스크립트·시각화 |
| 3 | 아이디어 및 데모 | 01:15 ~ 02:35 | 에이전틱 시스템 + 가드레일 시연, gRPC 커넥터·ERPNext API 매핑, Terraform·실시간 로그 노출 |
| 4 | 최종 클로징 | 02:35 ~ 03:00 | 비전 선포("Antifragility"), 국산 AI 생태계 비즈니스 가치, 통합 클로징 샷 |

## 핵심 인용

> [!source] docs/competition/[DOC] AI ERP OwnGo 대회 제출 영상 통합 스크립트_v0.8.md §1 서론 (선언)
> "우리는 기업의 자산을 단순 관리하는 도구를 넘어, 자율적으로 비즈니스 문제를 해결하는 AI 에이전트 'OwnGo'를 구축했습니다."

> [!source] docs/competition/[DOC] AI ERP OwnGo 대회 제출 영상 통합 스크립트_v0.8.md §2 공통 인트로
> "저희 OwnGo 팀 3인은 모두 현장에서 '데이터 품질'을 최우선으로 다뤄온 실무 경험을 바탕으로, 서로의 빈틈을 채워주며 새롭게 도전하는 하나의 원팀(One Team)입니다."

> [!source] docs/competition/[DOC] AI ERP OwnGo 대회 제출 영상 통합 스크립트_v0.8.md §3 시연 구성
> "사용자가 '재고 확인' 입력 시, 화면 옆에 해당 명령이 gRPC 커넥터 코드로 변환되어 ERPNext API 엔드포인트에 도달하는 과정을 1:1 매핑으로 오버레이 노출."

> [!source] docs/competition/[DOC] AI ERP OwnGo 대회 제출 영상 통합 스크립트_v0.8.md §4 최종 클로징
> "Antifragility: 위기를 기회로 바꾸는 기술력 / 국산 AI 생태계의 비즈니스 가치 실현"

## 팀원 역할 요약

| 역할 | 담당자 | 경력 | 핵심 어필 포인트 |
|------|--------|------|------------------|
| AI / Backend | 김광혁 | 5년 차 | 빅데이터 플랫폼 고도화, 오픈소스 개발, gRPC 연동 |
| Frontend / UX | 선윤호 | 교육 3년 + 실무 4년 | 사내 ERP·AI 지식 허브 PL, 대화형 UI |
| Infra / DevOps & Architecture | 서명원 | 12년 차 | 공공·금융 데이터 AA, CI/CD, 시스템 안정성 |

## 요약 분석

> [!analysis]
> 문서는 [sources/ai-champion-competition-2026-briefing.md](ai-champion-competition-2026-briefing.md)의 "문제 정의 명확성 + 3분 영상 첫인상 + 에이전틱 역량 강조" 3대 심사 변별 포인트를 각 블록에 그대로 대응시킨 구조다. §1은 문제 정의, §2는 팀 신뢰도(기술 실체), §3은 에이전틱·가드레일 시연, §4는 사업화 비전.

> [!analysis]
> §1의 나레이션이 "1번 버전(직관적)"과 "2번 버전(부드러운)" 두 톤으로 병기된 점은 최종 녹음 직전 A/B 선택을 전제한 것으로 읽힌다. 즉 이 문서는 "촬영·녹음 최종본"이 아니라 "제작 직전 합의본(v0.8)"이다.

> [!analysis]
> [registry/owngo-platform.md](../registry/owngo-platform.md)의 "OSS 비종속 코어 + 커넥터 격리" 원칙이 §3 데모(gRPC 커넥터 → ERPNext API 매핑)에 가시적으로 반영되어 있다. 아키텍처 v1.0의 §10(커넥터 아키텍처)와 정합.

## 결락

> [!gap]
> 문서 상단 "확인사항"에 "문구에 나오는 기술들의 활용여부 검토", "5개 국내 AI모델 활용여부 검토" 2건이 미해결 상태로 남아 있다. 국내 AI 트랙 선택 여부와 구체 모델 매칭은 별도 결정 문서가 필요하다.

> [!gap]
> §3 데모 구성 중 "데모시연 영상에 나레이션 포함할지 검토"가 미결로 기재되어 있다. 최종 편집 방향 확정 필요.

> [!gap]
> 원본 내 `[[OwnGo팀 역할]]`, `[[OwnGo팀 경력이력]]` Obsidian 링크가 참조하는 문서가 현재 `docs/`에 존재하지 않는다. 해당 원본 문서 확보 시 sources/ 페이지 추가 필요.
