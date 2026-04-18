# OwnGo 플랫폼 아키텍처 설계서

> 2026.04.02 | v1.0
> 딥리서치 기반 설계 산출물

---

## 목차

1. [핵심 용어 정의](#1-핵심-용어-정의)
2. [설계 원칙](#2-설계-원칙)
3. [시스템 아키텍처 개요](#3-시스템-아키텍처-개요)
4. [고객 흐름](#4-고객-흐름)
5. [OwnGo팀 흐름 — 새 OSS 온보딩](#5-owngo팀-흐름--새-oss-온보딩)
6. [GitHub App 기반 고객 리포지토리 관리](#6-github-app-기반-고객-리포지토리-관리)
7. [서비스 프로비저닝 파이프라인](#7-서비스-프로비저닝-파이프라인)
8. [CI/CD 및 배포 아키텍처](#8-cicd-및-배포-아키텍처)
9. [Staging 전략](#9-staging-전략)
10. [커넥터 아키텍처 — OSS 플러그인 시스템](#10-커넥터-아키텍처--oss-플러그인-시스템)
11. [지식 분리 원칙 — Platform Core vs Connector Vault](#11-지식-분리-원칙--platform-core-vs-connector-vault)
12. [하네스 3중 제어 설계](#12-하네스-3중-제어-설계)
13. [AI 커스터마이징 → 소스 관리 흐름](#13-ai-커스터마이징--소스-관리-흐름)
14. [멀티테넌트 인프라](#14-멀티테넌트-인프라)
15. [온톨로지 기반 AI 정확도 향상](#15-온톨로지-기반-ai-정확도-향상)
16. [채팅 인터페이스 및 UX](#16-채팅-인터페이스-및-ux)
17. [보안](#17-보안)
18. [기술 스택 종합](#18-기술-스택-종합)
19. [MVP 단계 구분](#19-mvp-단계-구분)

---

## 1. 핵심 용어 정의

| 용어                | 의미                                                                                                                          |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| **플랫폼**          | OwnGo가 운영하는 SaaS 시스템 전체. 고객 서비스를 생성·관리·AI 커스터마이징하는 중앙 시스템.                                   |
| **커넥터**          | 특정 OSS(ERPNext, OpenMetadata 등)의 API를 Claude Tool로 래핑한 플러그인. 커넥터가 있어야 Claude가 해당 OSS를 조작할 수 있다. |
| **하네스**          | AI의 행동 범위를 물리적으로 제한하는 울타리. `tools` 배열(허용 도구) + Pre/Post Hook(파라미터/결과 검증).                     |
| **온톨로지**        | OSS의 엔티티 간 관계를 정리한 지식 그래프. AI가 "어떤 파일/테이블을 봐야 하는지" 정확히 판단하게 돕는다.                      |
| **스킬**            | 반복되는 커스터마이징 작업을 사전 검증된 워크플로우로 패키징한 것. (예: "리포트 생성", "필드 추가")                           |
| **Connector Vault** | 특정 OSS에 종속된 지식·설정·스킬을 격리 보관하는 디렉토리. 플랫폼 코어에는 포함되지 않는다.                                   |
| **고객 리포**       | 고객의 GitHub 조직에 존재하는 리포지토리. OSS 코드 + 커스터마이징 형상이 고객 소유로 관리된다.                                |

---

## 2. 설계 원칙

| #   | 원칙                     | 설명                                                                                               |
| --- | ------------------------ | -------------------------------------------------------------------------------------------------- |
| 1   | **고객 코드 소유**       | 오픈소스 코드는 고객의 GitHub 리포지토리에 존재한다. 고객은 언제든 OwnGo를 떠날 수 있다.           |
| 2   | **OSS 비종속 코어**      | 플랫폼 코어 설계에 특정 OSS(ERPNext 등)가 등장하지 않는다. 모든 OSS 특화 로직은 커넥터로 격리한다. |
| 3   | **지식 분리**            | 플랫폼 공통 지식과 OSS 특화 지식을 물리적으로 분리한다. (CLAUDE.md 패턴 참조)                      |
| 4   | **GitHub App 중심 연동** | 고객 리포에 대한 모든 조작(코드 푸시, 브랜치 생성, PR 생성)은 GitHub App 설치 토큰을 통한다.       |
| 5   | **플랫폼 사이드 CI/CD**  | 빌드·배포 파이프라인은 OwnGo가 소유한다. 고객 리포에 CI 설정이 들어가지 않는다.                    |
| 6   | **임시 Staging**         | Staging은 공유 인프라에서 임시(ephemeral)로 생성·파괴한다. Production만 고객 전용 VM에 상주한다.   |

---

## 3. 시스템 아키텍처 개요

```text
┌───────────────────────────────────────────────────────────────────────┐
│                        OwnGo Platform Core                            │
│                                                                       │
│  ┌──────────────┐    ┌──────────────┐   ┌───────────────────────────┐ │
│  │  웹 서버     │    │  AI 엔진     │   │  빌드 서비스              │ │
│  │  (Java)      │◄──►│ (TypeScript) │   │  (Docker Build + Deploy)  │ │
│  │  REST/WS     │    │  Claude SDK  │   │  Webhook → Build → Push   │ │
│  └──────┬───────┘    └──────┬───────┘   └───────────┬───────────────┘ │
│         │                   │                       │                 │
│         │            ┌──────┴─────────┐             │                 │
│         │            │  하네스 코어   │             │                 │
│         │            │  Tool Registry │             │                 │
│         │            │  Hook Chain    │             │                 │
│         │            │  Audit Log     │             │                 │
│         │            └──────┬─────────┘             │                 │
│         │                   │                       │                 │
│         │     ┌─────────────┼─────────────┐         │                 │
│         │     │ Connector A │ Connector B │ ...     │                 │
│         │     │ (Vault A)   │ (Vault B)   │         │                 │
│         │     └─────────────┴─────────────┘         │                 │
├─────────┼───────────────────────────────────────────┼─────────────────┤
│         │           GitHub App                      │                 │
│         │    ┌───────────────────────────┐          │                 │
│         └────► 고객 GitHub Org           │          │                 │
│              │  ├── customer-oss-repo    ◄──────────┘                 │
│              │  │   (OSS + 커스터마이징) │                            │
│              │  └── .owngo/config.yaml   │                            │
│              └────────────────────────── ┘                            │
│                                                                       │
│              ┌──────────────────────────┐                             │
│              │  고객 VM (Production)    │                             │
│              │  Docker Compose          │                             │
│              │  ├── OSS 서비스들        │                             │
│              │  └── 모니터링            │                             │
│              └──────────────────────────┘                             │
│                                                                       │
│              ┌──────────────────────────┐                             │
│              │  OwnGo 공유 서버         │                             │
│              │  (Ephemeral Staging)     │                             │
│              │  Docker Compose per req  │                             │
│              └──────────────────────────┘                             │
└───────────────────────────────────────────────────────────────────────┘
```

---

## 4. 고객 흐름

```text
① 회원가입 + 로그인
   │
   ▼
② 서비스 생성
   ├── 대상 OSS 선택 (카탈로그에서)
   ├── GitHub 조직 연결 (GitHub App 설치)
   ├── 서비스 이름 입력
   └── [생성] 클릭
   │
   ▼
③ 프로비저닝 (자동)
   ├── 고객 GitHub Org에 리포지토리 생성
   ├── OwnGo 레퍼런스 버전의 OSS 코드 푸시
   ├── .owngo/config.yaml + CI/CD 메타데이터 생성
   ├── 고객 전용 VM 프로비저닝 (Terraform + cloud-init)
   ├── Docker 이미지 빌드 + VM 배포
   ├── Health Check 통과 대기
   └── Production 환경 준비 완료
   │
   ▼
④ 서비스 대시보드 진입
   ├── 상태: 배포 중 → 준비 완료
   ├── Production URL 표시
   └── [AI 채팅 열기] 버튼
   │
   ▼
⑤ AI 채팅으로 커스터마이징 요청
   │  "견적서에 할인율 필드를 추가해주세요"
   │
   ▼
⑥ AI가 변경 계획 제시
   │  [승인] [거부] [수정 요청]
   │
   ▼
⑦ 고객 승인 → Ephemeral Staging에서 변경 실행
   │  ├── 공유 서버에 임시 Staging 인스턴스 생성
   │  ├── Production DB 스냅샷 복원
   │  ├── AI가 변경 수행 + 자동 테스트
   │  └── Staging URL에서 고객이 직접 확인
   │
   ▼
⑧ Production 반영
   │  ├── 고객 리포에 변경 커밋 (또는 PR 생성)
   │  ├── 빌드 서비스가 새 이미지 빌드
   │  ├── Production VM에 배포 + Health Check
   │  ├── 변경 이력에 자동 기록
   │  └── Staging 인스턴스 자동 파괴
   │
   ▼
⑨ 변경 이력 관리
      ├── 소스 열람 (GitHub에서 직접, 또는 플랫폼 UI)
      ├── 되돌리기 (스냅샷 기반 + git revert)
      └── 다음 커스터마이징 요청 → ⑤로 복귀
```

**반복 구간**: ⑤~⑨는 고객이 원하는 만큼 반복. 각 변경은 독립적으로 추적·롤백 가능.

---

## 5. OwnGo팀 흐름 — 새 OSS 온보딩

```text
① 대상 OSS 선정 + 기술 조사
   │  API 구조, 커스터마이징 메커니즘, Docker 배포 방식 파악
   │
   ▼
② 커넥터 개발
   │  ├── OSSConnector 인터페이스 구현
   │  ├── 읽기/쓰기/실행 Tool Definition 정의
   │  ├── Docker 배포 템플릿 작성 (docker-compose.yml)
   │  └── Health Check 엔드포인트 정의
   │
   ▼
③ Connector Vault 구축
   │  ├── 아키텍처 문서 (모듈 구조, 데이터 흐름)
   │  ├── 코딩 컨벤션 (해당 OSS의 확장 규칙)
   │  ├── 금지 패턴 (알려진 위험 조작)
   │  └── 온톨로지 (O-1: 핵심 엔티티 30~50개, 15.2절 참조)
   │
   ▼
④ 하네스 구성
   │  ├── 프로파일별 허용 도구 목록 (standard/power/admin)
   │  ├── 보호 엔티티 차단 목록
   │  ├── Pre-Hook 규칙 (파라미터 검증, 환경 강제)
   │  └── Post-Hook 규칙 (감사 로그, 스냅샷, 에러 처리)
   │
   ▼
⑤ 스킬 패키징
   │  ├── 검증된 워크플로우를 YAML 스킬로 정의
   │  ├── 스킬별 allowed_tools 축소 (최소 권한)
   │  └── 플랫폼 태그로 어떤 OSS에서 실행 가능한지 명시
   │
   ▼
⑥ 내부 테스트
   │  ├── 다양한 커스터마이징 시나리오 실행
   │  ├── 비전공자 테스트 (성공률 ≥ 80%)
   │  └── 실패 케이스 → 프롬프트/Hook 보강
   │
   ▼
⑦ 레퍼런스 리포 + 배포 템플릿 준비
   │  ├── OwnGo가 관리하는 OSS 레퍼런스 리포지토리 (특정 버전 고정)
   │  ├── Terraform + cloud-init 템플릿
   │  └── Docker 이미지 빌드 파이프라인
   │
   ▼
⑧ 서비스 카탈로그 등록
      ├── 고객이 서비스 생성 시 선택할 수 있는 OSS 목록에 추가
      └── 요금 정책 설정
```

---

## 6. GitHub App 기반 고객 리포지토리 관리

### 6.1 GitHub App 등록 및 권한

OwnGo는 하나의 GitHub App을 등록하고, 각 고객이 자신의 GitHub 조직에 설치한다.

**필요 권한:**

| 권한             | 수준  | 용도                        |
| ---------------- | ----- | --------------------------- |
| `contents`       | write | 코드 읽기/쓰기, 브랜치 생성 |
| `pull_requests`  | write | PR 생성/관리                |
| `administration` | write | 리포지토리 생성             |
| `metadata`       | read  | 기본 리포 정보 (필수)       |
| `webhooks`       | write | 푸시 이벤트 구독            |

**인증 흐름:**

```text
고객이 GitHub App 설치
    → OwnGo가 installation webhook 수신
    → installation_id를 고객 레코드에 저장

API 호출 시:
    1. App 개인키로 JWT 서명
    2. POST /app/installations/{id}/access_tokens → 1시간 유효 토큰 발급
    3. 해당 토큰으로 고객 리포 조작
```

### 6.2 리포지토리 생성 패턴

**Create + Push 방식** (Fork나 Template이 아닌):

```text
1. POST /orgs/{customer_org}/repos → 빈 리포 생성
2. OwnGo 레퍼런스 OSS 코드를 초기 커밋으로 푸시
3. .owngo/config.yaml 추가
4. 브랜치 보호 규칙 설정
```

이 방식은 업스트림 링크 없이 깨끗한 히스토리를 제공하며, GitHub App 설치 토큰만으로 완전히 동작한다. Fork 방식의 제약(비공개 리포 크로스 org 불가, 조직 정책 차단 가능)을 회피한다.

### 6.3 AI 변경사항 반영 패턴

고객이 AI 커스터마이징을 승인하면:

```text
1. feature 브랜치 생성: POST /repos/{owner}/{repo}/git/refs
2. 변경 코드 커밋: PUT /repos/{owner}/{repo}/contents/{path}
3. PR 생성: POST /repos/{owner}/{repo}/pulls
4. (자동 머지 또는 고객 승인 후 머지)
5. main 머지 시 빌드 서비스가 webhook으로 감지 → 자동 배포
```

고객 요청의 성격에 따라 두 가지 모드:

| 모드              | 시점                 | 설명                                            |
| ----------------- | -------------------- | ----------------------------------------------- |
| **Direct Commit** | Staging 검증 완료 후 | 간단한 변경은 main에 직접 커밋                  |
| **PR 기반**       | 복잡한 변경          | feature 브랜치 + PR로 변경 내역을 투명하게 제시 |

### 6.4 업스트림 업데이트 동기화

OwnGo가 레퍼런스 버전을 업데이트할 때 고객 리포에 반영하는 방법:

```text
1. OwnGo 레퍼런스 리포에서 새 버전 태그
2. 각 고객 리포에 sync 브랜치 생성
3. 레퍼런스 변경사항을 sync 브랜치에 푸시
4. PR 생성: "OwnGo 업데이트 v1.2.3 → v1.3.0"
5. 고객이 PR 리뷰 + 머지 (고객이 타이밍 통제)
```

---

## 7. 서비스 프로비저닝 파이프라인

고객이 [서비스 생성]을 클릭하면 아래 파이프라인이 자동 실행된다:

```text
[서비스 생성 요청]
   │
   ▼
① GitHub 리포 생성
   ├── 고객 Org에 리포 생성 (GitHub App)
   ├── 레퍼런스 OSS 코드 푸시
   ├── .owngo/config.yaml 생성
   └── Webhook 등록 (push 이벤트 → OwnGo 빌드 서비스)
   │
   ▼
② VM 프로비저닝 (동시 진행)
   ├── Terraform: Azure VM 생성 (B2s, 2vCPU/4GB)
   ├── cloud-init: Docker + Docker Compose 설치
   ├── SSH 키 등록 (OwnGo 빌드 서비스용)
   └── DNS 등록: {customer}.owngo.kr
   │
   ▼
③ Docker 이미지 빌드
   ├── 고객 리포 클론 (GitHub App 토큰)
   ├── 커넥터별 빌드 파이프라인 실행
   ├── 이미지 → Azure ACR 푸시: owngo.azurecr.io/{customer}/{service}:{sha}
   │
   ▼
④ 배포
   ├── SSH로 VM 접속
   ├── docker compose pull && docker compose up -d
   │
   ▼
⑤ Health Check
   ├── start_period 대기 (OSS별 상이, 보통 60~120초)
   ├── HTTP 프로브로 서비스 상태 확인 (커넥터가 엔드포인트 정의)
   ├── 성공 시:
   │   ├── GitHub Commit Status API → 녹색 체크
   │   ├── 서비스 상태: "준비 완료"
   │   └── 고객에게 알림 (카카오 알림톡 또는 이메일)
   └── 실패 시:
       ├── 자동 재시도 (최대 3회)
       ├── 실패 지속 시 OwnGo팀 알림
       └── 서비스 상태: "배포 실패 — 확인 중"
```

---

## 8. CI/CD 및 배포 아키텍처

### 8.1 플랫폼 사이드 CI/CD

빌드·배포 파이프라인은 OwnGo가 소유한다. 고객 리포에는 CI 설정 파일이 존재하지 않는다.

```text
고객 리포에 push (AI 커밋 또는 수동)
    → GitHub App이 push webhook 전송
    → OwnGo Webhook Receiver (서명 검증)
    → Build Queue (중복 제거, 최신 커밋 우선)
    → Builder:
        1. 리포 클론 (GitHub App 설치 토큰)
        2. .owngo/config.yaml 읽기 → 커넥터 식별
        3. 커넥터별 빌드 파이프라인 실행
        4. Docker 이미지 빌드 + ACR 푸시
    → Deployer:
        5. SSH로 고객 VM 접속
        6. docker compose pull && up -d
        7. Health Check (커넥터 정의 엔드포인트)
    → Reporter:
        8. GitHub Commit Status API 업데이트
        9. 고객 대시보드 상태 갱신
```

### 8.2 `.owngo/config.yaml` 계약

고객 리포 루트에 위치하며, 플랫폼과 고객 리포 사이의 계약이다:

```yaml
platform: owngo
version: "1.0"

connector: erpnext # 사용할 커넥터 식별자
connector_version: "1.0.0"

service:
  name: "alpha-erp"
  domain: "alpha.owngo.kr"

deployment:
  method: docker-compose
  compose_file: docker-compose.yml
  health_check:
    endpoint: /api/method/ping
    timeout: 120s
    retries: 5

environments:
  production:
    vm_id: "azure-vm-xxxx"
    auto_deploy_branch: main
```

### 8.3 AI 엔진 ↔ 빌드/Staging 통합 (MCP)

AI 엔진이 빌드 서비스와 Staging 환경을 직접 조작할 수 있도록 MCP(Model Context Protocol) 서버로 노출한다:

```text
AI 엔진 (Claude Agent SDK)
    │
    ├── mcp__staging__provision     → Staging 인스턴스 생성
    ├── mcp__staging__deploy        → Staging에 변경 배포
    ├── mcp__staging__get_url       → Staging URL 조회
    ├── mcp__build__trigger         → 빌드 파이프라인 실행
    ├── mcp__build__get_status      → 빌드 상태 조회
    └── mcp__health__check          → Health Check 실행
```

MCP 도구는 하네스의 Tool Registry에 등록되어 프로파일/훅 제어를 받는다. AI가 "Staging에 배포하겠습니다"라고 판단하면 `mcp__staging__deploy` 도구를 호출하고, PreToolUse Hook이 tenant 격리와 파라미터를 검증한 뒤 실행된다.

---

## 9. Staging 전략

### 9.1 임시 Staging (Ephemeral Staging)

각 고객에 대해 상시 Staging VM을 운영하지 않는다. 대신, OwnGo가 관리하는 공유 서버에서 요청 시 임시 Staging 인스턴스를 생성한다.

```text
AI 커스터마이징 승인
    │
    ▼
① Staging 인스턴스 생성 (공유 서버)
   ├── Docker Compose로 OSS 풀스택 기동
   ├── 고유 프로젝트 이름: staging-{customer}-{timestamp}
   ├── Production DB 스냅샷 복원 (데이터 마스킹 적용)
   └── Traefik 리버스 프록시: staging-{customer}-{id}.owngo.kr
   │
   ▼
② AI 변경 수행 + 자동 테스트
   │
   ▼
③ 고객 검증 (Staging URL로 직접 확인)
   │
   ▼
④ 승인 → Production 반영
   ├── 고객 리포에 커밋/PR
   ├── 빌드 → Production VM 배포
   │
   ▼
⑤ Staging 인스턴스 파괴
   └── docker compose down -v (데이터 포함 완전 삭제)
```

### 9.2 공유 Staging 서버 사양

| 항목          | 사양                                |
| ------------- | ----------------------------------- |
| VM            | Azure B4ms (4vCPU, 16GB)            |
| 월 비용       | ~12만원                             |
| 동시 Staging  | 2~3개 인스턴스                      |
| 네트워크      | 격리된 Docker 네트워크 per 인스턴스 |
| 리버스 프록시 | Traefik (자동 서브도메인 라우팅)    |
| 생성 시간     | 3~8분 (DB 복원 포함)                |

### 9.3 대안: 간단한 변경의 경우

필드 추가, 리포트 생성 등 단순 변경은 Staging 없이 처리 가능:

| 변경 유형             | Staging 필요 여부 | 검증 방법                                      |
| --------------------- | ----------------- | ---------------------------------------------- |
| 필드 추가             | 불필요            | AI가 변경 미리보기 제시 → 직접 Production 적용 |
| 리포트/인쇄 양식 생성 | 불필요            | AI가 결과물 미리보기 → 직접 적용               |
| 비즈니스 로직 변경    | **필요**          | Staging에서 검증 후 Production 반영            |
| 다중 엔티티 연관 변경 | **필요**          | Staging에서 검증 후 Production 반영            |

---

## 10. 커넥터 아키텍처 — OSS 플러그인 시스템

### 10.1 OSSConnector 인터페이스

모든 커넥터는 공통 인터페이스를 구현한다:

```text
OSSConnector (Interface)
   ├── platform_name: string           # 식별자 (예: "erpnext")
   ├── get_tool_definitions(): Tool[]  # 이 OSS에서 사용 가능한 도구 목록
   ├── execute_tool(name, params): Result
   ├── health_check(endpoint): boolean
   ├── build_image(repo_path): Image   # Docker 이미지 빌드
   ├── get_ontology(): Graph           # 온톨로지 그래프
   └── get_blocked_entities(): string[] # 보호 엔티티 목록
```

### 10.2 커넥터 디렉토리 구조

각 커넥터는 독립된 패키지로 존재한다:

```text
connectors/
  ├── _interface/              # 공통 인터페이스 정의
  │   ├── connector.ts
  │   └── types.ts
  │
  ├── erpnext/                 # ERPNext 커넥터 (첫 번째)
  │   ├── connector.ts         # OSSConnector 구현
  │   ├── tools/               # Tool Definition들
  │   │   ├── read.ts          # frappe_get_list, frappe_get_doc 등
  │   │   ├── write.ts         # frappe_insert_doc, frappe_set_value 등
  │   │   └── execute.ts       # frappe_create_server_script 등
  │   ├── knowledge/           # Connector Vault
  │   │   ├── architecture.md
  │   │   ├── conventions.md
  │   │   └── blocked-entities.yaml
  │   ├── ontology/            # 온톨로지 빌더 + 데이터
  │   │   ├── builder.ts
  │   │   └── graph.json
  │   ├── skills/              # OSS 특화 스킬
  │   │   ├── build-report.yaml
  │   │   └── build-print.yaml
  │   ├── templates/           # 배포 템플릿
  │   │   ├── docker-compose.yml
  │   │   └── Containerfile
  │   └── health.ts            # Health Check 구현
  │
  └── generic/                 # 범용 커넥터 (Git/DB/API 기반)
      ├── connector.ts
      ├── tools/
      │   ├── git.ts           # git_read_file, git_write_file
      │   ├── db.ts            # db_get_schema, db_query_readonly
      │   └── api.ts           # api_call
      └── ...
```

### 10.3 커넥터별 Tool Registry

| 추상 인터페이스 | 설명                   | 프로파일 |
| --------------- | ---------------------- | -------- |
| list_entities   | 엔티티 목록 조회       | standard |
| get_entity      | 단일 엔티티 상세 조회  | standard |
| get_schema      | 스키마/메타데이터 조회 | standard |
| create_entity   | 엔티티 생성            | power    |
| update_entity   | 엔티티 수정            | power    |
| run_query       | 읽기 전용 쿼리 실행    | standard |
| create_script   | 스크립트/로직 생성     | power    |
| apply_patch     | 소스 코드 패치 적용    | admin    |

각 커넥터는 위 추상 인터페이스를 자신의 OSS API로 구현한다. 예를 들어, ERPNext 커넥터는 `list_entities`를 `frappe.client.get_list`로, 범용 커넥터는 `git_read_file` + `db_get_schema`로 구현한다.

---

## 11. 지식 분리 원칙 — Platform Core vs Connector Vault

### 11.1 4계층 지식 모델

Claude Code의 CLAUDE.md 계층 패턴을 차용한다:

| 계층                    | 위치                          | 내용                                       | 로딩 시점         |
| ----------------------- | ----------------------------- | ------------------------------------------ | ----------------- |
| **Platform Core**       | 플랫폼 시스템 프롬프트        | 하네스 규칙, 공통 안전 정책, 승인 프로토콜 | 항상              |
| **Connector Knowledge** | `connectors/{oss}/knowledge/` | OSS 아키텍처, 컨벤션, 금지 패턴            | 해당 OSS 세션 시  |
| **Customer Context**    | 고객 리포 `.owngo/knowledge/` | 고객사 비즈니스 규칙, 커스터마이징 이력    | 해당 고객 세션 시 |
| **Session Context**     | 대화 히스토리                 | 현재 요청의 맥락                           | 현재 세션         |

### 11.2 분리 규칙

- **Platform Core에 포함되면 안 되는 것**: 특정 OSS의 API 호출 방법, 특정 OSS의 엔티티 구조, 특정 OSS의 코딩 컨벤션
- **Connector Vault에 포함되어야 하는 것**: 해당 OSS의 모든 기술적 지식, 온톨로지, 스킬 정의, 배포 템플릿
- **고객 리포에 포함되어야 하는 것**: 고객사 고유 비즈니스 규칙, 커스터마이징 히스토리, 환경 설정

### 11.3 예시: ERPNext 지식이 플랫폼 코어에 들어가면 안 되는 이유

```text
❌ System Prompt: "DocType은 ERPNext의 기본 데이터 모델이며..."
   → 다른 OSS 세션에서도 불필요하게 로딩됨

✅ connectors/erpnext/knowledge/architecture.md:
   "DocType은 ERPNext의 기본 데이터 모델이며..."
   → ERPNext 커넥터가 활성화된 세션에서만 로딩
```

### 11.4 지식 자동 생성

새 OSS 온보딩(5절) 시, 프로젝트 분석을 통해 Connector Vault 지식을 반자동 생성한다:

- **빌드/테스트/린트 커맨드 자동 감지**: `package.json`, `Makefile`, `setup.py` 등에서 추출
- **기존 문서 자동 참조**: `@include`로 OSS 자체 `ARCHITECTURE.md`, `CONTRIBUTING.md` 연결
- **경로 범위 규칙**: 모듈별로 다른 컨벤션을 가진 대규모 프로젝트는 `.claude/rules/*.md`에 `paths` 프론트매터로 조건부 적용

```text
OSS 레퍼런스 리포 분석
    → 빌드 시스템 감지 (npm/maven/pip/cargo...)
    → 기존 문서 수집 (ARCHITECTURE.md, CONTRIBUTING.md, API docs)
    → 온톨로지 O-1 수작업 정리 (핵심 엔티티 30~50개)
    → Connector Vault 지식 파일 생성
    → AI 세션 시 `@include`로 자동 주입
```

### 11.5 컨벤션 작성 원칙: 설명보다 코드 예시

Connector Vault의 `knowledge/conventions.md`는 **추상적 설명이 아닌 코드 예시 기반**으로 작성한다. AI는 세 문단의 규칙 설명보다 Good/Bad 코드 스니펫 한 쌍에서 훨씬 정확하게 패턴을 학습한다.

```markdown
# connectors/erpnext/knowledge/conventions.md 작성 예시

## Custom Script 작성 규칙

​```javascript
// ✅ Good — validate 이벤트에서 필드 검증, frappe.throw로 중단
frappe.ui.form.on('Quotation', {
  validate(frm) {
    if (!frm.doc.customer_name) {
      frappe.throw(__('Customer Name is required'));
    }
  }
});

// ❌ Bad — before_save에서 검증, alert로 표시 (저장 중단 안 됨)
frappe.ui.form.on('Quotation', {
  before_save(frm) {
    if (!frm.doc.customer_name) {
      alert('Customer Name is required');
    }
  }
});
​```
```

**작성 원칙:**
- 각 컨벤션 항목에 Good/Bad 코드 쌍을 반드시 포함
- 플래그와 옵션까지 포함한 정확한 명령어 제공 (예: `bench --site site1.local run-tests --module Selling` — `bench test`만으로는 불충분)
- 금지 패턴은 "왜 금지인지" 한 줄 주석으로 이유 명시

---

## 12. 하네스 3중 제어 설계

### 12.1 아키텍처

```text
사용자 요청
    → [커넥터 선택] → [Tool Registry (고객+OSS+프로파일 기반)]
    → Claude API tools=[필터링된 도구 목록]
         │
    ┌────┘
    ▼
[PreToolUse Hook Chain]        [Tool Executor]         [PostToolUse Hook Chain]
1. tenant_isolation_check  →   커넥터.execute_tool()  →  1. result_validation
2. parameter_validation                                   2. data_sanitization
3. staging_enforcement                                    3. snapshot_creation
4. rate_limit_check                                       4. audit_log
5. audit_log_pre                                          5. vault_update_check
```

### 12.2 3중 제어 계층

| 계층                     | 역할                          | 우회 가능성 |
| ------------------------ | ----------------------------- | ----------- |
| System Prompt (소프트)   | AI 사고 방향 가이드           | 있음        |
| **tools 배열 (하드)**    | 목록에 없는 도구는 호출 불가  | **없음**    |
| **Pre/Post Hook (하드)** | 파라미터/권한 검증, 결과 검증 | **없음**    |
| **Managed 잠금 (하드)**  | 플랫폼 정책 우회 원천 차단    | **없음**    |

**Managed 잠금 설정** — 플랫폼이 최상위 정책으로 강제하는 설정:

| 잠금 설정                          | 효과                                        |
| ---------------------------------- | ------------------------------------------- |
| `allowManagedHooksOnly`            | 플랫폼이 정의한 훅만 실행, 사용자 훅 무시   |
| `allowManagedPermissionRulesOnly`  | 플랫폼이 정의한 권한 규칙만 존중            |
| `allowManagedMcpServersOnly`       | 플랫폼이 등록한 MCP 서버만 허용             |
| `availableModels`                  | 사용 가능한 모델을 제한 (비용 제어)         |

### 12.3 프로파일 시스템

| 프로파일 | 허용 범위                           | 대상                |
| -------- | ----------------------------------- | ------------------- |
| standard | 읽기 도구 + 안전한 쿼리             | 일반 사용자         |
| power    | standard + 쓰기/생성 도구           | 관리자              |
| admin    | power + 코드 패치 + Production 승격 | OwnGo팀/슈퍼 관리자 |

**워크플로우 단계별 Permission 모드 전환:**

| 워크플로우 단계            | Permission 모드      | 이유                                       |
| -------------------------- | -------------------- | ------------------------------------------ |
| ⑥ AI가 변경 계획 제시     | `plan`               | 읽기 전용, 코드 분석만 허용                |
| ⑦ 승인 → Staging에서 실행 | `acceptEdits`        | 파일 편집 자동 승인, Bash는 Hook으로 검증  |
| ⑧ Production 배포 자동화  | `bypassPermissions`  | Docker 컨테이너 격리 하에서 완전 자동 실행 |

### 12.4 스킬 시스템

```yaml
# 스킬 정의 예시 (플랫폼 태그 포함)
name: add-field
platform: "*" # 모든 OSS에서 실행 가능
profile: power
description: 엔티티에 새 필드 추가
allowed_tools:
  - get_schema
  - get_entity
  - update_entity
steps:
  - analyze: 대상 엔티티 스키마 조회
  - plan: 변경 계획 제시 + 사용자 확인
  - execute: 필드 추가 실행
  - verify: 변경 결과 검증
```

### 12.5 Staging → Production 승인 게이트

```text
AI 수정 (Staging)
    → 자동 테스트 (커넥터별 정의)
    → OwnGo Change Request 생성
    → 고객 확인 (Staging URL에서 직접 검증)
    → 승인 시:
        ├── 고객 리포에 커밋/PR
        ├── 빌드 서비스 → Production 배포
        └── Health Check 통과 → 완료
    → 반려 시:
        ├── Staging 인스턴스 파괴
        └── 변경 취소
```

### 12.6 SDK 제어 인터페이스

AI 엔진(TypeScript)이 Claude Code를 프로그래밍 방식으로 제어하는 핵심 인터페이스. stdin/stdout JSON 스트림 기반.

**세션 초기화 — 하네스 설정 주입:**

```typescript
// AI 엔진이 커스터마이징 세션을 시작할 때
const session = await query({
  prompt: userRequest,
  options: {
    cwd: customerRepoPath,
    systemPrompt: platformCorePrompt,          // 하네스 규칙, 승인 프로토콜
    appendSystemPrompt: connectorKnowledge,    // Connector Vault 지식
    permissionMode: 'plan',                    // 초기: 분석 모드
    maxBudgetUsd: 5.0,                         // 요청당 비용 상한
    maxTurns: 50,                              // 루프 최대 턴 수
    hooks: harnessHookConfig,                  // Pre/Post Hook 체인
    agents: {
      SecurityAuditor: { ... },                // 보안 감사 에이전트
      TestGenerator: { ... }                   // 테스트 생성 에이전트
    }
  }
})
```

**핵심 제어 메시지:**

| 메시지                | 방향          | 하네스에서의 용도                          |
| --------------------- | ------------- | ------------------------------------------ |
| `initialize`          | 엔진→CLI      | 세션별 시스템 프롬프트, 훅, 에이전트 설정  |
| `can_use_tool`        | CLI→엔진      | 도구 호출 시 플랫폼이 실시간 승인/거부     |
| `hook_callback`       | CLI→엔진      | 플랫폼 자체 훅 로직 실행 (감사, 비용 추적) |
| `interrupt`           | 엔진→CLI      | 사용자가 실행 중인 작업 중단               |
| `set_permission_mode` | 엔진→CLI      | plan→acceptEdits 전환 (승인 후)            |
| `rewind_files`        | 엔진→CLI      | 실패한 변경 되돌리기                       |
| `get_context_usage`   | 엔진→CLI      | 토큰 사용량 모니터링                       |

**스트림 출력 → WebSocket 전달:**

| SDK 출력 타입     | 프론트엔드 전달 내용             |
| ----------------- | -------------------------------- |
| `stream_event`    | AI 응답 실시간 스트리밍          |
| `tool_progress`   | "파일 분석 중...", "테스트 실행 중..." |
| `result`          | 턴 완료, 비용, 소요시간         |
| `hook_callback`   | 보안 스캔 결과, 테스트 통과 여부 |

### 12.7 에이전틱 루프와 리소스 제어

AI 엔진의 각 커스터마이징 세션은 Claude Code의 에이전틱 루프로 구동된다:

```text
사용자 요청
    → 컨텍스트 조립 (System Prompt + Connector Knowledge + Customer Context)
    → Claude 추론 + 도구 선택 (tool_use 블록)
    → Permission 모드 + Hook Chain 평가
    → 도구 실행 (커넥터.execute_tool)
    → 결과 → 대화에 추가
    → (도구 호출 없을 때까지 반복)
```

**컨텍스트 조립 — 4계층 지식 자동 주입:**

| 계층              | 조립 시점        | 내용                                         |
| ----------------- | ---------------- | -------------------------------------------- |
| Platform Core     | 항상             | 하네스 규칙, 공통 안전 정책                  |
| Connector Vault   | 커넥터 선택 후   | OSS 아키텍처, 온톨로지, 금지 패턴            |
| Customer Context  | 고객 리포 클론 후| `.owngo/knowledge/` 비즈니스 규칙            |
| Session Context   | 요청 분석 후     | 관련 서브그래프(온톨로지), 최근 커스터마이징 이력 |

**리소스 제어 파라미터:**

| 파라미터                          | 용도                              | 기본값       |
| --------------------------------- | --------------------------------- | ------------ |
| `--max-budget-usd`                | 요청당 API 비용 상한              | 5.0          |
| `--max-turns`                     | 에이전틱 루프 최대 턴 수          | 50           |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS`   | 응답당 최대 출력 토큰             | 모델 기본값  |
| `CLAUDE_CODE_MAX_CONTEXT_TOKENS`  | 컨텍스트 윈도우 제한              | 모델 기본값  |
| `API_TIMEOUT_MS`                  | API 요청 타임아웃                 | 300,000ms    |
| Hook `timeout`                    | 개별 훅 실행 시간 제한            | 60s          |

**컨텍스트 윈도우 관리:**
- 대규모 코드베이스 작업 시 자동 **컴팩트** 트리거 (오래된 메시지 요약)
- 대용량 도구 출력은 **임시 파일**로 저장, 모델에 파일 경로 + 미리보기만 전달
- 세션 **저장/재개** 지원: 복잡한 커스터마이징의 중단/재개 가능 (`--resume <session-id>`)

### 12.8 멀티에이전트 활용

복잡한 커스터마이징 요청은 서브에이전트를 병렬로 스폰하여 처리한다:

```text
[Orchestrator] — 사용자 요청 분석, 작업 분할, 결과 종합
     │
     ├── [SecurityAuditor]         보안 영향 분석 (백그라운드)
     ├── [TestGenerator]           테스트 코드 생성 (백그라운드)
     ├── [CompatibilityChecker]    하위 호환성 검증 (백그라운드)
     └── [CodeModifier]            실제 코드 수정 (포그라운드, Worktree 격리)
```

| 활용 패턴              | 설명                                                           |
| ---------------------- | -------------------------------------------------------------- |
| **병렬 분석**          | 코드 분석 + 보안 감사 + 호환성 검사를 동시 실행               |
| **Worktree 격리**      | 각 커스터마이징을 격리된 git worktree에서 작업, 메인에 영향 없음 |
| **백그라운드 테스트**  | 테스트 실행을 백그라운드로 진행하면서 다음 수정 작업 계속      |
| **에이전트 메모리**    | 프로젝트별 커스터마이징 경험이 축적 → 반복 요청 시 정확도 향상 |

**에이전트 정의 표준** — 각 에이전트는 아래 4요소를 반드시 포함한다:

| 요소 | 설명 | 예시 (SecurityAuditor) |
| ---- | ---- | ---------------------- |
| **페르소나** | 역할, 전문성, 출력물 | "보안 취약점을 탐지하는 시니어 보안 엔지니어. OWASP Top 10 기준 보고서 출력" |
| **명령어** | 실행 가능한 CLI 커맨드 (플래그 포함) | `npm audit --audit-level=high`, `semgrep --config=auto` |
| **경계** | 3단계 행동 규칙 | 항상: 읽기 전용 분석 / 질문: 의존성 업데이트 제안 / 절대 금지: 코드 직접 수정 |
| **코드 예시** | Good/Bad 패턴 | ✅ parameterized query / ❌ string concatenation SQL |

---

## 13. AI 커스터마이징 → 소스 관리 흐름

### 13.1 변경 형상의 고객 리포 반영

모든 AI 커스터마이징 결과는 고객의 GitHub 리포에 커밋된다.

```text
사용자 요청 → [커넥터 선택] → AI 커스터마이징 수행 (Staging)
    │
    ▼
변경 형상 추출 (커넥터별 방식)
    ├── 설정 기반 OSS: 설정 파일 / fixtures 내보내기
    ├── 코드 기반 OSS: 소스 코드 변경 + 패치 파일
    └── API 기반 OSS: 설정 JSON 내보내기
    │
    ▼
고객 리포에 반영 (GitHub App)
    ├── Direct Commit (단순 변경)
    └── PR (복잡 변경)
    │
    ▼
빌드 서비스 → Production 배포
```

### 13.2 고객 리포 구조

```text
{customer}-{oss}/
  .owngo/
    config.yaml              # 플랫폼 ↔ 리포 계약
    knowledge/               # 고객사 비즈니스 규칙 (AI 컨텍스트)
      business-rules.md
      customization-history.md
  src/                       # OSS 소스 코드 (레퍼런스 버전 기반)
    ...
  customizations/            # 커스터마이징 형상 (커넥터별 구조)
    ...
  docker-compose.yml         # 배포 정의
  README.md
```

### 13.3 코어 수정 원칙: "관리된 수정"

OSS 코어 수정은 "금지"가 아니라 "추적 가능하게 관리"한다.

**코어 수정 허용 조건 (5단계 게이트):**

1. 해당 OSS의 공식 확장 메커니즘(플러그인, API, 설정)으로 해결 불가능함을 확인
2. 수정 범위를 최소화 (함수/라인 단위)
3. `owngo-patch-` 접두사 커밋으로 태깅
4. 업스트림 PR 동시 제출 (머지되면 고객 리포에서 제거)
5. 업스트림 업데이트 시 충돌 가능성 문서화

---

## 14. 멀티테넌트 인프라

### 14.1 VM per Tenant

| 항목        | 사양                                   |
| ----------- | -------------------------------------- |
| VM          | Azure B2s (2vCPU, 4GB)                 |
| 월 비용     | ~5만원/고객                            |
| 격리        | 물리적 VM 수준 완전 격리               |
| 자동화      | Terraform + cloud-init                 |
| SSL         | Cloudflare 무료 플랜 (DNS + CDN + SSL) |
| 도메인      | `{customer}.owngo.kr` 서브도메인       |
| 백업        | Azure Blob Storage 자동 스냅샷         |
| 이미지 저장 | Azure ACR (Container Registry)         |

### 14.2 단계별 전환

| 고객 수 | 전략                        | 전환 트리거                    |
| ------- | --------------------------- | ------------------------------ |
| 1~5곳   | VM + Terraform + cloud-init | 즉시 시작                      |
| 5~15곳  | VM + Ansible 일괄 관리      | VM 관리 시간 > 주 10시간       |
| 15곳+   | AKS 전환 검토               | VM 관리 시간 > 전체 업무의 30% |

---

## 15. 온톨로지 기반 AI 정확도 향상

### 15.1 커넥터별 온톨로지 빌더

온톨로지 구축 파이프라인은 각 커넥터가 소유한다. 출력 포맷(관계 그래프 JSON)만 통일한다.

**정확도 향상 추정:**

| 시나리오              | 단순 RAG | 온톨로지 RAG | 개선폭 |
| --------------------- | -------- | ------------ | ------ |
| 단일 엔티티 필드 수정 | 80-90%   | 90-95%       | 소폭   |
| 다중 엔티티 연관 수정 | 50-65%   | 75-85%       | 유의미 |
| 비즈니스 로직 변경    | 30-50%   | 60-75%       | 유의미 |

### 15.2 단계별 구축

온톨로지는 **데이터 정밀도**와 **저장·조회 인프라** 두 축으로 진화한다. 아래 표는 두 축을 하나의 로드맵으로 통합한 것이다.

| 단계    | 데이터 정밀도                                  | 저장·조회 인프라                                                      | 비용       | 효과      |
| ------- | ---------------------------------------------- | --------------------------------------------------------------------- | ---------- | --------- |
| **O-1** | 핵심 엔티티 30~50개 수작업 정리                | JSON 그래프 파일 → AI 세션 시 System Prompt에 통째 주입               | 반나절/OSS | 즉시 체감 |
| **O-2** | 메타데이터 자동 파싱 → 전체 엔티티 관계 그래프 | JSON 파일 + 선별 주입 (요청 관련 서브그래프만 추출)                   | 1-2일/OSS  | 80%       |
| **O-3** | 소스코드 AST 파싱 → 메서드/호출 매핑           | PostgreSQL 저장 + API 기반 N홉 쿼리 (Apache AGE 확장으로 Cypher 가능) | 3-5일/OSS  | 95%       |
| **O-4** | + 런타임 호출 추적 (선택)                      | Neo4j (그래프 DB) — 영향도 분석, 그래프 알고리즘, GraphRAG 통합       | 2-3주      | 고도화    |

> **접두사 O-**: 온톨로지 고유 단계. MVP Phase 번호(19절)와 혼동 방지를 위해 별도 넘버링.
>
> - MVP Phase 1~3 → 온톨로지 O-1 (수작업 JSON, 컨텍스트 직접 주입)
> - MVP Phase 5 → 온톨로지 O-2 (자동 파싱)
> - MVP Phase 6+ → 온톨로지 O-3, O-4

**원칙: 온톨로지 "데이터"는 O-1부터 필수이고, "인프라"는 규모가 정당화될 때 진화한다.**

```text
O-1: JSON 파일 → Claude 컨텍스트에 직접 주입
  "Quotation → links_to: [Quotation Item, Pricing Rule, Sales Order]"
  → AI가 연관 엔티티를 놓치지 않고 수정 계획에 포함

O-2: JSON 파일 + 선별 주입
  사용자 요청 → 키워드로 관련 엔티티 탐색 → 2홉 서브그래프 추출
  → 필요한 부분만 컨텍스트에 주입 (토큰 효율)

O-3: PostgreSQL + Apache AGE
  SELECT * FROM cypher('ontology', $$
    MATCH (e:Entity {name: 'Quotation'})-[*1..2]-(related)
    RETURN related
  $$) AS (related agtype);
  → 결과를 Claude 컨텍스트에 주입

O-4: Neo4j
  MATCH path = (e:Entity {name: 'Quotation'})-[*1..3]-(affected)
  WITH affected, length(path) AS distance
  RETURN affected.name, distance, labels(affected)
  ORDER BY distance
  → 영향도 점수 계산, 위험도 자동 판단
```

### 15.3 통합 출력 포맷

```yaml
platform: "{connector_name}"
entities:
  EntityName:
    module: ModuleName
    fields: [...]
    links_to: [Entity1, Entity2]
    lifecycle_methods: [method1, method2]
    business_context: "이 엔티티의 비즈니스 역할 설명"
```

---

## 16. 채팅 인터페이스 및 UX

### 16.1 기술 스택

```text
프론트엔드: Vue 3 SPA (Vite)
  ├── @ai-sdk/vue        → useChat 컴포저블 (상태/스트리밍 관리)
  ├── ai-elements-vue    → shadcn-vue 기반 채팅 UI
  ├── markstream-vue     → 스트리밍 마크다운 + 코드 하이라이트
  └── 커스텀 컴포넌트     → 승인/거부 버튼, 미리보기 카드
```

### 16.2 UX 모드: 채팅 우선, 구조 탐색 보조

| 원칙                   | 설명                                           |
| ---------------------- | ---------------------------------------------- |
| Progressive Disclosure | 기본은 채팅만, 필요 시 구조 패널 열림          |
| 구조를 보여줘라        | 코드가 아닌 필드 테이블, 워크플로우 다이어그램 |
| 항상 되돌리기 보장     | 변경 결과 옆에 "되돌리기" 버튼                 |
| 전문 용어 번역         | 기술 용어 → 비즈니스 한글 표현                 |

### 16.3 레이아웃

```text
+-----------+--------------------+--------------+
| 변경이력  |     채팅 패널      | 컨텍스트     |
| 사이드바  |  [사용자 메시지]   | 패널         |
| (접이식)  |  [AI 응답]         | (접이식)     |
|           |  [미리보기 카드]   | [필드목록]   |
|           |  [적용/되돌리기]   | [다이어그램] |
|           |  [...입력창...]    |              |
+-----------+--------------------+--------------+
```

### 16.4 한국 중소기업 UX 핵심

| 원칙          | 설명                                         |
| ------------- | -------------------------------------------- |
| 한글 우선     | "Staging"→"테스트 서버", "Deploy"→"적용하기" |
| 단계별 가이드 | 점진적 공개, 필수 기능부터 시작              |
| 카카오 알림톡 | 승인 요청/배포 완료 알림                     |
| 엑셀 친화     | XLSX 가져오기/내보내기 눈에 띄게             |

### 16.5 AI 응답의 3단계 행동 표현

AI가 고객에게 보여주는 행동을 **3단계로 시각적으로 구분**하여 신뢰감과 안전감을 준다:

| 단계 | AI 표현 | 트리거 조건 | UI 표현 |
| ---- | ------- | ----------- | ------- |
| ✅ **자동 적용** | "필드를 추가했습니다" | standard 프로파일 + 단순 변경 + 자동 테스트 통과 | 결과 카드 + 되돌리기 버튼 |
| ⚠️ **확인 필요** | "이 변경은 다른 엔티티에 영향을 줍니다. 진행할까요?" | 다중 엔티티 연관 변경, 비즈니스 로직 수정 | 변경 계획 카드 + 승인/거부 버튼 |
| 🚫 **수행 불가** | "이 작업은 보안 정책상 수행할 수 없습니다" | Hook deny, 차단 목록, 보호 엔티티 접근 | 사유 설명 + 대안 제시 |

이 3단계는 하네스의 프로파일(12.3절) + Permission 모드 + Hook 결과에 의해 자동 결정되며, 고객이 기술적 세부사항을 몰라도 안전하게 작업을 진행할 수 있게 한다.

### 16.6 실시간 상태 전달 아키텍처

AI 커스터마이징 진행 상황을 고객에게 실시간으로 보여주는 데이터 흐름:

```text
Claude Code SDK (stream-json)
    │
    │  stream_event ──→ AI 응답 텍스트 스트리밍
    │  tool_progress ─→ "파일 분석 중...", "테스트 실행 중..."
    │  hook_callback ─→ 보안 스캔 통과, 린트 결과
    │  result ────────→ 턴 완료 + 비용 + 소요시간
    │
    ▼
AI 엔진 (TypeScript) ── 이벤트 정규화 + 한글 번역
    │
    ▼
Java 웹 서버 ── WebSocket 브로드캐스트
    │
    ▼
Vue 3 채팅 UI ── 실시간 렌더링
    ├── AI 응답 스트리밍 (markstream-vue)
    ├── 단계별 진행 표시 (분석 → 수정 → 테스트 → 검증)
    ├── 변경 파일 diff 미리보기
    └── 예상 비용 / 잔여 턴 표시
```

---

## 17. 보안

### 17.1 Prompt Injection 대응

핵심 전략: 방어가 아니라 **피해 범위 제한**.

```text
Layer 1: 입력 전처리 — 오버라이드 패턴 탐지, XML/JSON escape
Layer 2: System/User 격리 — Claude API 자체 격리 + <user_request> 태그 래핑
Layer 3: Tool 실행 검증 — tools 배열 + Pre-Hook (하드 제약, 최종 방어선)
```

### 17.2 데이터 보안

| 항목            | 정책                                                      |
| --------------- | --------------------------------------------------------- |
| ERP 실데이터    | API로 나가지 않는 아키텍처 (스키마만 전달, 데이터 마스킹) |
| GitHub App 토큰 | 1시간 만료, 리포 스코핑 가능 (최대 500개)                 |
| SSH 키          | Azure Key Vault에 보관                                    |
| Staging 데이터  | 마스킹 적용 후 복원, 검증 후 즉시 파괴                    |

### 17.3 고객 소유권 보장

| 항목        | 보장 내용                                                         |
| ----------- | ----------------------------------------------------------------- |
| 코드 소유   | 고객 GitHub Org에 존재, OwnGo 탈퇴 시 코드 유지                   |
| 데이터 소유 | 고객 VM에 존재, VM 접근 권한은 고객 보유                          |
| 탈퇴 절차   | GitHub App 제거 → OwnGo 접근 즉시 차단, VM/데이터는 고객에게 이관 |

---

## 18. 기술 스택 종합

```text
┌─────────────────────────────────────────────────────┐
│                    프론트엔드                       │
│  Vue 3 + Vite + Tailwind CSS                        │
│  ├── @ai-sdk/vue (AI 로직)                          │
│  ├── AI Elements Vue (채팅 UI)                      │
│  ├── markstream-vue (스트리밍 마크다운)             │
│  └── PrimeVue Unstyled (복잡 컴포넌트)              │
├─────────────────────────────────────────────────────┤
│              백엔드 — Java (웹 서버)                │
│  Spring Boot                                        │
│  ├── REST API + WebSocket (클라이언트 통신)         │
│  ├── 인증/인가, 세션 관리                           │
│  ├── GitHub App 인증 + Webhook 수신                 │
│  └── AI 엔진 호출 (gRPC 또는 내부 REST)             │
├─────────────────────────────────────────────────────┤
│         백엔드 — TypeScript/Node.js (AI 엔진)       │
│  OwnGo AI Core                                      │
│  ├── Claude Agent SDK (tool_use 3중 제어)           │
│  ├── 하네스 코어                                    │
│  │   ├── Tool Registry (OSS별 Tool 관리)            │
│  │   ├── Hook Chain (Pre/Post 미들웨어)             │
│  │   ├── Profile System (standard/power/admin)      │
│  │   └── Audit Log + Change Snapshot                │
│  ├── 커넥터 플러그인                                │
│  │   ├── connectors/_interface/ (공통 인터페이스)   │
│  │   ├── connectors/erpnext/ (첫 번째 커넥터)       │
│  │   └── connectors/generic/ (범용 커넥터)          │
│  └── 스킬 엔진 (YAML 워크플로우)                    │
├─────────────────────────────────────────────────────┤
│                  빌드 서비스                        │
│  ├── Webhook Receiver (GitHub push 이벤트)          │
│  ├── Docker Image Builder (Kaniko 또는 DinD)        │
│  ├── Deployer (SSH + docker compose)                │
│  └── Health Checker                                 │
├─────────────────────────────────────────────────────┤
│                    인프라                           │
│  ├── Azure VM per Tenant (B2s) — Production         │
│  ├── Azure VM 공유 (B4ms) — Ephemeral Staging       │
│  ├── Azure ACR — Docker 이미지 레지스트리           │
│  ├── Azure Blob Storage — 백업                      │
│  ├── Terraform + cloud-init — VM 자동화             │
│  ├── Cloudflare — DNS + SSL + CDN                   │
│  └── PostgreSQL — OwnGo 메타데이터                  │
├─────────────────────────────────────────────────────┤
│              AI 정확도 향상                         │
│  ├── 커넥터별 온톨로지 빌더                         │
│  │   ├── O-1~2: JSON 그래프 (컨텍스트 직접 주입)    │
│  │   ├── O-3: PostgreSQL + Apache AGE               │
│  │   └── O-4: Neo4j (그래프 알고리즘, GraphRAG)     │
│  ├── 4계층 지식 모델                                │
│  └── Prompt Caching (비용 30-50% 절감)              │
└─────────────────────────────────────────────────────┘
```

---

## 19. MVP 단계 구분

### Phase 1 — 하네스 프레임워크 + 첫 번째 커넥터 (3주)

| 영역            | 항목              | 설명                                                         |
| --------------- | ----------------- | ------------------------------------------------------------ |
| **하네스 코어** | Tool Registry     | ToolDefinition 모델 + OSS/프로파일/스킬 기반 필터링          |
|                 | Hook Chain        | Pre/Post Hook 미들웨어 체인                                  |
|                 | Profile System    | 3-Tier 프로파일 (standard/power/admin)                       |
|                 | Audit Logger      | 감사 로그 기록                                               |
| **커넥터**      | 커넥터 인터페이스 | OSSConnector 인터페이스 정의                                 |
|                 | 첫 번째 커넥터    | 읽기/쓰기 도구 구현                                          |
| **AI 연동**     | Claude API        | Claude Agent SDK tool_use 루프                               |
|                 | SDK 제어 프로토콜 | initialize, can_use_tool, hook_callback 메시지 처리 (12.6절) |
|                 | Orchestrator      | 상태 머신 (WAITING→ANALYZING→EXECUTING→REVIEWING→RESPONDING) |
|                 | 리소스 제어       | max-budget-usd, max-turns, 자동 컴팩트 (12.7절)             |

### Phase 2 — GitHub App + CI/CD + 배포 자동화 (3주)

| 영역            | 항목                    | 설명                          |
| --------------- | ----------------------- | ----------------------------- |
| **GitHub App**  | App 등록 + 인증         | JWT 서명, 설치 토큰 관리      |
|                 | 리포 생성 + 코드 푸시   | 프로비저닝 파이프라인         |
|                 | PR 생성 + 웹훅 수신     | AI 변경사항 반영              |
| **빌드 서비스** | Webhook Receiver        | push 이벤트 수신 + 서명 검증  |
|                 | Docker Builder          | 커넥터별 이미지 빌드          |
|                 | Deployer + Health Check | SSH 배포 + HTTP 프로브        |
| **Staging**     | Ephemeral Staging       | 공유 서버 Docker Compose 기반 |

### Phase 3 — 백엔드 API + 승인 플로우 (3주)

| 영역                | 항목                       | 설명                                         |
| ------------------- | -------------------------- | -------------------------------------------- |
| **AI 엔진 API**     | Hono 래핑                  | POST /chat (SSE), GET /tools, GET /audit     |
|                     | 스킬 엔진                  | YAML 스킬 로드 + 단계별 실행                 |
|                     | 사용자 확인 Tool           | present_options, request_confirmation        |
| **Java 웹 서버**    | Spring Boot                | REST API + WebSocket                         |
|                     | 인증/세션                  | 기본 로그인 + 대화 히스토리 (PostgreSQL)     |
|                     | 승인 게이트                | Change Request → 승인/반려 → Production 반영 |
| **Connector Vault** | 첫 번째 커넥터 지식 베이스 | 아키텍처, 컨벤션, 온톨로지 Phase 0           |

### Phase 4 — 프론트엔드 + 통합 검증 (3주)

| 영역            | 항목              | 설명                                      |
| --------------- | ----------------- | ----------------------------------------- |
| **프론트엔드**  | Vue 3 채팅 UI     | @ai-sdk/vue + AI Elements Vue (SSE)       |
|                 | 승인/거부 UI      | 미리보기 카드 + 적용/되돌리기 버튼        |
|                 | 변경 이력 뷰      | 감사 로그 타임라인                        |
| **통합 테스트** | E2E 파이프라인    | 자연어 → AI → 승인 → Staging → Production |
|                 | 비전공자 시나리오 | 성공률 ≥ 80% 목표                         |

### Phase 5 — 고도화 (2주)

| 영역              | 항목                  | 설명                                   |
| ----------------- | --------------------- | ------------------------------------- |
| **온톨로지**      | O-2 자동 파싱         | 메타데이터 → 관계 그래프 (15.2절 참조) |
| **스킬 추가**     | 2~3종 추가            | 범용 + OSS 특화                        |
| **프롬프트 튜닝** | 실패 케이스 기반 개선 | 조건부 XML 블록, System Prompt 최적화  |
| **인프라**        | 백업 자동화           | Azure Blob + cron                      |

### 제외 (Phase 6 이후)

- 범용 커넥터 (Git/DB/API)
- 추가 OSS 커넥터
- 소스 열람 UI (구조 탐색기)
- 온보딩 위자드
- 비주얼 Before/After diff
- AKS 전환
- 한국 세무 현지화
- 모바일 최적화
- 셀프 호스팅 배포

---

## 핵심 설계 결정 요약

| 결정 포인트        | 선택                      | 이유                                             |
| ------------------ | ------------------------- | ------------------------------------------------ |
| **코드 소유**      | 고객 GitHub Org           | 고객 소유권 보장, Lock-in 방지                   |
| **리포 생성 방식** | Create + Push (Fork 아님) | 깨끗한 히스토리, GitHub App 토큰만으로 동작      |
| **고객 리포 변경** | GitHub App                | 세밀한 권한 스코핑, 1시간 토큰, 조직 변동에 강건 |
| **CI/CD**          | 플랫폼 사이드             | 고객 리포에 CI 설정 불필요, 빌드 인프라 통제     |
| **Staging**        | Ephemeral (공유 서버)     | 고객당 상시 Staging 비용 절감 (65%↓)             |
| **OSS 지식**       | Connector Vault 격리      | 플랫폼 코어 OSS 비종속                           |
| **아키텍처**       | 커넥터 플러그인 패턴      | 다양한 OSS 지원 가능                             |
| **코어 수정**      | "관리된 수정"             | 금지가 아닌 추적 관리, owngo-patch 브랜치        |
| **AI 인터페이스**  | tool_use 기반 하이브리드  | Skill 매칭 + 자유 대화                           |
| **UX 모드**        | 채팅 우선, 구조 탐색 보조 | 비전공자 최적, 투명성 확보                       |
