---
lab:
    title: '03c - Azure PowerShell을 사용해 Azure 리소스 관리'
    module: '모듈 03 - Azure 관리'
---

# 랩 03c - Azure PowerShell을 사용해 Azure 리소스 관리

# 학생 실습 매뉴얼

## 랩 시나리오

Azure 포털과 Azure Resource Manager 템플릿을 사용하여 리소스 그룹을 기반으로 리소스를 프로비저닝하고 구성하는 기본 Azure 관리 기능을 탐색했으므로, Azure PowerShell을 사용하여 동등한 작업을 수행합니다. Azure PowerShell 모듈을 설치하지 않으려면 Azure Cloud Shell에서 사용 가능한 PowerShell 환경을 활용하십시오.

## 목표

이 과정에서, 우리는 다음과 같은 실습을 합니다 :

+ 작업 1: Azure Cloud Shell에서 PowerShell 세션을 시작
+ 작업 2: Azure PowerShell을 사용하여 리소스 그룹과 Azure 관리 디스크 생성
+ 작업 3: Azure PowerShell을 사용하여 관리 디스크 구성

## 설명

### 연습 1

#### 작업 1: Azure Cloud Shell에서 PowerShell 세션을 시작

이 작업에서는 Cloud Shell의 PowerShell 세션을 이용합니다.

1. Azure 포털 오른쪽 위의 아이콘을 클릭하여 **Azure Cloud Shell**을 실행한다.

1. **Bash** 또는 **PowerShell**을 선택하는 프롬프트 창에서 **PowerShell**을 선택한다. 

    >**참고**: **Cloud Shell**을 처음 실행한 경우, **탑재된 스토리지가 없음** 메시지가 표시됩니다. 이 랩에서 사용하고 있는 구독을 선택하고 **스토리지 만들기**를 클릭하십시오. 

1. 메시지가 나타나면 **스토리지 만들기**를 클릭하고, Azure Cloud Shell 창이 표시될 때까지 기다린다. 

1. Cloud Shell 왼쪽 위의 드롭다운 메뉴에 **PowerShell**이 표시되는지 확인한다.

#### 작업 2: Azure PowerShell을 이용해 리소스 그룹과 Azure 관리 디스크 생성

이 작업에서는 Cloud Shell 내의 Azure PowerShell 세션을 사용해 리소스 그룹과 Azure 관리 디스크를 생성할 것입니다. 

1. 이전 랩에서 만들었던 **az104-03b-rg1** 리소스 그룹과 같은 지역에 새 리소스 그룹을 만들기 위해 Cloud Shell의 PowerShell 세션에서 다음 명령어를 실행한다.

   ```pwsh
   $location = (Get-AzResourceGroup -Name az104-03b-rg1).Location

   $rgName = 'az104-03c-rg1'

   New-AzResourceGroup -Name $rgName -Location $location
   ```
1. 새로 생성한 리소스 그룹의 속성을 검색하려면 다음 명령을 실행한다.

   ```pwsh
   Get-AzResourceGroup -Name $rgName
   ```
1. 이 모듈의 지난 랩에서 만들었던 관리 디스크와 같은 속성을 가진 새 디스크를 만들기 위해 다음 명령어를 실행한다.

   ```pwsh
   $diskConfig = New-AzDiskConfig `
    -Location $location `
    -CreateOption Empty `
    -DiskSizeGB 32 `
    -Sku Standard_LRS

   $diskName = 'az104-03c-disk1'

   New-AzDisk `
    -ResourceGroupName $rgName `
    -DiskName $diskName `
    -Disk $diskConfig
   ```

1. 새로 생성한 디스크의 속성을 검색하려면 다음 명령을 실행한다. 

   ```pwsh
   Get-AzDisk -ResourceGroupName $rgName -Name $diskName
   ```

#### 작업 3: Azure PowerShell을 사용하여 관리 디스크 구성

이 작업에서는 Cloud Shell 내의 Azure PowerShell 세션을 사용해 Azure 관리 디스크의 구성을 관리할 것입니다. 

1. Azure 관리 디스크의 크기를 **64GB**로 늘리기 위해 Cloud Shell의 PowerShell 세션에서 다음 명령어를 실행한다.

   ```pwsh
   New-AzDiskUpdateConfig -DiskSizeGB 64 | Update-AzDisk -ResourceGroupName $rgName -DiskName $diskName
   ```

1. 변경 사항이 적용되었는지 확인하려면 다음을 실행한다.

   ```pwsh
   Get-AzDisk -ResourceGroupName $rgName -Name $diskName
   ```

1. 디스크 성능 SKU를 **Premium_LRS**로 변경하려면, Cloud Shell의 PowerShell 세션에서 다음 명령어를 실행한다.

   ```pwsh
   New-AzDiskUpdateConfig -Sku Premium_LRS | Update-AzDisk -ResourceGroupName $rgName -DiskName $diskName
   ```

1. 변경 사항이 적용되었는지 확인하려면 다음을 실행한다.

   ```pwsh
   (Get-AzDisk -ResourceGroupName $rgName -Name $diskName).Sku
   ```

#### 리소스 삭제

   >**참고**: 이 랩에서 배포한 리소스를 삭제하지 마십시오. 이 모듈의 다음 랩에서 참조할 것입니다.

#### 요약

이 랩에서 우리는

- Azure Cloud Shell에서 PowerShell 세션을 시작했습니다.
- Azure PowerShell을 이용해 리소스 그룹과 Azure 관리 디스크를 생성했습니다. 
- Azure PowerShell을 이용해 관리 디스크를 구성했습니다. 