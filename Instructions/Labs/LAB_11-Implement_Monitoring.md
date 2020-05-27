---
lab:
    title: '11 - 모니터링 구현'
    module: '모듈 11 - 모니터링'
---

# 랩 11 - 모니터링 구현


## 랩 시나리오

You need to evaluate Azure functionality that would provide insight into performance and configuration of Azure resources, focusing in particular on Azure virtual machines. To accomplish this, you intend to examine the capabilities of Azure Monitor, including Log Analytics. 

## 목표

이 과정에서, 우리는 다음과 같은 실습을 합니다 :

+ 작업 1: 랩 환경 프로비전
+ 작업 2: Create and configure an Azure Log Analytics workspace and Azure Automation-based solutions 
+ 작업 3: Review default monitoring settings of Azure virtual machines
+ 작업 4: Configure Azure virtual machine diagnostic settings
+ 작업 5: Review Azure Monitor functionality
+ 작업 6: Review Azure Log Analytics functionality


## 설명


### 작업 1: 랩 환경 프로비전

이 작업에서는 모니터링 시나리오를 테스트하는 데 사용할 가상 머신을 배포합니다. 

1. [Azure portal](https://portal.azure.com)에 로그인한다. 

1. Azure 포털 오른쪽 위의 아이콘을 클릭하여 **Azure Cloud Shell**을 실행한다.

1. **Bash** 또는 **PowerShell**을 선택하는 프롬프트 창에서 **PowerShell**을 선택한다. 

    >**참고**:  **Cloud Shell**을 처음 실행한 경우, **탑재된 스토리지가 없음** 메시지가 표시됩니다. 이 랩에서 사용하고 있는 구독을 선택하고 **스토리지 만들기**를 클릭하십시오.

1. Cloud Shell 창의 툴바에서 **파일 업로드/다운로드** 아이콘을 선택하고, **업로드**를 클릭하여 **\\Allfiles\\Labs\\11\\az104-11-vm-template.json** 과 **\\Allfiles\\Labs\\11\\az104-11-vm-parameters.json** 파일을 Cloud Shell 홈 디렉토리에 업로드한다. 

1. Cloud Shell 창에서 다음 명령을 실행하여 가상 머신을 호스팅할 리소스 그룹을 생성한다. (`[Azure_region]`을 가상 머신을 배포할 Azure 지역의 이름으로 대체한다):

    >**참고**: **Log Analytics Workspace Region** 목록에 있는 지역을 선택해야 합니다. 목록은 [Workspace mappings documentation](https://docs.microsoft.com/en-us/azure/automation/how-to/region-mappings)에서 확인하십시오.

   ```powershell
   $location = '[Azure_region]'

   $rgName = 'az104-11-rg0'

   New-AzResourceGroup -Name $rgName -Location $location
   ```

1. Cloud Shell 창에서 다음 명령을 실행하여, 업로드한 템플릿과 파라미터 파일을 이용해 첫 번째 가상 네트워크에 가상 머신을 배포한다. 

   ```powershell
   New-AzResourceGroupDeployment `
      -ResourceGroupName $rgName `
      -TemplateFile $HOME/az104-11-vm-template.json `
      -TemplateParameterFile $HOME/az104-11-vm-parameters.json `
      -AsJob
   ```

1. Cloud Shell 창을 최소화한 채 열어둔다.

    >**참고**: 배포가 완료될 때까지 기다리지 않고 다음 작업을 진행하십시오. 배포에는 약 3분 소요됩니다.


### 작업 2: Create and configure an Azure Log Analytics workspace and Azure Automation-based solutions

In this task, you will create and configure an Azure Log Analytics workspace and Azure Automation-based solutions 
이 작업에서는 Azure Log Analytics 작업 영역과 

1. Azure 포털에서 **Log Analytics 작업 영역**읓 찾아 선택하고, 블레이드에서 **+ 추가**를 클릭한다.

1. 다음 설정을 사용하여 새 Log Analytics 작업 영역을 생성한다. 

    | 설정 | 값 |
    | --- | --- |
    | 인스턴스 이름 | 고유한 이름 |
    | 구독 | 이 랩에서 사용하는 구독의 이름 |
    | 리소스 그룹 | 새로 만들기 **az104-11-rg1** |
    | 영역 | 이전 작업에서 가상 머신을 배포한 Azure 영역의 이름 |
    | 가격 책정 계층 | **종량제** |

    >**참고**: 반드시 이전 작업에서 가상 머신을 배포한 영역과 같은 영역을 사용해야 합니다. 

    >**참고**: 배포가 끝날 때까지 기다리십시오. 배포에는 약 1분 소요됩니다. 

1. Azure 포털에서 **Automation 계정**을 찾아 선택하고, 블레이드에서 **+ 추가**를 클릭한다.

1. **Automation 계정 추가** 블레이드에서 다음 설정을 사용하고 **만들기**를 클릭한다.

    | 설정 | 값 |
    | --- | --- |
    | 이름 | 고유한 이름 |
    | 구독 | 이 랩에서 사용하는 구독의 이름 |
    | 리소스 그룹 | **az104-11-rg1** |
    | 위치 | [Workspace mappings documentation](https://docs.microsoft.com/en-us/azure/automation/how-to/region-mappings) 목록에 있는 Azure 영역의 이름 |
    | Azure 실행 계정 만들기 | **예** |

    >**참고**: 반드시 [Workspace mappings documentation](https://docs.microsoft.com/en-us/azure/automation/how-to/region-mappings) 목록에 있는 Azure 영역을 사용하십시오.

    >**참고**: 배포가 끝날 때까지 기다리십시오. 이 작업은 약 3분 소요됩니다.

1. **Add Automation 계정** 블레이드에서 **새로 고침**을 클릭하여 Automation 계정이 생성된 것을 확인하고 선택한다. 

1. **구성 관리** 섹션에서 **인벤토리**를 클릭한다.

1. **인벤토리** 창의 **Log Analytics 작업 영역** 드롭다운 리스트에서 앞서 생성한 Analytics 작업 영역을 선택하고, **사용**을 클릭한다.

    >**참고**: Log Analytics 솔루션을 설치할 때까지 기다리십시오. 이 작업은 약 3분 소요됩니다.

    >**참고**: **변경 내용 추적** 솔루션은 자동으로 설치됩니다. 

1. Automation 계정 블레이드의 **업데이트 관리**를 선택하고 **사용**을 클릭한다.

    >**참고**: 설치가 완료될 때까지 가디리십시오. 이 작업은 약 5분 소요됩니다.


### 작업 3: Azure 가상 머신 기본 모니터링 환경 검토

In this task, you will review default monitoring settings of Azure virtual machines
이 작업에서는 Azure 가상 머신의 기본 모니터링 환경을 검토합니다. 

1. Azure 포털에서 **가상 머신**을 찾아 선택하고, 목록에서 **az104-11-vm0**를 클릭한다.

1. **az104-11-vm0** 블레이드에서 **모니터링** 섹션의 **메트릭**을 클릭한다.

1. **az104-11-vm0 - 메트릭** 블레이드 기본 차트의 **메트릭 네임스페이스**에 **가상 머신 호스트**만 사용할 수 있는 것을 확인한다.

    >**참고**: This is expected, since no guest-level diagnostic settings have been configured yet.

1. **메트릭** 드롭다운 리스트에서 이용 가능한 메트릭 목록을 검토한다.

    >**참고**: The list includes a range of CPU, disk, and network-related metrics that can be collected from the virtual machine host, without having access into guest-level metrics. 이 목록은 CPU의 범위, 디스크, 네트워크 관련 메트릭을 포함합니다. 

1. In the **METRICS** drop-down list, select **Percentage CPU**, in the **AGGREGATION** drop-down list, select **Avg**, and review the resulting chart. 

### 작업 4: Configure Azure virtual machine diagnostic settings

In this task, you will configure Azure virtual machine diagnostic settings.

1. On the **az104-11-vm0** blade, in the **Monitoring** section, click **Diagnostic settings**.

1. On the **Overview** tab of the **az104-11-vm0 - Diagnostic settings** blade, click **Enable guest-level monitoring**.

    >**Note**: Wait for the operation to take effect. This might take about 3 minutes.

1. Switch to the **Performance counters** tab of the **az104-11-vm0 - Diagnostic settings** blade and review the available counters.

    >**Note**: By default, CPU, memory, disk, and network counters are enabled. You can switch to the **Custom** view for more detailed listing.

1. Switch to the **Logs** tab of the **az104-11-vm0 - Diagnostic settings** blade and review the available event log collection options.

    >**Note**: By default, log collection includes critical, error, and warning entries from the Application Log and System log, as well as Audit failure entries from the Security log. Here as well you can switch to the **Custom** view for more detailed configuration settings.

1. On the **az104-11-vm0** blade, in the **Monitoring** section, click **Logs**. 

1. On the **az104-11-vm0 - Logs** blade, ensure that the Log Analytics workspace you created earlier in this lab is selected in the **Choose a Log Analytics Workspace** drop-down list and click **Enable**.

1. On the **az104-11-vm0 - Logs** blade, click **Enable**, select the Log Analytics workspace you created earlier in this lab from the **Choose a Log Analytics Workspace** drop-down list, and click **Enable** again.

    >**Note**: Do not wait for the operation to complete but instead proceed to the next step. The operation might take about 5 minutes.

1. On the **az104-11-vm0 - Logs** blade, in the **Monitoring** section, click **Metrics**.

1. On the **az104-11-vm0 - Metrics** blade, on the default chart, note that at this point, the **METRICS NAMESPACE** drop-down list, in addition to the **Virtual Machine Host** entry includes also the **Guest (classic)** entry.

    >**Note**: This is expected, since you enabled guest-level diagnostic settings.

1. In the **METRICS** drop-down list, review the list of available metrics.

    >**Note**: The list includes additional guest-level metrics not available when relying on the host-level monitoring only. 

1. In the **METRICS** drop-down list, select **Memory\Available Bytes**, in the **AGGREGATION** drop-down list, select **Avg**, and review the resulting chart. 

### 작업 5: Review Azure Monitor functionality

1. In the Azure portal, search for and select **Monitor** and, on the **Monitor - Overview** blade, click **Metrics**.

1. In the chart pane on the right side of the blade, in the **SCOPE** drop-down list, click **+ Select a scope**.

1. On the **Select a scope** blade, on the **Browse** tab, navigate to the **az104-11-rg0** resource group, expand it, select the **az104-11-vm0** virtual machine within that resource group, and click **Apply**.

    >**Note**: This gives you the same view and options as those available from the **az104-11-vm0 - Metrics** blade.

1. On the **Monitor - Metrics** blade, click **New alert rule**.

    >**Note**: Creating an alert rule from Metrics is not supported for metrics from the Guest (classic) metric namespace. This can be accomplished by using Azure Resource Manager templates, as described in the document [Send Guest OS metrics to the Azure Monitor metric store using a Resource Manager template for a Windows virtual machine](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/collect-custom-metrics-guestos-resource-manager-vm)

1. On the **Create rule** blade, in the **RESOURCE** section, click **Select**, on the **Select a resource** blade, navigate to the **az104-11-vm0** virtual machine entry, select the checkbox next to it, and click **Done**. 

1. On the **Create rule** blade, in the **CONDITION** section, click **Add**. 

1. On the **Configure signal logic** blade, in the list of signals, click **Percentage CPU**, in the **Alert logic** section, specify the following settings (leave others with their default values) and click **Done**:

    | Settings | Value |
    | --- | --- |
    | Threshold | **Static** |
    | Operator | **Greater than** |
    | Aggregation type | **Average** |
    | Threshold value | **2** |
    | Aggregation granularity (Period) | **1 minute** |
    | Frequency of evaluation | **Every 1 Minute** |

1. On the **Create rule** blade, in the **ACTION GROUPS (optional)** section, click **Create**.

1. On the **Add action group** blade, specify the following settings (leave others with their default values):

    | Settings | Value |
    | --- | --- |
    | Action group name | **az104-11-ag1** |
    | Short name | **az104-11-ag1** |
    | Subscription | the name of the Azure subscription you are using in this lab |
    | Resource group | **az104-11-rg1** |

1. On the **Add action group** blade, in the **Actions** section, specify the following settings (leave others with their default values):

    | Settings | Value |
    | --- | --- |
    | Action group name | **az104-11-ag1 email** |
    | Action Type | **Email/SMS/Push/Voice** |

1. In the **az104-11-ag1 email** action row, click **Edit details**

1. On the **Email/SMS/Push/Voice** blade, select the **Email** checkbox, type your email address in the **Email** textbox, leave others with their default values, click **OK**, and back on the **Add action group** blade, click **OK** again.

1. Back on the **Create rule** blade, specify the following settings (leave others with their default values):

    | Settings | Value |
    | --- | --- |
    | Alert rule name | **CPU Percentage above the test threshold** |
    | Description | **CPU Percentage above the test threshold** |
    | Severity | **Sev 3** |
    | Enable rule upon creation | **Yes** |

1. Click **Create alert rule** and close the **Create rule** blade.

    >**Note**: It can take up to 10 minutes for a metric alert rule to become active.

1. In the Azure portal, search for and select **Virtual machines**, and on the **Virtual machines** blade, click **az104-11-vm0**.

1. On the **az104-11-vm0** blade, click **Connect**, in the drop-down menu, click **RDP**, on the **Connect with RDP** blade, click **Download RDP File** and follow the prompts to start the Remote Desktop session.

    >**Note**: This step refers to connecting via Remote Desktop from a Windows computer. On a Mac, you can use Remote Desktop Client from the Mac App Store and on Linux computers you can use an open source RDP client software.

    >**Note**: You can ignore any warning prompts when connecting to the target virtual machines.

1. When prompted, sign in by using the **Student** username and **Pa55w.rd1234** password.

1. Within the Remote Desktop session, click **Start**, expand the **Windows System** folder, and click **Command Prompt**.

1. From the Command Prompt, run the following to copy the restore the **hosts** file to the original location:

   ```
   for /l %a in (0,0,1) do echo a
   ```

    >**Note**: This will initiate the infinite loop that should increase the CPU utilization above the threshold of the newly created alert rule.

1. Leave the Remote Desktop session open and switch back to the browser window displaying the Azure portal on your lab computer.

1. In the Azure portal, navigate back to the **Monitor** blade and click **Alerts**.

1. Note the number of **Sev 3** alerts and then click the **Sev 3** row.

    >**Note**: You might need to wait for a few minutes and click **Refresh**.

1. On the **All Alerts** blade, review generated alerts.


### 작업 6: Review Azure Log Analytics functionality


1. In the Azure portal, navigate back to the **Monitor** blade, click **Logs**. 

    >**Note**: You might need to click **Get Started** if this is the first time you access Log Analytics.

1. On the **Select a scope** blade, navigate to the **az104-11-rg0** resource group, expand it, select **a104-11-vm0**, and click **Apply**.

1. Click **Example queries** in the toolbar, in the **Get started with sample queries** pane, review each tab, locate **Virtual machine available memory**, and click **Run**.

1. Review the resulting chart and remove the line containing the following text:

   ```
   | where TimeGenerated > ago(1h)
   ```

    >**Note**: As the result, the **Time range** entry in the toolbar changed from **Set in query** to **Last 24 hours**. 

1. Rerun the query and examine the resulting chart.

1. On the **New Query 1** tab, on the **Tables** tab, review the list of **Virtual machines** tables.

1. In the list of tables in the **Virtual machines** section. 

    >**Note**: The names of several tables correspond to the solutions you installed earlier in this lab.

1. Hover the mouse over the **VMComputer** entry and click the **Preview data** icon.  

1. If any data is available, in the **Update** pane, click **See in query editor**.

    >**Note**: You might need to wait a few minutes before the update data becomes available.

1. Examine output displayed in the query results.

1. Click **Example queries** in the toolbar, in the **Get started with sample queries** pane, review each tab, locate **Virtual machine free disk space**, and click **Run**.


### Clean up resources

   >**Note**: Remember to remove any newly created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

1. In the Azure portal, open the **PowerShell** session within the **Cloud Shell** pane.

1. List all resource groups created throughout the labs of this module by running the following command:

   ```powershell
   Get-AzResourceGroup -Name 'az104-11*'
   ```

1. Delete all resource groups you created throughout the labs of this module by running the following command:

   ```powershell
   Get-AzResourceGroup -Name 'az104-11*' | Remove-AzResourceGroup -Force -AsJob
   ```

    >**Note**: The command executes asynchronously (as determined by the -AsJob parameter), so while you will be able to run another PowerShell command immediately afterwards within the same PowerShell session, it will take a few minutes before the resource groups are actually removed.


### Review

In this lab, you have:

- Provisioned the lab environment
- Created and configured an Azure Log Analytics workspace and Azure Automation-based solutions
- Reviewed default monitoring settings of Azure virtual machines
- Configured Azure virtual machine diagnostic settings
- Reviewed Azure Monitor functionality
- Reviewed Azure Log Analytics functionality
