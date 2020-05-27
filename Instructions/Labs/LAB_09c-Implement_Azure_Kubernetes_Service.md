---
lab:
    title: '09c - Azure Kubernetes Service 구현'
    module: '모듈 09 - 서버리스 컴퓨팅'
---

# 랩 09c - Azure Kubernetes Service 구현


## 랩 시나리오

Contoso는 Azure Container Instances에서 동작하는데 적합한 다수의 멀티 티어 애플리케이션을 보유합니다. 컨테이너화된 워크로드를 동작시킬 수 있는지 확인하기 위해 컨테이너 오케스트레이션 도구로 쿠버네티스를 사용할 수 있는지 평가합니다. 관리 오버헤드를 더 줄이기 위해 배포와 확장이 편리한 Azure Kubernetes Service를 테스트합니다.


## 목표

이 과정에서, 우리는 다음과 같은 실습을 합니다 :

+ 작업 1: Azure Kubernetes Service 클러스터 배포
+ 작업 2: Azure Kubernetes Service 클러스터에 파드 배포
+ 작업 3: Azure Kubernetes service 클러스터의 컨테이너화된 워크로드 확장


## 설명

### 작업 1: Azure Kubernetes Service 클러스터 배포

이 작업에서는 Azure 포털을 사용하여 Azure Kubernetes Services 클러스터를 배포합니다.

1. [Azure portal](https://portal.azure.com)에 로그인한다.

1. Azure 포털에서 **Kubernetes 서비스**를 찾아 클릭하고 블레이드에서 **+ 추가**를 클릭한다. 

1. **기본 사항** 탭에서 다음 설정을 사용한다. (다른 값은 기본 설정을 사용한다) 

    | 설정 | 값 |
    | ---- | ---- |
    | 구독 | 이 랩에서 사용하는 애저 구독의 이름 |
    | 리소스 그룹 | 새로 만들기 **az104-09c-rg1** |
    | Kubernetes 클러스터 이름 | **az104-9c-aks1** |
    | 지역 | 쿠버네티스 클러스터를 프로비전할 수 있는 지역의 이름 |
    | Kubernetes 버전 | 기본 값 사용 |
    | DNS name prefix | any valid, globally unique DNS host name |
    | 노드 크기 | 기본값 사용 |
    | 노드 개수 | **1** |

1. **다음: 노드풀 >** 을 클릭하고, 다음 설정을 사용한다. (다른 값은 기본 설정을 사용한다) 

    | 설정 | 값 |
    | ---- | ---- |
    | 가상 노드 | **사용 안 함** |
    | VM 확장 집합 | **사용 안 함** |
	
1. **다음: 인증 >** 을 클릭하고, 다음 설정을 사용한다. (다른 값은 기본 설정을 사용한다) 

    | 설정 | 값 |
    | ---- | ---- |
    | 서비스 사용자 | 기본 값 사용 |
    | RBAC(역할 기반 액세스 제어) | **사용** |


1. **다음: 네트워킹 >** 을 클릭하고, 다음 설정을 사용한다. (다른 값은 기본 설정을 사용한다)

    | 설정 | 값 |
    | ---- | ---- |
    | HTTP 애플리케이션 라우팅 | **아니요** |
    | 부하 분산 장치 | **Standard** |
    | 네트워크 구성 | **고급** |

1. **다음: 통합 >** 을 클릭하고, **container monitoring**을 **사용 안 함**으로 설정한다 **검토 + 만들기**를 클릭하고 유효성 검사를 통과하면 **만들기**를 클릭한다. 

    >**참고**: 클러스터를 만들 때 모니터링을 사용할 수 있지만, 이 랩에서는 모니터링을 다루지 않습니다.

    >**참고**: 배포가 끝날 때까지 기다리십시오. 이 작업은 약 10분 소요됩니다. 


### 작업 2: Azure Kubernetes Service 클러스터에 파드 배포

이 작업에서는 Azure Kubernetes Service 클러스터에 파드를 배포합니다.

1. 배포 블레이드에서 **리소스로 이동**을 클릭한다.

1. **az104-9c-aks1** Kubernetes 서비스 블레이드에서 **설정** 섹션의 **노드 풀**을 클릭한다.

1. **az104-9c-aks1 - 노드 풀** 클러스터가 노드 1개를 가진 하나의 풀로 구성되어 있는 것을 확인한다. 

1. Azure 포털 오른쪽 위의 아이콘을 클릭하여 **Azure Cloud Shell**을 실행한다.

1. Azure 포털에서 **Cloud Shell**의 **Bash** 세션을 시작한다.

    >**참고**: **Cloud Shell**을 처음 실행한 경우, **탑재된 스토리지가 없음** 메시지가 표시됩니다. 이 랩에서 사용하고 있는 구독을 선택하고 **스토리지 만들기**를 클릭하십시오. 

1. Cloud Shell 창에서 다음을 실행하여 AKS 클러스터에 액세스하기 위한 자격 증명을 검색한다. 

    ```sh
    RESOURCE_GROUP='az104-09c-rg1'

    AKS_CLUSTER='az104-9c-aks1'

    az aks get-credentials --resource-group $RESOURCE_GROUP --name $AKS_CLUSTER
    ``` 

1. **Cloud Shell** 창에서 다음 명령을 실행하여 AKS 클러스터에 대한 연결을 확인한다. 

    ```sh
    kubectl get nodes
    ```

1. **Cloud Shell** 창에서 출력을 검토하고, 클러스터가 구성하는 한 노드가 **Ready** 상태인지 확인한다.

1. **Cloud Shell** 창에서 다음을 실행하여 Docker Hub로부터 **nginx** 이미지를 배포한다. 

    ```sh
    kubectl create deployment nginx-deployment --image=nginx
    ```

    > **참고**: 배포의 이름(nginx-deployment)은 소문자로 입력하십시오. 

1. **Cloud Shell** 창에서 다음을 실행하여 Kubernetes 파드가 생성되었는지 확인한다. 

    ```sh
    kubectl get pods
    ```

1. **Cloud Shell** 창에서 다음을 실행하여 배포 상태를 확인한다.

    ```sh
    kubectl get deployment
    ```

1. **Cloud Shell** 창에서 다음을 실행하여 인터넷에서 파드를 사용할 수 있도록 한다. 

    ```sh
    kubectl expose deployment nginx-deployment --port=80 --type=LoadBalancer
    ```

1. **Cloud Shell** 창에서 다음을 실행하여 공용 IP 주소가 프로비전 되었는지 확인한다. 

    ```sh
    kubectl get service
    ```

1. **nginx-deployment** 항목의 **EXTERNAL-IP** 열의 값이 **\<pending\>**에서 공용 IP 주소로 변경될 때까지 명령을 다시 실행한다. **EXTERNAL-IP** 열의 공용 IP 주소를 기록해 둔다.

1. 브라우저 창을 열어서 기록해 둔 공용 IP 주소로 접속한다. 브라우저 페이지에 **Welcome to nginx!** 메시지가 출력되는 것을 확인한다. 


### 작업 3: Azure Kubernetes service 클러스터의 컨테이너화된 워크로드 확장

이 작업에서는 파드와 클러스터 노드의 개수를 확장합니다. 

1. From the **Cloud Shell** 창에서 다음을 실행하여 파드의 개수를 2로 증가시켜서 배포를 확장한다. 

    ```sh
    kubectl scale --replicas=2 deployment/nginx-deployment
    ```

1. **Cloud Shell** 창에서 다음을 실행하여 배포를 확장한 결과를 확인한다. 

    ```sh
    kubectl get pods
    ```

    > **참고**: 명령의 결과를 확인하고 파드 개수가 2개로 증가한 것을 확인합니다.

1. From the **Cloud Shell** 창에서 다음을 실행하여 노드 수를 2로 증가시켜서 클러스터를 확장한다.  

    ```sh
    az aks scale --resource-group $RESOURCE_GROUP --name $AKS_CLUSTER --node-count 2
    ```

    > **참고**: 추가 노드의 프로비전이 완료될 때까지 기다리십시오. 이 작업은 약 3분 소요됩니다. 만약 실패하면 `az aks scale` 명령을 재실행하십시오. 

1. **Cloud Shell** 창에 다음을 실행하여 클러스터 확장 결과를 확인한다. 

    ```sh
    kubectl get nodes
    ```

    > **참고**: 명령 결과를 검토하여 노드의 수가 2개로 증가한 것을 확인하십시오.

1. **Cloud Shell** 창에서 다음을 실행하여 배포를 확장한다.

    ```
    kubectl scale --replicas=10 deployment/nginx-deployment
    ```

1. **Cloud Shell** 창에서 다음을 실행하여 배포 확장의 결과를 확인한다.

    ```
    kubectl get pods
    ```

    > **참고**: 명령 결과를 검토하여 파드의 수가 10개로 증가한 것을 확인한다. 

1. **Cloud Shell** 창에서 다음을 실행하여 클러스터 노드에 파드가 배포된 것을 확인한다.

    ```
    kubectl get pod -o=custom-columns=NODE:.spec.nodeName,POD:.metadata.name
    ```

    > **참고**: 명령의 출력을 검토하고 파드가 두 노드에 분산되어 있는지 확인하십시오.

1. **Cloud Shell** 창에서 다음을 실행하여 배포를 삭제한다. 

    ```
    kubectl delete deployment nginx-deployment
    ```

1. **Cloud Shell** 창을 닫는다. 


### 리소스 삭제

   >**참고**: 사용하지 않는 새로 생성된 Azure 리소스를 제거하십시오. 사용하지 않는 리소스를 제거해야 예상치 못한 비용이 발생하지 않습니다.

1. Azure 포털에서 **Cloud Shell**의 **Bash** 세션을 시작한다.

1. 다음 명령을 실행하여 이 모듈의 실습에서 생성된 모든 리소스 그룹을 나열한다.

   ```sh
   az group list --query "[?starts_with(name,'az104-09c')].name" --output tsv
   ```

1. 다음 명령을 실행하여 이 모듈의 실습에서 생성한 모든 리소스 그룹을 삭제한다.

   ```sh
   az group list --query "[?starts_with(name,'az104-09c')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

    >**참고**: 이 명령은 비동기적으로 실행되므로( --nowait 매개 변수로 결정됨) 동일한 PowerShell 세션 내에서 즉시 다른 PowerShell 명령을 실행할 수 있지만, 리소스 그룹이 실제로 제거되기까지는 몇 분 정도 소요됩니다.


### 요약

이 랩에서 우리는

- Azure Kubernetes Service 클러스터를 배포했습니다. 
- Azure Kubernetes Service 클러스터에 파드를 배포했습니다.
- Azure Kubernetes service 클러스터의 컨테이너화된 워크로드를 확장했습니다. 