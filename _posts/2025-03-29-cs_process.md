---
title: "리눅스 - 프로세스 파헤치기"
excerpt: "프로세스 개념을 이해하고 실무까지 연결해봅니다"
categories:
  - Linux
tags:
  - Process
toc: true
toc_sticky: true
last_modified_at: 2025-03-29
---

## 1️⃣ 리눅스에서 프로세스란?

디스크에 있는 실행 파일이 메모리에 올라가 실행 중인 상태.

하나의 프로세스는 다음 정보들을 갖습니다:
- 고유한 PID (Process ID)
- 독립된 메모리 공간 (코드, 데이터, 힙, 스택)
- 실행 상태 (Running, Sleeping)
- 부모 프로세스 관계 (PPID)
- 열고 있는 파일, 파이프 등의 리소스

## 2️⃣ 리눅스 프로세스 메모리 구조

리눅스에서 하나의 프로세스는 아래 메모리 구조를 갖습니다:

```text
+-----------------+  높은 주소
|   Stack         |  함수 호출, 지역 변수
+-----------------+
|   Heap          |  malloc, new 로 할당된 동적 메모리
+-----------------+
|   BSS Segment   |  초기화되지 않은 전역/정적 변수
+-----------------+
|   Data Segment  |  초기화된 전역/정적 변수
+-----------------+
|   Text Segment  |  코드 (실행 명령어)
+-----------------+  낮은 주소
```
🔍 cat /proc/[pid]/maps 명령으로 확인 가능

## 3️⃣ 프로세스 생성과 실행 흐름

1. fork(): 부모 프로세스를 복제 -> 자식 프로세스 생성
2. exec(): 자식 프로세스의 메모리를 새로운 실행파일로 덮어씀
3. wait(): 부모가 자식의 종료를 기다림

### 예시

```C
pid_t pid = fork();
if (pid == 0) {
    // 자식 프로세스
    execl("/bin/ls", "ls", NULL);
} else {
    // 부모 프로세스
    wait(NULL);
}
```
💡 fork 이후 부모/자식은 서로 완전히 다른 메모리 공간을 가짐   
(단, 실제론 COW – Copy-on-write로 효율 처리)

## 4️⃣ 프로세스 상태 (ps 명령어에서 보는 것)

- R (Running): 실행 중 또는 실행 대기 중
- S (Sleeping): 대기 상태 (I/O 대기 등)
- D (Uninterruptible sleep): 종료 불가 상태 (디스크 I/O 대기 중 등)
- Z (Zombie): 종료됐지만 부모가 wait() 안 해서 남아있는 상태
- T (Stopped): 시그널로 멈춘 상태 (Ctrl+Z)

🔍 ps -eo pid,ppid,state,cmd로 확인 가능

## 5️⃣ 프로세스 간 통신 (IPC)

- 파이프 (pipe, FIFO): 간단한 데이터 흐름
- 메시지 큐 (msgget, msgsnd): 커널이 관리하는 큐
- 공유 메모리 (shmget, shmat): 메모리 직접 공유
- 세마포어 (semget, semop): 동기화 및 잠금

이런 IPC는 `System V`, `POSIX` 방식으로 나뉘며, `ipcs`, `ipcrm` 등으로 관리

## 6️⃣ 프로세스 관련 주요 명령어

| 명령어              | 설명            |
|:-----------------|:--------------|
| `ps aux`         | 현재 모든 프로세스 보기 |
| `top`, `htopw`   | 실시간 리소스 사용 확인 |
| `kill PID`       | 프로세스 종료       |
| `nice`, `renice` | 프로세스 우선순위 조정  |
| `strace`         | 시스템 호출 추적     |
| `lsof -p PID`    | 프로세스가 연 파일 확인 |
| `pmap PID`        | 메모리 앱 확인      |

## 7️⃣ /proc 가상 파일 시스템

- /proc/[pid]/cmdline : 실행 명령
- /proc/[pid]/status : 메모리, 상태, 부모 PID 등
- /proc/[pid]/fd/ : 열린 파일 디스크립터 목록
- /proc/[pid]/stat : 스케줄링 관련 정보
- /proc/meminfo, /proc/cpuinfo 등도 시스템 상태 파악에 사용
