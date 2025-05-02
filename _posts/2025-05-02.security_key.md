---
title: "SSH 보안 및 호스트키 검증"
excerpt: "원격 서버에 SSH로 접속할 때, 클라이언트와 서버 간에 일어나는 통신을 알아봅니다"
categories:
  - Security
tags:
  - SSH
  - Public key
  - Private key
toc: true
toc_sticky: true
last_modified_at: 2025-05-02
---

원격 서버에 SSH로 안전하게 접속하기 위해 클라이언트는 서버가 '진짜 내가 접속하려한 서버가 맞는지'를 확인해야 합니다. SSH는 비대칭 암호화와 호스트키 검증을 통해 이 과정을 자동화합니다.

## 공개키 & 개인키

### 비대칭 암호화 방식

- 개인키(private key): 본인맞 갖고 있는 비밀키(복호화에 사용)
- 공개키(public key): 상대방에게 공개해도 되는 키(암호화에 사용)

### 인증 흐름

1. 서버에 사용자(클라이언트)의 공개키를 등록(ssh-copy-id)
2. 접속 시 서버가 메시지를 공개키(1번에서 클라이언트가 전달한 공개키)로 암호화해 클라이언트에 전송
3. 클라이언트가 개인키로 복호화해 서버에 응답
4. 서버는 원본 메시지와 클라이언트의 응답을 비교해 일치하면 인증 완료

## 서버 호스트키

사용자가 `ssh-keygen`으로 만든 사용자키가 아닌 서버에서 생성된 키를 의미합니다. 
주로 SSH 서버를 처음 설치하거나, OS 설치 시점에 **자동**으로 생성됩니다.   
만약 키 파일이 없거나 재생성하고 싶다면 수동으로 `ssh-keygen`을 이용할 수 있습니다.

- 역할: "진짜 서버가 맞는지"를 클라이언트에 증명
- 키 종류: RSA, ECDSA, ED25519 등
- 용도를 구분해보면:
  - 호스트키: SSH 서버 자체 인증용
  - 사용자키: SSH 로그인(사용자 인증)용

### 호스트키 저장 위치

대부분의 리눅스 배포판에서는 호스트키를 `/etc/ssh/` 아래에 두고 관리합니다.

```bash
/etc/ssh/ssh_host_rsa_key           # 호스트키(비밀키)
/etc/ssh/ssh_host_rsa_key.pub       # 호스트키(공개키)
  
/etc/ssh/ssh_host_ecdsa_key
/etc/ssh/ssh_host_ecdsa_key.pub

/etc/ssh/ssh_host_ed25519_key
/etc/ssh/ssh_host_ed25519_key.pub
```

### fingerprint

- 긴 공개키를 사람이 보기 쉽게 **짧은 해시값**으로 표현
- 중간자 공격 여부를 빠르게 검증할 수 있음

### 호스트키 fingerprint 확인 방법

공개키 파일에서 직접 해시를 뽑아볼 수 있습니다. 예를 들어 RSA 키 fingerprint를 보려면:

```bash
ssh-keygen -l -f /etc/ssh/ssh_host_rsa_key.pub
```

- `-l`: fingerprint 출력
- `-f`: 지문을 확인할 공개키 파일 지정

## SSH 최초 접속 시 검증 과정

1. 클라이언트에 `~/.ssh/known_hosts` 파일이 없거나
2. `known_hosts`에 해당 서버의 fingerprint가 없으면
3. 다음과 같이 프롬프트를 띄워 사용자에게 확인 요청
  ```bash
  The authenticity of host 'example.com (203.0.113.5)' can't be established.
  ECDSA key fingerprint is SHA256:AbCdEfGhIjKlMnOpQrStUvWxYz0123456789+/=.
  Are you sure you want to continue connecting (yes/no/[fingerprint])?
  ```
  - `yes`: fingerprint를 `known_hosts`에 저장 -> 다음부터 묻지 않음
  - `no`: 접속 중단
  - `[fingerprint]`: 저장 없이 이번 한 번만 신뢰 -> 다음에 다시 묻게 됨


## 결론

- **서버 호스트키**는 서버 신원을 검증하기 위한 키 쌍으로 `/etc/ssh`에 자동 생성됩니다.
- **fingerprint**는 공개키를 사람이 확인하기 편한 해시값으로 압축한 것으로, 중간자 공격 방어의 핵심입니다.
- SSH 최초 접속 시 나오는 "yes/no/[fingerprint]"프롬프트는 클라이언트가 "이 서버를 믿어도 되나요?" 를 묻는 절차입니다.