# Section 5. 클라우드 아키텍처 검증

## 학습 목표
- 아키텍처 검증을 위한 체크리스트 설계 방법을 이해한다.
- AI를 활용해 취약점과 병목을 자동으로 탐지하는 접근법을 익힌다.
- 설계 입력값으로부터 개선 포인트를 자동 도출할 수 있다.

## 검증 체크리스트 설계
Azure Well-Architected Framework **5대 기둥**(비용 최적화·운영 우수성·성능 효율성·안정성·보안)을 기준으로 구성.

### 기둥별 체크 항목
- **안정성(Reliability)**: SPOF 제거 여부, 백업·복구 계획, Multi-AZ 구성
- **보안(Security)**: 최소권한 적용, 데이터 암호화(전송/저장), Private Endpoint 사용
- **성능 효율성**: 적정 SKU 선택, 캐싱 전략, Auto-scale 설정
- **비용 최적화 / 운영 우수성**: 유휴 리소스·예약 인스턴스 활용률, IaC 적용, 모니터링 체계

> **사례**: 체크리스트는 Yes/No 문항이 아닌 **근거(Evidence)를 함께 기록**하도록 설계해야 실효성이 높다.

### AI 검증 파이프라인
```
설계 정보 입력 → AI 분석(체크리스트) → 취약점/병목 탐지 → 개선안 리포트
```

## AI 기반 취약점/병목 자동 탐지
아키텍처 다이어그램·구성 정보를 AI에 입력하면 알려진 안티패턴을 빠르게 스캐닝.

### 대표 안티패턴
- 단일 VM으로만 운영되는 프로덕션 서비스
- `0.0.0.0/0` 전체 허용 NSG 규칙
- 백업 미설정 DB

### 성능 병목 패턴
- DB 커넥션 풀 고갈
- 캐시 미적용으로 인한 반복 쿼리
- 동기 호출 체인으로 인한 지연 누적

> **Azure 네이티브 도구 병행**: Azure Advisor(권고안), Microsoft Defender for Cloud(보안 점수)와 AI 분석을 상호 보완.
>
> **실무 팁**: AI 탐지 결과는 오탐(False Positive) 가능성이 있으므로 실제 리소스 구성과 반드시 교차 검증.

## 개선 포인트 자동 도출 절차
1. **설계 정보 정리**: 리소스 목록, 네트워크 구성, 트래픽 패턴을 텍스트·표로 정리
2. **AI 검증 요청**: Well-Architected 5대 기둥 기준 리뷰 요청
3. **개선안 우선순위화**: 영향도(고/중/저) × 조치 난이도로 매트릭스 작성
4. **개선 로드맵 수립**: 즉시 조치 / 단기(1개월) / 중장기(분기)로 구분

## Well-Architected Framework 5대 기둥 (공식)
| 기둥 | 핵심 관심사 | 적용 원칙 |
|------|-----------|----------|
| Reliability (안정성) | 복원력, 가용성, 복구 | 비즈니스 요구 기반 설계, 단순성 유지 |
| Security (보안) | 데이터 보호, 위협 탐지·완화 | 기밀성·무결성·가용성 보호 |
| Cost Optimization (비용 최적화) | 비용 모델링, 예산, 낭비 절감 | 사용량·요금제 최적화 |
| Operational Excellence (운영 우수성) | 통합 관측성, DevOps 관행 | 표준화·모니터링·안전한 배포 |
| Performance Efficiency (성능 효율성) | 확장성, 부하 테스트 | 수평 확장, 지속적 테스트·모니터링 |

*출처: Microsoft Learn — Azure Well-Architected Framework pillars*

## 실습 Lab
> 🧪 CLI 핸즈온: [해당 실습 바로가기](../labs/day2-labs.md#lab-5--클라우드-아키텍처-검증)

- **시나리오**: 단일 VM + 단일 리전 DB로 구성된 현재 아키텍처를 Gen AI로 검증
- **프롬프트 예시**:
  > "다음 Azure 아키텍처를 Well-Architected Framework 5대 기둥 기준으로 검토해줘: 단일 VM(D2s v5) 1대에서 웹서버 운영, 백업 미설정 Azure SQL Database 1대, 모든 NSG 규칙이 0.0.0.0/0 허용. 발견된 리스크와 개선안을 영향도 순으로 정리해줘."
- **점검 포인트**: 안정성(SPOF)·보안(개방 규칙)·운영(백업 미설정) 리스크 식별 여부 · 영향도×난이도 우선순위 · 즉시 조치 항목(보안) 구분

## 핵심 요약
- **Well-Architected Framework**: 5대 기둥 기반 체계적 검증
- **안티패턴 탐지**: 단일 인스턴스, 개방 규칙, 백업 미설정 등 반복 확인
- **AI + 네이티브 도구 병행**: Advisor·Defender for Cloud와 상호 보완
- **개선 우선순위화**: 영향도×난이도 매트릭스로 로드맵 수립
- **교차 검증**: AI 결과는 반드시 실제 구성과 대조

## 공식문서
- [Well-Architected Framework pillars](https://learn.microsoft.com/en-us/azure/well-architected/pillars)
- [Introduction to Azure Advisor](https://learn.microsoft.com/en-us/azure/advisor/advisor-overview)
- [Microsoft Defender for Cloud 개요](https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-cloud-introduction)
