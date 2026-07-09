# Section 1. 클라우드 아키텍처 기본 및 AI 페르소나 설정

## 학습 목표
- Azure 핵심 구성요소(VNet, Subnet, Load Balancer 등)의 역할과 관계를 설명할 수 있다.
- MSA·Serverless·Container 아키텍처의 차이와 적용 기준을 이해한다.
- GenAI를 활용해 비즈니스 요구사항으로부터 기본 아키텍처 구조를 도출할 수 있다.

## Azure 핵심 서비스 구성요소
- 리소스 계층: **구독(Subscription) > 리소스 그룹(Resource Group) > 개별 리소스**
- **VNet**: 리전 단위 논리 네트워크(AWS VPC 대응). CIDR 기반 주소공간 정의, Peering으로 VNet 간 연결
- **Subnet**: VNet 내부 IP 대역 분할 단위. NSG·Route Table을 Subnet 단위로 적용
- **Load Balancer (L4)**: 내부/공용 구성, TCP/UDP 분산, Health Probe로 상태 점검
- **Application Gateway (L7)**: HTTP(S) 라우팅, WAF 통합, URL 기반 라우팅

> **사례**: 온라인 쇼핑몰 — Application Gateway가 도메인별 트래픽을 Web/API Subnet으로 분기,
> 내부 Load Balancer가 각 Subnet 내 VM 간 부하 분산. 리소스 그룹에 태그(Tag)를 부여하면 비용·운영 관리가 용이.

### 3-Tier 웹 서비스 기본 구조
```
사용자 → Application Gateway → Web Tier (VMSS) → App Tier (VMSS) → Azure SQL Database
```

## MSA · Serverless · Container
- **MSA**: 모놀리식 대비 서비스 단위 독립 배포·확장 가능
- **Serverless (Azure Functions)**: 이벤트 기반 실행, 인프라 관리 부담 최소화. HTTP·Timer·Queue 트리거, 소비 계획(Consumption Plan) 종량 과금
- **Container**:
  - **AKS**: 관리형 Kubernetes, 노드 풀 단위 확장, Helm/GitOps 연계
  - **Azure Container Apps**: K8s 지식 없이 컨테이너 운영, KEDA 기반 이벤트 스케일링
- **선택 기준**: 트래픽 예측 가능성 · 실행 시간 · 상태 유지 여부 · 팀 운영 역량

> **실무 팁**: 스타트업 초기엔 Container Apps/Functions로 빠르게 출시 → 트래픽 증가 시 AKS로 단계적 전환.

## 컴퓨팅 모델 비교
| 항목 | Azure VM | AKS (컨테이너) | Azure Functions (서버리스) |
|------|----------|----------------|----------------------------|
| 관리 부담 | 높음 (OS·패치 직접) | 중간 (클러스터는 관리형) | 매우 낮음 (인프라 완전 위임) |
| 확장 방식 | VMSS 수동/자동 | 노드 풀·Pod 오토스케일 | 트리거 기반(0까지 축소) |
| 과금 방식 | 가동 시간 기준 | 노드 가동 시간 기준 | 실행 횟수·시간(종량제) |
| 적합 워크로드 | 상태 유지·레거시 | 마이크로서비스, 이식성 | 이벤트 처리, 짧은 비동기 |

*출처: Microsoft Learn — Choose an Azure compute service*

## AI 활용 아키텍처 도출 절차
1. **요구사항 정리**: 사용자 수, 트래픽 패턴, RTO/RPO, 보안·컴플라이언스 요건 문서화
2. **프롬프트 작성**: 서비스 목적, 제약조건, 우선순위(비용/성능/가용성) 명시
3. **초안 검토**: 제안된 서비스 조합(VNet, Compute, Storage, DB)을 Well-Architected 기준으로 검증
4. **반복 개선**: 부족한 요소(보안, 모니터링 등)를 추가 질의하며 완성도 향상

> AI 산출물은 **초안**이며, 반드시 아키텍트의 검증과 조정을 거쳐야 한다.

## 실습 Lab
> 🧪 CLI 핸즈온: [해당 실습 바로가기](../labs/day1-labs.md#lab-1--클라우드-아키텍처-기본-및-ai-페르소나-설정)

- **시나리오**: 월 방문자 50만 명 규모 이커머스 서비스의 기본 Azure 아키텍처 초안을 Gen AI로 도출
- **프롬프트 예시**:
  > "너는 Azure 클라우드 아키텍트다. 월간 방문자 50만 명, 피크 시간대 트래픽 집중, 예산 월 500만원 이내인 이커머스 웹서비스의 기본 아키텍처를 VNet, Subnet, 컴퓨팅, 스토리지, DB 구성 중심으로 제안해줘. 고가용성과 비용 효율을 함께 고려해줘."
- **점검 포인트**: 요구사항 충족 여부 · 보안/모니터링 누락 여부 · 서비스명-역할 매칭 정확성

## 핵심 요약
- **VNet/Subnet**: 네트워크 격리·분할의 기본 단위
- **LB / App Gateway**: 계층별 트래픽 분산 전략
- **MSA/Serverless/Container**: 서비스 특성에 맞는 컴퓨팅 모델 선택
- **AI 활용**: 요구사항 기반 아키텍처 초안 자동 생성으로 설계 시간 단축
- **검증 필수**: AI 산출물은 항상 Well-Architected 관점에서 재검토

## 공식문서
- [Azure Virtual Network 개요](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview)
- [Azure 애플리케이션 10가지 설계 원칙](https://learn.microsoft.com/en-us/azure/architecture/guide/design-principles/)
- [Azure Well-Architected Framework](https://learn.microsoft.com/en-us/azure/well-architected/)
