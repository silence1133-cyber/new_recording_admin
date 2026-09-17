---
document_id: "SPEC-CORE-P2-003"
title: "기본 녹취 솔루션 화면 목록 및 라우팅 체계 (Sitemap)"
version: "1.0.0"
stage: "Phase 2 - Approved"
last_updated: "2026-09-17"
---

# Phase 2. 화면 목록 및 라우팅 체계 (Sitemap Specification)

## 1. 화면 ID 명명 규칙
* 도메인 Prefix: AUTH(인증), OPS(운영관제), REC(녹취이력), QA(품질평가), SYS(시스템/보안)
* 본체 화면: [Prefix]-[순번] (예: REC-01)
* 팝업/모달: [Prefix]-[순번]-M[순번] (예: REC-02-M1)

## 2. 전체 화면 및 라우팅 목록표
| 화면 ID | GNB | LNB | 세부 화면명 | UI 타입 | 라우팅 경로 | 기본 접근 역할 |
|---|---|---|---|:---:|---|---|
| AUTH-01 | - | - | 로그인 (IP 검증) | Page | /login | ALL |
| AUTH-02 | - | - | 2차 인증 (OTP) | Modal | /login/mfa | Super Admin, Auditor |
| OPS-01 | 1. 실시간 모니터링 | 1.1 운영 대시보드 | 가동율 및 자원 현황 | Page | /ops/dashboard | Super, Auditor, Ops, QA |
| OPS-02 | 1. 실시간 모니터링 | 1.2 채널 관제 | 채널/내선 관제 그리드 | Page | /ops/channels | Super, Ops, QA |
| OPS-02-M1| 1. 실시간 모니터링 | 1.2 채널 관제 | 실시간 감청 사유 모달 | Modal | - | Super, Ops, QA |
| REC-01 | 2. 녹취 이력 관리 | 2.1 녹취 이력 조회 | 다차원 검색 및 마스킹 그리드 | Page | /recordings/search | ALL (역할별 필터) |
| REC-01-P1| 2. 녹취 이력 관리 | 2.1 녹취 이력 조회 | HTML5 파형 오디오 플레이어 | Drawer | - | ALL |
| REC-01-M1| 2. 녹취 이력 관리 | 2.1 녹취 이력 조회 | 마스킹 해제 신청 | Modal | - | Super, Ops |
| REC-02 | 2. 녹취 이력 관리 | 2.2 반출/다운로드 | 반출 신청/결재/다운로드 큐 | Page | /recordings/exports | Super, Auditor, Ops, QA |
| REC-02-M1| 2. 녹취 이력 관리 | 2.2 반출/다운로드 | 반출 사유 필수 입력 | Modal | - | Super, Ops, QA |
| REC-02-M2| 2. 녹취 이력 관리 | 2.2 반출/다운로드 | 반출 승인 결재 팝업 | Modal | - | Super, Auditor, Ops |
| REC-03 | 2. 녹취 이력 관리 | 2.3 보존 및 파기 | Legal Hold 및 파기 관리 | Page | /recordings/lifecycle | Super, Auditor, Ops |
| QA-01 | 3. QA 품질 평가 | 3.1 평가표 관리 | 평가 시트 템플릿 설정 | Page | /qa/templates | Super, Ops, QA |
| QA-02 | 3. QA 품질 평가 | 3.2 코칭 & 피드백 | 구간별 피드백 관리 | Page | /qa/coaching | Super, Ops, QA, Agent |
| QA-03 | 3. QA 품질 평가 | 3.3 QA 리포트 | 품질 통계 리포트 | Page | /qa/reports | Super, Auditor, Ops, QA, Agent |
| SYS-01 | 4. 시스템 & 보안 관리 | 4.1 조직 및 내선 | 무제한 조직도 (CRM 승계) | Page | /system/organizations | Super, Ops(조회) |
| SYS-01-M1| 4. 시스템 & 보안 관리 | 4.1 조직 및 내선 | 내선 매핑 설정 팝업 | Modal | - | Super |
| SYS-02 | 4. 시스템 & 보안 관리 | 4.2 사용자 및 권한 | 계정 & 접속 IP 통제 | Page | /system/users | Super |
| SYS-03 | 4. 시스템 & 보안 관리 | 4.3 감사 로그 | 불변 감사 로그 조회 | Page | /system/audit-logs | Super, Auditor |
| SYS-04 | 4. 시스템 & 보안 관리 | 4.4 정책 & 스토리지 | 옵션 토글 & 보관 주기 | Page | /system/policies | Super, Auditor |
