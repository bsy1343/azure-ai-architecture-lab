# Section 4. 비용·고가용성·DR 설계

## 학습 목표
- Availability Zone/Set 기반 고가용성 설계 원칙을 이해한다.
- DR Tier 개념과 Azure Site Recovery 기반 재해복구 전략을 수립할 수 있다.
- 기존 아키텍처를 입력해 AI로 비용 절감안을 도출할 수 있다.

## High Availability 설계 원칙
고가용성은 **단일 장애점(SPOF) 제거**에서 시작. Azure는 리전 내 AZ와 Availability Set 두 메커니즘 제공.

### 핵심 구성요소
- **Availability Zone (AZ)**: 리전 내 물리적으로 분리된 데이터센터 그룹. 전원·냉각·네트워크 독립 → 존 단위 장애 대응
- **Availability Set**: 동일 DC 내 장애 도메인·업데이트 도메인으로 VM 분산 배치
- **Zone-redundant 서비스**: Load Balancer(Standard), Azure SQL Database, Storage(ZRS) 등이 AZ 간 자동 이중화
- **SLA 조합**: 다중 AZ 배포 시 VM SLA 99.9% → **99.99%** 향상

> **사례**: 3개 AZ에 각각 VMSS 인스턴스를 분산하고 Standard LB로 트래픽 분산하면 특정 AZ 장애 시에도 서비스 지속.

### 리전 간 DR 구성
```
Primary Region (AZ1·AZ2·AZ3) ──[Azure Site Recovery 복제]──> Paired DR Region (Failover 대기)
```

## DR Tier 및 재해복구 전략
DR 전략은 **RTO(복구 목표 시간)** 와 **RPO(복구 목표 시점)** 요구사항에 따라 Tier 구분.

| DR Tier | 설명 | RTO |
|---------|------|-----|
| Backup & Restore | 가장 저비용 | 수 시간 ~ 수일 |
| Pilot Light | 최소 구성만 상시 유지, 장애 시 스케일업 | 수십 분 |
| Warm Standby | 축소된 운영 환경 상시 가동, 장애 시 확장 | 수 분 |
| Multi-Site (Active-Active) | 두 리전 동시 서비스, 비용 최고 | ≈ 0 |

- **Azure Site Recovery (ASR)**: VM 단위 리전 간 복제·장애 조치(Failover) 자동화. Paired Region 활용 권장

> **실무 팁**: Azure는 리전을 Paired Region으로 묶어 순차 업데이트·지리적 이중화 지원 → DR 대상 리전 선정 시 우선 고려.

### ASR RPO/RTO 참고값
| 항목 | 값 | 설명 |
|------|-----|------|
| RTO SLA | 약 1시간 | Azure-to-Azure 복제 기준 공식 SLA |
| RPO | 약 5분 | 일반 구성 기준 실측 수준 |
| 최소 복제 주기 | 약 30초 | Azure VM 간 설정 가능한 최소 간격 |

*출처: Microsoft Learn — Azure Site Recovery FAQ*

### 가용성 구성별 SLA
| 구성 | SLA |
|------|-----|
| 단일 VM (Premium SSD) | 99.90% |
| Availability Set (2대 이상) | 99.95% |
| Availability Zone (2개 이상 존) | 99.99% |

*출처: Microsoft Azure SLA for Virtual Machines*

## AI 비용 절감안 자동 생성 절차
1. **현황 정리**: 리소스 목록, 사용률(CPU/메모리), 현재 요금제(온디맨드/예약) 문서화
2. **AI 분석 요청**: Gen AI/Azure Advisor에 리소스 사용 패턴 입력 → 절감 기회 도출
3. **절감안 검토**: 예약 인스턴스 전환, 유휴 리소스 삭제, 스토리지 계층 조정, Right-sizing 등 항목별 절감액 산정
4. **우선순위 적용**: 서비스 영향도가 낮은 항목부터 단계적 적용

## 실습 Lab
> 🧪 CLI 핸즈온: [해당 실습 바로가기](../labs/day1-labs.md#lab-4--비용고가용성dr-설계)

- **시나리오**: 상시 가동 중인 D4s v5 VM 10대(평균 CPU 15%)에 대한 비용 절감안을 Gen AI로 도출
- **프롬프트 예시**:
  > "Azure에서 D4s v5 VM 10대를 24시간 상시 운영 중이며 평균 CPU 사용률이 15%다. Right-sizing, 예약 인스턴스, Azure Hybrid Benefit 관점에서 비용 절감 방안을 구체적인 예상 절감률과 함께 제안해줘."
- **점검 포인트**: 성능·가용성 영향 검토 · Right-sizing 후 목표 사용률(60~70%) 적절성 · 리스크 대비 효과 순 우선순위

## 핵심 요약
- **AZ / Availability Set**: 리전 내 고가용성 확보의 핵심 메커니즘
- **DR Tier**: Backup&Restore ~ Multi-Site까지 RTO/RPO 기준 선택
- **Azure Site Recovery**: 리전 간 자동 복제·장애조치
- **비용 절감**: Right-sizing, 예약 인스턴스, Hybrid Benefit이 대표 수단
- **AI 활용**: 기존 구성 입력만으로 절감 기회를 빠르게 식별

## 공식문서
- [Azure Reliability documentation](https://learn.microsoft.com/en-us/azure/reliability/overview)
- [About Azure Site Recovery](https://learn.microsoft.com/en-us/azure/site-recovery/site-recovery-overview)
- [Azure-to-Azure 복제 FAQ](https://learn.microsoft.com/en-us/azure/site-recovery/azure-to-azure-common-questions)
