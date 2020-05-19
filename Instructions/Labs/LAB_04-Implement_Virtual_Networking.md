---
lab:
    title: '04 - 가상 네트워킹 구현'
    module: '모듈 04 - 가상 네트워킹'
---

# 랩 04 - 가상 네트워킹 구현

# 학생 실습 매뉴얼

## 랩 시나리오

Azure 가상 네트워킹 기능을 살펴보십시오. 우선 Azure에 가상 네트워크를 생성하여 Azure 가상 시스템 몇 개를 호스팅할 것입니다. 네트워크 기반 분할을 구현할 예정이므로 가상 네트워크의 다른 서브넷에 배포하십시오. 또한 시간이 지남에 따라 개인 IP 주소와 공용 IP 주소가 변경되지 않도록 하십시오. Contoso 보안 요구 사항을 준수하려면 인터넷에서 액세스할 수 있는 Azure 가상 시스템의 공용 엔드포인트를 보호해야 합니다. 마지막으로 가상 네트워크와 인터넷에서 모두 Azure 가상 시스템에 DNS 이름 확인을 구현해야 합니다.

## 목표

이 과정에서, 우리는 다음과 같은 실습을 합니다 :

+ 작업 1: 가상 네트워크 생성 및 구성
+ 작업 2: 가상 네트워크에 가상 머신 배포
+ 작업 3: Azure 가상 머신의 사설 및 공용 IP 주소 설정
+ 작업 4: 네트워크 보안 그룹 구성
+ 작업 5: 내부 이름 확인을 위한 Azure DNS 구성
+ 작업 6: 외부 이름 확인을 위한 Azure DNS 구성

## 설명

### 연습 1

#### 작업 1: 가상 네트워크 생성 및 구성

이 작업에서는 Azure 포털을 사용해 다수의 서브넷이 있는 가상 네트워크를 만듭니다.

1. [Azure portal](https://portal.azure.com)에 로그인한다.

1. Azure 포털에서 **가상 네트워크**를 찾아 선택한다. **가상 네트워크** 블레이드에서 **+ 추가**를 클릭한다.

1. 다음 설정을 사용하여 가상 네트워크를 만든다. (다른 값은 기본 설정을 사용한다)

    | 설정 | 값 |
    | --- | --- |
    | 구독 | 이 랩에서 사용할 Azure 구독의 이름 |
    | 리소스 그룹 | 새로만들기 **az104-04-rg1** |
    | 이름 | **az104-04-vnet1** |
    | 지역 | 이 랩에서 사용할 구독에서 이용 가능한 지역의 이름 |
    | IPv4 주소 공간 | **10.40.0.0/20** |
    | 서브넷 이름 | **subnet0** |
    | 서브넷 주소 범위 | **10.40.0.0/24** |

    >**참고:** 가상 네트워크가 프로비전될 때까지 기다리십시오. 이 작업은 1분 미만 소요됩니다.

1. **가상 네트워크** 블레이드에서 **새로 고침**을 하고, **az104-04-vnet1**을 클릭한다.

1. **az104-04-vnet1** 가상 네트워크 블레이드에서 **서브넷**을 선택하고, **+ 서브넷**을 클릭한다. 

1. 다음 설정을 사용하여 서브넷을 만든다. (다른 값은 기본 설정을 사용한다)

    | 설정 | 값 |
    | --- | --- |
    | Name | **subnet1** |
    | 주소 범위 (CIDR 블록) | **10.40.1.0/24** |
    | 네트워크 보안 그룹 | **없음** |
    | 경로 테이블 | **없음** |

#### 작업 2:  가상 네트워크에 가상 머신 배포

이 작업에서는 ARM 템플릿을 사용하여 Azure 가상 머신을 가상 네트워크의 다른 서브넷에 배포할 것입니다.

1. 포털에서 오른쪽 위의 이아콘을 클릭하여 **Azure Cloud Shell**을 시작한다. 

1. **Bash** 또는 **PowerShell**을 선택하는 프롬프트 창에서 **PowerShell**을 선택한다.

    >**참고**: **참고**: **Cloud Shell**을 처음 실행한 경우, **탑재된 스토리지가 없음** 메시지가 표시됩니다. 이 랩에서 사용하고 있는 구독을 선택하고 **스토리지 만들기**를 클릭하십시오. 


1. Cloud Shell 창의 툴바에서 **파일 업로드/다운로드** 아이콘을 선택한다. 드롭다운 메뉴에서 **업로드**를 클릭하고, **\\Allfiles\\Labs\\04\\az104-04-vms-template.json** 와 **\\Allfiles\\Labs\\04\\az104-04-vms-parameters.json** 을 Cloud Shell의 홈 디렉토리에 업로드한다. 

    >**참고**: 파일을 각각 업로드 하십시오.

1. From the Cloud Shell pane, run the following to deploy two virtual machines by using the template and parameter files you uploaded:

   ```pwsh
   $rgName = 'az104-04-rg1'

   New-AzResourceGroupDeployment `
      -ResourceGroupName $rgName `
      -TemplateFile $HOME/az104-04-vms-template.json `
      -TemplateParameterFile $HOME/az104-04-vms-parameters.json
   ```

    >**Note**: This method of deploying ARM templates uses Azure PowerShell. You can perform the same task by running the equivalent Azure CLI command **az deployment create** (for more information, refer to [Deploy resources with Resource Manager templates and Azure CLI](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/deploy-cli).

    >**Note**: Wait for the deployment to complete before proceeding to the next task. This should take about 2 minutes.

1. Close the Cloud Shell pane.

#### Task 3: Configure private and public IP addresses of Azure VMs

In this task, you will configure static assignment of public and private IP addresses assigned to network interfaces of Azure virtual machines.

   >**Note**: Private and public IP addresses are actually assigned to the network interfaces, which, in turn are attached to Azure virtual machines, however, it is fairly common to refer to IP addresses assigned to Azure VMs instead.

1. In the Azure portal, search for and select **Resource groups**, and, on the **Resource groups** blade, click **az104-04-rg1**.

1. On the **az104-04-rg1** resource group blade, in the list of its resources, click **az104-04-vnet1**.

1. On the **az104-04-vnet1** virtual network blade, review the **Connected devices** section and verify that there are two network interfaces **az104-04-nic0** and **az104-04-nic1** attached to the virtual network.

1. Click **az104-04-nic0** and, on the **az104-04-nic0** blade, click **IP configurations**. 

    >**Note**: Verify that **ipconfig1** is currently set up with a dynamic private IP address.

1. In the list IP configurations, click **ipconfig1**.

1. On the **ipconfig1** blade, set **Assignment** to **Static**, leave the default value of **IP address** set to **10.40.0.4**.

1. On the **ipconfig1** blade, set **Public IP address** to **Enabled** and then click **IP address - Configure required settings**. 

1. On the **Choose public IP address blade**, click **+ Create new** and create a new public IP address with the following settings:

    | Setting | Value |
    | --- | --- |
    | Name | **az104-04-pip0** |
    | SKU | **Standard** |

1. Back on the **ipconfig1** blade, save the changes.

1. Navigate back to the **az104-04-vnet1** blade and repeat the previous six steps to change the IP address assignment of **ipconfig1** of **az104-04-nic1** to **Static** and associate **az104-04-nic1** with a new Standard SKU public IP address named **az104-04-pip1**.

1. Navigate back to the **az104-04-rg1** resource group blade, in the list of its resources, click **az104-04-vm0**, and from the **az104-04-vm0** virtual machine blade, note the public IP address entry.

1. Navigate back to the **az104-04-rg1** resource group blade, in the list of its resources, click **az104-04-vm1**, and from the **az104-04-vm1** virtual machine blade, note the public IP address entry.

    >**Note**: You will need both IP addresses in the last task of this lab. 


#### Task 4: Configure network security groups

In this task, you will configure network security groups in order to allow for restricted connectivity to Azure virtual machines.

1. In the Azure portal, navigate back to the **az104-04-rg1** resource group blade, and in the list of its resources, click **az104-04-vm0**.

1. On the **az104-04-vm0** blade, click **Connect**, in the drop-down menu, click **RDP**, on the **Connect with RDP** blade, click **Download RDP File** and follow the prompts to start the Remote Desktop session.

1. Note that the connection attempt fails.

    >**Note**: This is expected, because public IP addresses of the Standard SKU, by default, require that the network interfaces to which they are assigned are protected by a network security group. In order to allow Remote Desktop connections, you will create a network security group explicitly allowing inbound RDP traffic from Internet and assign it to network interfaces of both virtual machines.

1. In the Azure portal, search for and select **Network security groups**, and, on the **Network security groups** blade, click **+ Add**.

1. Create a network security group with the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Subscription | the name of the Azure subscription you are using in this lab |
    | Resource Group | **az104-04-rg1** |
    | Name | **az104-04-nsg01** |
    | Region | the name of the Azure region where you deployed all other resources in this lab |

    >**Note**: Wait for the deployment to complete. This should take about 2 minutes.

1. On the deployment blade, click **Go to resource** to open the **az104-04-nsg01** network security group blade. 

1. On the **az104-04-nsg01** network security group blade, in the **Settings** section, click **Inbound security rules**. 

1. Add an inbound rule with the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Source | **Any** |
    | Source port ranges | * |
    | Destination | **Any** |
    | Destination port ranges | **3389** |
    | Protocol | **TCP** |
    | Action | **Allow** |
    | Priority | **300** |
    | Name | **AllowRDPInBound** |

1. On the **az104-04-nsg01** network security group blade, in the **Settings** section, click **Network interfaces** and then click **+ Associate**.

1. Associate the **az104-04-nsg01** network security group with the **az104-04-nic0** and **az104-04-nic1** network interfaces.

    >**Note**: It may take up to 5 minutes for the rules from the newly created Network Security Group to be applied to the Network Interface Card.

1. Navigate back to the **az104-04-vm0** virtual machine blade.

    >**Note**: Now verify that you can successfully to the target virtual machine and sign in by using the **Student** username and **Pa55w.rd1234** password.

1. On the **az104-04-vm0** blade, click **Connect**, click **Connect**, in the drop-down menu, click **RDP**, on the **Connect with RDP** blade, click **Download RDP File** and follow the prompts to start the Remote Desktop session.

    >**Note**: This step refers to connecting via Remote Desktop from a Windows computer. On a Mac, you can use Remote Desktop Client from the Mac App Store and on Linux computers you can use an open source RDP client software.

    >**Note**: You can ignore any warning prompts when connecting to the target virtual machines.

1. When prompted, sign in by using the **Student** username and **Pa55w.rd1234** password.

    >**Note**: Leave the Remote Desktop session open. You will need it in the next task.

#### Task 5: Configure Azure DNS for internal name resolution

In this task, you will configure DNS name resolution within a virtual network by using Azure private DNS zones.

1. In the Azure portal, search for and select **Private DNS zones** and, on the **Private DNS zones** blade, click **+ Add**.

1. Create a private DNS zone with the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Subscription | the name of the Azure subscription you are using in this lab |
    | Resource Group | **az104-04-rg1** |
    | Name | **contoso.org** |

    >**Note**: Wait for the private DNS zone to be created. This should take about 2 minutes.

1. Click **Go to resource** to open the **contoso.org** DNS private zone blade. 

1. On the **contoso.org** private DNS zone blade, in the **Settings** section, click **Virtual network links**

1. Add a virtual network link with the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Link name | **az104-04-vnet1-link** |
    | Subscription | the name of the Azure subscription you are using in this lab |
    | Virtual network | **az104-04-vnet1** |
    | Enable auto registration | enabled |

    >**Note:** Wait for the virtual network link to be created. This should take less than 1 minute.

1. On the **contoso.org** private DNS zone blade, in the **Settings** section, click **Overview**

1. Verify that the DNS records for **az104-04-vm0** and **az104-04-vm1** appear in the list of record sets as **Auto registered**.

    >**Note:** You might need to wait a few minutes and refresh the page if the record sets are not listed.

1. Switch to the Remote Desktop session to **az104-04-vm0**, right-click the **Start** button and, in the right-click menu, click **Windows PowerShell (Admin)**.

1. In the Windows PowerShell console window, run the following to test internal name resolution of the **az104-04-vm1** DNS record set in the newly created private DNS zone:

   ```pwsh
   nslookup az104-04-vm1.contoso.org
   ```
1. Verify that the output of the command includes the private IP address of **az104-04-vm1** (**10.40.1.4**).

#### Task 6: Configure Azure DNS for external name resolution

In this task, you will configure external DNS name resolution by using Azure public DNS zones.

1. In the Azure portal, search for and select **DNS zones** and, on the **DNS zones** blade, click **+ Add**.

1. Create a DNS zone with the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Subscription | the name of the Azure subscription you are using in this lab |
    | Resource Group | **az104-04-rg1** |
    | Name | **contoso.org** |

    >**Note**: Wait for the DNS zone to be created. This should take about 2 minutes. 

1. Click **Go to resource** to open the **contoso.org** DNS zone blade. 

1. On the **contoso.org** DNS zone blade, click **+ Record set**.

1. Add a record set with the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Name | **az104-04-vm0** |
    | Type | **A** |
    | Alias record set | **No** |
    | TTL | **1** |
    | TTL unit | **Hours** |
    | IP address | the public IP address of **az104-04-vm0** which you identified in the third exercise of this lab |

1. Add a record set with the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Name | **az104-04-vm1** |
    | Type | **A** |
    | Alias record set | **No** |
    | TTL | **1** |
    | TTL unit | **Hours** |
    | IP address | the public IP address of **az104-04-vm1** which you identified in the third exercise of this lab |

1. On the **contoso.org** DNS zone blade, note the name of the **Name server 1** entry.

1. In the Azure portal, open the **PowerShell** session in **Cloud Shell** by clicking on the icon in the top right of the Azure Portal.

1. From the Cloud Shell pane, run the following to test external name resolution of the **az104-04-vm0** DNS record set in the newly created DNS zone (replace the placeholder `[Name server 1]` with the name of **Name server 1** you noted earlier in this task):

   ```pwsh
   nslookup az104-04-vm0.contoso.org [Name server 1]
   ```
1. Verify that the output of the command includes the public IP address of **az104-04-vm0**.

1. From the Cloud Shell pane, run the following to test external name resolution of the **az104-04-vm1** DNS record set in the the newly created DNS zone (replace the placeholder `[Name server 1]` with the name of **Name server 1** you noted earlier in this task):

   ```pwsh
   nslookup az104-04-vm1.contoso.org [Name server 1]
   ```
1. Verify that the output of the command includes the public IP address of **az104-04-vm1**.

#### Clean up resources

   >**Note**: Remember to remove any newly created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

1. In the Azure portal, open the **PowerShell** session within the **Cloud Shell** pane.

1. List all resource groups created throughout the labs of this module by running the following command:

   ```pwsh
   Get-AzResourceGroup -Name 'az104-04*'
   ```

1. Delete all resource groups you created throughout the labs of this module by running the following command:

   ```pwsh
   Get-AzResourceGroup -Name 'az104-04*' | Remove-AzResourceGroup -Force -AsJob
   ```

    >**Note**: The command executes asynchronously (as determined by the -AsJob parameter), so while you will be able to run another PowerShell command immediately afterwards within the same PowerShell session, it will take a few minutes before the resource groups are actually removed.

#### Review

In this lab, you have:

- Created and configured a virtual network
- Deployed virtual machines into the virtual network
- Configured private and public IP addresses of Azure VMs
- Configured network security groups
- Configured Azure DNS for internal name resolution
- Configured Azure DNS for external name resolution
