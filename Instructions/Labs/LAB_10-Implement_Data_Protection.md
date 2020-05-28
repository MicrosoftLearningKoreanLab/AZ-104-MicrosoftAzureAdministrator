---
lab:
    title: '10 - 데이터 보호 구현'
    module: '모듈 10 - 데이터 보호'
---

# 랩 10 - 가상 머신 백업


## 랩 시나리오

Azure 가상 머신과 온프레미스 컴퓨터에 호스팅된 파일을 백업하고 복구하기 위하여 Azure Recovery Services의 사용을 검토합니다. 또한 우발적이거나 악의적인 데이터 손실로부터 Recovery Services vault에 저장된 데이터를 보호하기 위한 방법을 확인합니다. 


## 목표

이 과정에서, 우리는 다음과 같은 실습을 합니다 :

+ 작업 1: 랩 환경 프로비전
+ 작업 2: Recovery Services 자격 증명 모음 생성
+ 작업 3: Azure 가상 머신 레벨 백업 구성
+ 작업 4: 파일 및 폴더 백업 구현
+ 작업 5: Azure Recovery Services agent를 이용하여 파일 복구 수행
+ 작업 6: Azure 가상 머신 스냅샷을 이용하여 파일 복구 수행 (선택 사항)
+ 작업 7: Azure Recovery Services 일시 삭제 기능 검토 (선택 사항) 


## 설명

### 작업 1: 랩 환경 프로비전

이 작업에서는 다른 백업 시나리오를 테스트하는 데 사용할 두 개의 가상 머신을 배포합니다. 

1. [Azure portal](https://portal.azure.com)에 로그인한다.

1. Azure 포털 오른쪽 위의 아이콘을 클릭하여 **Azure Cloud Shell**을 실행한다.

1. **Bash** 또는 **PowerShell**을 선택하는 프롬프트 창에서 **PowerShell**을 선택한다. 

    >**참고**: **Cloud Shell**을 처음 실행한 경우, **탑재된 스토리지가 없음** 메시지가 표시됩니다. 이 랩에서 사용하고 있는 구독을 선택하고 **스토리지 만들기**를 클릭하십시오. 

1. Cloud Shell 창의 툴바에서 **파일 업로드/다운로드** 아이콘을 클릭한다. 드롭다운 메뉴에서 **업로드**를 클릭하고, **\\Allfiles\\Labs\\10\\az104-10-vms-template.json** 와 **\\Allfiles\\Labs\\10\\az104-10-vms-parameters.json**을 Cloud Shell의 홈 디렉토리에 업로드한다. 

1. Cloud Shell 창에서 다음을 실행하여 가상 머신을 호스팅할 리소스 그룹을 생성한다. (`[Azure_region]`을 Azure 가상 머신을 배포할 지역의 이름으로 대체한다)

   ```powershell
   $location = '[Azure_region]'

   $rgName = 'az104-10-rg0'

   New-AzResourceGroup -Name $rgName -Location $location
   ```

1. Cloud Shell 창에서 다음을 실행하여 업로드한 템플릿과 파라미터 파일을 사용하여 첫 번째 가상 네트워크를 생성하고 가상 머신을 배포한다. 

   ```powershell
   New-AzResourceGroupDeployment `
      -ResourceGroupName $rgName `
      -TemplateFile $HOME/az104-10-vms-template.json `
      -TemplateParameterFile $HOME/az104-10-vms-parameters.json `
      -AsJob
   ```

1. Cloud Shell을 최소화한다. (창은 닫지 않는다)

    >**참고**: 배포가 끝날 때까지 기다리지 말고 다음 작업을 진행하십시오. 배포에는 약 5분 소요됩니다. 


### 작업 2: Recovery Services 자격 증명 모음 생성

이 작업에서는 Recovery Services 자격 증명 모음을 생성합니다. 

1. Azure 포털에서 **Recovery Services 자격 증명 모음**을 선택하고, **+ 추가**를 클릭한다.

1. **Recovery Services 자격 증명 모음 만들기** 블레이드에서 다음 설정을 사용한다.

    | 설정 | 값 |
    | --- | --- |
    | 구독 | 이 랩에서 사용할 구독의 이름 |
    | 리소스 그룹 | 새로 만들기 **az104-10-rg1** |
    | 자격 증명 모음 이름 | **az104-10-rsv1** |
    | 지역 | 이전 작업에서 두 개의 가상 머신을 배포한 지역의 이름 |

    >**참고**: 이전 작업에서 두 개의 가상 머신을 배포한 지역과 동일한 지역에 배포해야 합니다.

1. **검토 + 만들기**를 클릭하고, **만들기**를 클릭한다.

    >**참고**: 배포가 완료될 때까지 기다리십시오. 이 작업은 1분 미만 소요됩니다.

1. 배포가 끝나면 **리소스로 이동**을 클릭한다.. 

1. **az104-10-rsv1** Recovery Services 자격 증명 모음 블레이드 **설정** 섹션의 **속성**을 클릭한다.

1. **az104-10-rsv1 - 속성** 블레이드 **백업 구성** 레이블 아래 **업데이트**를 클릭한다. 

1. **백업 구성** 블레이드에서 **스토리지 복제 유형**을 **로컬 중복**이나 **지역 중복**으로 설정할 수 있다. 기본 설정인 **지역 중복**을 사용하고 블레이드를 닫는다.

    >**참고**: 이 설정은 백업 항목이 존재하지 않을 때만 할 수 있다. 

1. **az104-10-rsv1 - 속성** 블레이드로 돌아가서 **보안 설정** 레이블 아래 **업데이트**를 클릭한다. 

1. **보안 설정** 블레이드에서 **일시 삭제(Azure Virtual Machines 용)** 이 **사용**으로 설정되어 있는 것을 확인한다.

1. **보안 설정** 블레이드를 닫고, **az104-10-rsv1**  Recovery Services 자격 증명 모음 블레이드로 돌아가 **개요**를 클릭한다.


### 작업 3: Azure 가상 머신 레벨 백업 구성

이 작업에서는 Azure 가상 머신 레벨 백업을 구성합니다. 

   >**참고**: 이 작업을 시작하기 전에, 이 랩의 첫 번째 작업에서 시작한 배포가 완료되었는지 확인합니다. 

1. **az104-10-rsv1** Recovery Services 자격 증명 모음 블레이드에서 **+ 백업**을 클릭한다.

1. **백업 목표** 블레이드에서 다음 설정을 사용한다. 

    | 설정 | 값 |
    | --- | --- |
    | 작업이 실행되는 위치 | **Azure** |
    | 백업할 항목 | **가상 머신** |

1. **백업 목표** 블레이드에서 **백업**을 클릭한다. 

1. **백업 목표** 블레이드에서, **DefaultPolicy** 설정을 검토하고 드롭다운 목록 아래 **새 정책 만들기**를 클릭한다.

1. 다음 설정을 사용하여 새로운 백업 정책을 정의한다. (다른 값은 기본 설정을 사용한다)

    | 설정 | 값 |
    | ---- | ---- |
    | 정책 이름 | **az104-10-backup-policy** |
    | 빈도 | **매일** |
    | 시간 | **12:00 AM** |
    | 표준 시간대 | 로컬 타임 존 이름 |
    | 빠른 복구 스냅샷 보존 | **2**일 |

1. **확인**을 클릭하여 정책을 만든다. 가상 머신 레이블 아래의 **추가**를 클릭하여 **가상 머신 선택** 블레이드를 연다.

1. **가상 머신 선택** 블레이드에서 **az-104-10-vm0**를 선택하고 **확인**을 클릭한다. **백업** 블레이드에서 **백업 사용**을 클릭한다.

    >**참고**: 백업 설정이 완료될 때까지 기다리십시오. 이 작업은 약 2분 소요됩니다.

1. **az104-10-rsv1** Recovery Services 자격 증명 모음 블레이드의  **보호된 항목** 섹션에서 **백업 항목**을 선택하고, **Azure virtual machines**을 클릭한다.

1. **백업 항목 (Azure Virtual Machine)** 목록의 **az104-10-vm0**에서 **백업 사전 검사** 항목과 **마지막 백업 상태**를 확인하고, **az104-10-vm0**을 클릭한다.

1. **az104-10-vm0** 백업 항목에서 **지금 백업**을 클릭하고, **다음까지 백업 보존** 값을 기본으로 두고 **확인**을 클릭한다.

    >**참고**:  백업 작업이 끝날 때까지 기다리지 않고 다음 작업을 진행합니다.


### 작업 4: 파일 및 폴더 백업 구현

이 작업에서는 Azure Recovery Services를 사용하여 파일 및 폴더 백업을 구현합니다. 

1. Azure 포털에서 **가상 머신**을 찾아 선택하고, **az104-10-vm1**을 클릭한다.

1. **az104-10-vm1** 블레이드에서 **연결**을 클릭하고, **RDP**를 선택한다. **RDP를 사용하여 연결** 블레이드에서 **RDP 파일 다운로드**를 클릭하고 원격 데스크톱 세션을 시작한다.

    >**참고**: 이 단계는 Windows 컴퓨터에서 원격 데스크톱을 통해 연결하는 것을 말합니다. Mac에서는 Mac App Store에서 Remote Desktop Client를 사용할 수 있으며, Linux 컴퓨터에서는 오픈 소스 RDP 클라이언트 소프트웨어를 사용할 수 있습니다.

    >**참고**: 가상 머신에 연결할 때 출력되는 경고 메시지는 무시할 수 있습니다.

1. 원격 데스크톱에 연결되면 **Student** 계정과 **Pa55w.rd1234** 패스워드를 사용하여 로그인한다.

1. **az104-10-vm1** Azure 가상 머신의 원격 데스크톱 세션에서 **Server Manager** 창에서 **Local Server**를 클릭하고, **IE Enhanced Security Configuration**을 선택한 후, Administrators에 대해 **off**로 설정한다.

1. **az104-10-vm1** Azure 가상 머신에서 Internet Explore를 시작하여 [Azure portal](https://portal.azure.com)에 로그인한다.

1. Azure 포털에서 **Recovery Services vaults**를 찾아 선택하고, **Recovery Services vaults**의 **az104-10-rsv1**를 클릭한다.

1. **az104-10-rsv1** Recovery Services vault 블레이드에서 **+ Backup**을 클릭한다.

1. **Backup Goal** 블레이드에 다음 설정을 사용한다.

    | 설정 | 값 |
    | --- | --- |
    | Where is your workload running? | **On-premises** |
    | What do you want to backup? | **Files and folders** |

    >**참고**: 이 작업에서 사용하는 가상 머신이 Azure 환경에서 동작하지만,  이 작업에서 사용 중인 가상 시스템이 Azure에서 실행되고 있더라도 Windows Server 운영 체제를 실행하는 모든 온프레미스 컴퓨터에 적용할 수 있는 백업 기능을 평가하는 데 이 가상 머신을 활용할 수 있다.

1. **Backup Goal** 블레이드에서 **Prepare infrastructure**을 클릭한다.

1. **Prepare infrastructure** 블레이드에서 **Download Agent for Windows Server or Windows Client** 링크를 클릭한다. 

1. **Run**을 클릭하여 기본 설정으로  **MARSAgentInstaller.exe**를 설치한다.

    >**참고**: **Microsoft Azure Recovery Services Agent Setup Wizard**의 **Microsoft Update Opt-In**페이지에서 **I do not want to use Microsoft Update** 설치 옵션을 선택한다. 

1. **Proceed to Registration**을 클릭하여 **Register Server Wizard**를 시작한다. 

1. Azure portal이 있는 Internet Explorer 창으로 돌아가서 **Prepare infrastructure** 블레이드의 **Already downloaded or using the latest Recovery Server Agent** 체크 박스를 선택하고, **Download**를 클릭한다. 

1. **Save**를 클릭하여 vault credentials 파일을 Downloads 폴더에 저장한다. 

1. **Register Server Wizard** 창으로 돌아가서 **Vault Identification** 페이지의 **Browse**를 클릭한다.

1. **Select Vault Credentials** 창에서 **Downloads** 폴더를 찾아 다운로드했던 vault credentials 파일을 선택하고, **Open**을 클릭한다.

1. **Vault Identification** 페이지로 돌아가 **Next**를 클릭한다.

1. **Register Server Wizard**의 **Encryption Setting** 페이지에서 **Generate Passphrase**를 클릭한다.

1.  **Register Server Wizard**의 **Encryption Setting** 페이지에서 **Enter a location to save the passphrase** 드롭다운 목록 옆의 **Browse** 버튼을 클릭한다. 

1. **Browse For Folder** 창에서 **Documents** 폴더를 선택하고, **OK**를 클릭한다.

1. **Finish**를 클릭하고 **Microsoft Azure Backup** 안내 사항을 검토한 뒤, **Yes**를 클릭한다. 등록이 완료될 때까지 기다린다. 

    >**참고**: production 환경에서는 암호 파일을 백업되는 서버가 아닌 안전한 위치에 저장해야 합니다. 

1. **Register Server Wizard**의 **Server Registration** 페이지에서 파일 위치에 대한 경고를 확인하고, **Launch Microsoft Azure Recovery Services Agent**의 체크박스를 선택한 뒤, **Close**를 클릭한다. **Microsoft Azure Backup** 콘솔이 자동으로 실행된다. 

1. **Microsoft Azure Backup** 콘솔에서 **Actions** 창의 **Schedule Backup**을 클릭한다.

1. **Schedule Backup Wizard**의 **Getting started** 페이지에서 **Next**를 클릭한다.

1. **Select Items to Backup** 페이지에서 **Add Items**를 클릭한다.

1. **Select Items** 박스에서 **C:\\Windows\\System32\\drivers\\etc\\** 경로를 찾아서 **hosts**를 선택하고 **OK**를 클릭한다.

1. **Select Items to Backup** 페이지에서 **Next**를 클릭한다.

1. **Specify Backup Schedule** 페이지에서 **Day**로 설정된 것을 확인한다. **At following times (Maximum allowed is three times a day)** 박스 아래 첫 번째 드롭다운 리스트에서 **4:30 AM**을 선택하고 **Next**를 클릭한다. 

1. **Select Retention Policy** 페이지의 설정을 기본 값으로 두고 **Next**를 클릭한다.

1. **Choose Initial Backup type** 페이지의 설정을 기본 값으로 두고, **Next**를 클릭한다.

1. **Confirmation** 페이지에서 **Finish**를 클릭한다. 백업 스케줄이 생성되면 **Close**를 클릭한다. 
  
1. **Microsoft Azure Backup** 콘솔에서 Actions 창의 **Back Up Now**를 클릭한다.

    >**참고**: 예약된 백업을 생성하면 온디맨드 방식으로 백업을 실행할 수 있는 옵션이 제공됩니다.

1. Back Up Now Wizard의 **Select Backup Item** 페이지에서 **Files and Folders** 옵션이 선택된 것을 확인하고, **Next**를 클릭한다.

1. **Retain Backup Till** 페이지의 설정을 기본 값으로 두고, **Next**를 클릭한다.

1. **Confirmation** 페이지에서 **Back Up**을 클릭한다.

1. 백업이 완료되면 **Close**를 클릭하고 Microsoft Azure Backup을 닫는다. 

1. Azure portal이 있는 Internet Explorer 창으로 돌아와서 Recovery Services vault 블레이드의 **Backup items**을 클릭한다.

1. **az104-10-rsv1 - Backup items** 블레이드에서 **Azure Backup Agent**를 클릭한다.

1. **Backup Items (Azure Backup Agent)** 블레이드의 항목이 **az104-10-vm1.** 의 **C:\\** 드라이브를 참조하고 있는 것을 확인한다. 


### 작업 5: Azure Recovery Services agent를 이용하여 파일 복구 수행 (선택 사항)

이 작업에서는 Azure Recovery Services agent를 이용하여 파일 복구를 수행합니다. 

1. **az104-10-vm1** 원격 데스크톱 세션에서 **C:\\Windows\\System32\\drivers\\etc\\** 폴더의 **hosts** 파일을 삭제한다.

1. Microsoft Azure Backup 창으로 돌아가서 **Recover data**를 클릭하여 **Recover Data Wizard**를 시작한다.

1. **Getting Started** 페이지에서 **This server (az104-10-vm1.)** 옵션이 선택된 것을 확인하고, **Next**를 클릭한다.

1. **Individual files and folders** 옵션이 선택된 것을 확인하고 **Next**를 클릭한다.

1. **Select Volume and Date** 페이지의 **Select the volume** 드롭 다운 메뉴에서 **C:\\** 를 선택한다. available backups 설정을 기본으로 두고 **Mount**를 클릭한다.

    >**참고**: 작업이 완료될 때까지 기다리십시오. 이 작업은 약 2분 소요됩니다. 

1. **Browse And Recover Files** 페이지에서 복구 volume의 드라이브 문자를 확인하고, Robocopy 사용에 대한 팁을 검토한다.

1. **Start**를 클릭하여 **Windows System** 폴더의 **Command Prompt**를 시작한다.

1. Command Prompt 창에서 다음 명령을 실행하여 **hosts** 복구 파일을 원본 위치에 복사한다. (`[recovery_volume]`을 앞서 확인한  복구 volume의 드라이브 문자로 대체한다)

   ```
   robocopy [recovery_volume]:\Windows\System32\drivers\etc C:\Windows\system32\drivers\etc hosts /r:1 /w:1
   ```

1. **Recover Data Wizard**로 돌아가서 **Browse and Recover File** 창의 **Unmount**를 클릭한다.확인 창이 뜨면, **Yes**를 클릭한다.

1. 원격 데스크톱 세션을 종료한다. 


### 작업 6: Azure 가상 머신 스냅샷을 이용하여 파일 복구 수행 (선택 사항)

이 작업에서는 Azure 가상 머신 레벨 스냅샷 기반 백업을 이용하여 파일을 복구합니다. 

1. 랩 컴퓨터로 Azure 포털에 접속한다. 

1. Azure 포털 **가상 머신** 블레이드의 **az104-10-vm0**을 클릭한다.

1. **az104-10-vm0** 블레이드에서 **연결**을 클릭하고, **RDP**를 선택한다. **RDP를 사용하여 연결** 블레이드에서 **RDP 파일 다운로드**를 클릭하고 원격 데스크톱 세션을 시작한다.

    >**참고**: 이 단계는 Windows 컴퓨터에서 원격 데스크톱을 통해 연결하는 것을 말합니다. Mac에서는 Mac App Store에서 Remote Desktop Client를 사용할 수 있으며, Linux 컴퓨터에서는 오픈 소스 RDP 클라이언트 소프트웨어를 사용할 수 있습니다.

    >**참고**: 가상 머신에 연결할 때 출력되는 경고 메시지는 무시할 수 있습니다.

1. 원격 데스크톱에 연결되면 **Student** 계정과 **Pa55w.rd1234** 패스워드를 사용하여 로그인한다.

1. **az104-10-vm0** Azure 가상 머신의 원격 데스크톱 세션에서 **Server Manager** 창에서 **Local Server**를 클릭하고, **IE Enhanced Security Configuration**을 선택한 후, Administrators에 대해 **off**로 설정한다.

1. **Start**를 클릭하고, **Windows System** 폴더의 **Command Prompt**를 시작한다.

1. Command Prompt 창에서 다음을 실행하여 **hosts** 파일을 제거한다.

   ```
   del C:\Windows\system32\drivers\etc\hosts
   ```
 
   >**참고**: 이 파일은 가상 머신 레벨 스냅샷 기반 백업을 통해 복구할 예정입니다.

1. Azure 가상 머신 **az104-10-vm0**의 원격 데스크톱 세션에서 [Azure portal](https://portal.azure.com) 에 로그인한다.

1. Azure 포털에서 **Recovery Services vaults** 를 찾아 선택하고, **az104-10-rsv1**를 클릭한다.

1. **az104-10-rsv1** Recovery Services vault 블레이드에서 **Protected items** 섹션의 **Backup items**을 클릭한다.

1. **az104-10-rsv1 - Backup items** 블레이드에서 **Azure Virtual Machine**을 클릭한다.

1. **Backup Items (Azure Virtual Machine)** 블레이드에서 **az104-10-vm0**을 클릭한다.

1. **az104-10-vm0** Backup Item 블레이드에서 **File Recovery**을 클릭한다.

    >**참고**: 애플리케이션 정합성이 보장되는 스냅샷을 기반으로 백업이 시작된 직후 복구를 실행할 수 있는 옵션이 있습니다.

1. **File Recovery** 블레이드에서 기본 설정을 사용하고, **Download Executable**를 클릭한다. 

    >**참고**: 이 스크립트는 선택한 복구 지점의 디스크를 스크립트가 실행되는 운영 체제 내의 로컬 드라이브로 마운트합니다.

1. **Download**를 클릭하고, **Run**을 클릭하여 **IaaSVMILRExeForWindows.exe**을 실행한다. 

1. 명령 프롬프트에 패스워드를 입력하라는 메시지가 출력되면, **Password to run the script** 옆의 패스워드를 복사하여  명령 프롬프트 창에 붙여넣기 하고 **Enter**를 클릭한다.

    >**참고**: 마운트 과정을 보여주는 Windows PowerShell 창이 시작됩니다. 

    >**참고**: 이 작업에서 에러 메시지가 표시되면 Internet Explorer 창을 새로고침하고 마지막 세 단계를 반복하십시오.

1. 마운트 과정이 끝날 때까지 기다리고, Windows PowerShell 창의 정보성 메시지를 검토한다. **Windows**를 호스팅하는 volume에 할당된 드라이브 문자를 기록한다. 

1. 파일 탐색기에서 이전 단계에서 식별한 운영 체제 volume의 스냅샷을 호스팅하는 드라이브 문자로 이동하여 내용을 검토한다.

1. **Command Prompt** 창으로 돌아간다.

1. 명령 프롬프트에 다음 명령을 실행하여 원본 위치에 **hosts** 파일 복사하여 복구한다. (`[os_volume]`을 앞서 확인한 운영체제 volume의 드라이브 문자로 대체한다)

   ```
   robocopy [os_volume]:\Windows\System32\drivers\etc C:\Windows\system32\drivers\etc hosts /r:1 /w:1
   ```

1. **File Recovery** 블레이드로 돌아가서 Azure 포털의 **Unmount Disks**를 클릭한다.

1. 원격 데스크톱 세션을 종료한다.


### 작업 7: Azure Recovery Services 일시 삭제 기능 검토

1. 랩 컴퓨터의 Azure 포털에서 **Recovery Services 자격 증명 모음**을 찾아 선택한다. 목록에서 **az104-10-rsv1**를 클릭한다.

1. **az104-10-rsv1** Recovery Services 자격 증명 모음 블레이드에서 **보호된 항목** 섹션에서 **백업 항목**을 클릭한다.

1. **az104-10-rsv1 - 백업 항목** 블레이드에서 **Azure Backup Agent**를 클릭한다.

1. **백업 항목 (Azure Backup Agent)** 블레이드 목록에서 **az104-10-vm1**을 클릭한다.

1. **on az104-10-vm1.의 C:\\** 블레이드에서 **az104-10-vm1.** 링크를 클릭한다.

1. **az104-10-vm1.** 보호된 서버 블레이드에서 **삭제**를 클릭한다.

1. **삭제** 블레이드에 다음 설정을 사용한다. 

    | 설정 | 값 |
    | --- | --- |
    | 서버 이름 입력 | **az104-10-vm1.** |
    | 이유 | **개발/테스트 서버 재활용** |
    | 설정 | **az104 10 lab** |

    >**참고**: 서버 이름을 입력할 때, **.** (Trailing period)을 반드시 포함해야 합니다.  

1. **이 서버와 연결된 1 백업 항목의 백업 데이터가 있습니다."확인"을 클릭하면 모든 클라우드 백업 데이터가 영구적으로 삭제된다는 것을 이해하는 것으로 간주됩니다. 이 작업은 실행 취소할 수 없습니다. 이 삭제를 알리는 경고가 이 구독의 관리자에게 전송될 수 있습니다.** 에 체크하고 **삭제**를 클릭한다.

1. **az104-10-rsv1 - 백업 항목** 블레이드로 돌아가 **Azure Virtual Machines**을 클릭한다.

1. **백업 항목 (Azure Virtual Machine)** 블레이드에서 **az104-10-vm0**을 클릭한다.

1. **az104-10-vm0** 백업 항목 블레이드에서 **백업 중지**를 클릭한다. 

1. **백업 중지** 블레이드에서 **백업 데이터 삭제**를 선택하고 다음 설정을 사용한 후, **백업 중지**를 클릭한다.

    | 설정 | 값 |
    | --- | --- |
    | 백업 항목의 이름 입력 | **az104-10-vm0** |
    | 이유 | **기타** |
    | 설명 | **az104 10 lab** |

1. **az104-10-rsv1 - 백업 항목** 블레이드로 돌아가 **새로 고침**을 클릭한다.

    >**참고**: **Azure Virtual Machine**의 백업 항목 개수가 여전히 **1**인 것을 확인한다.

1. **Azure Virtual Machine**을 선택하고 **백업 항목 (Azure Virtual Machine)** 블레이드에서 **az104-10-vm0**를 클릭한다. 

1. **az104-10-vm0** 백업 항목 블레이드에서 삭제된 백업에 대해 **삭제 취소** 옵션이 있는 것을 확인할 수 있다. 

    >**참고**: 이 기능은 기본적으로 Azure 가상 머신 백업을 가능하게 하는 일시 삭제 기능을 통해 제공됩니다. 

1. **az104-10-rsv1** Recovery Services 자격 증명 모음으로 돌아가, **설정** 섹션의 **속성**을 클릭한다.

1. **az104-10-rsv1 - 속성** 블레이드에서 **보안 설정** 레이블 아래의 **업데이트**를 클릭한다.

1. **보안 설정** 블레이드에서 **일시 삭제 (Azure Virtual Machines 용)** 를 **사용 안 함**으로 설정하고 **저장**을 클릭한다.

    >**참고**: 이 작업은 이미 일시 삭제된 상태에 있는 항목에는 영향을 주지 않습니다. 

1. **보안 설정** 블레이드를 닫고 **az104-10-rsv1** Recovery Services 자격 증명 블레이드로 돌아가 **개요**를 클릭한다.

1. **az104-10-vm0** 백업 항목 블레이드로 돌아가서 **삭제 취소**를 클릭한다.  

1. **삭제취소 az104-10-vm0** 블레이드로 돌아가서 **삭제 취소**를 클릭한다. 

1. 삭제 취소 작업이 끝날 때까지 기다리십시오. 필요한 경우 브라우저 페이지를 새로고침하고, **az104-10-vm0** 백업 항목 블레이드에서 **백업 데이터 삭제**를 클릭한다.

1. **백업 데이터 삭제** 블레이드에서 다음 설정을 사용하고, **삭제**를 클릭한다.

    | 설정 | 값 |
    | --- | --- |
    | 백업 항목의 이름 입력 | **az104-10-vm0** |
    | 이유 | **기타** |
    | 설명 | **az104 10 lab** |


### 리소스 삭제

   >**참고**: 사용하지 않는 새로 생성된 Azure 리소스를 제거하십시오. 사용하지 않는 리소스를 제거해야 예상치 못한 비용이 발생하지 않습니다.

1. Azure 포털에서 **Cloud Shell**의 **PowerShell** 세션을 시작한다.

1. 다음 명령을 실행하여 이 모듈의 실습에서 생성된 모든 리소스 그룹을 나열한다.

   ```powershell
   Get-AzResourceGroup -Name 'az104-10*'
   ```

1. 다음 명령을 실행하여 이 모듈의 실습에서 생성한 모든 리소스 그룹을 삭제한다.

   ```powershell
   Get-AzResourceGroup -Name 'az104-10*' | Remove-AzResourceGroup -Force -AsJob
   ```

   >**참고**: 추가로, **AzureBackupRG_** 로 시작하는 자동 생성된 리소스 그룹을 삭제할 수도 있습니다. (이 리소스 그룹에 대한 추가 과금은 발생하지 않습니다)

    >**참고**: 이 명령은 비동기적으로 실행되므로( --nowait 매개 변수로 결정됨) 동일한 PowerShell 세션 내에서 즉시 다른 PowerShell 명령을 실행할 수 있지만, 리소스 그룹이 실제로 제거되기까지는 몇 분 정도 소요됩니다.


### 요약

이 랩에서 우리는

- 랩 환경을 프로비전 했습니다.
- Recovery Services 자격 증명 모음을 생성했습니다.
- Azure 가상 머신 레벨 백업을 구현했습니다. 
- 파일 및 폴더 백업을 구현했습니다.
- Azure Recovery Services agent를 사용하여 파일 복구를 수행했습니다.
- Azure 가상 머신 스냅샷을 이용하여 파일 복구를 수행했습니다.
- Azure Recovery Services 일시 삭제 기능을 검토했습니다.
