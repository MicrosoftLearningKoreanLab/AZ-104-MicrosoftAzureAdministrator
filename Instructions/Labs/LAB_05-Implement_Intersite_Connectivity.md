---
lab:
    title: '05 - Implement Intersite Connectivity'
    module: '모듈 05 - Intersite Connectivity'
---

# 랩 05 - 사이트 간 연결 구현
# 학생 실습 매뉴얼

## 랩 시나리오

Contoso는 보스턴, 뉴욕, 시애틀에 광역 메쉬 네트워크로 연결된 데이터 센터를 보유합니다. 이들은 완전히 연결되어 있습니다. Contoso의 온프레미스 네트워크 토폴로지를 반영하는 실험 환경을 구현하고 그 기능을 검증해야 합니다.

## 목표

이 과정에서, 우리는 다음과 같은 실습을 합니다 :

+ 작업 1: 랩 환경 프로비전
+ 작업 2: 로컬 및 글로벌 가상 네트워크 피어링 구성
+ 작업 3: 사이트 간 연결 테스트

#### 작업 1: 랩 환경 프로비전

이 작업에서는 세 개의 가상 머신을 분리된 네트워크에 배포합니다. 두개는 같은 Azure 지역에, 나머지 하나는 다른 Azure 지역에 배포합니다. 

1. [Azure portal](https://portal.azure.com)에 로그인한다.

1. Azure 포털 오른쪽 위의 아이콘을 클릭하여 **Azure Cloud Shell**을 실행한다.

1. **Bash** 또는 **PowerShell**을 선택하는 프롬프트 창에서 **PowerShell**을 선택한다. 

    >**참고**: **Cloud Shell**을 처음 실행한 경우, **탑재된 스토리지가 없음** 메시지가 표시됩니다. 이 랩에서 사용하고 있는 구독을 선택하고 **스토리지 만들기**를 클릭하십시오.

1. Cloud Shell 창의 툴바에서 **파일 업로드/다운로드** 아이콘을 선택한다. 드롭다운 메뉴에서 **업로드**를 클릭하고, **\\Allfiles\\Labs\\05\\az104-05-vnetvm-template.json** 와 **\\Allfiles\\Labs\\05\\az104-05-vnetvm-parameters.json**를 Cloud Shell의 홈 디렉토리에 업로드한다.

1. Cloud Shell 창에서 다음 명령을 실행하여 첫 번째 가상 네트워크와 가상 머신들을 호스팅할 리소스 그룹을 생성한다. (`[Azure_region_1]` 부분을 Azure 가상 머신을 배포할 Azure 지역의 이름으로 대체한다)

   ```pwsh
   $location = '[Azure_region_1]'

   $rgName = 'az104-05-rg0'

   New-AzResourceGroup -Name $rgName -Location $location
   ```
1. Cloud Shell 창에서 다음 명령을 실행하여 업로드한 템플릿과 파라미터 파일로 첫 번째 가상 네트워크를 만들고 가상 머신을 배포한다. 

   ```pwsh
   New-AzResourceGroupDeployment `
      -ResourceGroupName $rgName `
      -TemplateFile $HOME/az104-05-vnetvm-template.json `
      -TemplateParameterFile $HOME/az104-05-vnetvm-parameters.json `
      -nameSuffix 0 `
      -AsJob
   ```
1. Cloud Shell 창에서 다음 명령을 실행하여 두번째 가상 네트워크와 가상 머신을 배포할 두 번째 리소스 그룹을 생성한다.

   ```pwsh
   $rgName = 'az104-05-rg1'

   New-AzResourceGroup -Name $rgName -Location $location
   ```
1. Cloud Shell 창에서 다음 명령을 실행하여 업로드한 템플릿과 파라미터 파일로 두 번째 가상 네트워크를 만들고 가상 머신을 배포한다.  

   ```pwsh
   New-AzResourceGroupDeployment `
      -ResourceGroupName $rgName `
      -TemplateFile $HOME/az104-05-vnetvm-template.json `
      -TemplateParameterFile $HOME/az104-05-vnetvm-parameters.json `
      -nameSuffix 1 `
      -AsJob
   ```
1. Cloud Shell 창에서 다음 명령을 실행하여 세번째 가상 네트워크와 가상 머신을 배포할 세번째 리소스 그룹을 생성한다. (`[Azure_region_2]` 부분을 Azure 가상 머신을 배포할 Azure 지역의 이름으로 대체한다. 다른 가상 머신을 배포했던 지역과 다른 지역을 사용한다):

   ```pwsh
   $location = '[Azure_region_2]'

   $rgName = 'az104-05-rg2'

   New-AzResourceGroup -Name $rgName -Location $location
   ```
1. Cloud Shell 창에서 다음 명령을 실행하여 업로드한 템플릿과 파라미터 파일로 세 번째 가상 네트워크를 만들고 가상 머신을 배포한다.

   ```pwsh
   New-AzResourceGroupDeployment `
      -ResourceGroupName $rgName `
      -TemplateFile $HOME/az104-05-vnetvm-template.json `
      -TemplateParameterFile $HOME/az104-05-vnetvm-parameters.json `
      -nameSuffix 2 `
      -AsJob
   ```
    >**참고**: 다음 작업을 시작하기 전에 배포가 끝날 때까지 기다리십시오. 이 작업은 약 2분 소요됩니다. 

    >**참고**: 배포 상태를 확인하려면 이 작업에서 생성했던 리소스 그룹의 속성을 검토하십시오.
    

1. Cloud Shell 창을 닫는다. 

#### 작업 2: 로컬 및 글로벌 가상 네트워크 피어링 구성

이 작업에서는 이전 작업에서 배포했던 가상 네트워크 사이에 로컬 및 글로벌 피어링을 구성합니다. 

1. Azure 포털에서 **가상 네트워크**를 찾아 선택한다.

1. 이전 작업에서 배포했던 가상 네트워크를 검토한다. 먼저 배포했던 두 개의 가상 머신은 같은 지역에 배포되고, 나머지 가상 머신은 다른 Azure 지역에 배포되었는지 확인한다. 

    >**참고**: 세 개의 가상 네트워크를 배포할 때 사용했던 템플릿은 가상 네트워크의 IP 주소 범위가 중첩되지 않는 것을 보장합니다. 

1. 가상 네트워크 목록에서 **az104-05-vnet0**을 클릭한다.

1. **az104-05-vnet0** 가상 네트워크 블레이드에서 **설정** 섹션에 **피어링**을 선택하고 **+ 추가**를 클릭한다.

1. 다음 설정을 사용하여 피어링을 추가한다. (다른 값은 기본 설정을 사용한다)

    | 설정 | 값|
    | --- | --- |
    | az104-05-vnet0에서 원격 가상 네트워크(으)로 피어링 이름 | **az104-05-vnet0_to_az104-05-vnet1** |
    | 가상 네트워크 배포 모델 | **Resource manager** |
    | 구독 | 이 랩에서 사용할 구독의 이름 |
    | 가상 네트워크 | **az104-05-vnet1 (az104-05-rg1)** |
    | az104-05-vnet1에서 az104-05-vnet0(으)로 피어링 이름 | **az104-05-vnet1_to_az104-05-vnet0** |
    | az104-05-vnet0에서 az104-05-vnet1(으)로 가상 네트워크 액세스 허용 | **사용** |
    | az104-05-vnet1에서 az104-05-vnet0(으)로 가상 네트워크 액세스 허용 | **사용** |
    | az104-05-vnet1에서 az104-05-vnet0(으)로 전달되는 트래픽 허용 | **사용 안 함** |
    | az104-05-vnet0에서 az104-05-vnet1(으)로 전달되는 트래픽 허용 | **사용 안 함** |
    | 게이트웨이 전송 설정 구성 | **(체크하지 않음)** |

    >**참고**: 이 과정은 두개의 로컬 피어링을 설정합니다. - az104-05-vnet0 에서 az104-05-vnet1 로 향하는 피어링, az104-05-vnet1 에서 az104-05-vnet0로 향하는 피어링

1. **az104-05-vnet0** 가상 네트워크 블레이드에서 **설정** 섹션에 **피어링**을 선택하고 **+ 추가**를 클릭한다.

1. 다음 설정을 사용하여 피어링을 추가한다. (다른 값은 기본 설정을 사용한다)

    | 설정 | 값|
    | --- | --- |
    | az104-05-vnet0에서 원격 가상 네트워크(으)로 피어링 이름 | **az104-05-vnet0_to_az104-05-vnet2** |
    | 가상 네트워크 배포 모델 | **Resource manager** |
    | 구독 | 이 랩에서 사용할 구독의 이름 |
    | 가상 네트워크 | **az104-05-vnet2 (az104-05-rg2)** |
    | az104-05-vnet2에서 az104-05-vnet0(으)로 피어링 이름 | **az104-05-vnet2_to_az104-05-vnet0** |
    | az104-05-vnet0에서 az104-05-vnet2(으)로 가상 네트워크 액세스 허용 | **사용** |
    | az104-05-vnet2에서 az104-05-vnet0(으)로 가상 네트워크 액세스 허용 | **사용** |
    | az104-05-vnet2에서 az104-05-vnet0(으)로 전달되는 트래픽 허용 | **사용 안 함** |
    | az104-05-vnet0에서 az104-05-vnet2(으)로 전달되는 트래픽 허용 | **사용 안 함** |
    | 게이트웨이 전송 설정 구성 | **(체크하지 않음)** |

    >**참고**: 이 과정은 두개의 글로벌 피어링을 설정합니다. - az104-05-vnet0 에서 az104-05-vnet2 로 향하는 피어링,az104-05-vnet2 에서 az104-05-vnet0로 향하는 피어링

1. **가상 네트워크** 블레이드로 돌아가서 **az104-05-vnet1**를 클릭한다.

1. **az104-05-vnet1** 가상 네트워크 블레이드에서 **설정** 섹션에 **피어링**을 선택하고 **+ 추가**를 클릭한다.

1. 다음 설정을 사용하여 피어링을 추가한다. (다른 값은 기본 설정을 사용한다)

    | 설정 | 값|
    | --- | --- |
    | az104-05-vnet1에서 원격 가상 네트워크(으)로 피어링 이름 | **az104-05-vnet1_to_az104-05-vnet2** |
    | 가상 네트워크 배포 모델 | **Resource manager** |
    | 구독 | 이 랩에서 사용할 구독의 이름 |
    | 가상 네트워크 | **az104-05-vnet2 (az104-05-rg2)** |
    | az104-05-vnet2에서 az104-05-vnet1(으)로 피어링 이름 |  **az104-05-vnet2_to_az104-05-vnet1** |
    | az104-05-vnet1에서 az104-05-vnet2(으)로 가상 네트워크 액세스 허용 | **사용** |
    | az104-05-vnet2에서 az104-05-vnet1(으)로 가상 네트워크 액세스 허용 | **사용** |
    | az104-05-vnet2에서 az104-05-vnet1(으)로 전달되는 트래픽 허용 | **사용 안 함** |
    | az104-05-vnet1에서 az104-05-vnet2(으)로 전달되는 트래픽 허용 | **사용 안 함** |
    | 게이트웨이 전송 설정 구성 | **(체크하지 않음)** |

    >**참고**:  이 과정은 두개의 글로벌 피어링을 설정합니다. - az104-05-vnet1 에서 az104-05-vnet2 로 향하는 피어링, az104-05-vnet2 에서 az104-05-vnet1로 향하는 피어링

#### 작업 3: 사이트 간 연결 테스트

이 작업에서는 앞선 작업에서 로컬 및 글로벌 피어링을 통해 연결한 세 개의 가상 네트워크에 있는 가상 머신 사이의 연결성을 테스트합니다.

1. Azure 포털에서 **가상 머신**을 찾아 선택한다.

1. 가상 머신 목록에서 **az104-05-vm0**을 클릭한다.

1. **az104-05-vm0** 블레이드에서 **연결**을 클릭하고, **RDP**를 선택한다. **RDP를 사용하여 연결** 블레이드에서 **RDP 파일 다운로드**를 클릭하고 원격 데스크톱 세션을 시작한다. 

    >**참고**: 이 단계는 Windows 컴퓨터에서 원격 데스크톱을 통해 연결하는 것을 말합니다. Mac에서는 Mac App Store에서 Remote Desktop Client를 사용할 수 있으며, Linux 컴퓨터에서는 오픈 소스 RDP 클라이언트 소프트웨어를 사용할 수 있습니다.

    >**참고**: 가상 머신에 연결할 때 출력되는 경고 메시지는 무시할 수 있습니다.


1. 원격 데스크톱에 연결되면 **Student** 계정과 **Pa55w.rd1234** 패스워드를 사용하여 로그인한다.

1. **az104-05-vm0** 원격 데스크톱 세션 내의 **Start** 버튼을 우클릭하여, 메뉴에서 **Windows PowerShell (Admin)**를 클릭한다.

1. Windows PowerShell 콘솔 창에서 다음 명령을 사용하여 TCP 포트 3389를 통해 **az104-05-vm1**에 대한 연결성을 테스트한다. (사설 IP 주소 **10.51.0.4**)

   ```pwsh
   Test-NetConnection -ComputerName 10.51.0.4 -Port 3389 -InformationLevel 'Detailed'
   ```
    >**참고**: 테스트에 사용되는 TCP 3389 포트는 운영 체제 방화벽에 의해 기본적으로 허용됩니다.

1. 명령의 결과를 검토하고, 연결에 성공했음을 확인한다.

1. Windows PowerShell 콘솔 창에서 다음 명령을 사용하여 **az104-05-vm2**에 대한 연결성을 확인한다. (사설 IP 주소 **10.52.0.4**)

   ```pwsh
   Test-NetConnection -ComputerName 10.51.0.4 -Port 3389 -InformationLevel 'Detailed'
   ```
1. Azure 포털의 **가상 머신** 블레이드로 돌아간다. 

1. 가상 머신 목록에서 **az104-05-vm1**를 클릭한다.

1. **az104-05-vm1** 블레이드에서 *연결**을 클릭하고, **RDP**를 선택한다. **RDP를 사용하여 연결** 블레이드에서 **RDP 파일 다운로드**를 클릭하고 원격 데스크톱 세션을 시작한다. 

    >**참고**: 이 단계는 Windows 컴퓨터에서 원격 데스크톱을 통해 연결하는 것을 말합니다. Mac에서는 Mac App Store에서 Remote Desktop Client를 사용할 수 있으며, Linux 컴퓨터에서는 오픈 소스 RDP 클라이언트 소프트웨어를 사용할 수 있습니다.

    >**참고**: 가상 머신에 연결할 때 출력되는 경고 메시지는 무시할 수 있습니다.

1. 원격 데스크톱에 연결되면 **Student** 계정과 **Pa55w.rd1234** 패스워드를 사용하여 로그인한다.

1. **az104-05-vm1** 원격 데스크톱 세션 내의 **Start** 버튼을 우클릭하여, 메뉴에서 **Windows PowerShell (Admin)**를 클릭한다.

1. Windows PowerShell 콘솔 창에서 다음 명령을 사용하여 TCP 포트 3389를 통해 **az104-05-vm2**에 대한 연결성을 테스트한다. (사설 IP 주소 **10.52.0.4**)

   ```pwsh
   Test-NetConnection -ComputerName 10.52.0.4 -Port 3389 -InformationLevel 'Detailed'
   ```
    >**참고**: 테스트에 사용되는 TCP 3389 포트는 운영 체제 방화벽에 의해 기본적으로 허용됩니다.

1. 명령의 결과를 검토하고, 연결에 성공했음을 확인한다.

#### 리소스 삭제

   >**참고**: 사용하지 않는 새로 생성된 Azure 리소스를 제거하십시오. 사용하지 않는 리소스를 제거해야 예상치 못한 비용이 발생하지 않습니다.

1. Azure 포털에서 **Cloud Shell**의 **PowerShell** 세션을 시작한다.

1. 다음 명령을 실행하여 이 모듈의 실습에서 생성된 모든 리소스 그룹을 나열한다.

   ```pwsh
   Get-AzResourceGroup -Name 'az104-05*'
   ```

1. 다음 명령을 실행하여 이 모듈의 실습에서 생성한 모든 리소스 그룹을 삭제한다.

   ```pwsh
   Get-AzResourceGroup -Name 'az104-05*' | Remove-AzResourceGroup -Force -AsJob
   ```

    >**참고**: 이 명령은 비동기적으로 실행되므로( --nowait 매개 변수로 결정됨) 동일한 Bash 세션 내에서 즉시 다른 Azure CLI 명령을 실행할 수 있지만, 리소스 그룹이 실제로 제거되기까지는 몇 분 정도 소요됩니다.

#### 요약

이 랩에서 우리는

- 랩 환경을 프로비전 했습니다.
- 로컬 및 글로벌 가상 네트워크 피어링을 구성했습니다.
- 사이트 간 연결성을 테스트했습니다.
