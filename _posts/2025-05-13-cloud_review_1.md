---
title: "AWS 복습2 "
excerpt: ""
categories:
  - AWS
tags:
  - VPC
  - EC2
toc: true
toc_sticky: true
last_modified_at: 2025-05-13
---

## ALB → 프라이빗 EC2 간 패킷 포워딩 경로 (“Local” 라우팅)

퍼블릭 서브넷의 ALB가 프라이빗 서브넷 EC2로 요청을 전달할 때, NAT Gateway나 IGW를 거치지 않고 VPC 내부의 **로컬 라우팅**(`10.0.0.0/16 → local`)을 사용합니다.

1. 클라이언트 요청이 ALB 노드(퍼블릭 IP)로 도착
2. ALB가 **리버스 프록시**처럼 퍼블릭 요청을 종료(terminate)
3. ALB가 **새로운 커넥션**을 프라이빗 EC2의 사설 IP로 생성
4. 해당 패킷은 라우트 테이블의 `10.0.0.0/16 → local` 규칙에 따라 바로 전달

이 덕분에 프라이빗 인스턴스는 외부 노출 없이도 ALB를 통해 인바운드를 처리할 수 있으며, 퍼블릭 경로와 분리된 안전한 통신이 보장됩니다.

---

## Bastion Host 접속과 ProxyJump/ProxyCommand 설정

프라이빗 서브넷 EC2에 SSH로 접속할 때는 Bastion Host(점프 서버)를 경유하도록 설정하는데, SSH 설정 파일(`~/.ssh/config`)에 다음과 같이 작성하면 편리합니다.

### ProxyJump 예시 (SSH 7.3 이상 권장)

```ssh-config
Host bastion
    HostName <퍼블릭-IP 또는 DNS>
    User ec2-user
    IdentityFile ~/.ssh/bastion-key.pem

Host private-web
    HostName 10.0.x.y
    User ec2-user
    IdentityFile ~/.ssh/private-key.pem
    ProxyJump bastion
```

* `ssh private-web` 한 번으로 Bastion → 프라이빗 EC2 자동 연결

### ProxyCommand 예시 (SSH 버전 관계없이)

```ssh-config
Host private-web
    HostName 10.0.x.y
    User ec2-user
    IdentityFile ~/.ssh/private-key.pem
    ProxyCommand ssh -W %h:%p bastion
```

* `-W %h:%p` 옵션이 Bastion에서 최종 호스트(%h)와 포트(%p)로 터널링
* `bastion` 블록을 참고해 키·유저 정보를 자동 로드

---

## 서브넷별 라우트 테이블 구성 이유 (퍼블릭 vs 프라이빗)

### 퍼블릭 서브넷

* **라우팅**: `0.0.0.0/0 → IGW`
* **목적**: 인터넷-Facing ALB, Bastion, NAT Gateway와 같은 퍼블릭 리소스에 외부 트래픽 허용

### 프라이빗 서브넷

* **라우팅**: `0.0.0.0/0 → NAT Gateway`
* **로컬**: `10.0.0.0/16 → local` (ALB 포워딩, VPC 내부 통신)
* **목적**:

  1. 프라이빗 EC2가 퍼블릭 IP 없이 아웃바운드 인터넷 통신
  2. 인바운드는 오직 ALB(VPC 로컬 경로)나 Bastion을 통해서만 허용

서브넷을 명확히 구분해 라우트 테이블을 분리함으로써, 각 계층의 역할과 보안 경계를 확실히 유지할 수 있습니다.

---

## AZ별 NAT Gateway 분리의 고가용성·비용 최적화 목적

1. **AZ 장애 격리**

   * 각 AZ에 별도 NAT Gateway를 배치
   * 특정 AZ NAT GW 장애 시에도 다른 AZ의 프라이빗 서브넷은 정상 아웃바운드 유지

2. **크로스 AZ 데이터 전송 비용 절감**

   * 프라이빗 EC2는 **동일 AZ** NAT GW를 경유
   * 타 AZ NAT GW 사용 시 발생하는 데이터 전송 요금 방지

3. **단일 실패 지점 제거**

   * 하나의 NAT GW에 전체 VPC가 의존하지 않아, 가용성 및 복원력 강화
