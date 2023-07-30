---
title: 트랜스포트 계층(Transport Layer)(2)
date: 2023-07-18 00:00:00 +09:00
categories: [ Computer Science, Network ]
tags: [ Transport, UDP ]
math: true
---


# 연결지향형 트랜스포트: TCP

TCP가 신뢰적인 데이터 전송을 제공하기 위해 오류 검출, 재전송 누적 확인응답, 타이머, 순서 번호와 확인응답 번호를 위한 헤더 필드를 포함한 방법을 설명하겠습니다.

## TCP 연결

TCP는 애플리케이션 프로세스가 데이터를 다른 프로세스에게 보내기 전에 두 프로세스가 서로 '핸드셰이크'를 먼저 해야하므로 연결지향형(connection-oriented)입니다.
즉 데이터 전송을 보장하는 파라미터들을 각자 설정하기 위한 어떤 사전 세그먼트들을 보내야합니다.
TCP 포로토콜은 오직 종단 시스템에서만 동작하고 중간의 네트워크 요소(라우터와 링크 계층 스위치)에서는 동작하지 않으므로 중간의 네트워크 요소들은 TCP 연결 상태를 유지하지 않습니다.
(중간 라우터들은 이들의 연결은 보지 못하고 데이터그램만 봅니다.)

TCP 연결은 전이중 서비스(full-duplex service)를 제공합니다.
만약 A의 프로세스와 호스트 B의 프로세스 사이에 TCP 연결이 있다면 애플리케이션 계층 데이터는 B에서 A로 흐르는 동시에 A에서 B로 흐를 수 있습니다.
또한 TCP 연결은 단일 송신 동작으로 한 송신자가 여러 수신자에게 데이터를 전송하는 멀티캐스팅(multicasting)은 불가능합니다. 

한 호스트에서 동작하는 프로세스가 다른 호스트의 프로세스와 연결을 초기화하기를 원한다고 가정합니다. 
연결을 초기화하는 프로세스를 클라이언트 프로세스(client process), 다른 프로세스를 서버 프로세스(server process)라고 가정했을 때 클라이언트 애플리케이션 프로세스는 서버의 프로세스와 연결을 설정하기를 원한다고 TCP 클라이언트에게 먼저 알립니다. 
클라이언트가 먼저 특별한 TCP 세그먼트를 보냅니다. 
서버는 두 번째 특별한 TCP 세그먼트로 응답합니다. 
마지막으로 클라이언트가 세 번째 특별한 세그먼트로 다시 응답을 합니다. 
처음 2개의 세그먼트에는 페이로드 즉 애플리케이션 계층 데이터가 없지만 세 번째 세그먼트는 페이로드를 포함 할 수 있습니다. 
두 호스트 사이에 3개의 세그먼트가 보내지므로 이 연결 설정 절차는 흔히 세 방향 핸드셰이크(three-way handshake)라 부릅니다. 

일단 TCP 연결이 설정되면 두 애플리케이션 프로세스는 서로 데이터를 보낼 수 있게 됩니다. 
클라이언트 프로세스는 소켓(프로세스의 관문)을 통해 데이터의 스트림을 전달합니다. 
데이터가 관문을 통해 전달되면 이제 데이터는 클라이언트에서 동작하고 있는 TCP에 맡겨집니다. 
TCP는 초기 세 방향 핸드셰이크 동안 준비된 버퍼 중 하나인 연결의 송신 버퍼(send buffer)로 데이터를 보냅니다. 
세그먼트로 모아 담을 수 있는 최대 데이터 양은 최대 세그먼트 크기(MSS: maximum segment size)로 제한됩니다. 
MSS는 일반적으로 로컬 송신 호스트에 의해 전송될 수 있는 가장 큰 링크 계층 프레임의 길이(최대 전송 단위(MTU: maximum transmission unit))에 의해 결정되고 그 후에 TCP 세그먼트와 TCP/IP 헤더 길이(통상 40바이트)가 단일 링크 계층 프레임에 딱 맞도록 하여 정해집니다. 
이더넷과 PPP 링크 계층 프로토콜은 모두 1,500 바이트의 MTU를 갖기 때문에 MSS의 일반적인 값은 1460 바이트입니다. 
MSS가 헤더를 포함하는 TCP 세그먼트의 최대 크기가 아니라 세그먼트에 있는 애플리케이션 계층 데이터에 대한 최대 크기라는 점을 주의해야합니다. 

TCP는 TCP 헤더와 클라이언트 데이터를 하나로 짝지어 TCP 세그먼트(TCP segment)를 구성한 후 네트워크 계층 IP 데이터그램 안에 각각 캡슐화됩니다. 
이 세그먼트들이 네트워크로 송신 후 수신했을 때 세그먼트의 데이터는 TCP 연결의 수신 버퍼에 위치합니다. 
애플리케이션은 이 버포로부터 데이터의 스트림을 읽습니다. 

![tcp-flow](/assets/img/computer-science/network/transport-layer/tcp-flow.png)  

## TCP 세그먼트 구조

![tcp-segment](/assets/img/computer-science/network/transport-layer/tcp-segment.png)  

- 32 비트 순서 번호 필드(sequence number field)와 32 비트 확인응답 번호 필드(acknowledgement number field): 신뢰적인 데이터 전송 서비스 구현에서 TCP 송신자와
  수신자에 의해 사용됩니다.
- 16 비트 수신 윈도(receive window): 흐름 제어에 사용되는 필드입니다. 수신자가 받아들이려는 바이트의 크기를 나타내는 데 사용됩니다.
- 4 비트 헤더 길이 필드(header length field): 32 비트 워드 단위로 TCP 헤더의 길이를 나타냅니다. TCP 옵션 필드에 의해 가변적인 길이가 될 수 있습니다.(옵션 필드는 일반적인 TCP의
  길이가 20 바이트가 되도록 비어 있습니다.)
- 옵션 필드(option filed): 송신자와 수신자가 최대 세그먼트 크기(MMS)를 협상하거나 고속 네트워크에서 사용하기 위한 윈도 확장 요소로 이용됩니다. 타임스탬프 옵션 또한 정의됩니다.
- 6 비트 플래그 필드(flag field): ACK 비트는 확인응답 필드에 있는 값이 유용함을 가리키는데 사용됩니다. 즉 이 세그먼트는 성공적으로 수신된 세그먼트에 대한 확인응답을 포함합니다. RST, SYN,
  FIN 비트는 연결 설정과 해제에 사용됩니다. PSH 비트가 설정될 때 이것은 수신자가 데이터를 상위 계층에 즉시 전달해야한다는 것을 가리킵니다. 마지막으로 URG 비트는 이 세그먼트에서 송신 측 상위 계층
  개체가 '긴급'으로 표시하는 데이터임을 가리킵니다.

## 순서 번호와 확인응답 번호

TCP 세그먼트 헤더에서 가장 중요한 필드 두 가지는 순서 번호 필드와 확인응답 번호 필드입니다. 
이러한 필드들은 TCP의 신뢰적인 데이터 전송 서비스의 중대한 부분입니다. 
TCP는 데이터를 구조화되어 있지 않고 단지 순서대로 정렬되어 있는 바이트 스트림으로 봅니다. 
세그먼트에 대한 순서 번호는 세그먼트에 있는 첫 번째 바이트의 바이트 스트림 번호입니다. 
예를 들어 호스트 A에서 호스트 B로 데이터 스트림 전송을 원한다고 가정하겠습니다. 
호스트 A의 TCP는 데이터 스트림의 각 바이트에 암시적으로 순서 번호를 지정합니다. 
또 데이터 스트림은 500,000 바이트로 구성된 파일이라고 가정 하고 MSS는 1,000 바이트이고 데이터 스트림의 첫 번째 바이트는 0으로 설정했습니다. 
TCP는 데이터 스트림으로부터 500개의 세그먼트들을 구성합니다. 
첫번째 세그먼트는 순서 번호 0, 두 번째 세그먼트는 순서 번호 1,000, 세 번째 세그먼트는 순서 번호 2,000과 같은 식으로 할당됩니다.
각각의 순서 번호는 적절한 TCP 세그먼트의 헤더 내부의 순서 번호 필드에 삽입됩니다. 

확인 응답 번호는 TCP가 호스트 A가 호스트 B로 데이터를 송신하는 동안에 호스트 B로부터 데이터를 수신하게 해주는 전이중 방식임을 고려했을 때 호스트 B로부터 도착한 각 세그먼트는 B로부터 A로 들어온 데이터에 대한 순서 번호를 갖습니다. 
호스트 A가 자신의 세그먼트에 삽입하는 확인응답 번호는 호스트 A가 호스트 B로부터 기대하는 다음 바이트의 순서 번호입니다. 

### 정상 통신

호스트 A가 B로부터 0에서 535까지 번호가 붙은 모든 바이트를 수신했다고 예를 들겠습니다. 
그리고 호스트 B로 세그먼트를 송신하려고 합니다. 
호스트 A는 호스트 B의 데이터 스트림에서 536번째 바이트와 그다음에 오는 모든 바이트를 기다리고 있습니다. 
그래서 호스트 A는 세그먼트의 확인응답 번호 필드에 536을 삽입하고 그것을 B에 송신합니다. 

### 잃어버린 바이트

호스트 A가 호스트 B로부터 0~535의 바이트를 포함하는 어떤 세그먼트와 900~1,000의 바이트를 포함하는 또 다른 세그먼트를 수신했다고 예를 들겠습니다. 
어떤 이유 때문인지 호스트 A는 536~899의 바이트를 아직 수신하지 않았습니다. 
호스트 A는 B의 데이터 스트림을 재생성하기 위해 536번째(와 그다음의) 바이트를 아직 기다리고 있습니다. 
그러므로 B에 대한 A의 다음 세그먼트는 확인응답 번호 필드에 536을 가질 것입니다. 
TCP는 스트림에서 첫 번째 잃어버린 바이트까지의 바이트들까지만 확인응답하기 때문에 TCP는 누적 확인응답(cummulative acknowledgment)을 제공합니다.  

### 순서가 바뀐 통신 

호스트 A는 세 번째 세그먼트(900~1,000 값의 바이트)를 두 번째 세그먼트(536~899 값의 바이트)가 수신되기 전에 수신했습니다. 
즉 세 번째 세그먼트는 순서가 틀리게 도착했습니다. 
TCP 연결에서 순서가 바뀐 세그먼트를 수신할 때 호스트는 어떤 행동을 하는가에 대한 것입니다. 
흥미롭게도 TCP RFC는 여기에 어떤 규칙도 부여하지 않았고 TCP 구현 개발자에게 맡기고 있습니다. 
기본적으로 다음 두 가지 선택이 있습니다. 

1. 수신자가 순서가 바뀐 세그먼트를 즉시 버립니다. 
2. 수신자는 순서가 바뀐 데이터를 보유하고 빈 공간에 잃어버린 데이터를 채우기 위해 기다립니다. 

확실히 후자가 네트워크 대역폭 관점에서는 효율적이며 실제에서도 많이 취하는 방법입니다. 
실제로는 TCP 연결의 양쪽 모두 시작 순서 번호를 임의로 선택합니다.
이것은 두 호스트 사이에 이미 종료된 연결로부터 아직 네트워크에 남아 있던 세그먼트가 같은 두 호스트 간의 나중 연결(또한 이전의 연결과 같은 포트 번호를 사용해서 발생)에서 유요한 세그먼트로 오인될 확률을
최소화합니다.

## 왕복 시간(RTT) 예측과 타임아웃

TCP는 손실 세그먼트를 발견하기 위해 타임아웃/재전송 매커니즘을 사용합니다. 
분명 타임아웃은 세그먼트가 전송된 시간부터 긍정 확인응답될 때까지의 시간인 연결의 왕복 시간(RTT: round-trip time)보다 좀 커야합니다. 
만약 그렇지 않다면 불필요한 재전송이 발생하기 때문입니다. 

### 왕복 시간 예측

세그먼트에 대한 RTT 샘플은 세그먼트가 송신된 시간(즉, IP에서 넘겨진 시간)으로부터 그 세그먼트에 대한 긍정 응답이 도착한 시간까지의 시간 길입니다. 
모든 전송된 세그먼트에 대해 RTT 샘플을 측정하는 대신 대부분의 TCP는 한번에 하나의 RTT 샘플 측정만을 시행합니다. 
즉 어떤 시점에서 RTT 샘플은 전송되었지만 현재까지 확인응답이 없는 세그먼트 중 하나에 대해서만 측정되며 이는 대략 왕복 시간마다 RTT 샘플의 새로운 값을 얻게합니다. 
또한 TCP는 재전송한 세그먼트에 대핸 RTT 샘플은 계산하지 않으며 한 번 전송된 세그먼트에 대해서만 측정합니다. 

RTT 샘플 값은 라우터에서의 혼잡과 종단 시스템에서의 부하 변화와 같은 변동성 때문에 주어진 RTT 샘플 값은 불규칙적입니다. 
그렇기 때문에 RTT를 추정하기 위해서는 RTT 샘플 값의 평균값을 채택합니다. 
긍정 확인응답을 수신하고 새로운 RTT 샘플 값을 획득하자마자 TCP는 다음 공식에 따라 평균 RTT 샘플 값을 갱신합니다. 

$$ EstimatedRTT = (1 - \alpha) \times EstimatedRTT + \alpha \times SampleRTT $$  

* 권장되는 $$ \alpha $$의 값은 0.125(1/8)[RFC 6928]입니다.  

RTT의 예측 외에 RTT의 변화율을 측정하는 것도 매우 유용합니다. 
RTT 변화율이 의미하는 것은 RTT 샘플 값이 평균 RTT 샘플 값으로부터 얼마나 많이 벗어나는지에 대한 예측으로 정의합니다.

$$DevRtt = (1 - \beta) \times DevRTT + \beta \times \vert SampleRTT - EstimatedRTT \vert$$  

만일 RTT 샘플  값이 어떠한 변화도 없다면 DevDTT는 작을 것이며 그렇지 않다면 DevRTT는 클 것입니다. 
$$ \beta $$ 권장값은 0.25입니다. 

### 재전송 타임아웃 주기의 설정과 관리

분명히 타임아웃 주기는 평균 RTT 샘플 값보다 크거나 같아야합니다. 
그렇지 않다면 불필요한 재전송이 보내질 것입니다. 
그러나 타임아웃 주기는 평균 RTT 샘플보다 너무 크면 안됩니다. 
너무 크면 세그먼트를 잃었을 때 TCP는 세그먼트의 즉각적인 재전송을 하지 않게 됩니다. 
그러므로 타임아웃 값은 약간의 여윳값을 더한 값으로 설정하는 것이 바람직합니다.
평균 RTT 샘플 값에 많은 변동이 있을 때는 여웃값이 커야 하며 변동이 작을 때는 작야합니다. 
따라서 RTT 변화율 값이 역활을 하게 됩니다. 
이러한 모든 고려사항은 재전송 타임아웃 주기를 결정하는 TCP의 방식에서 사용됩니다. 

$$ TimeoutInterval = EstimatedRTT + 4 \times DevRTT $$  

초기 타임아웃 주기의 값으로 1초를 권고합니다[RFC 6298]. 
또한 타임아웃이 발생할 때 타임아웃 주기 값은 두 배로 하여 조만간 확인응답할 후속 세그먼트에게 발생할 수 있는 조기 타임아웃을 피하도록 합니다. 
그러나 세그먼트가 수신되고 평균 RTT 샘플 값이 수정되면 타임아웃 주기 값은 다시 위의 공식에 따라 계산됩니다.

해당 글은 [컴퓨터 네트워킹 하향식 접근(James F. Kurose, Keith W. Ross)](https://www.yes24.com/Product/Goods/45543957)을 읽고 공부한 내용을 정리한 글입니다.  

오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다.
 