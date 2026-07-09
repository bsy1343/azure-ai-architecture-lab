# Section 8. 운영 최적화 자동화

## 학습 목표
- 로깅 기반 장애 패턴 감지 방법을 이해한다.
- Latency 분석과 AI 기반 문제 탐지 기법을 익힌다.
- AI를 활용해 운영 개선안을 자동 생성하고 적용할 수 있다.

## Logging 기반 장애 패턴 감지
반복되는 로그 패턴(에러 코드, 예외 메시지)을 분석하면 장애의 근본 원인을 조기 식별.

### 핵심 구성요소
- **KQL 기반 이상 탐지**: 특정 시간대 에러 로그 급증(spike)을 감지하는 쿼리 작성
- **로그 상관관계 분석**: 여러 서비스 로그를 시간 축으로 정렬해 장애 전파 경로 추적
- **Azure Monitor Log Alert**: 특정 KQL 쿼리 결과가 임계값 초과 시 자동 알림
- **대표 패턴**: 특정 API 5xx 급증, DB 커넥션 타임아웃 반복, 특정 리전에서만 발생하는 오류

### 최적화 자동화 루프
```
Logs / Metrics → KQL 분석 → AI 이상탐지 → 개선안 자동생성 → 적용 & 검증
```

> **사례**: '지난 1시간 동안 5xx 에러가 평시 대비 10배 증가'와 같은 KQL 쿼리를 스케줄링하여 조기 경보 체계 구축.

## Latency 분석 & AI 기반 문제 탐지
응답 지연은 네트워크·컴퓨팅·DB·외부 종속성 등 여러 구간에서 발생 → **구간별 분해 분석** 필요.

### 핵심 구성요소
- Application Insights의 **종단 간 트랜잭션 추적**으로 지연 발생 구간(DB 쿼리, 외부 API 호출 등) 특정
- **P50/P95/P99 지연 분포**를 함께 봐야 꼬리 지연(Tail Latency) 문제를 놓치지 않음
- **AI 기반 이상 탐지**: 과거 정상 범위를 학습해 통계적으로 벗어난 지연 패턴 자동 플래깅 (Azure Monitor 동적 임계값 등)

> **실무 팁**: 지연 문제는 단일 원인보다 복합 원인(캐시 미스 + DB 부하 동시 발생)인 경우가 많아 다각도 로그를 함께 분석.

## 개선안 자동 생성 절차
1. **로그·메트릭 수집**: 장애 발생 시점 전후의 로그·메트릭·트레이스 정리
2. **AI 분석 요청**: 증상(에러율, 지연시간 변화)과 관련 로그를 입력해 원인 후보 요청
3. **가설 검증**: AI가 제시한 원인 후보를 실제 대시보드 데이터와 대조
4. **개선안 적용 및 회고**: 조치 후 효과 측정, 포스트모템(Postmortem) 문서로 정리

## KQL 쿼리 예시
```kusto
// 지난 1시간 5xx 에러 급증 탐지
requests
| where timestamp > ago(1h)
| where resultCode startswith "5"
| summarize ErrorCount = count() by bin(timestamp, 5m)
| where ErrorCount > 50

// P95 응답시간 추이 조회
requests
| where timestamp > ago(24h)
| summarize P95 = percentile(duration, 95) by bin(timestamp, 1h)
| render timechart
```

*출처: Microsoft Learn — KQL tutorial / Log queries in Azure Monitor*

## 실습 Lab
> 🧪 CLI 핸즈온: [해당 실습 바로가기](../labs/day2-labs.md#lab-8--운영-최적화-자동화)

- **시나리오**: 특정 시간대 API 응답 지연이 P95 500ms에서 3초로 급증한 상황을 Gen AI로 분석
- **프롬프트 예시**:
  > "Azure App Service 기반 API의 P95 응답시간이 평소 500ms에서 특정 30분 동안 3초로 급증했다. 같은 시간대 Azure SQL Database의 DTU 사용률이 95% 이상으로 치솟았고, 외부 결제 API 호출 실패율도 증가했다. 가능한 원인 가설과 우선 확인해야 할 지표, 단기/장기 개선안을 제안해줘."
- **점검 포인트**: 원인 가설과 증상(DB DTU 급증, 외부 API 실패)의 논리적 연결 · 단기 조치 vs 장기 개선 구분 · 효과 측정 지표 제시 여부

## 핵심 요약
- **KQL 기반 로그 분석**: 장애 패턴을 조기에 탐지하는 핵심 수단
- **구간별 지연 분해**: 네트워크·컴퓨팅·DB·외부 종속성 단위로 원인 특정
- **Tail Latency**: P95/P99까지 함께 관찰해야 놓치지 않는 문제
- **AI 기반 원인 분석**: 증상 입력만으로 원인 가설·개선안 초안 확보
- **포스트모템**: 개선 적용 후 회고로 재발 방지 체계화

## 공식문서
- [Learn common KQL operators (Tutorial)](https://learn.microsoft.com/en-us/kusto/query/tutorials/learn-common-operators)
- [Log Analytics tutorial](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-tutorial)
- [Log queries in Azure Monitor 개요](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-query-overview)
