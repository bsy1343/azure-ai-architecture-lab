# Section 7. 운영·모니터링 구조 설계

## 학습 목표
- Azure Monitor, Log Analytics, Application Insights의 구성 원리를 이해한다.
- 운영 기준 KPI를 선정하고 설계할 수 있다.
- AI를 활용해 운영 정책을 자동 추천받을 수 있다.

## Azure Monitor / Log Analytics / Application Insights
Azure Monitor는 메트릭·로그를 수집하는 통합 플랫폼. Log Analytics Workspace와 Application Insights가 핵심 구성요소.

### 핵심 구성요소
- **Azure Monitor Metrics**: 리소스별 실시간 성능 지표(CPU, 네트워크 등) 수집, 짧은 보존 기간
- **Log Analytics Workspace**: KQL(Kusto Query Language)로 로그 분석, 장기 보존·복합 쿼리 지원
- **Application Insights**: APM(애플리케이션 성능 모니터링), 요청 추적, 예외 로깅, 종속성 맵 제공
- **Diagnostic Settings**: 리소스 로그를 Log Analytics/Storage/Event Hub로 라우팅하는 설정

### 모니터링 아키텍처
```
VM / App Service → Azure Monitor Agent → Log Analytics Workspace → App Insights / Workbook → Alert → 운영팀
```

> **사례**: 웹앱 응답 지연 발생 시 Application Insights의 종속성 맵으로 DB 호출 구간 지연을 즉시 식별.

## 운영 기준 KPI 선정
좋은 KPI는 **SLI(Service Level Indicator)** 에 기반하며, 목표(SLO)와 허용 오차(Error Budget)를 함께 정의.

### KPI 유형
- **가용성 KPI**: 성공 요청 비율(예: 99.9% 이상)
- **성능 KPI**: P95/P99 응답 시간(예: 500ms 이내)
- **처리량·오류율 KPI**: 초당 요청 수(RPS), 큐 대기 시간, 5xx 에러 비율(예: 0.1% 미만)
- KPI별 알림 임계값을 설정하고 Azure Monitor Alert Rule로 자동화

> **실무 팁**: 모든 지표를 KPI로 삼기보다 비즈니스에 직결되는 **3~5개 핵심 지표**에 집중.

## AI 기반 운영 정책 자동 추천 절차
1. **서비스 특성 정리**: SLA 목표, 트래픽 패턴, 장애 허용 범위 문서화
2. **AI 질의**: Gen AI에 KPI 목표와 함께 알림 정책·대시보드 구성 추천 요청
3. **정책 검증**: 추천 임계값이 과거 트래픽 데이터와 부합하는지 확인
4. **Azure Monitor 적용**: Alert Rule, Action Group, Workbook 대시보드로 구현

## Azure Monitor 데이터 보존 기간
| 구성요소 | 기본 보존 | 최대 연장 |
|---------|----------|----------|
| Log Analytics 일반 테이블 | 30일 | 최대 2년 (Analytics 플랜) |
| Usage / AzureActivity 테이블 | 90일 (무료) | 최대 2년 |
| Application Insights | 90일 (무료) | 최대 2년 |
| Azure Monitor Metrics | 93일 | 장기 보관 시 Storage/Log Analytics 연계 |

*출처: Microsoft Learn — Manage data retention in a Log Analytics workspace*

## 실습 Lab
> 🧪 CLI 핸즈온: [해당 실습 바로가기](../labs/day2-labs.md#lab-7--운영모니터링-구조-설계)

- **시나리오**: P95 응답시간 500ms, 가용성 99.9% 목표인 API 서비스의 모니터링 정책을 Gen AI로 설계
- **프롬프트 예시**:
  > "Azure App Service로 운영되는 API 서비스의 목표가 P95 응답시간 500ms 이내, 가용성 99.9%다. Azure Monitor 기반으로 추적해야 할 핵심 메트릭과 Alert Rule 임계값, 로그 보존 기간을 제안해줘."
- **점검 포인트**: 메트릭이 가용성·성능 목표 커버 여부 · Alert 임계값 민감/둔감 여부 · 로그 보존 기간의 컴플라이언스 충족

## 핵심 요약
- **Azure Monitor**: 메트릭·로그 통합 관리 플랫폼
- **Log Analytics/KQL**: 심층 로그 분석의 핵심 도구
- **Application Insights**: 애플리케이션 성능·종속성 추적
- **KPI 설계**: SLI/SLO/Error Budget 기반 핵심 지표 3~5개 선정
- **AI 활용**: 트래픽 패턴 기반 알림·대시보드 정책 자동 추천

## 공식문서
- [Azure Monitor 개요](https://learn.microsoft.com/en-us/azure/azure-monitor/overview)
- [Log Analytics 데이터 보존 관리](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/data-retention-configure)
- [Get started with log queries](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/get-started-queries)
