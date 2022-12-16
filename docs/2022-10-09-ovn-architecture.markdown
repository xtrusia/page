---
layout: post
title: OVN Architecture
nav_order: 2
---

# OVN Architecture

```
이 문서는 계속 작업중입니다.
``` 

OVN(오픈 가상 네트워크Open Virtual Network)는 가상 네트워크 추상화를 지원하기 위한 시스템이다. OVN은 가상 네트워크 추상화를 네이티브 지원하기 위해 현존하는 OVS의 기능들을 보완한다. 여기에는 가상 L2와 L3 오버레이, 보안 그룹이 포함된다. DHCP와 같은 서비스 또한 가능한 기능이다. OVS처럼, OVN의 설계 목표는 거대한 규모에서도 운용이 가능한 제품으로써 운영 가능한 품질의 구현을 갖는 것이다.

OVN 배포는 다양한 구성 요소로 이뤄진다.

* 클라우드 관리 시스템(CMS, Cloud Management System)은 OVN의 말단 클라이언트(사용자 및 관리자를 통한)이다. OVN 통합integration은 CMS특정 플러그인과 관련 소프트웨어(아래 확인)의 설치를 요구한다. OVN은 처음에는 오픈스택OpenStack을 CMS로 잡았다. 일반적으로 하나의 CMS를 상정하지만, 서로 다른 OVN 배포 부분을 관리하는 다중 CMS 환경도 생각해 볼 수 있다.
물리 또는 가상 노드에 설치된 OVN 데이터베이스는 중심에 위치한다.
* 하나 이상의(또는 많은 수) 하이퍼바이저. 하이퍼바이저는 Open vSwitch를 구동해야 하며, OVS 소스 트리의 IntegrationGuide.rst에 서술된 인터페이스를 구현한다. Open vSwitch가 지원하는 하이퍼바이저 플랫폼은 어떤 것이든 사용 가능하다.
* 0개 이상의 게이트웨이. 게이트웨이는 터널과 물리 이더넷 포트 간 패킷 상호 포워딩을 통해 터널 기반 논리 네트워크를 물리 네트워크로 확장한다. 이는 비 가상화된 머신을 논리 네트워크에 참여할 수 있도록 한다. 게이트웨이는 물리 호스트, 가상 머신, 또는 vtep(5)기술을 지원하는 ASIC 기반 하드웨어 스위치여야 한다. 하이퍼바이저와 게이트웨이는 모두 전송 노드transport node 또는 섀시chassis라 불린다.

아래의 다이어그램은 OVN의 주 구성요소 및 관련 소프트웨어의 상호작용interact를 보여준다. 다이어그램의 최상단부터 보면, 다음과 같다.

* 위에서 정의한 클라우드 관리 시스템
* OVN/CMS 플러그인 는 OVN에 연결짓기 위한 CMS의 구성요소이다. 플러그인의 주 목적은 CMS의 환경설정 데이터베이스에 CMS 특정 포멧으로 저장된 논리 네트워크 환경설정에 대한 CMS의 개념을 OVN이 이해할 수 있는 중간 표현으로 해석하는 것이다. 이 구성요소는 필수적으로 CMS에 특정되어 있다. 따라서, 새 플로그인은 OVN에 통합된 각 CMS를 위해 개발될 필요가 있다. 아래 다이어그램에 있는 모든 구성요소는 CMS에 독립적이다.
* OVN Northbound 데이터베이스는 OVN/CMS 플러그인에 의해 전달된passed down 논리 네트워크 환경설정의 중간 표현을 수신한다. 데이터베이스 스키마는 CMS에서 사용된 개념을 impedance matched 구현한다. 따라서, 논리 스위치, 라우터, ACL 등의 개념을 직접 지원한다. 자세한 사항은 ovn-nb(5)을 보라 OVN Northbound 데이터베이스는 클라이언트가 두가지이다. 상단의 OVN/CMS 플러그인과 하단의 ovn-northd이다.
* ovn-northd(8) 은 상위 OVN Northbound 데이터베이스, 그리고 하위 OVN Southbound 데이터베이스에 접속한다. 이는 논리적 네트워크 환경설정을 기존conventional 네트워크 개념으로 변환한다. 이는 OVN Northbound 데이터베이스에서 취해져서, 하단의 OVN Southbound에 있는 논리 데이터패스datapath 흐름flows으로 변환된다.
* OVN Southbound 데이터베이스는 시스템의 중심이다. 이 데이터베이스의 클라이언트는 상단의 ovn-northd(8), 하단 모든 전송 노드에 있는 ovn-controller(8)이다. OVN Southbound 데이터베이스는 세 종류의 데이터를 포함한다. 
  하이퍼바이저 또는 다른 노드에 닿는 방법을 지정하는 물리 네트워크(PN) 테이블, 논리 데이터패스 흐름이라 불리는 논리 네트워크를 서술하는 논리 네트워크(LN), 논리 네트워크 구성요소를 물리 네트워크에 연결짓는 바인딩Binding 테이블이 그것이다. 하이퍼바이저는 PN과 Port_Binding 테이블을 구성하며, ovn-northd(8)은 LN 테이블을 구성한다. 
  
  OVN Southbound 데이터베이스 성능은 여러 개의 전송 노드와 함께 규모 조정이 가능해야 한다. 이는 병목현상 때문에 ovsdb-server(1)를 대상으로 한 작업이 필요하다. 사용 가능성을 위한 클러스터링이 필요할 수 있다.

나머지 구성요소는 각 하이퍼바이저마다 존재한다.

* ovn-controller(8)은 각 하이퍼바이저 및 소프트웨어 게이트웨이에 존재하는 OVN의 에이전트agent이다. Northbound는 OVN Southbound 데이터베이스에 접속하여 OVN 환경설정과 그 상태를 학습하고, 하이퍼바이저의 상태를 PN 테이블 및 바인딩 테이블의 섀시Chassis 컬럼을 구성한다. Southbound는 네트워크 트래픽 제어를 위해 OpenFlow 컨트롤러인 ovs-vswitchd(8)에 접속하고, 로컬 ovsdb-server(1)에 접속하여 Open vSwitch 환경설정을 모니터링 및 제어한다.

* ovs-vswitchd(8)와 ovsdb-server(1)은 Open vSwitch의 기존 구성요소이다.

                                         CMS
                                          |
                                          |
                              +-----------|-----------+
                              |           |           |
                              |     OVN/CMS Plugin    |
                              |           |           |
                              |           |           |
                              |   OVN Northbound DB   |
                              |           |           |
                              |           |           |
                              |       ovn-northd      |
                              |           |           |
                              +-----------|-----------+
                                          |
                                          |
                                +-------------------+
                                | OVN Southbound DB |
                                +-------------------+
                                          |
                                          |
                       +------------------+------------------+
                       |                  |                  |
         HV 1          |                  |    HV n          |
       +---------------|---------------+  .  +---------------|---------------+
       |               |               |  .  |               |               |
       |        ovn-controller         |  .  |        ovn-controller         |
       |         |          |          |  .  |         |          |          |
       |         |          |          |     |         |          |          |
       |  ovs-vswitchd   ovsdb-server  |     |  ovs-vswitchd   ovsdb-server  |
       |                               |     |                               |
       +-------------------------------+     +-------------------------------+

# OVN에서 정보 흐름

OVN 흐름에서의 환경설정 데이터는 north에서 south로 흐른다. CMS는 그 OVN/CMS 플러그인과 northbound 데이터베이스를 통해 논리 네트워크 환경설정을 ovn-northd로 전달한다. 결국, ovn-northd는 환경설정을 저수준 형태로 변환하고, 이를 southbound 데이터베이스를 통해 모든 섀시로 전달한다.

OVN 흐름에서의 상태 정보는 south에서 north로 흐른다. OVN은 현재 몇 가지의 상태 정보만을 제공한다. 먼저, ovn-northd는 northbound Logical_Switch_Port 테이블의 up 컬럼을 구성한다. 만약 southbound Port_Binding 테이블에 논리 포트의 chassis 컬럼이 비어있다면, up을 true로 설정한다. 그렇지 않으면 false로 설정한다. 이는 CMS가 VM의 네트워킹이 시작되었을 때 이를 탐지할 수 있도록 한다.

둘째로, OVN은 그 환경설정이 실현되었음을 CMS에 피드백한다. 즉, CMS가 제공한 환경설정이 효과를 발생할 때 마다 피드백한다. 이 기능은 CMS가 순차 번호 프로토콜sequence number protocol에 참여해야 한다. 이는 다음과 같이 동작한다.

1. CMS가 northbound 데이터베이스에 환경설정을 갱신하면, 같은 트랜잭션의 일부로써, NB_Global 테이블의 nb_cfg 컬럼의 값을 늘린다. (이는 CMS가 환경설정이 언제 실현되었는지 알기 원할 때에만 필요하다.)

2. ovn-northd가 northbound 데이터베이스의 주어진 스냅샷snapshot에 따라 southbound 데이터베이스를 갱신하면, 같은 트랜잭션의 일부로써 northbound NB_Global의 nb_cfg를 복사하여 southbound 데이버테이스의 SB_Global 테이블로 복사한다. (따라서, 두 데이터베이스의 옵저버 모니터링은 southbound 데이터베이스와 northbound가 서로 정보를 주고받은 시점을 알 수 있다.)

3. ovn-northd가 southbound 데이터베이스 서버로부터 그 변경이 적용됨에 대한 확인confirmation을 수신한 후에, northbound NB_Global 테이블의 sb_cfg를 내려보낸(pushed down) nb_cfg 버전으로 갱신한다. (따라서, CMS 또는 기타 옵저버는 southbound 데이터베이스에 접속 없이 southbound 데이터베이스의 정보 전달 시점을 알 수 있다. )

4. 각 섀시의 ovn-controller 프로세스는 갱신된 nb_cfg를 포함한 갱신된 southbound 데이터베이스를 수신한다. 이 프로세스는 결국 섀시의 Open vSwitch 인스턴스에 설치된 물리 흐름flow를 갱신한다. Open vSwitch로부터 물리 흐름이 갱신되었다는 확답을 수신하면, southbound 데이터베이스의 고유(its own) Chassis 레코드를 갱신한다.

5. ovn-northd는 southbound 데이터베이스에 있는 모든 Chassis 레코드의 nb_cfg 컬럼을 모니터링한다. 이들 모든 레코드의 최소값을 계속 추적하며, 이를 northbound NB_Global 테이블의 hv_cfg 컬럼에 복제한다. (따라서, CMS 또는 기타 옵저버는 언제 모든 하이퍼바이저가 northbound 환경설정을 가지는 지 알 수 있다.)

# 섀시 설정
OVN 배포에서의 각 섀시는 OVN에만 할당된 Open vSwitch 브릿지로 설정되어야 한다. 이는 통합 브릿지(integration bridge)라 불린다. 시스템 시작 스크립트는 ovn-controller를 구동 하기 전에 이 브릿지를 생성한다. 만약 이 브릿지가 ovn-controller 시작 시에 존재하지 않으면, 아래에 제안된 기본 환경설정을 가지고 자동으로 생성될 것이다. 통합 브릿지의 포트는 다음을 포함한다.

* 모든 섀시에서, OVN이 논리 네트웍 연결을 유지하기 위해 사용하는 터널 포트. ovn-controller는 이 터널 포트를 추가, 갱신, 삭제한다.

* 하이퍼바이저에서, 모든 VIF는 논리 네트워크에 연결(attached)된다. 하이퍼바이저 자체, 또는 Open vSwitch 및 하이퍼바이저(IntegrationGuide.rst에 서술됨.) 사이의 통합이 이를 관리한다(이는 OVN의 일부이거나, OVN에서 새로이 가져온 것은 아니다. 이는 OVS를 지원하는 하이퍼바이저가 이미 수행하고 있는 기존 통합 작업이다.).

* 게이트웨이에서, 논리 네트워크 연결을 위한 물리 포트. 시스템 시작 스크립트는 ovn-controller를 시작 하기 전에, 이 포트를 브릿지에 추가한다. 이는 더 복잡한 설정에서, 다른 브릿지로의 패치 포트(patch port)일 수 있다. 물리 포트 대신에 말이다.

다른 포트는 통합 브릿지에 연결되어서는 안된다. 특히, 기저 네트워크에 연결된 물리 포트(논리 네트워크에 연결된 물리 포트인 게이트웨이 포트와는 반대이다.)는 통합 브릿지에 연결되어서는 안된다. 대신 기저 물리 포트는 별도의 Open vSwitch 브릿지에 연결되어야 한다(사실 어떤 브릿지에도 연결될 필요는 없다.).

통합 브릿지는 다음에 서술된 바와 같이 설정되어야 한다. 아래와 같은 설정의 각 의미는 ovs-vswitchd.conf.db(5)에 문서화되어 있다.

- fail-mode=secure
    ovn-controller 시작 전에, 독립된 논리 네트워크 사이의 패킷 전환을 피한다. 자세한 정보는 ovs-vsctl(8)의 Controller Failure Settings 부분을 참고하라.
- other-config:disable-in-band=true
    통합 브릿지를 위한 in-band 제어 흐름을 억제한다. OVN은 원격 컨트롤러 대신 지역 컨트롤러(유닉스 도메인 소켓을 통한)를 사용하기 때문에, 이는 일반적이지는 않다.(추가요망). 그러나 같은 시스템의 다른 브릿지가 in-band 원격 컨트롤러를 갖도록 하는 것이 가능하다. 그리고 이 경우, in-band 제어가 본래 설정된 흐름을 억제한다. 더 자세한 정보는 문서를 참고하라.

통합 브릿지의 관습명은 br-int이다. 그러나 다른 이름도 사용 가능하다.

# 논리 네트워크
논리 네트워크는 물리 네트워크와 같은 개념이다. 그러나, 이는 터널 및 기타 캡슐화를 통해 물리 네트워크로부터 분리된다. 이는 논리 네트워크가 물리 네트워크가 사용중인 것과는 별도의 IP 및 중첩된 다른 주소 공간을 충돌 없이 가질 수 있게 한다. 논리 네트워크 토폴로지는 구동중인 물리 네트워크의 토폴로지와는 관계 없이 할당이 가능하다.

OVN에서의 논리 네트워크 개념은 다음을 포함한다.

* 논리 스위치: 이더넷 스위치의 논리 버전

* 논리 라우터: IP 라우터의 논리 버전. 논리 스위치와 라우터는 복잡한 토폴로지에 접속 가능하다.

* 논리 데이터패스(datapath): 이는 OpenFlow 스위치의 논리 버전이다. 논리 스위치와 라우터는 모두 논리 데이터패스로 구현된다.

* 논리 포트: 이는 논리 스위치와 논리 라우터의 입출력 연결 지점을 표현한다. 논리 포트의 일반적인 형태는 다음과 같다.
  
    * VIF를 나타내는 논리 포트

    * 논리 스위치와 물리 네트워크 간 연결 지점을 나타내는 Localnet 포트. 이들은 통합 브릿지와 별도의 Open vSwitch 브릿지 사이에 OVS 패치 포트로 구현된다. 이는 기저 물리 포트가 연결된다.

    * 논리 스위치와 논리 라우터 간 연결 지점을 나타내는 논리 패치 포트. 어떤 경우에는 논리 라우터간 연결도 가능하다. 각 지점에 논리 패치 포트의 쌍이 존재한다.

    * 논리 스위치와 VIF간의 지역 연결 지점을 표현하는 Localport 포트. 이 포트는 모든 섀시(특정 부분에 속하지 않는.)에 존재하며, 여기서 나오는 트래픽은 터널을 통하지 않는다. localport는 지역을 대상으로 하는 트래픽만 생성할 것으로 기대한다. 보통 수신하는 요청에 반응하는 경우 말이다. 한 예로, OpenStack Neutron 이 localport를 사용하여 모든 하이퍼바이저에 상주하는 VM에게 메타데이터를 제공하는데 사용한다. 메타데이터 프록시 프로세스는 이 모든 호스트에 있는 포트에 연결된다. 같은 네트워크에 있는 모든 VM은 같은 IP/MAC 주소를 사용하여 터널을 통과하지 않고도 도달이 가능하다. 더 자세한 정보는 다음을 확인하라. https://docs.openstack.org/developer/networking-ovn/design/metadata_api.html.

# VIF의 생에 주기
독립되어 표현된 테이블(Table)과 그 스키마는 이해하기가 어렵다. 아래는 그 예이다.

하이퍼바이저의 VIF는 하이퍼바이저에서 직접 구동중인 VM 또는 컨테이너에 연결된 가상 네트워크 인터페이스이다(이는 VM 내에서 구동되는 컨테이너의 인터페이스와는 다르다.).

이 예제의 단계는 OVN과 OVN Northbound 데이터베이스 스키마의 상세를 참조한다. 이 데이터베이스의 전체 내용은 ovn-sb(5)와 ovn-nb(5)를 각각 살펴보라.

1. VIF의 생애 주기는 CMS 관리자가 CMS 사용자 인터페이스나 API를 이용해서 새 VIF를 생성하고, 이를 스위치(OVN를 통해 논리 스위치로 구현된 것.)에 추가할 떄 시작된다. CMS는 그 고유의 환경설정을 갱신한다. 여기에는 해당 VIF와 관련된 고유의, 영구적인 식별자 vif-id와 이더넷 주소 mac이 포함된다.

2. CMS 플러그인은 새 VIF를 포함시키기 위해 Logical_Switch_Port 테이블에 행을 추가함으로써 OVN Northbound 데이터베이스를 갱신한다. 새 행에는, name은 vif-id를, mac은 mac을 뜻하며, switch는 OVN 논리 스위치의 Logical_Switch 레코드를 가리킨다. 또한 다른 열들도 적절히 초기화된다.

3. ovn-northd는 OVN Northbound 데이터베이스 갱신을 수신한다. 이제 대응되는 갱신을 OVN Southbound 데이터베이스에 수행한다. 새 포트를 반영하기 위해 OVN Southbound 데이터베이스 Logical_Flow 테이블을 추가함으로써 말이다. 예를 들어, 새 포트의 MAC 주소로 향하는 패킷을 인지하기 위핸 플로우(flow) 추가 및 새 포트를 포함하는 브로드캐스트 및 멀티캐스트 패킷을 전달하기 위한 플로우 갱신이 그것이다. 또한 Binding 테이블의 레코드를 생성하고, chassis를 식별하는 열을 제외한 모든 열을 populate한다.

4. 모든 하이퍼바이저에서, ovn-controller는 이전 단계에서 ovn-northd가 수행한 Logical_Flow 테이블의 갱신을 수신한다. 해당 VIF를 소유하는 VM이 전원이 꺼져 있는 한, ovn-controller는 많은 일을 수행할 수 없다. 예를 들어, VIF가 실제로는 어디에도 존재하지 않기 떄문에, 해당 VIF에서 패킷을 보내거나 수신하는 것을 할당할 수 없다.

5. 언젠가 사용자는 해당 VIF를 갖는 VM의 전원을 켤 것이다. 켜진 VM이 존재하는 하이퍼바이저에서, 하이퍼바이저와 Open vSwitch(IntegrationGuide.rst에 서술됨) 사이의 통합은 해당 VIF를 OVN 통합 브릿지에 추가하고, 해당 인터페이스가 새 VIF의 인스턴스화라는 것을 가리키기 위해 vif-id를 external_ids:iface-id에 저장한다(이 코드는 OVN에서 새로 구현된 것이 아니다. 이는 OVS를 지원하는 하이퍼바이저가 이미 수행하고 있는 통합 작업이다.).

6. 켜진 VM이 있는 하이퍼바이저에서, ovn-controller는 새 인터페이스의 external_ids:iface-id를 인지한다. OVN Southbound DB에서, 이는 external_ids:iface-id로부터의 논리 포트를 하이퍼바이저에 연결하는 열인 Binding 테이블의 chassis 열을 갱신한다. 그 후, ovn-controller는 지역 하이퍼바이저의 OpenFlow 테이블을 갱신하여 해당 VIF를 출입하는 패킷을 적절히 처리하게 한다.

7. OpenStack을 포함하는 몇몇 CMS 시스템에서는 네트워크가 완전히 준비된 경우에만 VM을 구동한다. 이를 지원하기 위해, ovn-northd는 Binding 테이블에 있는 열을 위해 갱신된 chassis 열을 인지하고, 해당 VIF가 이제 사용 가능하다는 것을 가리키기 위해 OVN Northbound 데이터베이스의 Logical_Switch_Port 테이블의 up 열을 갱신하도록 한다. 이 기능을 사용하는 CMS는 VM의 실행을 허용한다.

8. VIF가 상주하고 있지 않은 모든 하이퍼바이저에서, ovn-controller는 Binding 테이블에 poulated된 열을 인지한다. 이는 ovn-controller에 논리 포트의 물리 위치를 제공한다. 따라서, 각 인스턴스는 그 스위치(OVN DB Logical_Flow 테이블의 논리 데이터패스 흐름을 기반으로)의 OpenFlow 테이블을 갱신하여 해당 VIF에 출입하는 패킷은 터널을 통해 적절히 처리된다.

9. 끝으로, 해당 VIF를 갖는 VM의 전원을 끊을 것이다. 꺼진 VM이 있는 하이퍼바이저에서, VIF는 OVN 통합 브릿지에서 삭제된다.

10. 전원이 꺼진 VM을 갖는 하이퍼바이저에서, ovn-controller는 VIF가 삭제되었음을 인지한다. 따라서, Binding 테이블의 논리 포트를 나타내는 Chassis 열의 내용을 삭제한다.

11. 모든 하이퍼바이저에서, ovn-controller는 Binding 테이블의 논리 포트를 위한 행에서 빈 Chassis 열을 인지한다. 이는 ovn-controller가 더 이상 해당 논리 포트의 물리적 위치를 알지 못한다는 것이다. 따라서, 각 인스턴스는 이를 반영하기 위해 그 OpenFlow 테이블을 갱신한다.

12. 결국 VIF(또는 그 VM전체)가 더 이상 누구에게도 필요하지 않은 경우, 관리자는 CMS 사용자 인터페이스 또는 API를 사용해서 VIF를 삭제한다. CMS는 그 환경설정을 갱신한다.

13. CMS 플러그인은 OVN Northbound 데이터베이스에서 VIF를 삭제한다. Logical_Switch_Port 테이블의 열을 삭제함으로써 말이다.

14. ovn-northd는 OVN Northbound 갱신을 수신하고, OVN Southbound 데이터베이스를 그에 맞게 갱신한다. OVN Southbound 데이터베이스에서 삭제된 VIF와 관련된 Logical_Flow 테이블과 Binding 테이블에서 열을 삭제 또는 갱신한다.

15. 모든 하이퍼바이저에서, ovn-controller는 이전 단계에서 ovn-northd가 수행한 Logical_Flow 테이블의 갱신을 수신한다. ovn-controller는 해당 갱신을 반영하기 위해 OpenFlow 테이블을 갱신한다. 할 수 있는게 많진 않지만, 이전 단계에서 Binding 테이블로부터 삭제가 되었을 때 이미 해당 VIF는 도달할 수 없게 되었을 것이다.

# VM 내의 컨테이너 인터페이스의 생애 주기
