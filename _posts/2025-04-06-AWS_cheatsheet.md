---
title: "AWS SAA 시험 대비 핵심 Cheatsheet"
excerpt: ""
categories:
  - AWS
toc: true
toc_sticky: true
last_modified_at: 2025-04-06
---

# 🧠 AWS SAA 시험 대비 핵심 Cheatsheet

> 📌 AWS Certified Solutions Architect – Associate 시험 준비용 핵심 개념 요약 정리입니다. 자주 출제되는 서비스, 개념, 제한 사항 위주로 정리했습니다.

---

## 📦 EC2 (Elastic Compute Cloud)

- 인스턴스 타입: `t2.micro`, `m5.large` 등 – 목적에 따라 선택
- 스토리지:
  - EBS (Elastic Block Store): 영구 저장, AZ 내에서만 사용 가능
  - Instance Store: 일시적, 인스턴스 종료 시 데이터 삭제
- 구매 옵션:
  - On-Demand: 일반적인 사용, 시간 단위 과금
  - Spot: 최대 90% 저렴, 갑작스런 종료 가능
  - Reserved: 1년/3년 약정, 할인 적용
- 보안:
  - 보안 그룹(SG): Stateful
  - NACL: Stateless, 서브넷 단위로 적용

---

## ☁️ S3 (Simple Storage Service)

- 객체 스토리지, 무제한 저장
- 스토리지 클래스:
  - Standard / IA / One Zone-IA / Glacier / Glacier Deep Archive
- 버전 관리 지원 (버전별로 복원 가능)
- 정적 웹 호스팅 가능
- S3 정책:
  - 버킷 정책, IAM 정책, ACL
- 암호화:
  - SSE-S3 / SSE-KMS / SSE-C / Client-side

---

## 🛡️ IAM (Identity and Access Management)

- 리소스 접근 제어
- 구성요소:
  - 사용자 / 그룹 / 역할(Role) / 정책(Policy)
- 정책 작성: JSON 형식
- **Least Privilege Principle** 적용
- MFA(다단계 인증) 권장

---

## 🌐 VPC (Virtual Private Cloud)

- AWS에서 네트워크를 사용자 정의
- 구성요소:
  - 서브넷(Public/Private)
  - 인터넷 게이트웨이(IGW)
  - NAT Gateway (Private Subnet에서 인터넷 접근)
  - 라우팅 테이블
  - NACL, 보안 그룹
- Peering: VPC 간 통신 (cross-region 가능, transitive 불가)

---

## 📈 CloudWatch

- 모니터링 도구
- 기능:
  - 메트릭 수집 (CPU, RAM 등)
  - 알람 설정
  - 로그 수집
- 커스텀 메트릭 가능 (CLI/SDK 활용)

---

## 📜 CloudTrail

- 계정 내 API 호출 기록 추적
- 보안 감사, 문제 해결에 유용
- S3에 로그 저장 가능

---

## ⚙️ Auto Scaling & ELB

### Auto Scaling
- EC2 인스턴스를 자동으로 늘리거나 줄임
- 조건: CPU 사용률, 네트워크 트래픽 등

### ELB (Elastic Load Balancer)
- 트래픽 분산
- 종류:
  - ALB: HTTP/HTTPS (Layer 7)
  - NLB: TCP/UDP (Layer 4)
  - CLB: 구버전

---

## 💾 RDS (Relational Database Service)

- 관리형 관계형 DB
- 지원 DB: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server
- 백업 자동화, 멀티 AZ 구성 가능
- 리전 간 복제: Read Replica 사용
- 암호화: KMS 기반

---

## 🛰️ Route 53

- 도메인 이름 서비스(DNS)
- 레코드 타입: A, AAAA, CNAME, MX 등
- 헬스 체크 기능 포함
- 라우팅 정책:
  - Simple / Weighted / Latency / Failover / Geolocation

---

## 🏗️ Well-Architected Framework 핵심 원칙

1. **운영 우수성 (Operational Excellence)**
2. **보안 (Security)**
3. **신뢰성 (Reliability)**
4. **성능 효율성 (Performance Efficiency)**
5. **비용 최적화 (Cost Optimization)**

---

## 🧪 시험 대비 팁

- 문제는 **시나리오 기반**이 많음 (정답 2~3개 중 최적 선택)
- 핵심 키워드 캐치:
  - "비용 최소화" → Spot, S3 IA, Lambda 등
  - "고가용성" → Multi-AZ, ASG, ELB
  - "보안 강화" → IAM 역할, VPC 보안 그룹, KMS
  - "운영 간편성" → RDS, Elastic Beanstalk, Fargate

---

> ✍️ 이 요약본은 학습용으로 지속 업데이트 예정입니다. 각 항목에 대해 실제 실습을 통해 감을 익히는 것이 가장 중요합니다.