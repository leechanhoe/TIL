![](https://velog.velcdn.com/images/dodo4723/post/c97f1fb5-f5bb-4d2e-b58d-0a28b0cd747f/image.png)

<br>
<br>
<br>
<br>

## 네트워크 프로토콜?

**통신하기 위한 절차를 규정한 것이자 컴퓨터끼리 대화하기 위해 필요한 공통 언어.** 방법, 절차, 언어라는 역할마다 계층 구조로 구분되어 있어 사용하는 네트워크 서비스에 따라 각각 최적의 프토로콜을 조합하여 사용할 수 있다.

<br>
<br>
<br>

**`DNS`** : TCP/IP 네트워크에서 호스트의 도메인 이름(컴퓨터명)으로 검색하여 해당 IP주소를 취득하는 서비스. DNS 서버는 클라이언트의 도메인 이름 질의를 받아 데이터베이스에서 알맞은 IP주소를 반환.

**`DHCP`** : **D**ynamic **H**ost **C**onfiguration **P**rotocol. 네트워크 내의 컴퓨터에 IP 주소나 서브넷 마스크같은 **네트워크 정보를 자동으로 설정**하기 위한 프로토콜. ISP로 인터넷에 접속할 경우, DHCP를 통해 네트워크 설정을 취득함.

![](https://velog.velcdn.com/images/dodo4723/post/b7ba6419-a04e-4fe3-bba3-b15742137c6e/image.jpg)


**`NetBIOS`** : 네트워크 서비스를 이용하기 위한 기본적인 입출력을 정의한 인터페이스. 처음에는 NetBEUI라는 전송 계층의 인터페이스였지만, 현재는 TCP/IP가 보급되며 다른 프로토콜에서도 이 인터페이스가 제공되고 있음.

**`PPP`** : **P**oint에서 **P**oint 사이를 연결하는 회선을 확정하여 네트워크 회선으로 이용 가능하게 하는 **P**rotocol. 전화 회선을 사용할때 사용. 접속할 때 최초로 사용자 인증을 실시하고, 문제가 없으면 프로토콜이나 에러 정정의 방법등을 주고받아 사양을 결정함.

![](https://velog.velcdn.com/images/dodo4723/post/52c0acfb-0cbd-485a-9035-89734fe704aa/image.jpg)


**`PPPoE`** : **PPP** **o**ver **E**thernet. PPP를 이더넷에서 구현하기 위한 프로토콜. 전화 회선을 이용하는 다이얼 업 접속에서 ASDL 같은 상시 접속 환경으로 많이 바뀌면서 둘 다 사용하기 위해 고안됨.

**`PPTP`** : **P**oint to **P**oint **T**unneling **P**rotocol. 가상적인 다이얼업 접속을 통해 암호화 통신을 수행하여 전용선 접속과 같이 이용하기 위한 프로토콜. VPN 구축에 사용되는 암호화 기술로, 패킷을 암호화하여 안전한 전용 터널로 터널링 처리를 한다.

**`방화벽`** : 외부/내부 네트워크 사이의 경계에 구축하여 통신을 차단하고 필요한 서비스에 대해서만 통과하도록 설정함. 소프트웨어나 라우터등 형태는 다양함.

**`프록시 서버`** : 내부 네트워크의 컴퓨터를 대행하여 외부 네트워크에 엑세스하는 서버. 방화벽에서 실행됨. 내부에서 외부로의 엑세스를 집중적으로 관리할 수 있고, 제한하고 싶은 요청을 불가로 설정하고, 취득한 데이터를 캐시로 활용하는 등 응용 범위가 다양하다.

![](https://velog.velcdn.com/images/dodo4723/post/ce96453c-adaa-49f0-83d9-2fc0afe5e8bd/image.jpg)


**`패킷 필터링`** : 방화벽 구현 방식의 가장 기본적인 기능. 허용되는 패킷 규칙은 송신처나 수신처의 IP주소, TCP/UDP 같은 프로토콜 종류, 포트번호등을 지정함으로써 결정됨.

**`NAT`** : 사설 IP 주소와 공인 IP주소를 상호 변환하는 기술. 패킷이 인터넷으로 나갈 때 송신처를 공인 IP주소로 재작성해 송신하고, 테이블에 기억해 두었다가 인터넷에서 받은 패킷의 수신처 주소를 사설IP 주소로 재작성해 LAN에 보냄.

**`IP 마스커레이드(NAPT)`** : NAT와 달리, TCP나 UDP의 포트 번호까지 포함하여 변환함. 일 대 다수의 변환이 가능하다. 포트 번호를 고정해야하는 경우나, 동일 IP접속은 하나로 한정하는 서비스에선 제약이 있다.

![](https://velog.velcdn.com/images/dodo4723/post/39d64ad7-3424-49d4-8ae0-8cbf750e0aa2/image.jpg)