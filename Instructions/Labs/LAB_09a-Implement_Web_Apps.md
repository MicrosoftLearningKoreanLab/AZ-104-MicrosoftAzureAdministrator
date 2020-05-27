---
lab:
    title: '09a - Web Apps 구현'
    module: '모듈 09 - 서버리스 컴퓨팅'
---

# 랩 09a - Web Apps 구현


## 랩 시나리오

현재 온 프레미스 데이터 센터에서 호스팅되고 있는 Contoso의 웹 사이트를 호스팅하기 위한 Azure Web Apps의 사용을 평가해야 합니다. 이 웹 사이트들은 PHP 런타임 스택을 사용하여 윈도우 서버에서 실행되고 있습니다. Azure Web Apps 배포 슬롯을 활용하여 DevOps Practice를 구현할 수 있는 방법도 결정해야 합니다.


## 목표

이 과정에서, 우리는 다음과 같은 실습을 합니다 :
  
+ 작업 1: Azure web app 생성
+ 작업 2: staging 배포 슬롯 생성
+ 작업 3: web app 배포 설정 구성
+ 작업 4: staging 배포 슬롯에 코드 배포
+ 작업 5: staging 슬롯 교환
+ 작업 6: Azure web app의 자동 확장 구성 및 테스트
  

## 설명

### 작업 1: Azure web app 생성

이 작업에서는 Azure web app을 생성합니다. 

1. [**Azure portal**](http://portal.azure.com)에 로그인합니다. 

1. Azure 포털에서 **App services**를 검색하고 선택한다. **App Services** 블레이드에서 **+ 추가**를 클릭한다.

1. **기본** 탭에 다음 설정을 사용한다. (다른 값은 기본 설정을 사용한다)

    | 설정 | 값 |
    | --- | ---|
    | 구독 | 이 랩에서 사용할 구독 |
    | 리소스 그룹 | 새로 만들기 **az104-09a-rg1** |
    | Web app 이름 | 고유한 이름 |
    | 게시 | **코드** |
    | 런타임 스택 | **PHP 7.3** |
    | 운영 체제 | **Windows** |
    | 지역 | Azure web apps을 프로비전할 수 있는 지역의 이름 |
    | App service 계획 | 기본 설정 사용 |

1. **다음 : 모니터링 >**을 클릭하고 **Application Insights 사용**을 **아니요**로 설정한다. **검토 + 만들기**를 클릭하고, 유효성 검사를 통과하면 **만들기**를 클릭한다. 

    >**참고**: 일반적으로는 **Application Insights**를 사용하지만 이 랩에서 해당 기능은 필요하지 않습니다. 

    >**참고**: web app 생성이 끝날 때까지 기다리십시오. 이 작업은 약 1분 소요됩니다. 

1. 배포 블레이드에서 **리소스로 이동**을 클릭한다. 


### 작업 2: staging 배포 슬롯 생성

이 작업에서는 staging 배포 슬롯을 생성합니다. 

1. 새로 배포된 web app의 블레이드에서 **URL**을 클릭하여 새 브라우저 탭에서 기본 웹 페이지를 띄운다.  

1. 브라우저 탭을 닫고 Azure 포털로 돌아가 web app 블레이드의 **배포** 섹션에서 **배포 슬롯**을 클릭한다. 

    >**참고**: 현재 web app에는 **프로덕션**으로 라벨링된 하나의 배포 슬롯이 존재합니다. 

1. **+ 슬롯 추가**를 클릭하고, 다음 설정을 사용하여 슬롯을 추가한다.

    | 설정 | 값 |
    | --- | ---|
    | 이름 | **staging** |
    | 다음의 설정 복제 | **설정을 복제하지 않음**|

1. **배포 슬롯**으로 돌아와서 새로 생성된 staging 슬롯을 클릭한다.

    >**참고**: staging 슬롯의 속성을 나타내는 블레이드에 접근합니다. 

1. staging 슬롯의 블레이드를 검토하고 **프로덕션**에 할당된 URL과 다른 것을 확인한다. 


### 작업 3: web app 배포 설정 구성

이 작업에서는 web app 배포 설정을 구성합니다. 

1. 배포 슬롯 블레이드의 **배포** 섹션에 **배포 센터**를 클릭한다.

    >**참고:** staging 슬롯 블레이드에서 작업을 진행하십시오.

1. **지속적인 배포 (CI/CD)** 섹션에서 **Local Git**을 선택하고 **계속**을 클릭한다.

1. **App Service 빌드 서비스**를 선택하고 **계속**을 클릭한 뒤, **마침**을 클릭한다. 

1. **Git Clone Url**을 메모장에 복사해둔다. 

    >**참고:** 이 랩의 다음 작업에서 **Git Clone Url**이 필요합니다.

1. 툴바의 **배포 자격 증명**을 클릭한다.  

1. **사용자 자격 증명**을 클릭한다.

1. 필요한 정보를 입력하고, **자격 증명 저장**을 클릭한다.

    | 설정 | 값 |
    | --- | ---|
    | 사용자 이름 | 고유한 이름 |
    | 암호 | **Pa55w0rd1234** |

    >**참고:** 이 랩의 다음 작업에서 해당 자격 증명이 필요합니다. 


### 작업 4: staging 배포 슬롯에 코드 배포

이 작업에서는 staging 배포 슬롯에 코드를 배포합니다. 

1. Azure 포털 오른쪽 위의 아이콘을 클릭하여 **Azure Cloud Shell**을 실행한다.

1. **Bash** 또는 **PowerShell**을 선택하는 프롬프트 창에서 **PowerShell**을 선택한다. 

    >**참고**: **Cloud Shell**을 처음 실행한 경우, **탑재된 스토리지가 없음** 메시지가 표시됩니다. 이 랩에서 사용하고 있는 구독을 선택하고 **스토리지 만들기**를 클릭하십시오. 

1. Cloud Shell 창에서 다음 명령을 실행하여 web app을 위한 코드가 있는 원격 저장소를 클론한다. 

   ```sh
   git clone https://github.com/Azure-Samples/php-docs-hello-world
   ```
 
1. Cloud Shell 창에서 다음 명령을 실행하여 현재 위치를 새로 생성된 샘플 web app 코드의 로컬 저장소로 지정한다. 

   ```powershell
   Set-Location -Path $HOME/php-docs-hello-world/
   ```

1. Cloud Shell 창에서 다음 명령을 입력하여 원격 git을 추가한다. (`[deployment_user_name]` 과 `[git_clone_url]`을 각각 이전 작업에서 저장해둔 **배포 자격**과 **Git Clone Url**로 대체한다)

   ```sh
   git remote add [deployment_user_name] [git_clone_url]
   git remote add krazure123 https://krazure-staging.scm.azurewebsites.net:443/krazure.git
   ```

>**참고**: `git remote add` 값은 고유한 값이기만 하면 **배포 자격**의 사용자 이름과 같지 않아도 됩니다.

1. Cloud Shell 창에서 다음 명령을 실행하여 로컬 저장소에 있는 샘플 web app 코드를 Azure web app staging 배포 슬롯에 푸시한다.(`[deployment_user_name]`을 이전 작업에서 사용한 **배포 자격**으로 대체한다)

   ```sh
   git push [deployment_user_name] master
   git push krazure123 master
   ```

1. 인증 창이 뜨면 `[deployment_user_name]`과 패스워드(**Pa55w0rd1234**)를 입력한다. 

1. Cloud Shell 창을 닫는다.

1. staging 슬롯 블레이드에서 **개요**를 클릭하고 **URL** 링크로 접속하여 기본 웹페이지를 표시한다. 

1. 브라우저 페이지에 **Hello World!** 메시지가 표시되는 것을 확인하고 탭을 닫는다. 


### 작업 5: staging 슬롯 교환

이 작업에서는 staging 슬롯을 **프로덕션** 슬롯과 교환합니다. 

1. web app의 **프로덕션** 슬롯이 보이는 블레이드로 이동한다. 

1. **배포** 섹션의 **배포 슬롯**을 선택하고 툴바에서 **교환** 아이콘을 클릭한다. 

1. **교환** 블레이드에서 기본 설정을 검토하고 **교환**을 클릭한다. 

1. web app **프로덕션** 슬롯의 **개요**를 클릭하고, 새 브라우저 탭에서 **URL** 링크로 접속한다. 

1. 기본 웹 페이지가 **Hello World!** 페이지로 바뀐 것을 확인한다.


### 작업 6: Azure web app 자동 확장 구성 및 테스트

이 작업에서는 Azure web app의 자동 확장을 구성하고 테스트합니다.  

1. web app의 **프로덕션** 슬롯이 보이는 블레이드에서 **설정** 섹션의 **스케일 아웃 (App Service 계획)**을 클릭한다.

1. **사용자 지정 자동 크기 조정**을 클릭한다. 

    >**참고**: web app을 수동으로 크기 조정할 수 있는 옵션도 있습니다.

1. **메트릭 기준 크기 조정** 기본 옵션을 사용하고 **+ 규칙 추가**를 클릭한다.

1. **크기 조정 규칙** 블레이드에서 다음 설정을 사용한다. (다른 값은 기본 설정을 사용한다)

    | 설정 | 값 |
    | --- |--- |
    | 메트릭 원본 | **현재 리소스** |
    | 시간 집계 | **최대값** |
    | 메트릭 네임스페이스 | **App Service 계획 표준 메트릭** |
    | 메트릭 이름 | **CPU Percentage** |
    | 연산자 | **보다 큼** |
    | 크기 조정 작업을 트리거하는 메트릭 임계값 | **10** |
    | 기간(분) | **1** |
    | 시간 조직 통계 | **최대값** |
    | 작업 | **다음을 기준으로 개수 늘이기** |
    | 인스턴스 수 | **1** |
    | 휴지 기간(분)) | **5** |

    >**참고**: 이 값은 대기 시간 없이 가능한 빨리 자동 확장을 트리거하려는 목적이기 때문에 실제 구성과는 차이가 있습니다. 

1. **추가**를 클릭하고 다음 설정을 사용한다. (다른 값은 기본 설정을 사용한다)

    | 설정 | 값 |
    | --- |--- |
    | 인스턴스 최소값 | **1** |
    | 인스턴스 최대값 | **2** |
    | 인스턴스 기본값 | **1** |

1. **저장**을 클릭한다.

1. Azure 포털 오른쪽 위의 아이콘을 클릭하여 **Azure Cloud Shell**을 실행한다.

1. **Bash** 또는 **PowerShell**을 선택하는 프롬프트 창에서 **PowerShell**을 선택한다. 

1. Cloud Shell 창에서 다음 명령을 실행하여 Azure web app의 URL을 식별한다. 

   ```powershell
   $rgName = 'az104-09a-rg1'

   $webapp = Get-AzWebApp -ResourceGroupName $rgName
   ```

1. Cloud Shell 창에서 다음 명령을 실행하여 web app에 HTTP 요청을 보내는 무한 루프를 시작한다. 

   ```powershell
   while ($true) { Invoke-WebRequest -Uri $webapp.DefaultHostName }
   ```

1. Cloud Shell 창을 최소화하고(닫지 않는다), web app 블레이드 **모니터링** 섹션의 **프로세스 탐색기**를 클릭한다.

    >**참고**: 프로세스 탐색기는 인스턴스 수와 해당 리소스 활용률 모니터링을 용이하게 합니다. 

1. 몇 분 동안 인스턴스 개수와 활용률을 모니터링한다.

    >**참고**: 변경된 목록을 반영하려면 페이지를 **새로 고침**합니다.

1. 인스턴스 개수가 2개로 늘어나면, Cloud Shell 창을 다시 열고 **Ctrl +C**를 입력하여 스크립트를 종료합니다. 

1. Cloud Shell 창을 닫는다. 


### 리소스 삭제

   >**참고**: 사용하지 않는 새로 생성된 Azure 리소스를 제거하십시오. 사용하지 않는 리소스를 제거해야 예상치 못한 비용이 발생하지 않습니다.

1. Azure 포털에서 **Cloud Shell**의 **PowerShell** 세션을 시작한다.

1. 다음 명령을 실행하여 이 모듈의 실습에서 생성된 모든 리소스 그룹을 나열한다.

   ```powershell
   Get-AzResourceGroup -Name 'az104-09a*'
   ```

1. 다음 명령을 실행하여 이 모듈의 실습에서 생성한 모든 리소스 그룹을 삭제한다.

   ```powershell
   Get-AzResourceGroup -Name 'az104-09a*' | Remove-AzResourceGroup -Force -AsJob
   ```

    >**참고**: 이 명령은 비동기적으로 실행되므로( --nowait 매개 변수로 결정됨) 동일한 PowerShell 세션 내에서 즉시 다른 PowerShell 명령을 실행할 수 있지만, 리소스 그룹이 실제로 제거되기까지는 몇 분 정도 소요됩니다.


### 요약

이 랩에서 우리는

- Azure web app을 생성했습니다.
- staging 배포 슬롯을 생성했습니다.
- web app 배포 설정을 구성했습니다.
- staging 배포 슬롯에 코드를 배포했습니다.
- staging 슬롯을 교환했습니다.
- Azure web app의 자동 확장을 구성하고 테스트했습니다.
