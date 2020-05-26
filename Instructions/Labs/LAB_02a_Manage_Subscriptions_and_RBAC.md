---
lab:
    title: '02a - 구독 및 RBAC 관리'
    module: '모듈 02 - 거버넌스 및 규정 준수'
---

# 랩 02a - 구독 및 RBAC 관리


## 랩 시나리오

Contoso에 있는 Azure 리소스 관리를 향상하기 위해서 다음 기능을 구현합니다. 

- Contoso의 Azure 구독을 모두 포함하는 관리 그룹을 생성합니다.

- 지정된 Azure Active Directory 사용자에게 관리 그룹의 모든 구독에 대한 지원 요청을 제출할 수 있는 권한을 부여합니다. 해당 사용자의 권한은 다음과 같이 제한됩니다.

    - 지원 요청 티켓 생성
    - 리소스 그룹 보기


## 목표

이 과정에서, 우리는 다음과 같은 실습을 합니다 :

+ 작업 1: 관리 그룹 생성
+ 작업 2: 사용자 지정 RBAC 역할 생성 
+ 작업 3: RBAC 역할 할당


## 설명

### 작업 1: 관리 그룹 생성

이 작업에서는 관리 그룹을 생성하고 설정합니다. 

1. [Azure portal](https://portal.azure.com)에 로그인한다.

1. Search for and select **관리 그룹**을 찾아 선택한다. **관리 그룹** 블레이드에서 **+ 관리 그룹을 사용하여 시작**을 클릭한다.

1. 다음 설정을 사용하여 관리 그룹을 생성한다.

    | 설정 | 값 |
    | --- | --- |
    | 관리 그룹 ID | **az104-02-mg1**|
    | 관리 그룹 표시 이름 | **az104-02-mg1**|

1. 목록에서 새로 생성된 관리 그룹을 클릭하고 **details**를 클릭한다.

1. **az104-02-mg1** 블레이드에서 **+ 구독 추가**를 클릭하고, 이 랩에서 사용 중인 구독을 관리 그룹에 추가한다.

    >**참고**: Azure 구독의 ID를 클립보드에 복사합니다. 다음 작업에 사용할 예정입니다. 


### 작업 2: 사용자 지정 RBAC 역할 생성 

이 작업에서는 사용자 정의 RBAC 역할을 생성합니다. 

1. 랩 컴퓨터에서 **\\Allfiles\\Labs\\02\\az104-02a-customRoleDefinition.json** 파일을 Notepad로 열고, 내용을 검토한다.

   ```json
   {
      "Name": "Support Request Contributor (Custom)",
      "IsCustom": true,
      "Description": "Allows to create support requests",
      "Actions": [
          "Microsoft.Resources/subscriptions/resourceGroups/read",
          "Microsoft.Support/*"
      ],
      "NotActions": [
      ],
      "AssignableScopes": [
          "/providers/Microsoft.Management/managementGroups/az104-02-mg1",
          "/subscriptions/SUBSCRIPTION_ID"
      ]
   }
   ```

1. JSON 파일의 `SUBSCRIPTION_ID`를 클립보드에 복사한 구독 ID로 대체하고 저장한다.  

1. Azure 포털 오른쪽 위의 아이콘을 클릭하여 **Azure Cloud Shell**을 실행한다.

1. **Bash** 또는 **PowerShell**을 선택하는 프롬프트 창에서 **PowerShell**을 선택한다. 

    >**참고**: **Cloud Shell**을 처음 실행한 경우, **탑재된 스토리지가 없음** 메시지가 표시됩니다. 이 랩에서 사용하고 있는 구독을 선택하고 **스토리지 만들기**를 클릭하십시오.  

1. Cloud Shell 창의 툴바에서 **파일 업로드/다운로드** 아이콘을 선택하고, **업로드**를 클릭하여 **\\Allfiles\\Labs\\02\\az104-02a-customRoleDefinition.json** 파일을 Cloud Shell 홈 디렉토리에 업로드한다.

1. Cloud Shell 창에서 다음 명령을 실행하여 사용자 지정 역할 정의를 생성한다. 

   ```powershell
   New-AzRoleDefinition -InputFile $HOME/az104-02a-customRoleDefinition.json
   ```

1. Cloud Shell 창을 닫는다.


### 작업 3: RBAC 역할 할당

이 작업에서는 Azure Active Directory 사용자를 생성하고, 이전 작업에서 생성한 RBAC 역할을 해당 사용자에게 할당하며, 사용자가 RBAC 역할 정의에 지정된 작업을 수행할 수 있는지 확인합니다.

1. Azure 포털에서 **Azure Active Directory**를 클릭한다. Azure Active Directory 블레이드에서 **사용자**를 클릭하고, **+ 새 사용자**를 클릭한다.

1. 다음 설정을 사용하여 새 사용자를 생성한다. (다른 값은 기본 설정을 사용한다)

    | 설정 | 값 |
    | --- | --- |
    | 사용자 이름 | **az104-02-aaduser1**|
    | 이름 | **az104-02-aaduser1**|
    | 암호 | 암호 직접 만들기 |
    | 초기 암호 | **Pa55w.rd124** |

    >**참고**: **사용자 이름**을 **클립 보드에 복사**합니다. 다음 과정에서 사용됩니다.

1. Azure 포털의 **az104-02-mg1** 관리 그룹으로 돌아가 **details**을 클릭한다.

1. **액세스 제어 (IAM)** 를 선택하고, **+ 추가**와 **역할 할당 추가**를 차례로 클릭한다. 새로 생성한 사용자 계정에 **Support Request Contributor (Custom)** 역할을 할당한다. 

1. **InPrivate** 모드로 브라우저 창을 열어 새로 생성한 계정으로 [Azure portal](https://portal.azure.com)에 로그인하고, 암호를 변경한다.

    >**참고**: 사용자 이름은 클립보드에 복사했던 것을 사용하십시오.

1. Azure 포털에서 **리소스 그룹**을 찾아 선택하고, az104-02-aaduser1 사용자 계정이 모든 리소스 그룹에 접근할 수 있는 것을 확인한다. 

1.  Azure 포털에서 **모든 리소스**를 찾아 선택하고, 사용자가 리소스를 확인할 수 없다는 것을 확인한다.

1. Azure 포털에서 **도움말 + 지원**을 선택하고 **+ 새 지원 요청**을 클릭한다. 

1. **기본 사항** 탭의 **문제 유형**에서 **서비스 및 구독 제한(할당량)** 을 선택한다. 이 랩에서 사용 중인 구독이 **Subscription** 드롭다운 목록에 나열되어 있는 것을 확인한다.

    >**참고**: **Subscription** 드롭다운 목록에 이 랩에서 사용 중인 구독이 존재하면, 사용 중인 계정에 구독 관련 지원을 요청할 권한이 있음을 확인할 수 있습니다.

    >**참고**: 만약 **서비스 및 구독 제한(할당량)** 옵션이 보이지 않으면, Azure 포털에 다시 로그인합니다.

1. 지원 요청을 그만두고, 로그아웃한 뒤 InPrivate 브라우저 창을 닫는다. 


### 리소스 삭제

   >**참고**: 더 이상 사용하지 않는 새로 생성된 Azure 리소스를 제거하십시오. 

   >**참고**: 사용하지 않는 리소스를 제거해야 예상치 못한 비용이 발생하는 것을 막을 수 있지만, 이 랩에서 생성한 리소스는 추가 과금을 발생하지 않습니다. 

1. Azure 포털에서 **Azure Active Directory**를 찾아 선택하고 **사용자**를 클릭한다.

1. **사용자 - 모든 사용자** 블레이드에서 **az104-02-aaduser1**를 클릭한다.

1. **az104-02-aaduser1 - 프로필** 블레이드에서 **개체 ID** 속성 값을 복사한다. 

1. Azure 포털에서 **Cloud Shell** 내의 **PowerShell** 세션을 시작한다.

1. Cloud Shell 창에서 다음 설정을 사용하여 사용자 정의 역할 할당을 삭제한다. (`[object_ID]`를 이전 작업에서 복사한 **az104-02-aaduser1** 계정의 **object ID**로 대체한다):

   ```powershell
   $scope = (Get-AzRoleAssignment -RoleDefinitionName 'Support Request Contributor (Custom)').Scope

   Remove-AzRoleAssignment -ObjectId '[object_ID]' -RoleDefinitionName 'Support Request Contributor (Custom)' -Scope $scope
   ```

1. Cloud Shell 창에서 다음 명령을 실행하여 사용자 정의 역할을 삭제한다. 

   ```powershell
   Remove-AzRoleDefinition -Name 'Support Request Contributor (Custom)' -Force
   ```

1. Azure 포털에서 **Azure Active Directory**의 **사용자 - 모든 사용자** 블레이드로 돌아가 **az104-02-aaduser1** 사용자 계정을 삭제한다.

1. Azure 포털의 **az104-02-mg1** 관리 그룹 블레이드에서 **details**를 클릭한다.

1. Azure 구독 이름의 오른쪽 끝에 **줄임표** 아이콘을 우클릭하고, **이동**을 클릭한다.

1. **이동** 블레이드에서 구독이 원래 속한 관리 그룹을 선택하고 **저장**을 클릭한다.

1. **관리 그룹** 블레이드로 돌아가 **az104-02-mg1** 관리 그룹을 선택하고, **삭제**를 클릭한다.


### 요약

이 랩에서 우리는

- 관리 그룹을 구현했습니다.
- 사용자 정의 RBAC 역할을 생성했습니다. 
- RBAC 역할을 할당했습니다.