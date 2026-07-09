# Section 3. 컴퓨팅·스토리지 설계

## 학습 목표
- Azure VM, VMSS, Blob Storage, Azure SQL Database의 특징과 활용 전략을 이해한다.
- 성능과 비용을 함께 고려한 스펙 산정 원리를 익힌다.
- 트래픽 기준으로 컴퓨팅 스펙을 산정할 수 있다.

## 컴퓨팅·스토리지 핵심 구성요소
- **VM 시리즈**: 웹서버(D시리즈, 범용) · 배치·연산(F시리즈, 컴퓨팅 최적화) · DB(E시리즈, 메모리 최적화)
- **VMSS (Scale Sets)**: 동일 구성 VM 자동 확장·축소 (AWS ASG 대응). Scale-out 규칙(CPU/메모리/큐 길이), Availability Zone 균등 분산
- **Blob Storage**: Hot/Cool/Archive 계층으로 접근 빈도별 비용 최적화. LRS/ZRS/GRS 중복 옵션
- **Azure SQL Database**: DTU/vCore 두 과금 모델, 자동 튜닝·자동 백업 내장, Hyperscale로 대용량 확장

### 구성 흐름
```
Client → Load Balancer → VMSS (Web/App) → Azure Cache for Redis → Azure SQL DB / Blob Storage
```

> **사례**: 이미지 업로드가 많은 서비스는 수명 주기 관리 정책으로 최근 파일은 Hot, 오래된 파일은 Cool/Archive로 자동 이동해 비용 절감.

## 성능 & 비용 기준 스펙 산정
- **CPU 산정**: (초당 요청 수 × 요청당 평균 CPU 처리시간) ÷ 목표 CPU 사용률(예: 70%) → 필요 vCore
- **메모리 산정**: 세션·캐시 데이터 크기 × 동시 접속자 수 + 여유분(20~30%)
- **스토리지 IOPS**: DB 트랜잭션 처리량(TPS) 기준으로 Premium SSD 등 Managed Disk 등급 선택
- **비용 최적화**: 예약 인스턴스(Reserved VM), Azure Hybrid Benefit, Spot VM → 최대 **72%** 절감

> **실무 팁**: Azure Pricing Calculator + Azure Advisor 병행 → 스펙 산정과 비용 검증 동시 수행.

## 트래픽 기준 스펙 산정 절차
1. **트래픽 프로파일 수집**: 시간대별·요일별 요청 수, 피크 대비 평시 비율 분석
2. **인스턴스당 처리량 측정**: 부하 테스트로 VM 1대의 초당 요청 수(RPS) 확인
3. **필요 인스턴스 수 계산**: 피크 RPS ÷ 인스턴스당 RPS × 여유율(1.2~1.3)
4. **VMSS 오토스케일 규칙 설정**: CPU 70% 초과 시 확장, 30% 미만 시 축소

## Azure Storage 중복(Redundancy) 옵션
| 옵션 | 복제 방식 | 연간 내구성 | 특징 |
|------|----------|------------|------|
| LRS | 동일 DC 내 3중 복제 | 11개 9 | 최저 비용, DC 재해에 취약 |
| ZRS | 3개 이상 가용영역 복제 | 12개 9 | 가용영역 장애에도 읽기/쓰기 유지 |
| GRS | LRS + 보조 리전 비동기 복제 | 16개 9 | 리전 재해 대비, 보조 리전 기본 읽기 불가 |
| RA-GRS | GRS + 보조 리전 읽기 허용 | 16개 9 | 리전 장애 시 보조 리전에서 읽기 가능 |

*출처: Microsoft Learn — Azure Storage redundancy*

## Azure SQL Database 구매 모델
| 항목 | DTU 모델 | vCore 모델 |
|------|---------|-----------|
| 과금 단위 | CPU·메모리·IO를 묶은 DTU | vCPU·메모리 개별 산정 |
| 티어 | Basic / Standard / Premium | 프로비저닝 / 서버리스 |
| 서버리스 | 미지원 | 사용량 기반 자동 일시중지·재개 |
| 적합 대상 | 예측 가능한 소규모 워크로드 | 세밀한 조정이 필요한 중대형 |

*출처: Microsoft Learn — Azure SQL Database purchasing models*

## 실습 Lab
> 🧪 CLI 핸즈온: [해당 실습 바로가기](../labs/day1-labs.md#lab-3--컴퓨팅스토리지-설계)

- **시나리오**: 피크 시간대 초당 1,000건 요청을 처리해야 하는 서비스의 VMSS 스펙을 Gen AI로 산정
- **프롬프트 예시**:
  > "Azure VMSS로 피크 시간대 초당 1,000건 HTTP 요청을 처리해야 하는 웹 서비스가 있다. VM 1대(D2s v5)가 초당 80건을 처리할 수 있다고 가정할 때 필요한 최소/최대 인스턴스 수와 오토스케일 규칙(CPU 기준)을 제안해줘."
- **점검 포인트**: 인스턴스 수 여유율 포함 여부 · 오토스케일 임계값 합리성 · 예약 인스턴스/Spot VM 적용 가능성

## 핵심 요약
- **VM 시리즈**: 워크로드 특성별 D/F/E 선택
- **VMSS**: 오토스케일 기반 컴퓨팅 확장의 핵심
- **Blob 계층화**: Hot/Cool/Archive로 비용 최적화
- **Azure SQL**: DTU/vCore 모델과 Hyperscale 확장
- **스펙 산정**: 트래픽·성능·비용을 함께 고려한 역산 접근

## 공식문서
- [Sizes for virtual machines in Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes)
- [Access tiers for blob data](https://learn.microsoft.com/en-us/azure/storage/blobs/access-tiers-overview)
- [Azure SQL Database purchasing models](https://learn.microsoft.com/en-us/azure/azure-sql/database/purchasing-models)
