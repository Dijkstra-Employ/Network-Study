## 계층별 전송 단위

[ 애플리케이션 ] - Message

[ 전송 계층 ] - Segment

[ 네트워크 계층 ] - IP 패킷의 datagram

데이터와 헤더 부분으로 나뉘며 헤더의 필드들이 어떤 기능을 위해 사용되는지 이해하는 것이 프로토콜을 이해하는 것이다.

<br />

## UDP

UDP가 지원하는 기능은 UDP 헤더를 보면 알 수 있다.

`source port` , `dest port` → Multiplexing, Demultiplexing

`checksum` → 에러 detection

![image](https://github.com/user-attachments/assets/3b9602f7-f2c3-416f-960e-b9e11c802b5b)


많은 링크 계층 프로토콜이 오류 검사를 제공하는데, 왜 UDP가 체크섬을 제공할까?

이는 출발지와 목적지 사이의 모든 링크가 오류 검사를 제공한다는 보장이 없기 때문이다.

UDP는 오류 검사를 제공하지만, 오류를 회복하기 위한 어떤 일도 하지 않는다.

<br />

### TCP보다 UDP를 더 선호하는 이유

- UDP는 TCP와 달리 혼잡 제어를 하지 않는다.
  혼잡 제어를 하게 되면 메시지 송신자를 제한하게 된다.
  실시간 애플리케이션은 최소 전송률을 요구할 때가 있으며, 지나치게 지연되는 세그먼트 전송을 원하지 않거나 조금의 데이터 손실은 허용할 수도 있다.
- 연결 설정이 없다.
  TCP는 3-way-handshake를 통한 지연시간이 발생하지만 UDP는 연결 과정이 없다.
- 연결 상태가 없음
  UDP는 연결 상태를 유지하지 않으며 혼잡 제어 파라미터, 순서 번호와 확인 응답 번호 등의 파라미터를 기록하지 않기에 TCP보다 좀 더 많은 액티브 블라이언트를 수용할 수 있다.
- 작은 패킷 헤더 오버헤드
  UDP는 제공하는 기능이 적기에 헤더의 크기가 작다.

<br />

## TCP

`reliable data transfer` : rdt

신뢰적인 채널에서는 전송된 데이터가 손상되거나 손실되지 않는다.

그리고 모든 데이터는 전송된 순서 그대로 전달된다. 이것이 TCP가 인터넷 애플리케이션에게 제공하는 서비스 모델이다.

[유한 상태 머신, FSM]

전이를 일으키는 이벤트는 변화를 표기하는 가로선 위에 나타낸다.

그리고 이벤트가 발생했을 때 취해지는 액션은 가로선 아래에 나타낸다.

데이터 전송 프로토콜의 송신 측은 `rdt_send()` 호출로 위쪽으로부터 호출될 것이다.

수신 측에서는 상위 계층으로 전달된 데이터를 넘길 것이다.

수신측에서 `rdt_rcv()`는 패킷이 채널의 수신 측으로부터 도착했을 때 호출된다.

rdt 프로토콜이 상위 계층에 데이터를 전달하려고 할 때 `delever_data()`를 호출한다.

<br />

### rdt1.0

- reliable transfer over a reliable channel

![image](https://github.com/user-attachments/assets/5437c7d3-8365-496c-87a4-ad1997901f81)

통신 채널이 신뢰할 수 있기 때문에 sender는 계속 패킷을 만들어 보내주기만 하면 된다.

receiver는 패킷을 계속 받기만 하면 된다.

<br />

### rdt2.0

- channel with bit errors

![image](https://github.com/user-attachments/assets/843b9030-b255-4831-9fa9-0b78c6289f8b)

underlying network가 LOSS는 안나지만 ERROR가 발생하는 상황

receiver 측에서 `checksum`을 확인하여 ACKs(1), NAKs(0) 응답을 보낸다.

- ACKs: 정상적으로 잘 받았다.
- NAKs: 전달받은 데이터에서 에러가 있다.

<br />

sender는 응답을 기다렸다 ACKs면 다음 메시지를 보내고, NAKs면 재전송한다.

ACKs, NAKs 메시지를 sender에게 보내는 도중에 에러가 발생할 경우 sender가 무조건 재전송을 할 경우

sender와 receiver의 싱크가 안맞을 수 있다.

→ `sequence number`가 필요하다.

<br />

receiver는 sender가 보낸 메시지와 sequence number를 비교하여 처리한다.

중복된 메시지이든 아니든 receiver는 응답 메시지를 보낸다.

중복된 메시지의 경우 응답 메시지를 보내고, 해당 메시지는 처리하지 않고 버린다.

<br />

```
💡 컴퓨터 네트워크 설정에서 이러한 재전송을 기반으로 하는 신뢰적인 데이터 전송 프로토콜은 `자동 재전송 요구, ARQ 프로토콜` 로 알려져 있다.

또한, rdt2.0은 송신자는 수신자가 현재의 패킷을 정확하게 수신했음을 확신하기 전까지 새로운 데이터를 전달하지 않을 것이며, 이를 `전송 후 대기 프로토콜`이라고 한다.
```

<br />

### rdt2.1

수신자로부터 송신자까지의 긍정확인응답과 부정 확인응답을 모두 포함한다.

순서가 바뀐 패킷이 수신되면, 수신자는 이미 전에 수신한 패킷에 대한 긍정 확인응답을 전송한다.

손상된 패킷이 수신되면, 수신자는 부정 확인응답을 전송한다.

NAK을 전송하는 것 대신, 가장 최근에 정확하게 수신된 패킷에 대해 ACK를 송신함으로써 NAK을 송신하는 것과 같은 효과를 얻을 수 있다. 같은 패킷에 대해 2개의 ACK를 수신한 송신자는 수신자가 두 번 ACK한 패킷의 다음 패킷을 정확하게 수신하지 못했다는 것을 안다.

<br />

### rdt2.2

- a NAK-free protocol

receiver는 sender가 보낸 메시지가 비정상적인 경우 마지막으로 받은 정상 메시지의 sequence number를 ACK 응답과 함께 보낸다.

sender는 자신이 보낸 sequence number와 전달받은 sequence number를 비교하여 문제가 발생했는지 판단한다.

**checksum, sequence number가 헤더 필드에 포함된다.**

<br />

### rdt3.0

- channels with errors and loss

underlying network에서 LOSS가 날 수도, ERROR가 날 수도 있는 상황

sender는 메시지를 보낸 후 일정 시간(timer)동안 응답 메시지가 안오면, LOSS로 판단하여 재전송한다.

timer 값이 짧거나 길면 그에 따른 장단점이 있기 때문에, 적절한 timer 값을 설정해야 한다.
