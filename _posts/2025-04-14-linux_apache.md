---
title: "Linux Service - 🔥 Apache 설정"
excerpt: "🔥 Apache 설정 우선순위, 어디가 먼저 적용될까?에 관한 답변입니다"
categories:
  - Linux
tags:
  - WebServer
  - Apache
toc: true
toc_sticky: true
last_modified_at: 2025-04-14
---

Apache HTTP 서버를 설정하다 보면 가장 헷갈리는 질문 중 하나가 바로 이것입니다.

> “conf/httpd.conf에 설정이 있고, conf.d/vhosts.conf에도 설정이 있는데…
> 도대체 어떤 게 먼저 적용되나요?”

본 포스팅을 통해 설정파일 우선순위에 관한 혼동을 정리해보겠습니다.
- ✔️ 실제 테스트 환경 기준
- ✔️ 실무적인 해석
- ✔️ curl localhost로 바로 확인 가능한 방식   

순서대로 설명해보겠습니다.

---

## ✅ Apache 설정 파일 구조 간단 정리

Apache는 설정 파일을 아래와 같이 구성합니다:

- `httpd.conf` : 메인 설정 파일
- `conf.d/*.conf` : 보조 설정 모듈들, 예: ssl.conf, vhosts.conf 등
- httpd.conf 내에 다음과 같은 포함 구문이 존재합니다:

```conf
IncludeOptional conf.d/*.conf
```
즉, Apache는 다음 순서로 설정을 읽습니다:
1. `httpd.conf` 상단부터 쭉 읽고
2. `IncludeOptional`에 따라 `conf.d/*.conf`를 읽습니다

## ⚖️ 그런데… 어떤 설정이 “적용”되는지는 다르다?

단순히 "어떤 파일이 먼저 읽혔는가"보다 더 중요한 것이 있습니다.   
바로...
> "요청이 어떤 VirtualHost와 매칭되는가"

### ✅ Apache의 요청 처리 로직

Apache 는 요청이 들어오면 아래 순서로 판단합니다:

| 단계 | 설명                                       |       
|:---|:-----------------------------------------|
| 1  | 요청된 `Host` 헤더 또는 IP 주소/포트를 확인            | 
| 2  | 등록된 `<VirtualHost`> 설정들과 비교              | 
| 3  | 가장 먼저 정확히 매칭된 VirtualHost의 설정을 적용        | 
| 4  | 매칭이 없으면 `httpd.conf`의 Main Server 설정을 적용 |

---

## 💡 예시로 바로 이해하기

### ✔️ 기본 설정 (httpd.conf)

```conf
DocumentRoot "/var/www/html"
```

### ✔️ 추가 설정 (conf.d/vhosts.conf)

```conf
<VirtualHost *:80>
    ServerName mysite.local
    DocumentRoot "/var/html/mysite"
</VirtualHost>
```

### ❓ curl localhost 실행 시?

- VirtualHost에 `ServerName localhost`가 정의되어 있다면 → VirtualHost 설정 사용
- 없다면 → 기본 설정(`/var/www/html`)이 적용됨

---

## 정리

| 조건                             | 적용되는 설정                       |       
|:-------------------------------|:------------------------------|
| 요청한  Host가 특정 VirtualHost와 매칭됨 | 그 VirtualHost 설정이 우선          |       
| 매칭되는 VirtualHost가 없음           | `httpd.conf`의 메인 설정 사용        |       
| 같은 디렉티브가 여러 파일에 있음          | 나중에 로드된 설정이 우선 (단, 매칭되는 경우에만) |       

---

## 실무에선 어떻게 적용해보면 좋을까?

주의: 실제와 다를 수 있음.

- `httpd.conf` 에는 공통 설정만 두고,
- 모든 도메인별/서비스별 설정은 `conf.d/vhosts.conf`에 분리 관리
- 도메인 매칭이 필요한 경우, `ServerName`과 `ServerAlias`를 반드시 명시
- 디버깅 시에는 무조건 `apachectl -S`로 매핑 상황을 먼저 확인

## 📌 결론

> Apache 설정은 “파일 순서”보다 “**요청 매칭**”이 우선입니다.
> VirtualHost가 있는 경우, 요청의 Host 값에 맞는 설정이 모든 것을 결정합니다.