# azure-ai-architecture-lab

**AI 기반 클라우드 아키텍처 설계 (Azure) — 학습 정리 & 실습**

> 2일 / 16시간 과정 (이론 6h · 실습 10h) 정리 노트
> 실습환경: Microsoft Azure · Gen AI · Mermaid-AI

AI(Gen AI)를 활용해 Azure 클라우드 아키텍처를 **설계 → 검증 → 문서화 → 운영 최적화**까지
자동화하는 실무 흐름을 다루는 과정입니다. 각 Section은 개념 → 상세 → 실무 팁 → 실습(Lab) → 핵심 요약 구조로 정리했습니다.

## 과정 목표
- AI를 활용해 클라우드 아키텍처 설계·검증·최적화를 자동화하는 실무 능력 확보
- 네트워크·보안·컴퓨팅·스토리지 핵심 요소를 AI 기반으로 자동 분석·추천
- DevOps·IaC(Bicep/Terraform) 연계 운영 자동화의 기초 확보

## 목차

### Chapter 1. AI 기반 클라우드 아키텍처 기초 설계
| # | Section | 핵심 키워드 |
|---|---------|------------|
| 1 | [클라우드 아키텍처 기본 및 AI 페르소나 설정](./ch1-foundation/01-basics-and-ai-persona.md) | VNet, Subnet, LB/App Gateway, MSA/Serverless/Container |
| 2 | [네트워크·보안 설계](./ch1-foundation/02-network-security.md) | Hub-Spoke, NSG, Entra ID, RBAC, Private Endpoint |
| 3 | [컴퓨팅·스토리지 설계](./ch1-foundation/03-compute-storage.md) | VM/VMSS, Blob 계층, Azure SQL, 스펙 산정 |
| 4 | [비용·고가용성·DR 설계](./ch1-foundation/04-cost-ha-dr.md) | AZ/Availability Set, DR Tier, ASR, 비용 절감 |

### Chapter 2. AI 기반 아키텍처 검증·문서화·운영
| # | Section | 핵심 키워드 |
|---|---------|------------|
| 5 | [클라우드 아키텍처 검증](./ch2-verification-ops/05-architecture-verification.md) | Well-Architected 5대 기둥, 안티패턴 탐지 |
| 6 | [아키텍처 문서 자동 생성](./ch2-verification-ops/06-doc-automation.md) | HLD/LLD, Mermaid-AI, Sequence/Infra Diagram |
| 7 | [운영·모니터링 구조 설계](./ch2-verification-ops/07-monitoring.md) | Azure Monitor, Log Analytics/KQL, App Insights, KPI |
| 8 | [운영 최적화 자동화](./ch2-verification-ops/08-ops-optimization.md) | KQL 이상탐지, Latency 분석, 개선안 자동생성 |

## 🧪 실습 (Hands-on Labs)
Azure Cloud Shell(CLI)로 리소스를 직접 만들고, GenAI **Before/After**(모호한 질문 vs 구체적 프롬프트)를 비교하는 실습입니다. 각 Lab은 위 이론 Section과 1:1로 대응됩니다.

| 실습 | 대응 이론 | 내용 |
|------|----------|------|
| [Lab 0. 공통 환경 구성](./labs/lab-00-setup.md) | — | Cloud Shell, 리소스 그룹, GenAI 계정, (Day 2) 모니터링 스택 |
| [Day 1. 기초 아키텍처 (Lab 1~4)](./labs/day1-labs.md) | Section 1~4 | VNet/Subnet/LB, NSG/RBAC, VMSS/오토스케일/스토리지, Advisor/DR/정리 |
| [Day 2. 검증·문서화·운영 (Lab 5~8)](./labs/day2-labs.md) | Section 5~8 | Well-Architected 검증, Mermaid HLD, App Insights/KQL/Alert, 이상탐지/포스트모템 |
| [종합 실습 (Capstone) 🎫](./labs/capstone-ticketing.md) | Section 1~8 전체 | 티켓 예매 서비스 — 순간 트래픽 200배 폭증 시나리오 통합 설계 챌린지 |

> ⚠️ **비용 주의**: 실습은 VM·SQL·App Service 등 과금 리소스를 생성합니다. 각 Day 마지막의 `az group delete`로 반드시 정리하세요.

### 🎫 종합 실습 (Capstone) 소개
Day 1~2에서 배운 8개 Section을 **하나의 시나리오로 통합**하는 부록 실습입니다. "티켓 오픈 10분에 트래픽이 200배 폭증하는" 이벤트성 서비스를 설계 → 검증 → 문서화 → 운영까지 관통하며, 정답이 정해지지 않은 **설계 챌린지** 형태입니다. 예약 스케일링·캐싱 전략·비대칭 비용 최적화 등 실무 난제를 다루고, AI 산출물을 비판적으로 검증하는 훈련이 핵심입니다.

## 관통하는 핵심 원칙
- **AI 산출물은 항상 초안(draft)** — 아키텍트의 검증·조정을 반드시 거친다.
- **Well-Architected Framework 5대 기둥**(안정성·보안·비용 최적화·운영 우수성·성능 효율성)이 검증의 기준.
- **교차 검증(Cross-check)** — AI 탐지 결과는 실제 리소스 구성과 반드시 대조.

## 참고
- [Azure Well-Architected Framework](https://learn.microsoft.com/en-us/azure/well-architected/)
- [Azure Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/browse/)
- [Mermaid 공식 문서](https://mermaid.js.org)

- [강의](https://url.kr/1asqve)
- [교재](https://flunti.notion.site/ai-azure-lab)
- [Azure Infra Lab](https://psedu.gitbook.io/azure-infra-lab)

# 강사메일
- wsjang@gkn.co.kr
