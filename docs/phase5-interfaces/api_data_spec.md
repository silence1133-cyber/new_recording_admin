---
document_id: "SPEC-CORE-P5-001"
title: "기본 데이터 인터페이스 및 REST API 입출력(I/O) 규격서"
version: "1.0.0"
stage: "Phase 5 - Draft"
last_updated: "2026-09-21"
---

# Phase 5. 데이터 인터페이스 및 REST API 입출력(I/O) 규격서

## 1. 인터페이스 설계 개요 및 원칙

1. **엔진 독립형 데이터 추상화**:
   * 하부 패킷 수집 엔진이나 특정 PBX/CTI 하드웨어 종속성을 배제하고 순수 녹취 메타데이터 및 거버넌스 중심의 데이터 모델을 구축한다.
2. **공통 응답 포맷 준수 (Standard Response Envelope)**:
   * 모든 REST API 응답은 `success`, `code`, `message`, `data`의 단일 구조로 캡슐화하여 반환한다.
3. **무결성 및 불변 감사 추적 (Audit Trail WORM)**:
   * 청취, 다운로드, 실시간 감청, 마스킹 해제, IP 차단 로그는 수정/삭제가 불가능한 Write-Once 테이블에 필수 기록한다.
4. **보안 스트리밍 세션 강제**:
   * 파일 직접 경로 노출을 차단하며 30분 만료 서명 토큰(Signed Token) 기반의 Range Chunk 스트리밍을 제공한다.

---

## 2. 핵심 데이터베이스 스키마 정의 (Core DB Schema)

### 2.1 통화 메타데이터 테이블 (`tb_rec_call_info`)
* **설명**: 수집 완료된 개별 통화 녹취 건의 기준 정보 및 라이프사이클 플래그를 관리한다.

| 컬럼명 | 데이터 타입 | Nullable | 기본값 | 설명 |
|:---|:---|:---:|:---:|:---|
| `call_id` | VARCHAR(64) | NO | PK | 통화 고유 식별자 (예: REC_20260921_0012) |
| `start_time` | DATETIME | NO | - | 통화 시작 일시 |
| `end_time` | DATETIME | NO | - | 통화 종료 일시 |
| `duration` | INT | NO | 0 | 통화 시간 (초 단위) |
| `call_type` | VARCHAR(10) | NO | 'IN' | 호 유형 (`IN`, `OUT`, `INTERNAL`) |
| `agent_id` | VARCHAR(32) | NO | - | 담당 상담원 사번/ID |
| `agent_name` | VARCHAR(50) | NO | - | 상담원 성명 |
| `org_id` | VARCHAR(32) | NO | - | 상담원 소속 조직 ID (`tb_sys_org.org_id`) |
| `ext_no` | VARCHAR(20) | NO | - | 내선 번호 (Extension) |
| `channel_no` | INT | YES | - | 물리 수집 채널 번호 |
| `ani` | VARCHAR(32) | NO | - | 발신 고객 전화번호 (평문 인덱싱용) |
| `dnis` | VARCHAR(32) | YES | - | 인입 수신 대표번호 |
| `file_path` | VARCHAR(255)| NO | - | 스토리지 물리 볼륨 상대 경로 |
| `file_size` | BIGINT | NO | 0 | 오디오 파일 용량 (Byte) |
| `file_hash` | VARCHAR(64) | NO | - | 파일 위·변조 검증 SHA-256 해시 |
| `legal_hold_yn`| CHAR(1) | NO | 'N' | 영구 보존 연장 여부 (`Y`/`N`) |
| `legal_hold_reason`| VARCHAR(255)| YES| - | 영구 보존 지정 사유/사건번호 |
| `retention_expire_date`| DATE | NO | - | 보존 주기 만료 일자 (파기 예정일) |
| `purge_status`| VARCHAR(20) | NO | 'ACTIVE'| 상태 (`ACTIVE`, `PENDING_PURGE`, `PURGED`) |

---

### 2.2 무제한 계층 조직 마스터 (`tb_sys_org`)
* **설명**: CRM 조직 마스터를 승계하여 깊이(Depth) 제한 없는 트리 구조를 지원한다.

| 컬럼명 | 데이터 타입 | Nullable | 기본값 | 설명 |
|:---|:---|:---:|:---:|:---|
| `org_id` | VARCHAR(32) | NO | PK | 조직 노드 고유 ID |
| `parent_org_id` | VARCHAR(32) | YES | NULL | 상위 조직 ID (최상위 ROOT는 NULL, 재귀 관계) |
| `org_name` | VARCHAR(100)| NO | - | 부서/조직 명칭 |
| `depth_level` | INT | NO | 1 | 계층 깊이 (1부터 N까지 무제한) |
| `node_path` | VARCHAR(500)| NO | - | 노드 전체 경로 (예: /ROOT/HQ01/CENTER02/TEAM01) |
| `sort_order` | INT | NO | 1 | 동일 레벨 내 정렬 순서 |
| `crm_sync_key` | VARCHAR(64) | YES | - | 고객사 CRM 조직 고유 매핑 키 |
| `status` | VARCHAR(20) | NO | 'ACTIVE'| 상태 (`ACTIVE`, `EXPIRED`(CRM 삭제 부서 이력용)) |
| `updated_at` | DATETIME | NO | CURRENT | 최종 갱신 일시 |

---

### 2.3 사용자 계정 및 접속 IP 화이트리스트 (`tb_sys_user`, `tb_sys_user_ip`)
* **설명**: 단일 IP 완전 일치(Exact Match)와 서브넷 대역(CIDR) 통제를 구분하여 저장한다.

#### tb_sys_user (사용자 마스터)
| 컬럼명 | 데이터 타입 | Nullable | 기본값 | 설명 |
|:---|:---|:---:|:---:|:---|
| `user_id` | VARCHAR(32) | NO | PK | 관리자 계정 ID |
| `user_name` | VARCHAR(50) | NO | - | 성명 |
| `password_hash` | VARCHAR(255)| NO | - | 단방향 암호화 비밀번호 (BCrypt) |
| `org_id` | VARCHAR(32) | NO | - | 소속 조직 노드 ID |
| `role_code` | VARCHAR(30) | NO | - | `ROLE_SUPER_ADMIN`, `ROLE_AUDITOR`, `ROLE_OPS_ADMIN`, `ROLE_QA_LEAD`, `ROLE_AGENT` |
| `ip_policy_type`| VARCHAR(20) | NO | 'EXACT' | IP 검증 정책 (`EXACT`: 단일 IP 완전 일치, `CIDR`: 서브넷 대역) |
| `account_status`| VARCHAR(20) | NO | 'ACTIVE'| 상태 (`ACTIVE`, `LOCKED`, `EXPIRED`) |
| `fail_count` | INT | NO | 0 | 로그인 실패 횟수 (5회 실패 시 잠김) |

#### tb_sys_user_ip (허용 접속 IP 목록 - 1:N 매핑)
| 컬럼명 | 데이터 타입 | Nullable | 기본값 | 설명 |
|:---|:---|:---:|:---:|:---|
| `seq` | BIGINT AUTO | NO | PK | 순번 |
| `user_id` | VARCHAR(32) | NO | FK | 사용자 ID |
| `allowed_ip` | VARCHAR(45) | NO | - | 단일 IP 주소 또는 CIDR 블록 (예: 10.240.40.100 또는 10.240.10.0/24) |
| `description` | VARCHAR(100)| YES| - | IP 설명 (예: IDC 관리 콘솔 전용선) |

---

### 2.4 불변 감사 로그 테이블 (`tb_sys_audit_log`)
* **설명**: 모든 핵심 인터랙션(청취, 다운로드, 감청, 인증 실패 등)을 보존하는 WORM 테이블이다.

| 컬럼명 | 데이터 타입 | Nullable | 설명 |
|:---|:---|:---:|:---|
| `log_id` | BIGINT AUTO | NO (PK) | 순차 증가 감사 로그 ID |
| `event_time` | DATETIME | NO | 이벤트 발생 타임스탬프 |
| `event_type` | VARCHAR(40) | NO | `LOGIN_SUCCESS`, `UNAUTHORIZED_IP_ATTEMPT`, `PLAYBACK`, `EXPORT_REQUEST`, `EXPORT_APPROVED`, `EXPORT_DOWNLOAD`, `WIRETAP_START`, `UNMASK_REQUEST` |
| `actor_id` | VARCHAR(32) | NO | 행위자 ID (미인증 시 'ANONYMOUS') |
| `actor_ip` | VARCHAR(45) | NO | 접속 클라이언트 실제 IP (Exact Match 대조값) |
| `target_id` | VARCHAR(64) | YES | 대상 통화 ID 또는 반출 신청 번호 |
| `reason_code` | VARCHAR(50) | YES | 사유 코드 (`MINWON`, `LEGAL`, `QA_COACHING`, `AUDIT` 등) |
| `reason_detail`| TEXT | YES | 관리자가 직접 입력한 상세 근거 사유 |
| `result_status`| VARCHAR(10) | NO | `SUCCESS`, `FAIL`, `REJECTED` |

---

## 3. 표준 RESTful API 입출력 규격

### 3.1 공통 응답 봉투 규격 (Standard Envelope)

```json
{
  "success": true,
  "code": "OK_200",
  "message": "요청이 정상 처리되었습니다.",
  "data": {}
}

에러 응답시 :
{
  "success": false,
  "code": "AUTH_401_IP_MISMATCH",
  "message": "등록된 단일 관리자 IP와 일치하지 않아 접근이 차단되었습니다.",
  "data": null
}

```

### 3.2