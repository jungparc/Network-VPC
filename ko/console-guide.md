## Network > VPC > 콘솔 사용 가이드

본 문서에서는 콘솔에서 VPC를 다룰 때 필요한 내용을 기술합니다.

## VPC

VPC는 여러 서브넷을 가질 수 있기 때문에 서브넷을 분할하여 사용하는 경우 충분히 큰 네트워크를 설정해야 합니다. VPC 네트워크를 기술하는 방법은 [CIDR Notation](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing)을 사용하여 기술 할 수 있습니다. 모든 VPC는 [프라이빗 네트워크](https://en.wikipedia.org/wiki/Private_network)를 구성할 수 있는 아래 3개의 주소 영역에 있어야 하며 링크 로컬 주소는 사용할 수 없습니다. 또한 적어도 24bit-256개 보다 큰 네트워크 영역을 지정해야 합니다.

### 프라이빗 네트워크

RFC1918 | IP 주소 영역 | 사용 가능한 주소 개수
-------- | ---------- | -----------------
24bit block | 10.0.0.0/8 | 16,777,216
20bit block | 172.16.0.0/12 | 1,047,576
16bit block | 192.168.0.0/16 | 65,536

### 링크 로컬 주소

169.254.0.0/16에 포함되는 65,536개의 IP 주소는 사용할 수 없습니다.

### 예제

예시 | 사용 가능 여부
-------- | ---------- 
10.0.0.0/8 | 사용 가능합니다.
10.0.0.0/16 | 사용 가능합니다.
10.0.0.0/24 | 사용 가능합니다.
10.0.0.0/28 | 사용이 불가합니다. 범위가 너무 작습니다.
172.16.0.0/16 | 사용 가능합니다.
172.16.0.0/8 | 사용이 불가합니다. 사용 가능 범위를 벗어났습니다.
192.168.0.0/16 | 사용 가능합니다. 기본 사용 범위로 지정됩니다.
192.168.0.0/24 | 사용 가능합니다.
192.253.0.0/24 | 사용이 불가합니다. 사용 가능 범위를 벗어났습니다. 

<br>


최초 Compute 와 Network 상품을 사용하면 아래와 같은 항목을 자동으로 구성합니다.

항목 | 이름 | 요약
-------- | ---------- | --------------
VPC | Default Network | 192.168.0.0/16 범위의 VPC 1개가 만들어집니다.
서브넷 | Default Network | 192.168.0.0/24 범위의 서브넷 1개가 만들어집니다.
라우팅 테이블 | vpc-[id] | VPC 아이디의 일부를 이름으로 갖는 라우팅 테이블 1개가 만들어집니다.
인터넷 게이트웨이 | ig-[id] | 라우팅 테이블 아이디의 일부를 이름으로 갖는 인터넷 게이이트웨이 1개가 만들어집니다.
보안 그룹 | default | default라는 이름을 갖는 보안 그룹 1개가 만들어집니다.

초기 구성이 아니라 VPC를 추가하는 경우는 아래 항목을 구성합니다.

항목 | 이름 | 요약
-------- | ---------- | --------------
VPC | 지정한 이름 | 지정한 범위의 VPC 1개가 만들어집니다.
서브넷 | - | 생성되지 않음
라우팅 테이블 | vpc-[id] | VPC 아이디의 일부를 이름으로 갖는 라우팅 테이블 1개가 만들어집니다.
인터넷 게이트웨이 | - | 생성되지 않기 때문에 별도 생성후 연결해야 합니다.
보안 그룹 | - | 추가로 생성되지 않습니다.

VPC와 각 항목의 Quota는 아래와 같습니다.

항목 | 최대값
-------- | ---------- 
VPC | 3
서브넷 | VPC당 10 
인터넷 게이트웨이 | 3 
플로팅 IP | 제한 없음
라우팅 테이블 | VPC당 10 
라우트 | 라우팅 테이블당 10


> [참고]
VPC를 삭제하기 위해서 서브넷을 모두 삭제 할 수 있는 상태에서만 가능하며, 그런 경우라면 서브넷, 라우팅 테이블, 인터넷 게이트웨이와 함께 삭제합니다.

* VPC는 다른 VPC와 완전히 격리되어 트래픽으로부터 안전합니다.

* VPC는 프라이빗 네트워크이기 때문에 인터넷에서 직접 액세스는 불가능합니다.

* VPC 내 모든 요소는 VLAN을 사용할 수 없습니다.

* Region을 넘어서는 트래픽에 대해서는 로컬 통신을 제공하지 않습니다.

* 인터넷 게이트웨이 없이는 VPC 내 모든 인스턴스가 인터넷에 연결되지 않습니다.

* 과도하게 전송되는 "Broadcast, Multicast, Unknown Unicast"는 예고없이 차단될 수 있습니다.


## 서브넷

VPC는 서브넷으로 나누어 작은 네트워크 여러개를 구성할 수 있습니다. 다만 서브넷의 경우에는 VPC 주소 범위에 포함되어야 하며 주소 길이가 같거나 작아야 합니다. 예를 들면 192.168.0.0/16의 경우 192.168.0.0 ~ 192.168.255.255까지 총 65536개의 IP 주소를 사용할 수 있습니다. 또한 가장 작은 서브넷은 28bit이며 이것 보다 작게 구성할 수는 없습니다. 서브넷도 VPC와 같이 CIDR 표기법을 사용합니다.

서브넷이 생성되면 VPC에 포함된 기본 라우팅 테이블에 자동으로 연결됩니다. 이때 게이트웨이 IP는 자동으로 지정됩니다. 

> [참고]
서브넷 삭제는 인스턴스나 로드 밸런서 등이 포함되어 있지 않은 비어있는 서브넷의 경우에만 가능합니다. 또한 서브넷이 연결되어 있는 라우팅테이블에 해당 서브넷으로 향하는 라우트가 없어야 합니다.

* 인스턴스가 생성되면 지정한 서브넷으로부터 한개의 IP 주소를 할당받습니다. (Fixed IP라고 합니다.)

* 인스턴스가 부팅되면 DHCP를 통하여 IP 주소가 인스턴스에 적용됩니다.

* 서브넷의 주소 범위를 수정할 수 없습니다.

* 같은 VPC 내에서 서로 다른 서브넷의 범위가 중복되거나 겹치게 생성할 수 없습니다.

* 다른 VPC에서는 서브넷의 범위가 중복되거나 겹칠 수 있습니다.

* 인스턴스에 할당된 MAC 주소가 아닌 경우 네트워크 상에서 차단될 수 있습니다. 따라서 VPN 서비스를 인스턴스에서 구동하는 경우 동작하지 않을 수 있습니다.

* 인스턴스에 여러개의 서브넷을 연결하는 경우 인스턴스 내 OS에서 적절한 라우팅 설정이 필요합니다.

* 동일한 VPC 내 두 개의 서브넷은 완전히 격리된 것이 아닙니다. 보안 그룹을 사용하여 인스턴스를 보호하세요.

* 서브넷은 서로 다른 가용성 영역에 걸쳐서 로컬 통신을 지원합니다. 로컬 통신은 과금하지 않습니다.

서브넷의 "정적 라우트" 설정을 이용해, 서브넷 내의 인스턴스들이 부팅 시 인스턴스 내의 라우팅 테이블에 설정해야 할 라우팅 룰을 전달할 수 있습니다.

* "정적 라우트"에 등록한 라우팅 룰은 인스턴스가 요청한 DHCP 요청에 대한 응답의 "classless-static-routes" 옵션에 포함되어 전송되며, 인스턴스에서 구동 중인 DHCP 클라이언트에서 이 옵션의 내용을 라우팅 테이블에 등록합니다.

* "classless-static-routes" 옵션의 반영 방식은 인스턴스에서 실행 중인 OS 종류, 배포판 또는 DHCP 클라이언트 버전에 따라 다르지만, 일반적으로 인스턴스 부팅 등 DHCP 클라이언트가 처음 구동되었을 때 반영되고 DHCP 임대 갱신 시에는 반영되지 않습니다. 그러므로 서브넷의 "정적 라우트"를 편집했을 때, 실행 중인 인스턴스에는 변경 내용이 즉시 적용되지 않을 수 있어 가급적 실행 중인 대상 인스턴스들을 재부팅하는 것이 좋습니다.

* "정적 라우트"는 라우팅할 패킷의 목적지 CIDR과, 대상 패킷을 전달할 게이트웨이 정보로 구성됩니다.<br>CIDR이 "0.0.0.0/0"인 정적 라우트를 생성하면, 인스턴스의 기본 게이트웨이를 서브넷의 게이트웨이가 아닌 다른 IP로 변경할 수 있습니다.<br>게이트웨이는 라우팅 테이블의 "라우트"와는 달리 텍스트로 입력하며, 서브넷 내에 아직 할당되지 않은 IP도 지정할 수 있습니다.


## 라우팅 테이블

라우팅 테이블은 VPC와 함께 생성되며 VPC가 삭제되는 경우 함께 삭제됩니다. 라우팅 테이블은 VPC에 복수개가 생성될 수 있으며 기본 라우팅 테이블이 아니라면 명시적으로 삭제할 수 있습니다. 서브넷은 적어도 하나의 라우팅 테이블에 연결되어야 하며, 복수개의 라우팅 테이블이 하나의 인터넷 게이트웨이를 공유할 수 없습니다.

라우팅 테이블 목록을 지정하는 경우 상세 화면에 요약된 정보가 표기되며, "라우트" 탭을 이용하여 경로를 추가할 수 있습니다.

> [참고]
경로를 추가하게 되는 경우 VPC내에 도달 가능한 영역을 지정해야만 추가할 수 있습니다. 그 외에는 실패 메시지가 발생합니다.

* 라우팅 테이블에 포함되는 서브넷의 게이트웨이는 자동으로 추가됩니다.

* "기본 라우팅 테이블"은 삭제할 수 없습니다. VPC 삭제 시 함께 삭제됩니다.

* 서브넷의 게이트웨이와 인터넷 게이트웨이는 라우트 목록에서 삭제할 수 없습니다.

* 라우팅 테이블과 인터넷 게이트웨이 연결을 끊게 되면 인터넷 연결이 끊어집니다.

* 라우팅 테이블의 라우트를 생성하여 특정 CIDR에 대해서 게이트웨이를 지정할 수 있습니다. 라우트를 잘못 설정하면 통신이 단절될 수 있으므로 주의하시기 바랍니다.

라우팅 테이블은 생성되는 위치에 따라 "분산형 라우팅(DVR)" 방식과 "중앙 집중형 라우팅(CVR)" 방식으로 생성할 수 있습니다.

* 분산형 라우팅(DVR) 방식

    * 라우팅 테이블에 연결된 서브넷에 포함되는 인스턴스가 위치하는 하이퍼바이저마다 라우팅 테이블이 생성되는 방식으로, 라우팅 테이블이 사용되는 위치에 고루 분산되므로 안정성과 고가용성을 제공합니다. 

* 중앙 집중형 라우팅(CVR) 방식

    * 중앙에 하나의 라우팅 테이블이 생성되고, 라우팅 테이블에 연결된 서브넷에 포함되는 인스턴스들의 트래픽이 하나의 라우팅 테이블에 집중되는 방식입니다.<br>게이트웨이나 방화벽 등과 같이, 하나의 지점을 통해 트래픽을 제어하는 경우 사용됩니다.

분산형 라우팅(DVR) 방식은 NHN Cloud에서 기본으로 제공하는 방식입니다. 안정성, 고가용성 및 트래픽 분산의 장점이 있어 특별한 경우가 아니면 분산형 라우팅(DVR) 방식을 사용하시는 것을 권고합니다.

라우팅 테이블의 방식을 변경하는 것도 가능하며, 변경 시 라우팅 테이블을 재구성하게 되어 완료되기까지 약 1분간 외부 통신 및 서브넷 간 통신이 단절되므로 주의하시기 바랍니다.

