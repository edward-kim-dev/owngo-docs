---
title: 1주차 26-04-01 회의 (OwnGo 사업전략)
type: source
created: 2026-04-20
updated: 2026-04-20
sources:
  - docs/planning/1주차 26-04-01 회의 (OwnGo 사업전략).md
tags: [planning, strategy, meeting, mvp, business-model]
related: []
---

# 1주차 26-04-01 회의 (OwnGo 사업전략)

> [!source] docs/planning/1주차 26-04-01 회의 (OwnGo 사업전략).md
> 작성자: 서명원, 작성일: 2026-04-11

## 목적
OwnGo 플랫폼의 초기 사업 전략, 제품 방향성, 그리고 10주간의 기술적 실행 로드맵(MVP)을 정의함.

## 핵심 내용 요약
- **문제 정의**: 기존 ERP는 수동 입력(기록의 감옥)에 의존하여 실패함. 10~50인 규모 제조/유통기업의 현장 통제력과 데이터 무결성 확보가 목표.
- **제품 전략**: 
  - 'Money-View'(머니뷰)를 중심으로 역설계. 실시간 공헌 이익 산출 및 의사결정 지원.
  - AI OCR로 전표/거래명세서 인식을 통한 데이터 수집 자동화.
  - AI 제안 + 오너의 최종 승인(Human-in-the-Loop) 루프로 신뢰성 확보.
- **비즈니스 모델**:
  - Basic(가시성), Standard(최적화), Premium(지능화) 3단계 구독 모델 및 On-Premise/Private Cloud 하이브리드 운영.
- **MVP 개발 로드맵 (10주)**:
  - Phase 1 (1~2주): 코어 온톨로지 정의, 데이터 무결성 셋업. (Harness 3중 가드레일 설계)
  - Phase 2 (3~6주): AI OCR 모듈 및 간이 공정 등록 UI 개발.
  - Phase 3 (7~8주): 머니뷰 대시보드 및 AI 승인 루프 모바일 구현.
  - Phase 4 (9~10주): 무결성 검증, 클로즈드 베타 시연.

> [!analysis]
> 10주 MVP 로드맵은 기존 아키텍처 문서에서 정의된 Harness 구조 및 Ontology 설계를 실행하기 위한 구체적 계획임. ERPNext의 기본 구조 변경을 제한하고(Configuration Drift 방지) 커스텀 Fixtures 기반 배포를 목표로 함.
