---
lab:
    title: '03a - Azure 포털을 사용해 Azure 리소스 관리'
    module: '모듈 03 - Azure 관리'
---

# 랩 03a - Azure 포털을 사용해 Azure 리소스 관리


## 랩 시나리오

리소스 그룹을 기반으로 리소스 프로비저닝 및 구성과 관련된 기본 Azure 관리 기능을 탐색합니다. 리소스 그룹 간의 리소스 이동 실습을 포함합니다. 또한 디스크 리소스가 실수로 삭제되지 않도록 보호하는 옵션도 살펴볼 것입니다. 디스크의 성능 특성과 크기를 수정하는 작업도 수행합니다.


## 목표

이 과정에서, 우리는 다음과 같은 실습을 합니다 :

+ 작업 1: 리소스 그룹을 생성하고 리소스를 배포
+ 작업 2: 리소스 그룹 간 리소스 이동
+ 작업 3: 리소스 잠금 구현 및 테스트


## 설명

### 작업 1: 리소스 그룹을 생성하고 리소스를 배포

이 작업에서는 리소스 그룹 및 디스크를 생성하기 위해 Azure 포털을 사용합니다.

1. [Azure portal](https://portal.azure.com)에 로그인한다.

1. **리소스 그룹**을 찾아 선택한다. 

1. **리소스 그룹** 블레이드에서, **+ 추가** 를 선택하고 다음 설정을 사용하여 리소스 그룹을 만든다.

    |설정|값|
    |---|---|
    |구독| 이 랩에서 사용할 구독의 이름 |
    |리소스 그룹| **az104-03a-rg1**|
    |영역| 이 랩에서 사용할 구독에서 이용 가능한 지역 |

1. **검토 + 만들기**를 클릭한 다음 **만들기**를 클릭한다.

1. Azure 포털에서 **Disks**를 검색하고 선택한다. **+ 추가**를 클릭하고, 다음 설정을 사용한다.

    |설정|값|
    |---|---|
    |구독| 리소스 그룹을 만든 구독의 이름 |
    |리소스 그룹| **az104-03a-rg1** |
    |디스크 이름| **az104-03a-disk1** |
    |지역| 리소스 그룹을 만들었던 지역 |
    |가용성 영역| **없음** |
    |원본 유형| **없음** |

    >**참고**: 리소스를 만들 때, 새로운 리소스 그룹을 만들거나 기존의 리소스 그룹을 선택해 사용할 수 있습니다.

1. 디스크 유형과 크기를 각각 **표준 HDD** 와 **32 GiB**로 변경한다.

1. **검토 + 만들기**를 클릭하고 **만들기**를 클릭한다.

    >**참고**: 디스크가 다 만들어질 때까지 기다리세요. 이 작업은 1분 미만 소요됩니다. 


### 작업 2: 리소스 그룹 간 리소스 이동

이 작업에서는 이전 작업에서 만든 디스크 리소스를 새로운 리소스 그룹으로 이동할 것입니다. 

1. **리소스 그룹**을 찾아 선택한다. 

1. **리소스 그룹** 블레이드에서, 이전 작업에서 만들었던 **az104-03a-rg1** 리소스 그룹을 클릭한다.

1. 리소스 그룹의 **개요** 블레이드에서 앞서 이전 작업에서 만들었던 디스크 리소스의 체크박스를 선택한다. 리소스 그룹 툴바의 **이동**을 클릭한 뒤, 드롭다운 리스트에서 **다른 리소스 그룹으로 이동**을 선택한다.

    >**참고**: 이 방법은 여러 리소스를 한 번에 옮기는 작업을 지원합니다.

1. **리소스 이동** 블레이드에서 **새 그룹 만들기**를 클릭한다.

1. **리소스 그룹** 텍스트 입력 칸에 **az104-03a-rg2**를 입력한다. **새 리소스 ID를 사용하려는 경우 이동한 리소스와 연결된 도구 및 스크립트를 업데이트하지 않으면 정상적으로 작동하지 않는다는 점을 이해합니다**의 체크 박스를 선택하고, **확인**을 클릭한다.

    >**참고**: 작업이 완료될 때까지 기다리지 말고 다음 작업으로 넘어갑니다. 리소스 이동은 약 10분이 소요됩니다. 소스 또는 대상 리소스 그룹의 작업 로그 항목을 모니터링하여 작업이 완료되었는지 확인할 수 있습니다. 다음 작업을 완료한 후 모니터링 하십시오.


### 작업 3: 리소스 잠금 구현 및 테스트

이 작업에서는 디스크 리소스를 포함하는 Azure 리소스 그룹에 대해 리소스 잠금을 적용할 것입니다.

1. Azure 포털에서 **Disks**를 검색하고 선택한다. **+ 추가**를 클릭하고, 다음 설정을 사용한다.

    |설정|값|
    |---|---|
    |구독| 이 랩에서 사용하고 있는 구독의 이름 |
    |리소스 그룹| 새로 만들기 **az104-03a-rg3** |
    |디스크 이름| **az104-03a-disk2** |
    |지역| 이 랩에서 다른 리소스 그룹을 만들었던 지역의 이름 |
    |가용성 영역| **없음** |
    |원본 유형| **없음** |

1. 디스크 유형과 크기를 각각 **표준 HDD** 와 **32 GiB**로 변경한다.

1. **검토 + 만들기**를 클릭하고 **만들기**를 클릭한다.

1. Azure 포털에서 **리소스 그룹**을 찾아 선택한다. 

1. 리소스 그룹 목록에서 **az104-03a-rg3** 리소스 그룹을 찾아 클릭한다.

1. **az104-03a-rg3** 리소스 그룹 블레이드에서 **잠금**을 클릭한다. **+ 추가**를 클릭하고, 다음 설정을 사용한다.

    |설정|값|
    |---|---|
    |잠금 이름| **az104-03a-delete-lock** |
    |잠금 유형| **삭제** |

1. **az104-03a-rg3** 리소스 그룹 블레이드에서 **개요**를 클릭한다. 리소스 리스트에서, 이전 작업에서 생성했던 디스크의 체크 박스를 선택한다. 툴바에 있는 **삭제**를 클릭한다. (**태그 지정** 오른쪽에 있는 **삭제**를 클릭한다)

1. **선택한 모든 리소스를 삭제하시겠습니까?** 라는 창이 뜨면, 텍스트 상자에 **예**를 입력하고 **삭제**를 클릭한다.

1. 삭제 실패를 알리는 에러 메시지를 확인할 수 있다.

    >**참고**: 오류 메시지에 따르면, 리소스 그룹 수준에 적용된 삭제 잠금으로 인해 생긴 오류입니다.

1. 다시 **az104-03a-rg3** 리소스 그룹으로 돌아가서 **az104-03a-disk2** 디스크 리소스를 클릭한다.

1. **az104-03a-disk2** 블레이드에서 **설정** 섹션에 있는 **구성**을 클릭하고, 스토리지 유형과 크기를 각각 **프리미엄 SSD**와 **64 GiB**로 설정하고 저장한다. 변경에 성공한 것을 확인한다. 

    >**참고**: 리소스 그룹 수준 잠금은 삭제에 대해서만 적용했기 때문에 예상된 결과입니다.


### 리소스 삭제

   >**참고**: 이 랩에서 사용한 리소스를 지우지 마십시오. 이 모듈에 대한 다음 랩에서 사용할 예정입니다. 이 랩에서 만들었던 리소스 잠금만 삭제하십시오.

1. **az104-03a-rg3** 리소스 그룹 블레이드에서 **잠금**을 클릭한다. **az104-03a-delete-lock** 오른편의 **삭제**를 클릭해 잠금을 삭제한다. 


#### 요약

이 랩에서 우리는

- 리소스 그룹을 만들고 리소스를 배포했습니다.
- 리소스 그룹 간 리소스를 이동했습니다.
- 리소스 잠금을 구현하고 테스트했습니다.
