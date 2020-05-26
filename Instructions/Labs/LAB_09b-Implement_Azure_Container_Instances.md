---
lab:
    title: '09b - Azure Container Instances 구현'
    module: '모듈 09 - 서버리스 컴퓨팅'
---

# 랩 09b - Azure Container Instances 구현


## 랩 시나리오

Contoso는 가상화된 워크로드를 위한 새로운 플랫폼을 찾고자 합니다. 이 목표를 달성하기 위해 활용할 수 있는 여러 컨테이너 이미지를 확인 합니다.  컨테이너 관리를 최소화기 위하여 Docker 이미지 배포를 위한 Azure Container Instance를 사용합니다. 


## 목표

이 과정에서, 우리는 다음과 같은 실습을 합니다 :

+ 작업 1: Azure Container Instance를 사용하여 Docker 이미지 배포
+ 작업 2: Azure Container Instance의 기능 검토


## 설명

### 작업 1: Azure Container Instance를 사용하여 Docker 이미지 배포

이 작업에서는 웹 애플리케이션의 새로운 컨테이너 인스턴스를 만듭니다. 

1. [Azure portal](https://portal.azure.com)에 로그인한다. 

1. Azure 포털에 **Container instances**를 검색하고 선택한다. **Container instances** 블레이드에서 **+ 추가**를 클릭한다. 

1. **기본 사항** 탭에서 다음 설정을 사용한다. (다른 값은 기본 설정을 사용한다) 

    | 설정 | 값 |
    | ---- | ---- |
    | 구독 | 이 랩에서 사용할 구독의 이름 |
    | 리소스 그룹 | 새로 만들기 **az104-09b-rg1** |
    | 컨테이너 이름 | **az104-9b-c1** |
    | 지역 | Azure container instances를 프로비전할 수 있는 지역의 이름|
    | 이미지 소스 | **빠른 시작 이미지** |
    | 이미지 | **microsoft/aci-helloworld (Linux)** |

1. **다음: 네트워킹 >**을 클릭하고, 다음 설정을 사용한다. (다른 값은 기본 설정을 사용한다) 

    | 설정 | 값 |
    | --- | --- |
    | DNS 이름 레이블 | 고유한 DNS 호스트 이름 |
	
    >**참고**: 생성한 컨테이너는 dns-name-label.region.azurecontainer.io에서 공개적으로 접근 가능합니다. **DNS 이름 레이블을 사용할 수 없음** 오류 메시지가 표시되면 다른 값을 지정하십시오.

1. **다음: 고급 >**을 클릭한다. 기본 설정을 검토한 뒤 **검토 + 만들기**를 클릭하고, **만들기**를 클릭한다. 

    >**참고**: 배포가 완료될 때까지 기다리십시오. 이 작업은 약 3분 소요됩니다. 

    >**참고**: 기다리는 동안 [code behind the sample application](https://github.com/Azure-Samples/aci-helloworld)의 \app 폴더에서 내용을 확인하십시오. 


### 작업 2: Azure Container Instance의 기능 검토

이 작업에서는 컨테이너 인스턴스의 배포를 검토합니다. 

1. 배포 블레이드에서 **리소스로 이동**을 클릭한다. 

1. **개요** 블레이드에서 **상태**가 **실행 중**인 것을 확인한다. 

1. 컨테이너 인스턴스의 **FQDN** 값을 복사하여, 새 브라우저 창을 열고 해당 URL로 접속한다.

1. **Welcome to Azure Container Instance** 페이지를 확인한다.

1. 브라우저 탭을 닫고 Azure portal로 돌아와 **설정** 섹션의 **컨테이너**를 선택하고, **로그** 탭을 클릭한다. 

1. 브라우저에 애플리케이션을 표시하여 생성된 HTTP GET 요청을 나타내는 로그 항목을 확인한다.


### 리소스 삭제

   >**참고**: 사용하지 않는 새로 생성된 Azure 리소스를 제거하십시오. 사용하지 않는 리소스를 제거해야 예상치 못한 비용이 발생하지 않습니다.

1. Azure 포털에서 **Cloud Shell**의 **PowerShell** 세션을 시작한다.

1. 다음 명령을 실행하여 이 모듈의 실습에서 생성된 모든 리소스 그룹을 나열한다.

   ```powershell
   Get-AzResourceGroup -Name 'az104-09b*'
   ```

1. 다음 명령을 실행하여 이 모듈의 실습에서 생성한 모든 리소스 그룹을 삭제한다.

   ```powershell
   Get-AzResourceGroup -Name 'az104-09b*' | Remove-AzResourceGroup -Force -AsJob
   ```

    >**참고**: 이 명령은 비동기적으로 실행되므로( --nowait 매개 변수로 결정됨) 동일한 PowerShell 세션 내에서 즉시 다른 PowerShell 명령을 실행할 수 있지만, 리소스 그룹이 실제로 제거되기까지는 몇 분 정도 소요됩니다.


### 요약

이 랩에서 우리는

- Azure Container Instance를 사용하여 Docker 이미지를 배포했습니다. 
- Azure Container Instance의 기능을 검토했습니다. 
