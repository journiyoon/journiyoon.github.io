---
title: "리눅스 사용자 계정 충돌 및 잔여 데이터 정리 트러블슈팅"
excerpt: "리눅스 실습 도중 uid 충돌로 인한 해결방법을 알아봅니다"
categories:
  - Linux
tags:
  - Linux_User
toc: true
toc_sticky: true
last_modified_at: 2025-03-31
---

리눅스 시스템에서 사용자 계정 관리 중 **UID 충돌, 사용자 삭제 후 남은 파일, sudo 인증 캐시, 고아 UID 파일**과 같은 문제를 겪을 수 있습니다. 
실제 겪은 사례를 바탕으로 문제 발생부터 해결까지 정리해보겠습니다.

## ✅ 문제 1: 사용자 UID 충돌
### 🔍 증상

`/etc/passwd` 파일을 확인했을 때 두 사용자 (`vagrant`, `test`) 이 **동일한 UID(1000)**를 갖고 있었습니다.
```bash
vagrant:x:1000:1000::/home/vagrant:/bin/bash
test1:x:1000:1001::/home/test1:/bin/bash
```
- id test1 입력 시, vagrant 사용자 정보가 출력됨
- usermod, userdel 명령이 오류를 내며 작동하지 않음

### 📌 원인

- 리눅스는 UID를 기준으로 사용자를 식별하기 때문에, UID가 중복되면 시스템이 두 사용자를 사실상 한 사용자로 인식함

### 🛠 해결

1. **root 또는 다른 사용자(admin 계정)** 로 로그인 (중요!!!정리하려는 사용자의 프로세스가 열려있으면 절대 해결되지 않음)
2. `UID` 충돌 사용자 프로세스 종료
   ```bash
   sudo pkill -u 1000
   ```
3. `test1` UID 변경
   - `1501` uid는 임의의 것, 사용중이지 않은 uid를 선택해야함
      ```bash
      sudo usermod -u 1501 test1
      sudo groupmod -g 1501 test1
      ```
4. 홈 디렉터리 권한 정리
   ```bash
   sudo chown -R test1:test1 /home/test1
   ```
---

## ✅ 문제 2: 사용자 삭제 후 남아있는 홈 디렉터리
### 🔍 증상

`userdel test1` 후에도 `/home/test1`, `/var/spool/mail/test1` 등의 디렉터리와 파일이 남아있음.

### 📌 원인

- `userdel` 기본 옵션은 홈 디렉터리를 삭제하지 않음
- `userdel -r 사용자명` 옵션을 사용해야 디렉터리도 함께 삭제됨

### 🛠 해결

- 불필요하면 직접 삭제:
  ```bash
  sudo rm -rf /home/test1 /var/spool/mail/test1
  ```
- 혹은 재사용할 경우 UID/GID 맞춰서 소유권 이전:
  ```bash
  sudo chown -R vagrant:vagrant /home/test1
  ```
---

## ✅ 문제 3: 고아 UID/GID 파일 존재
### 🔍 증상

`find / -nouser -o -nogroup` 명령으로 다음과 같은 결과 출력:
```bash
/var/spool/mail/user01
/run/sudo/ts/user01
```

### 📌 원인
- 계정은 삭제됐지만, 파일이나 인증 캐시가 남아 있음
- 이 상태로 두면 UID 재사용 시 **보안 문제** 발생 가능

### 🛠 해결

- 필요 없는 파일은 정리:
```bash
sudo rm -f /run/sudo/ts/user01 /var/spool/mail/user01
```
- 보존이 필요하다면 소유권 변경:
```bash
sudo chown root:root /var/spool/mail/user01
```

## 정리

- 사용자 삭제 시에는 항상 `-r` 옵션 사용(필요의 경우를 확인)
- UID는 중복되지 않게 관리 (`/etc/passwd`, `/etc/group` 점검)
- 주기적으로 `find / -nouser -o -nogroup` 명령으로 시스템 청소