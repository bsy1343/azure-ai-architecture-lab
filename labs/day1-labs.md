# Day 1 실습 — 기초 아키텍처 설계 (Lab 1~4)

> 대응 이론: [Chapter 1](../ch1-foundation/) · 먼저 [Lab 0 환경 구성](./lab-00-setup.md)을 완료하세요.
> 사용 도구: Azure Cloud Shell · Azure Portal · ChatGPT/Gemini · [Mermaid Live](https://mermaid.live)

각 Lab은 **CLI로 직접 리소스 생성 → GenAI Before/After 비교 → 체크포인트 검증** 흐름입니다.

---

## Lab 1 — 클라우드 아키텍처 기본 및 AI 페르소나 설정
> 이론: [Section 1](../ch1-foundation/01-basics-and-ai-persona.md)
> **목표**: VNet·Subnet·Load Balancer를 직접 생성하고, ChatGPT로 기본 아키텍처 초안을 도출.

### Task 1. VNet과 3-Tier Subnet 생성
```bash
# VNet + Web Subnet 동시 생성
az network vnet create \
  --resource-group $USER-rg \
  --name vnet-lab \
  --address-prefix 10.10.0.0/16 \
  --subnet-name snet-web \
  --subnet-prefix 10.10.1.0/24

# App Subnet 추가
az network vnet subnet create \
  --resource-group $USER-rg \
  --vnet-name vnet-lab \
  --name snet-app \
  --address-prefix 10.10.2.0/24

# 확인
az network vnet subnet list --resource-group $USER-rg --vnet-name vnet-lab --output table
```
**예상 결과**: `snet-web`(10.10.1.0/24), `snet-app`(10.10.2.0/24) 두 Subnet이 표로 출력.

### Task 2. Standard Load Balancer 생성
> 💡 교재는 L7에 Application Gateway를 쓰지만, 실습에서는 L4 **Standard Load Balancer**로 트래픽 분산 개념을 익힙니다. App Gateway는 Task 4에서 개념만 질의.

```bash
# 프런트엔드 공용 IP
az network public-ip create --resource-group $USER-rg --name pip-lb-web --sku Standard --allocation-method Static

# Load Balancer
az network lb create --resource-group $USER-rg --name lb-web --sku Standard \
  --public-ip-address pip-lb-web --frontend-ip-name feip-web --backend-pool-name bepool-web

# Health Probe + 분산 규칙
az network lb probe create --resource-group $USER-rg --lb-name lb-web --name probe-http --protocol Tcp --port 80
az network lb rule create --resource-group $USER-rg --lb-name lb-web --name rule-http \
  --protocol Tcp --frontend-port 80 --backend-port 80 \
  --frontend-ip-name feip-web --backend-pool-name bepool-web --probe-name probe-http
```
> ✅ `az network lb rule list --resource-group $USER-rg --lb-name lb-web --output table`에서 `rule-http` 조회 시 성공. 교재의 "Health Probe로 상태 점검"이 이 단계.

### Task 3. Azure Functions로 서버리스 체험
```bash
az storage account create --name stlabfn$USER --resource-group $USER-rg --location koreacentral --sku Standard_LRS

az functionapp create --resource-group $USER-rg --consumption-plan-location koreacentral \
  --runtime node --runtime-version 22 --functions-version 4 \
  --name func-lab-$USER --storage-account stlabfn$USER
```
**예상 결과**: `"state": "Running"` (1~2분 소요). Consumption Plan은 요청 없을 때 0으로 축소 → 교재의 "트리거 기반 자동 스케일(0까지 축소)"을 실제 확인.

### Task 4. GenAI Before/After — 요구사항 → 기본 아키텍처
```
# Before (모호한 질문)
Azure로 이커머스 웹사이트 아키텍처 만들어줘

# After (구체적 프롬프트 — Section 1 실습과 동일)
너는 Azure 클라우드 아키텍트야. 월간 방문자 50만 명, 피크 시간대 트래픽 집중,
예산은 월 500만원 이내인 이커머스 웹서비스의 기본 아키텍처를 VNet, Subnet,
컴퓨팅, 스토리지, DB 구성 중심으로 제안해줘. 고가용성과 비용 효율을 함께 고려해줘.
```
- [ ] 요구사항(트래픽·예산·가용성) 충족 여부
- [ ] 보안·모니터링 누락 여부
- [ ] Azure 서비스명-역할 매칭 정확성

### Task 5. Mermaid-AI로 다이어그램 생성
```
사용자 → Application Gateway → Web Tier(VMSS) → App Tier(VMSS) → Azure SQL Database
로 이어지는 3-Tier 아키텍처를 Mermaid graph TD 문법으로 그려줘
```
생성 코드를 [mermaid.live](https://mermaid.live)에 붙여 렌더링 → Section 1의 "3-Tier 기본 아키텍처"와 논리 일치 비교.

### Task 6. 리소스 정리 (선택)
```bash
az functionapp delete --resource-group $USER-rg --name func-lab-$USER
```
> VNet과 Load Balancer는 Lab 2~4에서 계속 사용하므로 남겨둡니다.

### ✅ Lab 1 완료 확인
- [ ] `snet-web`, `snet-app` 생성됨
- [ ] LB `rule-http` 규칙 정상 조회
- [ ] Function App `Running` 상태
- [ ] ChatGPT 모호 vs 구체 질문 결과 차이 비교
- [ ] mermaid.live에서 3-Tier 다이어그램 렌더링

---

## Lab 2 — 네트워크·보안 설계
> 이론: [Section 2](../ch1-foundation/02-network-security.md)
> **목표**: NSG로 최소권한 트래픽 정책을 구성하고 RBAC 역할 구조 확인.

### Task 1. Data Subnet 추가 (3-Tier 완성)
```bash
az network vnet subnet create --resource-group $USER-rg --vnet-name vnet-lab \
  --name snet-data --address-prefix 10.10.3.0/24
```

### Task 2. NSG 생성 및 최소권한 규칙
```bash
az network nsg create --resource-group $USER-rg --name nsg-web

# 인터넷에서 443만 허용
az network nsg rule create --resource-group $USER-rg --nsg-name nsg-web \
  --name Allow-HTTPS-Inbound --priority 100 --direction Inbound --access Allow \
  --protocol Tcp --destination-port-ranges 443 --source-address-prefixes Internet

# Web Subnet에 연결
az network vnet subnet update --resource-group $USER-rg --vnet-name vnet-lab \
  --name snet-web --network-security-group nsg-web

az network nsg rule list --resource-group $USER-rg --nsg-name nsg-web --output table
```
> 💡 우선순위 숫자가 낮을수록 먼저 적용. 기본 규칙(`AllowVnetInBound` 65000, `AllowAzureLoadBalancerInBound` 65001, `DenyAllInBound` 65500)보다 낮은 값을 써야 우리 규칙이 먼저 평가됩니다.

### Task 3. GenAI Before/After — 3-Tier NSG 규칙안
```
Azure 3-Tier 아키텍처(Web/App/Data Subnet)에 대해 각 Subnet의 NSG
인바운드/아웃바운드 규칙을 표 형태로 제안해줘. Web Subnet은 인터넷에서
443만 허용, App Subnet은 Web Subnet에서만 접근 허용, Data Subnet은
App Subnet에서만 접근을 허용하는 최소권한 원칙을 적용해줘.
```
제안받은 App Subnet 규칙 하나를 실제 구현:
```bash
az network nsg create --resource-group $USER-rg --name nsg-app
az network nsg rule create --resource-group $USER-rg --nsg-name nsg-app \
  --name Allow-From-Web-Subnet --priority 100 --direction Inbound --access Allow \
  --protocol Tcp --destination-port-ranges 8080 --source-address-prefixes 10.10.1.0/24
az network vnet subnet update --resource-group $USER-rg --vnet-name vnet-lab \
  --name snet-app --network-security-group nsg-app
```
- [ ] Subnet 간 최소권한 준수
- [ ] 우선순위 충돌/중복 없음
- [ ] 관리용 포트(RDP 3389/SSH 22) 인터넷 미개방

### Task 4. Azure RBAC 역할 확인
```bash
az role assignment list --output table
az role definition list --name "Contributor" --output table
```
> 💡 NSG가 "네트워크 계층"을 통제한다면, RBAC는 "누가 이 리소스를 관리할 수 있는가"를 통제. Entra ID 신원에 RBAC를 매핑하는 것이 최소권한 접근 제어의 핵심.

### ✅ Lab 2 완료 확인
- [ ] `snet-data` 추가로 3-Tier 완성
- [ ] `nsg-web`에 `Allow-HTTPS-Inbound` 적용·연결
- [ ] ChatGPT 제안 NSG 규칙 중 하나 구현
- [ ] `az role assignment list` 결과 확인

---

## Lab 3 — 컴퓨팅·스토리지 설계
> 이론: [Section 3](../ch1-foundation/03-compute-storage.md)
> **목표**: VMSS 오토스케일 구성, Blob 계층·Azure SQL 서버리스 실습.
> ⚠️ VM·SQL을 생성하므로 Lab 4 마지막에 반드시 리소스 그룹 정리.

### Task 1. VMSS 생성 (Lab 1 LB 백엔드 풀 연결)
```bash
az vmss create --resource-group $USER-rg --name vmss-web \
  --image Ubuntu2204 --admin-username azureuser --generate-ssh-keys \
  --instance-count 2 --vm-sku Standard_B2s \
  --vnet-name vnet-lab --subnet snet-web \
  --lb lb-web --backend-pool-name bepool-web --upgrade-policy-mode Automatic
```
**예상 결과**: 2~3분 후 `Succeeded`. `az vmss list-instances ... --output table`로 2대 조회.

### Task 2. CPU 기준 오토스케일 규칙
```bash
az monitor autoscale create --resource-group $USER-rg --resource vmss-web \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale-web --min-count 2 --max-count 6 --count 2

az monitor autoscale rule create --resource-group $USER-rg --autoscale-name autoscale-web \
  --condition "Percentage CPU > 70 avg 5m" --scale out 2
az monitor autoscale rule create --resource-group $USER-rg --autoscale-name autoscale-web \
  --condition "Percentage CPU < 30 avg 5m" --scale in 1
```

### Task 3. Blob Storage 계층(Hot/Cool)
```bash
az storage account create --name stlabdata$USER --resource-group $USER-rg \
  --location koreacentral --sku Standard_LRS --kind StorageV2 --access-tier Hot
az storage container create --account-name stlabdata$USER --name images --auth-mode login
```
> 💡 수명 주기 관리(Hot→Cool→Archive 자동 이동)는 Portal의 스토리지 계정 > 수명 주기 관리, 또는 `az storage account management-policy create`로 구성.

### Task 4. GenAI Before/After — 트래픽 기준 스펙 산정
```
Azure VMSS로 피크 시간대 초당 1,000건 HTTP 요청을 처리해야 하는 웹
서비스가 있다. VM 1대(D2s v5)가 초당 80건을 처리할 수 있다고 가정할
때 필요한 최소/최대 인스턴스 수와 오토스케일 규칙(CPU 기준)을 제안해줘.
```
Task 2의 `--min-count 2 --max-count 6`과 비교:
- [ ] 인스턴스 수 여유율(1.2~1.3배) 포함
- [ ] 오토스케일 임계값(70%/30%) 합리성

### Task 5. Azure SQL Database 서버리스 (선택)
```bash
az sql server create --name sql-lab-$USER --resource-group $USER-rg \
  --location koreacentral --admin-user sqladmin --admin-password "본인만의_강력한_암호!"

az sql db create --resource-group $USER-rg --server sql-lab-$USER --name db-lab \
  --edition GeneralPurpose --family Gen5 --capacity 1 \
  --compute-model Serverless --auto-pause-delay 60
```
> 💡 `Serverless` + `auto-pause-delay 60` → 60분 무활동 시 자동 일시중지로 비용 최소화 (vCore 서버리스 모델 실체험).
> ⚠️ 비밀번호는 예시 그대로 쓰지 말고 대/소문자·숫자·특수문자 포함 8자 이상 본인 값으로 변경.

### ✅ Lab 3 완료 확인
- [ ] VMSS 인스턴스 2대 실행 중
- [ ] 오토스케일 규칙(확장/축소) 등록
- [ ] Blob 컨테이너 생성
- [ ] ChatGPT 인스턴스 수 계산 결과와 우리 설정 비교

---

## Lab 4 — 비용·고가용성·DR 설계
> 이론: [Section 4](../ch1-foundation/04-cost-ha-dr.md)
> **목표**: Azure Advisor로 절감 기회를 조회하고, AI로 절감안 도출 후 리소스 정리.

### Task 1. VMSS 인스턴스 분포 확인 (AZ 개념)
```bash
az vmss list-instances --resource-group $USER-rg --name vmss-web --output table
```
> 💡 리전/구독 정책상 AZ 지정 생성이 제한될 수 있음. 운영 환경에서는 `az vmss create` 시 `--zones 1 2 3`으로 3개 가용영역에 분산 → VM SLA 99.9% → 99.99% 향상.

### Task 2. Azure Advisor 권장사항 조회
```bash
az advisor recommendation list --output table
az advisor recommendation list --category Cost --output table
```
> 💡 생성 직후 계정은 권장사항이 거의 없을 수 있음 → Task 3의 예시 시나리오로 진행.

### Task 3. GenAI Before/After — 비용 절감안
```
Azure에서 D4s v5 VM 10대를 24시간 상시 운영 중이며 평균 CPU 사용률이
15%다. Right-sizing, 예약 인스턴스, Azure Hybrid Benefit 관점에서
비용 절감 방안을 구체적인 예상 절감률과 함께 제안해줘.
```
- [ ] 성능·가용성 영향 검토
- [ ] Right-sizing 후 목표 사용률(60~70%) 적절성
- [ ] 리스크 대비 효과 순 우선순위

### Task 4. DR 전략 구조 확인 + 리소스 다이어그램
```bash
# Paired Region 개념 확인
az account list-locations --output table
```
DR Tier 정리 질의:
```
Azure Site Recovery를 이용한 Backup&Restore, Pilot Light, Warm
Standby, Multi-Site(Active-Active) 4가지 DR 전략의 RTO/RPO와
예상 비용 수준을 표로 정리해줘
```
리소스 목록 → 다이어그램 자동 생성:
```bash
az resource list --resource-group $USER-rg --output table
```
```
아래 Azure 리소스 목록과 연결 관계를 참고해서 Mermaid graph TD로
아키텍처 다이어그램을 그려줘.
[위 표 붙여넣기]
VNet/Subnet은 subgraph로 중첩하고, 트래픽 흐름 순서(사용자→LB→
컴퓨팅→데이터)로 화살표 방향을 맞추고, 계층별로 색을 구분해줘.
```

### Task 5. 전체 리소스 정리
```bash
az group delete --name $USER-rg --yes --no-wait
az group show --name $USER-rg --output table   # 완료 시 ResourceNotFound
```
> ⚠️ `$USER-rg` 안 모든 리소스(VNet, VMSS, Storage, SQL 등)가 삭제됩니다.

### ✅ Lab 4 완료 확인 & 마무리
- [ ] Advisor 결과(또는 예시)로 AI 절감안 도출
- [ ] DR Tier별 RTO/RPO 정리
- [ ] `az group delete`로 리소스 정리
- [ ] Lab 1~4 전부 완료
