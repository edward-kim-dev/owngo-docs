---
title: OwnGo Platform
type: registry
created: 2026-04-18
updated: 2026-04-18
sources:
  - docs/architecture/owngo-platform-architecture-v1.0.md
tags: [platform, product, saas]
related:
  - wiki/sources/owngo-platform-architecture-v1.0.md
---

# OwnGo Platform

## 식별 정보

| 항목 | 값 |
|------|-----|
| **정식 명칭** | OwnGo Platform |
| **약어/별명** | OwnGo, 플랫폼 (내부 문맥에서) |
| **분류** | SaaS 제품 — DevOps Integrated Management Platform (OSS 기반) |
| **현재 상태** | 설계 단계 (v1.0 아키텍처 합의) |
| **최신 업데이트** | 2026-04-18 |

## 정의

> [!source] docs/architecture/owngo-platform-architecture-v1.0.md §1 핵심 용어 정의
> "플랫폼: OwnGo가 운영하는 SaaS 시스템 전체. 고객 서비스를 생성·관리·AI 커스터마이징하는 중앙 시스템."

## 구성 요소 (v1.0 아키텍처 기준)

- **웹 서버**: Java(Spring Boot) — REST/WebSocket, GitHub App 인증/Webhook 수신
- **AI 엔진**: TypeScript/Node.js — Claude Agent SDK + 하네스 코어 + 커넥터 플러그인
- **빌드 서비스**: Webhook Receiver → Docker Image Builder → Deployer → Health Checker
- **하네스 코어**: Tool Registry + Hook Chain + Profile System + Audit Log
- **커넥터**: OSS별 플러그인 (초기: 범용 커넥터 + 첫 OSS 커넥터)
- **인프라**: Azure VM(B2s per Tenant / B4ms 공유 Staging) + ACR + Blob + Cloudflare + PostgreSQL

## 핵심 설계 원칙

1. 고객 코드 소유 (고객 GitHub Org)
2. OSS 비종속 코어 (OSS 특화는 커넥터로 격리)
3. 지식 분리 (Platform Core / Connector / Customer / Session 4계층)
4. GitHub App 중심 연동
5. 플랫폼 사이드 CI/CD
6. 임시 Staging (Ephemeral)

## MVP 일정 (v1.0)

> [!source] docs/architecture/owngo-platform-architecture-v1.0.md §19
> "Phase 1 — 하네스 프레임워크 + 첫 번째 커넥터 (3주) / Phase 2 — GitHub App + CI/CD + 배포 자동화 (3주) / Phase 3 — 백엔드 API + 승인 플로우 (3주) / Phase 4 — 프론트엔드 + 통합 검증 (3주) / Phase 5 — 고도화 (2주)"

- **총 일정**: 14주 (Phase 1~5)
- **Phase 6+ 제외**: 범용 커넥터, 추가 OSS 커넥터, 소스 열람 UI, 온보딩 위자드, AKS 전환 등

## 도메인·용어

관련 핵심 용어는 [sources/owngo-platform-architecture-v1.0.md](../sources/owngo-platform-architecture-v1.0.md) §1 참조. 개별 topics/ 페이지는 문서 축적에 따라 점진적으로 분리 예정.

## 상태 이력

- 2026-04-18 — 설계 단계 (v1.0 아키텍처 설계서 접수)

> [!gap]
> 아직 릴리스 일정, 초기 타겟 OSS(첫 커넥터), 팀 역할 분담 등은 본 문서만으로 확정되지 않음. 후속 문서 확보 필요.
