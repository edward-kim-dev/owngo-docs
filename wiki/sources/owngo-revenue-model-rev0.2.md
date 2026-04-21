---
title: OwnGo 핵심 수익 모델 기획안 Rev 0.2
type: source
created: 2026-04-21
updated: 2026-04-21
sources:
  - docs/[DOC] 수익모델 Rev 0.2.md
tags: [business-model, revenue, saas, pricing, ai-agent, b2b-saas, lock-in, marketplace]
related:
  - wiki/sources/owngo-business-strategy-meeting-260401.md
  - wiki/sources/erp-system-painpoints.md
  - wiki/registry/owngo-platform.md
---

# OwnGo 핵심 수익 모델 기획안 Rev 0.2

> [!source] docs/[DOC] 수익모델 Rev 0.2.md
> 작성자: (미확인), Rev 0.2

## 목적

AI 에이전트 기반 ERP/RPA 플랫폼 OwnGo의 핵심 수익 구조를 '생성 ➡️ 실행 ➡️ 확장' 단계별로 설계한 기획안. 강력한 락인(Lock-in) 효과를 기반으로 5가지 수익 축을 정의한다.

## 핵심 전제

> [!source] docs/[DOC] 수익모델 Rev 0.2.md
> "자연어를 통한 즉각적인 시스템 구축은 사용자에게 강력한 편의성을 제공함과 동시에, 축적되는 데이터와 자동화된 워크플로우로 인해 **강력한 락인(Lock-in) 효과**를 창출합니다. 이를 기반으로 '생성 ➡️ 실행 ➡️ 확장'의 단계별 수익 구조를 설계합니다."

## 수익 모델 5축 요약

### 1. SaaS 구독 모델 (구축 복잡도 기반 티어)

> [!source] docs/[DOC] 수익모델 Rev 0.2.md
> AI 에이전트가 생성하는 '시스템과 데이터베이스의 깊이'를 기준으로 플랜을 차등화.

| 티어 | 대상 | 주요 기능 |
|------|------|-----------|
| **Starter** (Free/Low-cost) | 진입 모객용 | 단일 테이블 단순 워크플로우 (방문자 기록, 연차 신청 등) |
| **Business** (Core Revenue) | 중소 기업 | 관계형 DB·ERP 모듈 연동, 외부 API 플러그인 (국세청 조회, 은행 계좌 연동) |
| **Enterprise** | 대기업 | 기업 전용 온톨로지 구축·에이전트 튜닝·폐쇄형 환경 (온프레미스/클라우드 선택) |

### 2. RPA 컴퓨팅 종량제 (Usage-Based)

> [!source] docs/[DOC] 수익모델 Rev 0.2.md
> "RPA 커스텀 지원 전면 차단 + 플랫폼 기반 스케줄링. AI가 구축한 RPA(자동화 워크플로우)가 실행될 때마다 '액션 크레딧' 차감."

- 시스템 '생성' 허들은 낮추고, '동작' 시 서버 리소스(컴퓨팅)에 대해 과금.
- 고객 비즈니스 성장 = OwnGo 수익 증가, 동반 성장형 구조.

> [!analysis]
> 액션 크레딧 모델은 Zapier·Make 등 기존 자동화 플랫폼의 과금 방식과 유사하나, OwnGo는 ERP 맥락 특화 워크플로우에 집중하는 점에서 차별화 여지 존재. 과금 단위(크레딧 가격 체계)는 미정.

> [!gap]
> 액션 크레딧의 구체적인 단가 및 번들 정책이 문서에 정의되어 있지 않음. 검증 필요.

### 3. 인프라 배포 환경별 라이선스 (Hosting vs. Self-Hosted)

> [!source] docs/[DOC] 수익모델 Rev 0.2.md
> "**Self-Hosted**: 보안에 민감하거나 자체 Linux/Windows 서버 환경을 보유한 기업 대상. AI 에이전트가 설계한 아키텍처와 DB를 독립적인 **Docker 컨테이너 이미지**로 추출(Export)하여 고객사 인프라에 직접 배포. 초기 도입비(라이선스)와 연간 유지보수(SLA) 계약을 통한 고수익 창출."

| 유형 | 형태 | 수익 구조 |
|------|------|-----------|
| **Managed Cloud** | OwnGo 클라우드 호스팅 | 월/연간 구독 |
| **Self-Hosted** | Docker 이미지 Export → 고객사 배포 | 초기 라이선스 + 연간 SLA |

### 4. OwnGo Builder API · 화이트라벨링 (B2B2B)

> [!source] docs/[DOC] 수익모델 Rev 0.2.md
> "'자연어 해독 ➡️ 오픈소스 ERP 매핑'이라는 OwnGo의 코어 엔진 자체를 타 비즈니스 솔루션에 공급하는 생태계 확장 모델."

- 타 SaaS·협회 전용 프로그램·레거시 시스템에 OwnGo 대화형 에이전트 UI/API를 모듈 형태로 탑재.
- 화이트라벨링 지원으로 타사 고객 트래픽 기반 API 호출량 수익 확보.

### 5. OwnGo Marketplace (스스로 성장하는 플랫폼)

> [!source] docs/[DOC] 수익모델 Rev 0.2.md
> "OwnGo에 solver로 등록된 개발자 및 기업이 개발한 프로그램을 OwnGo 마켓플레이스에 등록하고 판매."

- 판매 수수료를 통한 수익 창출.
- 다양한 외부 솔루션으로 OwnGo 생태계 확장 및 사용자 참여 유도.

## 전략적 핵심 — Lock-in과 Export의 딜레마

> [!source] docs/[DOC] 수익모델 Rev 0.2.md
> "**진입은 쉽게**: 초기에는 사용자가 자연어로 시스템을 제약 없이 만들어보도록 허용하여 락인(Lock-in) 효과를 극대화합니다."
> "**이탈은 비용으로**: 축적된 데이터(엑셀 형태)의 반출은 허용하되, **AI가 구성한 '워크플로우 로직'과 '아키텍처' 자체를 내보내기(Export)하거나 외부 시스템과 연동(Webhook/API)하려는 시점**에 강력한 과금 게이트(Business 이상 플랜)를 설정하는 것이 B2B 수익화의 핵심입니다."

> [!analysis]
> 수익 구조는 Notion·Airtable의 SaaS 구독 + AWS Lambda의 컴퓨팅 종량제 + Salesforce의 Self-Hosted 라이선스를 결합한 하이브리드 모델에 해당. 락인 설계가 수익 모델 전체의 핵심 레버.

## 미결 사항 (Gap)

> [!gap]
> - 각 티어별 구체적인 월정액 가격 미정의.
> - 액션 크레딧 단가 및 번들 정책 미정의.
> - Self-Hosted 라이선스 기준 금액 및 SLA 범위 미정의.
> - Marketplace 수수료율 미정의.
> - Rev 0.1과의 차이점(변경 이력)이 문서에 기술되어 있지 않음.
