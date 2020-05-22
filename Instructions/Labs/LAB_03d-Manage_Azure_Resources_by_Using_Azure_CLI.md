---
lab:
    title: '03d - Azure CLI를 사용해 Azure 리소스 관리'
    module: '모듈 03 - Azure 관리'
---

# 랩 03d - Azure CLI를 사용해 Azure 리소스 관리
# 학생 실습 매뉴얼

## 랩 시나리오

Azure 포털, Azure Resource Manager 템플릿, Azure PowerShell을 사용하여 리소스 그룹을 기반으로 리소스를 프로비저닝하고 구성하는 기본 Azure 관리 기능을 탐색했으므로, Azure CLI를 사용하여 동등한 작업을 수행합니다. Azure CLI를 설치하지 않으려면 Azure Cloud Shell에서 사용 가능한 Bash 환경을 활용하십시오.

## 목표

이 과정에서, 우리는 다음과 같은 실습을 합니다 :

+ 작업 1: Azure Cloud Shell에서 Bash 세션을 시작
+ 작업 2: Azure CLI를 사용하여 리소스 그룹과 Azure 관리 디스크 생성
+ 작업 3: Azure CLI를 사용하여 관리 디스크 구성

## 설명

### 연습 1

#### 작업 1: Azure Cloud Shell에서 Bash 세션을 시작

이 작업에서는 Cloud Shell의 Bash 세션을 이용합니다.

1. Azure 포털 오른쪽 위의 아이콘을 클릭하여 **Azure Cloud Shell**을 실행한다.

1. **Bash** 또는 **PowerShell**을 선택하는 프롬프트 창에서 **Bash**를 선택한다. 

    >**참고**:  **Cloud Shell**을 처음 실행한 경우, **탑재된 스토리지가 없음** 메시지가 표시됩니다. 이 랩에서 사용하고 있는 구독을 선택하고 **스토리지 만들기**를 클릭하십시오. 

1. 메시지가 나타나면 **스토리지 만들기**를 클릭하고, Azure Cloud Shell 창이 표시될 때까지 기다린다.

1. 왼쪽 위의 드롭다운 메뉴에 **Bash**가 표시되는지 확인한다.


#### 작업 2:  Azure CLI를 사용하여 리소스 그룹과 Azure 관리 디스크 생성

이 작업에서는 Cloud Shell 내의 Azure CLI 세션을 사용해 리소스 그룹과 Azure 관리 디스크를 생성할 것입니다. 

1. 이전 랩에서 만들었던 **az104-03c-rg1** 리소스 그룹과 같은 지역에 새 리소스 그룹을 만들기 위해 Cloud Shell의 Bash 세션에서 다음 명령어를 실행한다.

   ```sh
   LOCATION=$(az group show --name 'az104-03c-rg1' --query location --out tsv)

   RGNAME='az104-03d-rg1'

   az group create --name $RGNAME --location $LOCATION
   ```
1. 새로 생성한 리소스 그룹의 속성을 검색하려면 다음 명령을 실행한다.

   ```sh
   az group show --name $RGNAME
   ```
1. 이 모듈의 지난 랩에서 만들었던 관리 디스크와 같은 속성을 가진 새 디스크를 만들기 위해 Cloud Shell의 Bash 세션에서 다음 명령어를 실행한다.

   ```sh
   DISKNAME='az104-03d-disk1'

   az disk create \
   --resource-group $RGNAME \
   --name $DISKNAME \
   --sku 'Standard_LRS' \
   --size-gb 32
   ```
>**참고**: 여러 줄로 된 구문을 사용할 때는 각 줄이 후행 공백 없이 백슬래시("\")로 끝나는지, 각 줄의 시작 부분에 공백이 없는지 확인하십시오.

1. 새로 생성한 디스크의 속성을 검색하려면 다음 명령을 실행한다.

   ```sh
   az disk show --resource-group $RGNAME --name $DISKNAME
   ```

#### 작업 3: Azure CLI를 사용하여 리소스 그룹과 Azure 관리 디스크 생성

이 작업에서는 Cloud Shell 내의 Azure CLI 세션을 사용해 Azure 관리 디스크의 구성을 관리할 것입니다. 

1. Azure 관리 디스크의 크기를 **64GB**로 늘리기 위해 Cloud Shell의 Bash 세션에서 다음 명령어를 실행한다.

   ```sh
   az disk update --resource-group $RGNAME --name $DISKNAME --size-gb 64
   ```

1. 변경 사항이 적용되었는지 확인하려면 다음을 실행한다.

   ```sh
   az disk show --resource-group $RGNAME --name $DISKNAME --query diskSizeGb
   ```

1. 디스크 성능 SKU를 **Premium_LRS**로 변경하려면, Cloud Shell의 Bash 세션에서 다음 명령어를 실행한다.

   ```sh
   az disk update --resource-group $RGNAME --name $DISKNAME --sku 'Premium_LRS'
   ```

1. 변경 사항이 적용되었는지 확인하려면 다음을 실행한다.

   ```sh
   az disk show --resource-group $RGNAME --name $DISKNAME --query sku
   ```

#### 리소스 삭제

   >**참고**: 사용하지 않는 새로 생성된 Azure 리소스를 제거하십시오. 사용하지 않는 리소스를 제거해야 예상치 못한 비용이 발생하지 않습니다.

1. Azure 포털에서 **Cloud Shell**의 **Bash** shell 세션을 시작한다.

1. 다음 명령을 실행하여 이 모듈의 실습에서 생성된 모든 리소스 그룹을 나열한다.

   ```sh
   az group list --query "[?starts_with(name,'az104-03')].name" --output tsv
   ```

1. 다음 명령을 실행하여 이 모듈의 실습에서 생성한 모든 리소스 그룹을 삭제한다.

   ```sh
   az group list --query "[?starts_with(name,'az104-03')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

>**참고**: 이 명령은 비동기적으로 실행되므로( --nowait 매개 변수로 결정됨) 동일한 Bash 세션 내에서 즉시 다른 Azure CLI 명령을 실행할 수 있지만, 리소스 그룹이 실제로 제거되기까지는 몇 분 정도 소요됩니다.

#### 요약

이 랩에서 우리는

- Azure Cloud Shell에서 Bash 세션을 시작했습니다.
- Azure CLI를 이용해 리소스 그룹과 Azure 관리 디스크를 생성했습니다.
- Azure CLI를 이용해 관리 디스크를 구성했습니다.