---
title: "CI/CD 환경에서 암호화 및 시크릿 관리가 중요한 이유"
excerpt: ""
categories:
  - DevOps
tags:
  - CI/CD
  - Secrets
toc: true
toc_sticky: true
last_modified_at: 2025-03-27
---

오늘은 개발 또는 운영 관점에서 보안에 대해 이야기해보려 합니다. 
그전에 저의 경험을 이야기하며 보안의 중요성을 상기시켜 보겠습니다.

## 초보 개발자의 쓰라린 보안 교훈

개개발자로 첫발을 내디딘 시절, 호기심은 많았지만 지식은 부족했습니다.  
블록체인을 처음 배우던 날, 큰 실수를 저질렀습니다.

처음 배우는 기술에 들떠서 메타마스크 지갑 정보를 아무런 고민 없이 깃헙 저장소에 올렸습니다.  
**"누가 내 정보를 보겠어?"라는 순진한 생각이 불행의 시작이었습니다.**

무라카미 다카시 플라워 이벤트에 친구 추천으로 응모했고, 운이 좋게도 당첨됐습니다.  
설렘을 안고 NFT 수령을 위해 지갑에 이더리움을 충전했습니다. 
그리고 수초 뒤.. **이더리움이 알 수 없는 지갑으로 순식간에 사라졌습니다.**

거래 내역을 확인하자 모든 게 분명해졌습니다.  
깃헙에 무심코 올렸던 지갑 정보가 해커의 표적이 되었던 것입니다.  
이 부주의함으로 인해 암호화폐와 귀중한 NFT 수령 기회까지 모두 잃었습니다.

이 쓰라린 경험을 통해 보안의 중요성을 뼈저리게 느끼게 되었고, 앞으로 다시는 이런 실수를 반복하지 않겠다고 다짐했습니다.

### 배운 교훈

- **개인 정보와 보안 자격 증명은 절대 공개된 저장소에 업로드하지 않는다.**
- **암호화폐 지갑과 같은 민감한 정보는 항상 철저히 관리해야 한다.**
- 기술 학습 과정에서 **보안은 가장 중요한 덕목**이다.

---

이제 본격적으로 CI/CD 환경에서의 시크릿 관리 방법론을 살펴보겠습니다.

CI/CD 파이프라인은 개발 단계에서 배포 단계까지 자동화를 통해 속도와 효율성을 높이는 것이 목표입니다. 
이 과정에서 데이터베이스 비밀번호, API 키, SSH 키, OAuth 토큰 같은 민감한 정보(Secrets) 가 자주 사용됩니다. 
이러한 정보를 안전하게 관리하지 않으면:
- 코드 저장소(GitHub 등)에 민감한 정보가 노출될 위험
- 로그나 빌드 결과물에서의 민감 정보 유출
- 악의적인 접근으로 인한 보안 사고 발생 가능성

=> 따라서 CI/CD 환경에서의 안전한 시크릿 관리가 필수적입니다.

## 🛠️ CI/CD 시크릿 관리 방법론

### ① Secrets와 코드의 완벽한 분리 (Code & Secrets Separation)

- 절대 민감한 정보를 코드 리포지토리에 직접 포함하지 않음
- 환경 변수나 별도 시크릿 저장소를 통해 전달하여 보안성을 유지

✅ 권장 방법
- 환경 변수로 파이프라인 내에서 전달
- Secrets Management 솔루션에서 런타임 시 시크릿 조회

❌ 피해야 할 방법
- 하드코딩된 시크릿(소스코드에 직접 포함)
- 설정파일(.env 파일 등)을 깃 저장소에 커밋하는 것(실수로 공개 저장소에 노출될 위험)

---

### ② Secrets 관리 전용 도구의 사용

시크릿 관리 도구는 가장 필수로 학습하고 사용해야할 솔루션입니다.

#### 🔹 대표적인 솔루션 및 특징

| 솔루션                    | 설명 및 특징                                         | 클라우드 서비스 통합                      |
|:-----------------------|:------------------------------------------------|:---------------------------------|
| HashiCorp Vault        | 가장 인기 있는 오픈소스 시크릿 관리 도구로, 동적 시크릿 생성, 만료기간 설정 가능 | AWS, Azure, Kubernetes와 통합 가능    |
| AWS Secrets Manager    | AWS 환경에서 시크릿을 안전하게 저장, 주기적 자동 교체 기능 제공          | AWS 환경 전용 (Lambda, ECS 등과 손쉬운 통합 |
| Azure Key Vault        | Azure 클라우드에서 사용되는 시크릿, 암호화 키 저장 및 관리            | Azure DevOps와 네이티브 통합 가능         |
| GitHub Actions Secrets | GitHub Actions에서 사용할 시크릿을 암호화된 형태로 관리           | GitHub Actions 내에서만 사용 가능        |
| GitLab CI/CD Variables | GitLab CI/CD에서 환경 변수로 관리하는 시크릿                  | GitLab 환경에서 CI/CD 파이프라인 내 통합     |

### ③ 실제 CI/CD 파이프라인에서 시크릿을 가져오는 방법 예시
### 🔑 예시 1) GitHub Actions & HashiCorp Vault

>흐름
> Vault에 시크릿 저장 → GitHub Action에서 Vault 인증 → 런타임에 시크릿 가져와서 배포

```yaml
# .github/workflows/deploy.yml 예시
name: Deploy with Vault Secret

on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: hashicorp/vault-action@v2.7.4
        with:
        kurl: https://vault.yourdomain.com
          token: ${{ secrets.VAULT_TOKEN }}
          secrets: |
            secret/data/database db_password | DB_PASSWORD ;

      - name: Use Secret
        run: |
          echo "Deploying with database password: $DB_PASSWORD"
          # DB_PASSWORD를 사용해 배포 작업 진행
```

> 설명
> GitHub Secrets에 VAULT_TOKEN 저장
> 파이프라인은 Vault에서 DB_PASSWORD를 런타임에 가져와 환경 변수로 사용합니다.
> 배포 과정에서 DB_PASSWORD가 코드나 로그에 남지 않도록 주의합니다.


이 외에도 동적 시크릿 활용도 있습니다. 
보관도 중요하지만 시크릿이 노출된 경우(보안 사고 발생 시), 시크릿 무효화 및 변경이 가능한 모니터링 설정이 필수입니다.

## 추가로 고려할 보안적 관점

- DevOps 팀 구성원별로 최소 권한 원칙 적용
- Secret 접근 권한은 필요 시에만 부여하고 로그를 기록하여 관리
- 보안 감사(Security Audit) 쥐기적 실시로 보안 취약점 점검 및 개선