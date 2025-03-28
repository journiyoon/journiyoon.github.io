---
title: "passwd파일 속 nobody의 존재는?"
excerpt: "nobody의 목적과 존재 이유에 대해 알아봅니다"
categories:
  - Linux
tags:
  - permission
toc: true
toc_sticky: true
last_modified_at: 2025-03-28
---

`/etc/passwd` 파일에는 다양한 사용자 정보가 있습니다. 
그 중 `nobody`는 이름부터 어딘가 특이해 보입니다. 
호기심이 생겨 찾아본 김에, 간단히 정리하겠습니다.

## ✅ nobody의 목적과 존재 이유

### 1. 최소 권한 사용자 (Least Privilege)

`nobody`는 시스템 상에서 **어떠한 권한도 거의 없는 사용자**입니다. 
홈 디렉터리도 없고, 소유한 파일도 없으며, 일반적인 쉘 접근도 불가능합니다.
> 즉, 시스템에서 꼭 실행되긴 해야 하지만, 절대로 중요한 리소스나 권한을 주면 안 되는 프로세스에 이 계정을 사용합니다.

### 2. 비신뢰 프로세스 격리용

예를 들면, 어떤 네트워크 서비스를 실행하는데, 외부 사용자로부터 요청을 받아야 한다면, 혹시 모를 취약점에 대비해서 `nobody` 사용자로 실행하게 합니다. 
이렇게 하면 해당 서비스가 침해되더라도, 시스템 전체에 영향을 줄 수 있는 권한이 없기 때문에 **피해를 최소화**할 수 있습니다.

### 3. 권한 없는 기본 계정

시스템이나 데몬이 권한 검사를 할 때, 마땅한 소유자를 찾지 못할 경우 `nobody`로 대체되기도 합니다. 
예를 들면, NFS 공유 설정에서 `all_squash` 옵션을 사용하면, 모든 접근을 `nobody`로 매핑합니다.

## 📁 예시로 보는 /etc/passwd 속 nobody

```bash
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:Kernel Overflow User:/:/sbin/nologin
```
- UID: 65534 (보통 가장 낮거나 사용되지 않는 UID 중 하나)
- GID: 65534 (같은 이름의 그룹 nobody)
- 홈 디렉터리: 없음
- 셸: `/sbin/nologin` -> 로그인 불가

## 🔐 실무에서는?

- Apache, Nginx 같은 웹서버가 `nobody`로 실행되는 경우도 있었지만, 요즘은 더 세분화된 전용 사용를 따로 만듭니다.
- Docker 컨테이너에서도 UID 65534 (`nobody`)를 쓰는 경우가 많습니다.
