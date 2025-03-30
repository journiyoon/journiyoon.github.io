---
title: "불변 인프라(Immutable Infrastructure)란?"
excerpt: ""
categories:
  - DevOps 
toc: true
toc_sticky: true
last_modified_at: 2025-03-30
---

## 개념

- **Immutable(불변)**: 한 번 만들어진 서버나 인프라 구성 요소는 수정하지 않고, 변경이 필요할 경우 **새로운 인스턴스를 배포**하여 대체하는 방식
- 기존 방식은 "서버를 띄운 상태에서 수정" -> mutable
- 불변 인프라는 "새 버전을 만들어 기존 버전과 바꿔치기" -> immutable

### DevOps에서:

- CI/CD 파이프라인에서 이미지를 만들어 테스트 후, 그 이미지를 배포하여 운영 환경을 구성 -> 기존 이미지와 바꿔치기 후 운영
- Docker, Terraform, Ansible, Kubernetes 같은 도구들과 밀접하게 연관됨

### 클라우드에서:

- AWS EC2 Auto Scaling 그룹에서 새로운 AMI로 롤링 배포
- Kubernetes에서 새 버전의 컨테이너로 Pod 교체 -> 전형적인 불변 인프라

## 장점

1. 신뢰성 향상
   - 수정이 없으므로 상태 꼬임이 없음
2. 디버깅이 간편함
   - 재현 가능한 환경 (일관된 이미지로 배포하므로)
3. 배포 자동화에 적합한 방식
   - GitOps/CI-CD에 알맞는 방식

## 단점

1. 초기 셋업이 복잡할 수 있음
2. 이미지 생성 시간이 오래 걸릴 수 있음

## 결론

불변 인프라는 서버를 수정하는 것이 아닌, 새로 만들어 교체하는 방식입니다. 
Docker, Kubernetes, CI/CD 파이프라인과 함께 쓰이며 일관성과 안정성을 제공합니다. 
클라우드 자동화와 데브옵스 문화에 핵심적인 인프라 패턴입니다.
