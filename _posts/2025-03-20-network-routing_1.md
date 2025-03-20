---
title: "라우팅 뿌시기 - 1"
excerpt: "패킷 트레이서 실습으로 라우팅 프로토콜을 이해해봅니다."
categories:
  - Network
tags:
  - Routing Protocol
toc: true
toc_sticky: true
last_modified_at: 2025-03-20
---

![img.png](img.png)
네트워크 토폴리지

## RIP 구성 단계

### 1) 라우터 특권 EXEC 모드 진입

```bash
Router3> enable
Router3#
```

### 2) 글로벌 설정 모드로 진입

```bash
Router# configure terminal
Router(config)#
```

### 3) RIP 프로토콜 활성화 

```bash
Router3(config)# router rip
Router3(config-router)#
```

### 4) RIP 버전 지정

디폴트 RIP는 버전 2이므로, RIP 버전 2를 명시적으로 지정해야 합니다. 
```bash
Router3(config-router)# version 2
```

### 5) 네트워크 지정

RIP 라우팅을 적용할 네트워크를 등록합니다. 여기서는 Router 3을 기준으로 합니다. 
```bash
Router3(config-router)# network 10.128.0.0
Router3(config-router)# network 192.168.4.0
```

### 6) 클래스풀 자동 요약 해제

서브넷 마스크가 클래스풀 경계를 벗어나거나(예: /9) 네트워크가 여러 개로 분산되어 있으면, 자동 요약을 끄는 것이 좋습니다.
```bash
Router3(config-router)# no auto-summary
```

### 7) 설정 종료 및 저장

```bash
Router3(config-router)# end
Router3#
Router3# copy running-config startup-config
```
이렇게 하면 현재 동작 중인 설정이 라우터 부팅 시에도 적용되도록 저장됩니다.

## 참고 사항

1. RIP 버전 2를 사용해도, 디플트로는 클래스풀 기반 요약이 동작하므로 `no auto-summary`를 반드시 사용해야 합니다.
2. 설정 후 show ip route, show ip protocols 명령을 통해 RIP 라우팅 정보가 정상적으로 학습되고 있는지 확인합니다.
3. VLSM(가변 길이 서브넷 마스크)를 지원하는 버전 2를 사용합니다.
4. `copy running-config startup-config`를 사용하여 현재 실행 중인 설정을 비휘발성 메모리인 NVRAM 에 저장하여, 라우터가 재부팅되더라도 해당 설정이 유지되되록 하는 명령어 

## auto-summary

RIPv2 표준은 VLSM 을 지원하지만, Cisco IOS 구버전에서 'auto-summary'가 기본으로 활성화되어, 
클래스 경계를 기준으로 자동 요약을 수행하도록 되어 있었습니다. 이는 RIPv1과의 호환성이나, 일반적인 클래스풀 네트워킹 습관 때문에 남았있는 방식입니다.

### 확인 방법

특권 모드에서 조회합니다.
```bash
Router3# show ip protocols
```

`automatic network summarization is in effect`라고 표시될 수 있습니다. 
따라서 'no auto-summary' 를 명시적으로 넣어주어야, 10.0.0.0/9와 같은 클래스풀 경계를 벗어난 대역을 사용할 때 문제가 생기지 않습니다.

불연속 네트워크(Discontiguous Network)란 동일한 IP대역을 사용하지만 물리적으로 서로 떨어져 있어서, 
중간 경로가 전혀 다른 네트워크 대역으로 구성되어 하나로 이어지지 않는 형태를 말합니다. 

예를 들면:

- A지점: `10.0.0.0/16` 대역을 사용
- B지점: `10.1.0.0/16` 대역을 사용
- A지점과 B지점 사이에는 `172.17.0.0` 대역을 사용하는 다른 네트워크가 존재하면

이때, 10.x.x.x 대역이 물리적으로는 A와 B로 분산되어, 중간에 전혀 다른 IP 대역이 들어가 있기 때문에 불연속이 된 것입니다. 
불연속 네트워크에서 제대로 된 요약 정보를 전달하지 못해, 잘못된 라우팅, 라우팅 불안정이 발생할 수 있습니다. 
따라서 RIP, EIGRP, OSPF 등 클래스리스 라우팅 프로토콜과 정확한 서브넷 정보가 필요합니다.