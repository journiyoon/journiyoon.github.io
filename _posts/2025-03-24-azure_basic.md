---
title: "Azure App Service + GitHub Actions 맛보기"
excerpt: ""
categories:
  - Azure
toc: true
toc_sticky: true
last_modified_at: 2025-03-24
---

클라우드 DevOps & SRE 직무를 준비하며, 포트폴리오를 어떻게 시작해야 할지 막막했습니다.
그래서 하루만에 구축 가능한 실습 프로젝트로 **Azure App Service + GitHub Actions 기반 CI/CD 자동화**를 구현해보고,
ARM Template 및 Application Insights까지 확장해봤습니다.
이 글은 그 과정을 정리한 실습 기록이자 포트폴리오 템플릿입니다.

---

## ✅ 프로젝트 개요

### 🎯 목표
- Azure에서 Flask 웹앱을 실행하고, GitHub Actions로 코드 변경 시 자동 배포
- Application Insights로 모니터링 추가
- ARM Template로 리소스를 코드화

### 🧰 사용 스택
- **Azure App Service** (웹앱 호스팅)
- **GitHub Actions** (CI/CD 자동화)
- **Flask** (간단한 웹앱)
- **Application Insights** (모니터링)
- **ARM Template** (Infrastructure as Code)

---

## 🛠️ 1. Azure App Service + GitHub Actions 연동

### 1) Flask 앱 코드 (간단하게)
```python
# app.py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello from Azure App Service + GitHub Actions!"
```

### 2) GitHub Actions 자동 생성
- Azure Portal → App Service → 배포 센터
- GitHub 저장소 연결 → GitHub Actions 자동 생성됨
- `.github/workflows/azure-webapps-python.yml` 워크플로우 확인

### 3) 배포 결과
- 코드 push → Actions 실행 → Azure 자동 배포
- `https://my-flask-app-xxxx.azurewebsites.net` 접속 확인

---

## 📈 2. Application Insights 연결

1. App Service → Application Insights → **연결 활성화**
2. 웹 요청 발생시키면 자동으로 로그 수집
3. Insights → 성능, 실패, 요청 수 등 시각화 확인 가능

> 운영 관점의 관측성(Observability)을 실습으로 이해할 수 있었음

---

## 🧾 3. ARM Template로 리소스 관리

### 1) 기존 리소스를 템플릿으로 추출
Azure Portal → 리소스 그룹 → **템플릿 내보내기**

### 2) GitHub Actions로 ARM Template 자동 배포
```yaml
# .github/workflows/deploy-arm.yml
name: Deploy ARM Template

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      - run: |
          az deployment group create \
            --resource-group devops-rg \
            --template-file azure-deploy.json \
            --parameters appServiceName=my-flask-app
```

### 3) GitHub Secrets 등록
```bash
az ad sp create-for-rbac --name "github-actions-sp" --sdk-auth
```
- 출력된 JSON을 GitHub Secrets에 `AZURE_CREDENTIALS` 이름으로 저장

---

## 🧩 아키텍처 다이어그램

```
[GitHub] → [GitHub Actions] → [Azure App Service]
                                      ↓
                           [Application Insights]
                   ↑
[ARM Template / Terraform] (코드 기반 리소스 관리)
```

---

## 💬 회고 및 배운 점

- App Service를 활용하면 초심자도 쉽게 클라우드 앱을 띄울 수 있다
- GitHub Actions는 공식 연동만으로도 배포 자동화가 매우 간단함
- Application Insights 연결로 실시간 모니터링까지 구현 가능
- ARM Template을 활용해 코드로 리소스를 정의하고 배포하는 IaC 원리 체득

---

## ✅ 마무리 멘트 (커피챗/면접용)

> "하루 동안 Azure App Service에 Flask 앱을 배포하고, GitHub Actions로 CI/CD 자동화를 구성했습니다. 이후 Application Insights로 모니터링을 추가하고, ARM Template으로 인프라를 코드화해 자동 배포까지 구현해보며 DevOps 실무 흐름을 체험했습니다."

---

필요하신 분은 위 내용을 포트폴리오나 발표 자료로 가공해서 사용하셔도 좋습니다 :)

