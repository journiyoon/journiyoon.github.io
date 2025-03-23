---
title: "프로젝트 준비: AWS 리소스 모니터링"
excerpt: ""
categories:
  - AWS
toc: true
toc_sticky: true
last_modified_at: 2025-03-23
---

## 개요

프로젝트 시작 전 AWS의 비용과 리소스 모니터링 방식을 알아봅니다. 
아직은 클라우드 관리 실력이 부족할 수 있기 때문에, 비용이 가장 큰 걱정입니다. 
따라서, 비용과 리소스 사용량 모니터링을 무엇보다 먼저 구축하고 싶은 마음이 큽니다. 
현재 상황에 적절한 방식을 모색해 봅니다. 

## 요구 사항

- AWS 비용을 실시간(또는 매우 짧은 주기)으로 조회 및 시각화
- AWS 서비스(EC2, RDS 등)의 사용량(CPU, 메모리, 네트워크 등)을 실시간으로 파악 및 시각화
- 오픈 소스 활용 경험

## 1. AWS 비용 모니터링 방법
### 📌 사용 서비스: AWS CloudWatch + AWS Cost Explorer + Grafana

① AWS Cost Explorer 활성화
- AWS Billing Dashboard에서 Cost Explorer를 활성화합니다.

② AWS CloudWatch Billing 지표 활성화
- AWS Billing에서 CloudWatch로 지표를 내보낼 수 있도록 활성화
  - Billing Dashboard → Billing preferences → Receive Billing Alerts 활성화

③ Grafana로 AWS 비용 실시간 모니터링
- Grafana를 설치하거나 AWS Managed Grafana 사용
- AWS 플러그인으로 Grafana와 AWS 연동
- AWS CloudWatch의 Billing Metrics를 Grafana 대시보드에서 실시간 표현

> 주의
> AWS 비용 데이터는 일반적으로 몇 시간의 지연이 존재합니다.
> 완벽한 실시간은 어려우며, 매우 짧은 시간(수시간 내)에 비용 변화를 추적하는 것은 가능합니다.

## 2. AWS 리소스 사용량 실시간 모니터링 방법
### 📌 사용 서비스: AWS CloudWatch + Prometheus + Grafana

① AWS CloudWatch 세부 모니터링 설정
-EC2, RDS 등 CloudWatch의 상세 모니터링(1분 간격 수집)을 활성화
  → (AWS EC2 → Monitoring → Enable Detailed Monitoring)

② CloudWatch와 Prometheus 연동
- CloudWatch Exporter for Prometheus를 사용해 CloudWatch 데이터를 Prometheus로 실시간 내보냅니다.
- AWS 자격 증명(Access Key, Secret Key)을 설정 후 Prometheus에 연결

③ Grafana에 Prometheus 데이터 시각화
- Grafana에 Prometheus 데이터 소스 연결
- EC2 CPU, 메모리, 네트워크, RDS 등 다양한 리소스 사용량을 실시간으로 그래프로 시각화하여 구성

## 구성 정리

| 구분       | 사용하는 도구 및 서비스                                            | 목적 및 용도                                     |
|:---------|:---------------------------------------------------------|:--------------------------------------------|
| 비용 모니터링  | AWS Cost Explorer, CloudWatch Billing, Metrics, Grafana  | AWS 비용 시각화 및 관리                             | 
| 리소스 모니터링 | AWS CloudWatch, Prometheus, CloudWatch Exporter, Grafana | AWS 리소스 사용량(CPU, Memory, Network 등 ) 실시간 추적 |

## 고려 사항 (선택)

- AWS API를 활용해 오픈 소스 라이브러리인 CloudWatch Exporter, AWS Cost Exporter 사용을 고려
  - AWS 지표를 Prometheus 형식으로 제공
  - Grafana와 원활한 연동 가능

### ✅ 정확한 기술스택 구성 및 활용 오픈소스
| 구분	             | 사용하는 기술 및 오픈소스 라이브러리                    |
|:----------------|:----------------------------------------|
| AWS 비용 API      | 	AWS Cost Explorer API                  |
| AWS 리소스 API     | 	AWS CloudWatch API                     |
| Exporter (오픈소스) | 	AWS Cost Exporter, CloudWatch Exporter |
| 데이터 수집 및 저장     | 	Prometheus                             |
| 시각화	            | Grafana                                 |