---
document_id: "SPEC-CORE-P1-001"
title: "기본 녹취 솔루션 관리자 콘솔 요구사항 정의 및 거버넌스 정책서"
version: "1.2.0"
stage: "Phase 1 - Approved"
last_updated: "2026-09-17"
---

# Phase 1. 요구사항 정의 & 기본 정책 수립 (Core Recording Web Admin)

## 1. 아키텍처 원칙
* **엔진 독립성**: 특정 CTI/SIP 드라이버 및 STT 하부 연동 종속성을 분리한 독립 코어로 설계.
* **거버넌스 옵션화**: 청취 사유, 결재 단계, 감청 사유 등의 통제 여부를 시스템 설정에서 ON/OFF 토글 지원.
* **무제한 확장성**: CRM 조직 승계를 바탕으로 무제한 Depth 계층형 조직 지원.

## 2. 사용자 역할 (RBAC)
* ROLE_SUPER_ADMIN (슈퍼 관리자)
* ROLE_AUDITOR (보안 감사관)
* ROLE_OPS_ADMIN (센터장 / 운영 총괄)
* ROLE_QA_LEAD (팀장 / QA 관리자)
* ROLE_AGENT (일반 상담사)

## 3. 네트워크 접근 보안 (IP 화이트리스트)
* **상위 관리자 (Super Admin / Auditor / Ops Admin)**: CIDR 대역 불가, 사전 등록된 단일 IP 100% 완전 일치(Exact Match) 시에만 접근 허용.
* **일반 권한자 (QA Lead / Agent)**: 지정된 사내 인트라넷 서브넷 대역(CIDR) 허용.

## 4. 미디어 통제 & 마스킹 정책
* **청취 (Playback)**: 사유 입력 팝업 없이 원클릭 스트리밍 (감사 로그 자동 기록).
* **다운로드 (Export)**: 사유 강제 입력 필수 (설정에 따라 관리자 승인 단계 연계).
* **마스킹**: 정부 공식 비식별화 가이드라인 100% 준수 (전화번호, 성명, 주민번호, 카드번호 등).
