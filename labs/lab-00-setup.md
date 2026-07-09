# Lab 0. 공통 환경 구성

> 사용 도구: Azure Cloud Shell(브라우저 터미널) · Azure Portal · ChatGPT/Gemini(무료) · [Mermaid Live Editor](https://mermaid.live)

모든 Lab에서 공통으로 사용하는 환경입니다. 실습 시작 전 한 번만 진행합니다.

## Task 0-1. Azure 로그인 및 Cloud Shell 활성화
1. [https://portal.azure.com/](https://portal.azure.com/) 접속 → 제공 계정으로 로그인
2. 상단 `>_` (Cloud Shell) 아이콘 클릭 → 최초 실행 시 **Bash** 선택
3. 스토리지 마운트는 기본값(임시 스토리지 없이 계속 / 자동 생성)으로 진행
4. 로그인 계정·구독 확인:
   ```bash
   az account show --output table
   ```
   `Name`, `SubscriptionId`, `TenantId` 등이 표로 출력되면 정상.

> 💡 Cloud Shell은 브라우저에서 바로 열리는 리눅스 터미널로, 로컬에 Azure CLI 설치가 불필요합니다(최신 `az` 사전 설치됨).

## Task 0-2. 공통 리소스 그룹 및 변수
Day 1의 모든 실습은 하나의 리소스 그룹(`$USER-rg`) 안에서 진행합니다.
```bash
az group create --name $USER-rg --location koreacentral

USER=$USER
echo $USER
```
`"provisioningState": "Succeeded"`가 포함된 JSON이 나오면 정상.

> 💡 스토리지 계정·App Service·SQL 서버 이름은 **전역적으로 고유**해야 하므로 `$USER` 접미사로 충돌을 방지합니다. 리전은 편의상 `koreacentral`로 통일.

## Task 0-3. GenAI 계정 준비
1. [https://chat.openai.com](https://chat.openai.com/) (또는 허용된 Gemini 등) 로그인
2. 새 대화창을 하나 열어 Day 내내 같은 창에서 프롬프트를 이어서 입력

> 💡 같은 대화창을 유지하면 이전 맥락(리소스 그룹·리전 등)을 AI가 기억해 조건 반복 입력이 줄어듭니다.

## ✅ Lab 0 완료 확인
- [ ] `az account show` 결과에 내 구독 이름이 출력된다
- [ ] `az group create` 결과에 `"provisioningState": "Succeeded"`가 보인다
- [ ] GenAI 대화창이 열려 있다

---

## Day 2 추가 환경 (Lab 5~8 시작 전)
Day 2는 "운영 중인 서비스를 관측한다"는 시나리오라 관측 대상 App Service와 모니터링 스택을 먼저 띄웁니다.

```bash
# (리소스 그룹은 Day 1과 동일 이름/리전 재사용)
az group create --name $USER-rg --location koreacentral

# 샘플 Web App
az appservice plan create --name plan-lab-web --resource-group $USER-rg --sku B1 --is-linux
az webapp create --resource-group $USER-rg --plan plan-lab-web --name webapp-lab-$USER --runtime "NODE:18-lts"
curl -I https://webapp-lab-$USER.azurewebsites.net   # HTTP/2 200 확인

# Log Analytics + Application Insights
az monitor log-analytics workspace create --resource-group $USER-rg --workspace-name law-lab --location koreacentral
az extension add -n application-insights --upgrade
az monitor app-insights component create --app appi-lab --location koreacentral \
  --resource-group $USER-rg --workspace law-lab --application-type web

# Instrumentation Key 조회 후 Web App 연결
IKEY=$(az monitor app-insights component show --app appi-lab --resource-group $USER-rg --query instrumentationKey -o tsv)
az webapp config appsettings set --resource-group $USER-rg --name webapp-lab-$USER \
  --settings APPINSIGHTS_INSTRUMENTATIONKEY=$IKEY ApplicationInsightsAgent_EXTENSION_VERSION=~3
```

> ✅ Portal의 `appi-lab` 개요 화면에서 "라이브 메트릭"이 활성화되면 연결 정상. 이 연결 구조가 곧 Section 7의 **Diagnostic Settings** 개념입니다.

---

> ⚠️ **비용/정리**: 실습 종료 후에는 반드시 리소스 그룹을 삭제해 비용 발생을 막습니다.
> ```bash
> az group delete --name $USER-rg --yes --no-wait
> ```
