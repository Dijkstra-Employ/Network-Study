## IPv4와 IPv6

현재 우리는 IPv4를 사용하고 있다.

IPv6으로 넘어가지 못한 이유는 두 가지가 있다.

<br />

1. 4에서 6로 바꾸기 위해선 라우터 장비를 바꿔야 한다.
   라우터는 IPv4만 인식하기 때문이다.
   라우터는 한 개인이 관리하는 것이 아닌 수많은 사람이 관리하는 것이기 때문에 생태계를 바꾸는 것이 쉽지 않다.<br />
2. IPv6로 바꾸지 않아도 네트워크 생태계가 잘 동작하고 있다.

위의 이유로 근본적인 이유를 해결하지 않고 IP 주소를 적게 사용하기 위한 방법을 찾아 적용하고 있다.

<br />

### 어떤 식으로 IP 주소를 배정하는가

임의로 클라이언트가 요구할 때마다 IP 주소를 새로 배정하는 방식은 Scalability 문제가 생긴다.

즉, 포워딩 테이블에 해당 IP 주소에 대해 어디로 나가야하는지에 대한 정보를 모두 저장해야 하는데, 그 크기가 매우 커지게 된다.

그래서 IP 주소를 계층화하여 사용한다.

네트워크 ID 부분과 호스트 ID 부분으로 나누었다.

→ **같은 네트워크에 속한 호스트는 네트워크 ID가 같다.**

<br />

포워딩 테이블은 네트워크 ID에 해당하는 값에 매칭시키도록 테이블을 관리한다.

이를 위해 Subnet mask를 사용한다.

<br />

### DHCP

`Dynamic Host Configuration Protocol`

애플리케이션 계층 프로토콜

기관은 ISP로부터 주소 블록을 획득하여, 개별 IP 주소를 기관 내부의 호스트와 라우터 인터페이스에 할당한다.

호스트에 IP 주소를 할당하는 것은 수동으로 구성이 가능하지만, 일반적으로 동적 호스트 구성 프로토콜(DHCP)을 더 많이 사용한다.

네트워크 관리자는 해당 호스트가 네트워크에 접속하고자 할 때마다 동일한 IP 주소를 받도록 하거나, 다른 임시 IP 주소를 할당하도록 DHCP를 설정한다.

DHCP는 호스트 IP 주소를 할당뿐만 아니라, 서브넷 마스크, 첫 번째 홉 라우터(디폴트 게이트웨이) 주소나 로컬 DNS 서버 주소 같은 추가 정보를 얻게 해준다.

<br />

DHCP는 클라이언트-서버 프로토콜이다.

클라이언트는 일반적으로 IP 주소를 포함하며 네트워크 설정을 위한 정보를 얻고자 새롭게 도착한 호스트다.

일반적으로, 각 서브넷은 DHCP 서버를 가진다.

만약 서버가 현재 서브넷에 없다면, 해당 네트워크에 대한 DHCP 서버 주소를 알려줄 DHCP 연결 에이전트가 필요하다.

<br />

### DHCP IP 할당 동작 과정

<p align="center">
  <img src="https://github.com/user-attachments/assets/d6a322a2-1325-4489-a01a-13c09d81c256" width="70%"/>
</p>

1. **DHCP discover**<br />
   새롭게 도착한 호스트는 상호작용할 DHCP를 발견한다.
   호스트는 자신이 접속될 네트워크의 IP 주소를 알지 못하며 해당 네트워크의 DHCP 서버 주소도 모른다.
   이때 DHCP 클라이언트는 DHCP discover 메시지를 포함하는 IP 데이터그램을 목적지 주소를 255.255.255.255로 설정하여 브로드캐스트한다.

2. **DHCP offer**<br />
   DHCP discover 메시지를 받은 DHCP 서버는 DHCP offer 메시지를 클라이언트에게 응답한다.
   이때에도 브로드캐스트 주소를 사용하여 서브넷의 모든 노드로 이 메시지를 브로드캐스트 한다.
   서브넷에는 여러 DHCP가 존재하기 때문에, 클라이언트는 여러 DHCP 제공 메시지로부터 가장 최적의 위치에 있는 DHCP 서버를 선택한다.
   각각의 서버 제공 메시지는 수신된 discover 메시지의 트랜잭션 ID, 클라이언트에 제공된 IP 주소, 네트워크 마스크, IP 주소 임대 기간을 포함한다.
3. **DHCP request**<br />
   클라이언트는 서버로부터 받은 메시지들 중 선택하여 선택한 IP 주소를 사용하겠다고 브로드캐스트한다.
   브로드캐스트하는 이유는, offer를 받은 다른 DHCP 서버들에게도 알리기 위해서다.
4. **DHCP ACK**<br />
   서버는 request 메시지에 대한 응답 메시지를 보낸다.
   클라이언트가 DHCP ACK 메시지를 받으면, 상호작용은 종료되고 클라이언트는 DHCP 할당 IP 주소를 임대 기간동안 사용할 수 있다.

<br />

## NAT

`Network Address Translation`

내부적으로만 서로 다른 사설 IP를 할당하고, 외부로 나갈 땐 사설 IP를 공인 IP (gateway의 IP 주소)로 변환하여 전달한다.

IP 전환시 포트번호까지 바꾼 이유는 사설 IP 내에선 포트 번호가 겹칠 수 있기 때문이다.

NAT를 사용해서 공인 IP의 사용을 줄이고 내부적으로 사용하는 사설 IP가 늘어남으로써 IPv4를 아직까지 사용할 수 있는 것이다.

<br />

### NAT 사설 IP 사용의 문제

하나의 문제는, 사설 IP를 사용하는 호스트는 클라이언트의 역할로서만 동작하기 때문에, 서버로서의 역할은 할 수 없다.

따라서 사설 IP 내에서 서버를 만들어 배포해도 정상적으로 동작하지 않는다.

정상동작하기 위해선 NAT table에 직접 IP 주소와 포트번호를 작성하여 추가해주어야 한다.

또 하나는, 라우터에서 공인 IP를 사설 IP로 변환하여 호스트를 특정하게 되는데, 이때 Destination IP 주소는 동일하고 Port 번호만 달라져 그 port 번호로 호스트를 특정한다.

원래 IP 주소는 호스트를 구분하는데 사용하고, port 번호는 프로세스를 특정하기 위해 사용한다.

하지만 그 역할이 아닌 다른 역할로서 사용하고 있기 때문에, 실제 port 번호의 제용도를 사용하지 못한다.

<br />

## IPv6

IPv4보다 큰 IP 주소 공간의 필요에 따라, 새로운 IP 프로토콜인 IPv6가 개발되었다.

<br />

### 변화된 사항

- **확장된 주소 기능**<br />
  IPv6는 IP 주소 크기를 32비트에서 128비트로 확장했다.
  IPv6는 유니캐스트, 멀티캐스트 주소뿐만 아니라, 새로운 주소 형태인 애니캐스트 주소가 도입되었다.

- **간소화된 40바이트 헤더**<br />
- **흐름 레이블링**<br />
  비 디폴트 품질 서비스나 실시간 서비스 같은 특별한 처리를 요청하는 송신자에 대해 특정 흐름에 속하는 패킷 레이블링을 가능하게 한다고 한다.

<br />

### IPv6 데이터그램 포맷

<p align="center">
  <img src="https://github.com/user-attachments/assets/2b307301-f65a-406c-9ff9-61cc19aaba2c" width="70%"/>
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/594f11fd-53b3-433b-ab6f-7bb1bc064514" width="70%"/>
</p>

- **버전**<br />
  4비트 필드로 IP 버전 번호를 인식한다.

- **트래픽 클래스**<br />
  8비트 필드는 흐름 내의 SMTP 이메일 같은 애플리케이션의 데이터그램보다 VoIP같은 특정 애플리케이션 데이터그램에 우선순위를 부여하는 데 사용된다.
- **흐름 레이블**<br />
  20비트 필드는 데이터그램의 흐름을 인식하는데 사용된다.
- **페이로드 길이**<br />
  16비트의 필드이며 IPv6 데이터그램에서 고정 길이 40바이트 패킷 헤더 뒤에 나오는 바이트 길이이다.(부호 없는 정수)
- **다음 헤더**<br />
  이 필드는 데이터그램의 내용이 전달될 프로토콜을 구분한다. IPv4의 프로토콜 필드와 같다.

- **홉 제한**<br />
- **출발지와 목적지 주소**<br />
- **데이터**<br />
  데이터그램이 목적지에 도착하면 IP 데이터그램에서 페이로드를 제거한 후, 다음 헤더 필드에 명시한 프로토콜에 전달한다.

<br />

### IPv4와 달리 사라진 필드

- 단편화 재결합<br />
  IPv6에서는 단편화와 재결합을 출발지와 목적지만이 수행한다.
  라우터가 받은 IPv6 데이터그램이 너무 커서 출력 링크로 전달할 수 없다면, 라우터는 데이터그램을 폐기하고 오류 메시지를 송신자에게 보낸다.
  송신자는 데이터를 IP 데이터그램 크기를 줄여서 다시 보낸다.
  단편화와 재결합은 시간이 걸리므로 라우터에서 이 기능을 삭제하고 종단 시스템이 하게 하면 네트워크에서 IP 전달 속도가 증가한다.
- 헤더 체크섬
  트랜스포트 계층 프로토콜과 데이터 링크 프로토콜은 체크섬을 수행하므로 IP 설계자는 네트워크 계층의 체크섬 기능이 반복되는 것으로 생략해도 될 것이라 생각했다.
- 옵션
  옵션 필드는 더 이상 표준 IP 헤더 필드가 아니다.
