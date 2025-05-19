---
title: "Terraform 복습"
excerpt: "개요 및 트러블슈팅 내용 정리입니다"
categories:
  - DevOps
tags:
  - IaC
  - Terraform
toc: true
toc_sticky: true
last_modified_at: 2025-05-19
---

## 학습 목표

* Terraform의 선언형 IaC 개념 이해
* Terraform 워크플로우(Write → Init & Plan → Apply → State) 숙지
* AWS EC2 Key Pair를 이용한 SSH 인증 흐름 파악
* 보안 그룹 설정에 따른 SSH 접속 문제 해결
* Terraform 초기화(init) 및 Plan 오류 해결 방안
* AWS CLI, VSCode 설치 및 환경 구성
* HCL 구성 파일 작성법 및 데이터 소스(data) 활용
* 가용 영역(AZ)별 인스턴스 타입 지원 차이 이해 및 필터링 적용
* Terraform 리소스 정리(tf destroy) 방법 확인

---

## 핵심 키워드 & 강의 요약

* **선언형 IaC**: `.tf` 파일에 원하는 상태를 기술하면 Terraform이 자동으로 API 호출
* **워크플로우**: Write → `terraform init` → `terraform plan` → `terraform apply` → State 관리
* **Key Pair**: SSH 비밀번호 로그인 대신 공개키 기반 인증
* **Security Group**: “All traffic”의 Source가 SG 자신인 경우 외부에서 접속 불가
* **Init 오류**: 락 파일 불일치 → `terraform init` 수행 필요
* **AZ 필터링**: 지원하지 않는 AZ에서 특정 인스턴스 타입은 에러 발생
* **AWS CLI & VSCode**: CLI 설치, Terraform 확장 프로그램으로 개발 편의성 향상
* **Data vs Resource**: 기존 리소스 조회(data) vs 새로운 리소스 생성(resource)

---

## Terraform 개요

Terraform은 HashiCorp에서 제공하는 **선언형 IaC(Infrastructure as Code) 도구**입니다.
사람이 읽고 쓸 수 있는 설정파일로 클라우드 및 온프레미스 리소스를 정의하고 관리할 수 있습니다. 주요 특징은 다음과 같습니다:
- 인프라 정의: 가상머신, 네트워크, 스토리지부터 DNS, SaaS 구성까지 다양한 리소스를 `.tf`파일에 선언
- 플러그인 기반 확장성: AWs, Azure, GCP, Kubernetes, GitHub 등 수천개의 프로바이더를 통해 API 호출 방식으로 리소스 프로비저닝
- 버전 관리: 설정 파일을 Git 같은 버전 관리 시스템으로 관리하여 이력 추적 및 협업 지원
- 일관된 워크플로우:
  1. **Write**: `.tf` 파일에 리소스, 데이터, 변수, 출력값(Output) 등을 선언
  2. **Init & Plan**:

    * `terraform init` → 필요한 Provider 플러그인 설치
    * `terraform plan` → 현재 상태(state)와 선언된 설정 간 차이를 계획(Plan)으로 출력
  3. **Apply**:

    * `terraform apply` → Plan 결과대로 리소스 생성, 변경, 삭제
    * 완료 후 `terraform.tfstate`에 실제 리소스 정보를 저장
  4. **State 관리**:

    * 다음 Plan/Apply 시 무엇이 변경되었는지 정확히 파악

이러한 과정을 통해 인프라의 전체 라이프사이클을 안전하고 효율적으로 운영할 수 있습니다.

---

## Terraform 워크플로우

```bash
# 1. 디렉터리 초기화
terraform init

# 2. 실행 계획 확인
terraform plan

# 3. 실제 적용
terraform apply
# → 완료 후 자동으로 state 파일 업데이트

# 4. 리소스 정리
terraform destroy
```

* `.tf` 파일이 위치한 디렉터리 전체가 하나의 워크스페이스로 다뤄집니다.
* 락 파일(`.terraform.lock.hcl`)은 설치된 Provider 버전을 고정하며, 초기화하지 않으면 Plan 단계에서 오류가 발생합니다.

---

## Key Pair를 이용한 SSH 인증

AWS EC2 인스턴스는 기본적으로 **공개키 기반 SSH 인증**만 허용합니다.
비밀번호 로그인은 보안상 비활성화되어 있기 때문입니다.

1. 로컬에서 키 페어 생성

   ```bash
   ssh-keygen -t ed25519 -f ~/.ssh/tf-keygen
   ```
2. Terraform 설정 파일에 공개키 전달

   ```hcl
   resource "aws_key_pair" "host" {
     key_name   = "tf-keygen"
     public_key = file("~/.ssh/tf-keygen.pub")
   }
   ```
3. EC2 인스턴스 생성 시 `key_name` 속성 사용

   ```hcl
   resource "aws_instance" "instance" {
     ami                         = "ami-05377cf8cfef186c2"
     instance_type               = "t2.micro"
     key_name                    = aws_key_pair.host.key_name
     subnet_id                   = data.aws_subnets.default_subnets.ids[0]
     associate_public_ip_address = true
     tags = {
       Name     = "instance"
       Practice = "One"
     }
   }
   ```
4. SSH 접속

   ```bash
   ssh -i ~/.ssh/tf-keygen ec2-user@<public_ip>
   ```

---

## SSH 접속 문제와 보안 그룹 설정

* **문제 증상**: EC2 Key Pair는 정상 등록됐으나 SSH 연결이 타임아웃됨
* **원인**: 보안 그룹 인바운드에 “All traffic”을 허용했지만 Source가 같은 SG로 설정되어 있어 외부 접속 차단

  * 보안 그룹 내에서만 통신 가능
* **해결 방법**: 인바운드 규칙에 `Type: SSH, Port: 22, Source: 0.0.0.0/0` 또는 CIDR 블럭을 명시적으로 추가

---

## Terraform 초기화 및 Plan 오류

### 락 파일 불일치 오류

```
Error: Inconsistent dependency lock file
...
required by this configuration but no version is selected
```

* **원인**: `.tf` 설정과 `.terraform.lock.hcl` 간 Provider 버전 불일치
* **조치**: `terraform init` 실행 → 락 파일 갱신

### Default VPC 조회 오류

```
Error: no matching EC2 VPC found
with data.aws_vpc.default_vpc
```

* **원인**: 삭제된 Default VPC가 없으므로 `data "aws_vpc" "default_vpc" { default = true }` 실패
* **조치**: AWS Console → Actions → Create Default VPC → Terraform 재실행

---

## 가용 영역(AZ)별 인스턴스 타입 이슈

```
Error: creating EC2 Instance: Unsupported: Your requested instance type (t2.micro)
is not supported in your requested Availability Zone (ap-northeast-2d)
```

* **원인**: 특정 AZ에서는 `t2.micro`를 지원하지 않음
* **해결**: `aws_subnets` 데이터 소스에 AZ 필터 추가

  ```hcl
  data "aws_subnets" "default_subnets" {
    filter {
      name   = "vpc-id"
      values = [data.aws_vpc.default_vpc.id]
    }
    filter {
      name   = "availability-zone"
      values = ["ap-northeast-2a", "ap-northeast-2c"]
    }
  }
  ```
* **효과**: 지원 AZ 내 서브넷만 조회 → `terraform apply` 성공

---

## HCL 구성 파일 작성법

### Provider 설정

```hcl
provider "aws" {
  profile = "default"
  region  = "ap-northeast-2"
}
```

### Resource 예시

```hcl
resource "aws_instance" "web" {
  ami           = "resolve:ssm:/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64"
  instance_type = "t3.micro"
  tags = {
    Name = "HelloWorld"
  }
}
```

* `resource` 레이블(`web`)은 Terraform 내 참조용 로컬 식별자
* 실제 AWS Console 태그와는 별개임

### Data 소스 예시

* 이미 존재하는 리소스 조회 시 사용

```hcl
data "aws_vpc" "default_vpc" {
  default = true
}
data "aws_subnets" "default_subnets" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default_vpc.id]
  }
}
```

---

## tf destroy를 통한 자원 정리

```bash
terraform destroy
```

* 실행 전 Plan을 통해 삭제 대상 확인
* `yes` 입력 시 관리 대상 리소스 모두 삭제
* 실습 후 비용 발생 방지를 위해 꼭 수행

