---
document_id: "SPEC-CORE-P2-001"
title: "기본 녹취 솔루션 정보 구조도 (IA) 및 사이트맵 명세"
version: "1.0.0"
stage: "Phase 2 - Approved"
last_updated: "2026-09-17"
---
# Phase 2. 정보 구조 (Information Architecture, IA)
---
document_id: "SPEC-CORE-P2-001"
title: "기본 녹취 솔루션 정보 구조도 (IA) 및 사이트맵 명세"
version: "1.0.0"
stage: "Phase 2 - Approved"
last_updated: "2026-09-17"
---

## 1. 개요 및 설계 원칙
* **코어 도메인 집중화**: 외부 통신 엔진(특화 CTI/SIP, AI STT/TA)의 물리적 종속성을 배제하고, 순수 녹취 코어(Pure Recording Core) 운영, 이력 반출, QA 평가, 컴플라이언스 관리에 초점을 맞춘 4대 도메인으로 구성한다.
* **무제한 계층 반영**: 조직 관리 체계는 고정 뎁스를 배제하고, CRM 마스터 조직을 그대로 승계하는 무제한 계층 구조를 전제로 메뉴를 라우팅한다.
* **권한 기반 가시성 제어**: RBAC 등급 및 접속 IP 인가 정책에 따라 각 메뉴 노드의 노출 여부를 동적으로 통제한다.

---

## 2. 최상위 정보 구조도 (Mermaid LR)

`mermaid
graph LR
    classDef root fill:#1e293b,stroke:#0f172a,stroke-width:2px,color:#ffffff,font-weight:bold;
    classDef gnb fill:#2563eb,stroke:#1d4ed8,stroke-width:2px,color:#ffffff,font-weight:bold;
    classDef lnb fill:#f8fafc,stroke:#cbd5e1,stroke-width:1px,color:#0f172a;
    classDef leaf fill:#ffffff,stroke:#e2e8f0,stroke-width:1px,color:#475569;

    Root["기본 녹취 코어 Admin"]:::root

    %% GNB Modules
    GNB1["1. 실시간 모니터링"]:::gnb
    GNB2["2. 녹취 이력 관리"]:::gnb
    GNB3["3. QA 품질 평가"]:::gnb
    GNB4["4. 시스템 & 보안 관리"]:::gnb

    Root --> GNB1
    Root --> GNB2
    Root --> GNB3
    Root --> GNB4

    %% 1. 실시간 모니터링
    GNB1 --> M1_1["1.1 운영 대시보드"]:::lnb
    M1_1 --> M1_1_1["채널 가동 현황 위젯"]:::leaf
    M1_1 --> M1_1_2["스토리지/장애 알림"]:::leaf

    GNB1 --> M1_2["1.2 채널 관제"]:::lnb
    M1_2 --> M1_2_1["채널/내선 상태 그리드"]:::leaf
    M1_2 --> M1_2_2["실시간 감청 (옵션 통제)"]:::leaf

    %% 2. 녹취 이력 관리
    GNB2 --> M2_1["2.1 녹취 이력 조회"]:::lnb
    M2_1 --> M2_1_1["다차원 복합 검색"]:::leaf
    M2_1 --> M2_1_2["정부 표준 마스킹 그리드"]:::leaf
    M2_1 --> M2_1_3["원클릭 파형 플레이어"]:::leaf

    GNB2 --> M2_2["2.2 반출/다운로드"]:::lnb
    M2_2 --> M2_2_1["반출 사유 필수 입력"]:::leaf
    M2_2 --> M2_2_2["결재 승인 / 암호화 다운로드"]:::leaf

    GNB2 --> M2_3["2.3 보존 및 파기"]:::lnb
    M2_3 --> M2_3_1["Legal Hold (보존 연장)"]:::leaf
    M2_3 --> M2_3_2["만료 데이터 파기 승인"]:::leaf

    %% 3. QA 품질 평가
    GNB3 --> M3_1["3.1 평가표 관리"]:::lnb
    M3_1 --> M3_1_1["동적 평가 시트 템플릿"]:::leaf

    GNB3 --> M3_2["3.2 코칭 & 피드백"]:::lnb
    M3_2 --> M3_2_1["구간 태깅 피드백 전달"]:::leaf

    GNB3 --> M3_3["3.3 QA 평가 리포트"]:::lnb
    M3_3 --> M3_3_1["상담원/부서별 통계 리포트"]:::leaf

    %% 4. 시스템 & 보안 관리
    GNB4 --> M4_1["4.1 조직 및 내선"]:::lnb
    M4_1 --> M4_1_1["무제한 조직 트리 / CRM 동기화"]:::leaf
    M4_1 --> M4_1_2["내선(Extension) 매핑 관리"]:::leaf

    GNB4 --> M4_2["4.2 사용자 및 권한"]:::lnb
    M4_2 --> M4_2_1["계정 관리 & 단일/대역 IP 통제"]:::leaf
    M4_2 --> M4_2_2["역할(RBAC) 및 접근 권한"]:::leaf

    GNB4 --> M4_3["4.3 감사 로그 (Audit)"]:::lnb
    M4_3 --> M4_3_1["청취 / 다운로드 로그"]:::leaf
    M4_3 --> M4_3_2["감청 / 마스킹 해제 로그"]:::leaf

    GNB4 --> M4_4["4.4 정책 & 스토리지"]:::lnb
    M4_4 --> M4_4_1["기능 통제 옵션 설정 (Toggle)"]:::leaf
<<<<<<< HEAD
    M4_4 --> M4_4_2["보존 수명주기 / 마스킹 규칙"]:::leaf
=======
    M4_4 --> M4_4_2["보존 수명주기 / 마스킹 규칙"]:::leaf
>>>>>>> 86d8d130acb99c90758e7e8badee28572cb58596
