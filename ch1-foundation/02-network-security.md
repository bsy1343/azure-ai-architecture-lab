# Section 2. 네트워크·보안 설계

## 학습 목표
- VNet 설계 패턴과 서브넷 구성 전략을 수립할 수 있다.
- Azure AD(Entra ID) 기반 인증과 NSG 기반 트래픽 제어 원리를 이해한다.
- 요구조건으로부터 AI를 활용해 보안정책안을 자동 생성할 수 있다.

## VNet 설계 패턴 / 서브넷 구성
- VNet 주소공간은 향후 확장(Peering, 온프레미스 연결)을 고려해 여유 있게 설계
- Public/Private Subnet 분리로 인터넷 노출 리소스 최소화가 기본 원칙

### 핵심 구성요소
- **Hub-Spoke 패턴**: 중앙 Hub VNet에 공통 서비스(방화벽, VPN Gateway) 배치, Spoke VNet에서 워크로드 분리 운영
- **3-Tier Subnet**: Web/App/Data Subnet 분리, Subnet 간 트래픽은 NSG로 통제
- **VNet Peering**: 동일·타 리전 VNet 간 저지연 연결. ⚠️ 전이적(Transitive) 라우팅 미지원
- **Private Endpoint**: PaaS 서비스(Storage, SQL DB 등)를 VNet 내부 사설 IP로 연결해 인터넷 노출 차단

### 보안 구성 흐름
```
인터넷 → Application Gateway (Public Subnet) → NSG 필터링 → VM/VMSS (Private Subnet) → Azure AD (RBAC 인증)
```

> **사례**: Hub-Spoke 구조에서 Azure Firewall을 Hub에 배치하면 모든 Spoke의 아웃바운드 트래픽을 중앙에서 통제.

### Hub-Spoke 토폴로지
```
                Hub VNet
        (Azure Firewall + VPN/ExpressRoute Gateway)
                    │ Peering
        ┌───────────┴───────────┐
   Spoke VNet A            Spoke VNet B
   Web/App (Private)       Data Tier (Private)
      + NSG                   + NSG
```
모든 아웃바운드·인터넷 트래픽은 Hub의 Azure Firewall을 경유해 중앙 통제.

## Entra ID(Azure AD) · NSG 보안 원리
- **Entra ID**: 신원(Identity) 관리 중심. RBAC와 결합해 최소 권한 원칙 구현
- **NSG**: Subnet·NIC 단위로 인바운드/아웃바운드 필터링하는 **상태 저장(Stateful) 방화벽**

### 핵심 구성요소
- **Azure RBAC**: 구독/리소스 그룹/리소스 범위(Scope)별 소유자·기여자·읽기 권한 할당
- **조건부 액세스**: 위치·기기 상태·위험도 기반 로그인 정책
- **NSG 규칙**: 우선순위(Priority) 숫자가 낮을수록 먼저 적용, 5-tuple 기반 필터링
- **Azure Firewall/WAF**: NSG보다 상위 계층에서 SQL Injection, XSS 등 앱 수준 위협 방어

> **실무 팁**: AWS Security Group(인스턴스 단위)과 달리 Azure NSG는 Subnet과 NIC 두 곳에 모두 적용 가능 → 규칙 중복·충돌에 주의.

### NSG 기본 보안 규칙 (인바운드)
| 우선순위 | 규칙 이름 | 동작 | 설명 |
|---------|----------|------|------|
| 65000 | AllowVnetInBound | 허용 | 동일 VNet 내부 트래픽 허용 |
| 65001 | AllowAzureLoadBalancerInBound | 허용 | LB Health Probe 트래픽 허용 |
| 65500 | DenyAllInBound | 차단 | 그 외 모든 인바운드 차단 |

*기본 규칙은 삭제 불가, 더 높은 우선순위 규칙으로만 재정의 가능. (출처: Microsoft Learn — NSG overview)*

## AI 보안정책안 자동 생성 절차
1. **자산 분류**: Subnet별 리소스 유형·민감도 등급 정리
2. **요구조건 입력**: 허용 포트/프로토콜, 접근 주체(팀, 서비스)를 Gen AI에 전달
3. **초안 생성**: NSG 규칙 목록(우선순위·방향·포트·소스) 및 RBAC 역할 할당안 도출
4. **최소권한 검증**: 광범위한 규칙(`0.0.0.0/0` 등)이 없는지 재검토 후 적용

## 실습 Lab
> 🧪 CLI 핸즈온: [해당 실습 바로가기](../labs/day1-labs.md#lab-2--네트워크보안-설계)

- **시나리오**: 3-Tier 아키텍처(Web/App/Data Subnet)에 대한 NSG 규칙안을 Gen AI로 도출
- **프롬프트 예시**:
  > "Azure 3-Tier 아키텍처(Web/App/Data Subnet)에 대해 각 Subnet의 NSG 인바운드/아웃바운드 규칙을 표로 제안해줘. Web Subnet은 인터넷에서 443만 허용, App Subnet은 Web Subnet에서만, Data Subnet은 App Subnet에서만 접근을 허용하는 최소권한 원칙을 적용해줘."
- **점검 포인트**: Subnet 간 최소권한 준수 · 우선순위 충돌/중복 · 관리용 포트(RDP/SSH) 인터넷 개방 여부

## 핵심 요약
- **Hub-Spoke / 3-Tier**: 대표적인 VNet 설계 패턴
- **Private Endpoint**: PaaS 사설 연결로 노출 최소화
- **Entra ID + RBAC**: 신원 기반 최소권한 접근 제어의 핵심
- **NSG**: Subnet/NIC 단위 상태 저장 트래픽 필터링
- **AI 활용**: 요구조건 기반 보안정책 초안 생성으로 검토 시간 단축

## 공식문서
- [Network security groups 개요](https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview)
- [Hub-spoke network topology](https://learn.microsoft.com/en-us/azure/architecture/networking/architecture/hub-spoke)
- [Azure networking services 개요](https://learn.microsoft.com/en-us/azure/networking/networking-overview)
