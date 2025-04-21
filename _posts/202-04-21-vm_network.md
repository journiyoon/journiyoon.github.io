---
title: " VirtualBox NAT/Host-Only 환경에서 VM 간 접속"
excerpt: "VM 간 특이한 IP 인식 현상을 위한 실습과 분석 정리"
categories:
  - Network
tags:
  - VM
toc: true
toc_sticky: true
last_modified_at: 2025-04-21
---

## 문제 상황

- 1대의 Physical Host 위에 2개의 VirtualBox VM 구동 중
-  VM은 NAT + Host-Only 인터페이스를 동시에 사용

#### VM 네트워크 구성

|VM	|Host-Only IP	|NAT IP 대역|
|:------|:----|:------|
|client-vm|	192.168.56.11	|10.0.2.0/24|
|db-vm|	192.168.57.11|	10.0.2.0/24|




- MySQL 서버는 db-vm에 설치되어 있으며, user는 `web@192.168.56.11` 로 생성 완료
- client-vm에서 `mysql -u web -p -h 192.168.57.11` 명령어로 접속 시도
- ❌ “Access denied for user ‘web’@‘192.168.57.1’” 에러 발생

---

### ❓ 왜 client-vm은 192.168.56.11인데, MySQL 서버에서는 192.168.57.1로 보일까?

이 지점에서 VirtualBox의 네트워크 동작 방식과 라우팅에 대한 이해가 필요합니다.

> ####  의심 포인트
> 1.	라우팅 테이블에 따라 어떤 인터페이스로 나갈지 결정됨
> 2.	Host-Only 네트워크는 서로 다른 대역 간 직접 통신이 불가
> 3.	NAT를 타고 나가면 VirtualBox는 자신의 가상 라우터 IP로 Source NAT를 수행

---

##  실제 확인 절차 (Rocky Linux 기준)

### 1. db-vm에서 패킷 캡처

```bash
sudo dnf install -y tcpdump
sudo tcpdump -i any port 3306 -nn
```

이후 client-vm에서 접속 시도:

```bash
mysql -u web -p -h 192.168.57.11
```

##### 결과 예시:

```bash
IP 192.168.57.1.54321 > 192.168.57.11.3306: Flags [S]...
```

👉 MySQL 서버는 192.168.57.1이 접속을 시도했다고 인식

---

## 2. client-vm에서 라우팅 테이블 확인

```bash
ip route
```

예시 출력:

```bash
default via 10.0.2.2 dev eth0
192.168.56.0/24 dev eth1 proto kernel scope link src 192.168.56.11
```

❗ 192.168.57.0/24 대역이 없음 → **기본 경로(NAT)를 통해 접속됨**

---

## 3. MySQL 로그 확인 (db-vm)

```bash
sudo grep 'Access denied' /var/log/mysqld.log
```

출력 예시:

```bash
Access denied for user 'web'@'192.168.57.1' (using password: YES)
```

MySQL은 192.168.57.1에서 접속한 것으로 오인함

---
 
## 정리: 왜 이런 일이 벌어졌는가?

|항목|	설명|
|:----|:----|
|예상|	client-vm (192.168.56.11) → db-vm (192.168.57.11) 직접 접속|
|실제|	client-vm이 라우팅 테이블상 NAT로 우회 → VirtualBox가 NAT 주소인 192.168.57.1로 변환 후 전달|
|결과|	MySQL 서버에서는 접속 IP가 192.168.56.11이 아니라 192.168.57.1로 보임|

---

## 해결 방향 (선택 사항)

```bash
# client-vm에서 명시적으로 라우팅 추가
sudo ip route add 192.168.57.0/24 via 192.168.56.1 dev eth1
```

또는 db-vm에서 다음처럼 user를 재생성할 수도 있습니다:

```sql
CREATE USER 'web'@'192.168.57.1' IDENTIFIED BY 'qwer';
```
⸻

## 마무리

이 문제는 단순한 사용자 설정 이슈처럼 보일 수 있지만, 실제로는 가상화 환경의 네트워크 구조와 라우팅 정책, 그리고 NAT의 동작 방식을 이해해야만 명확히 설명할 수 있는 문제입니다.

**패킷 캡처(tcpdump)**, **라우팅 확인(ip route)**, **로그 분석(mysqld.log)** 3가지를 조합하면 대부분의 네트워크 트러블슈팅이 가능해집니다.