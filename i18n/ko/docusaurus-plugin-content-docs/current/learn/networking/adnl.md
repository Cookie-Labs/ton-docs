# ADNL 프로토콜

구현:

- [ADNL GitHub 리포지토리](https://github.com/ton-blockchain/ton/tree/master/adnl)

## 개요

TON의 기반은 Abstract Datagram Network Layer (ADNL)입니다.

이는 **UDP** 상에서 작동하며, **IPv4** (미래에는 IPv6)에서 작동하는 오버레이형 P2P(peer-to-peer) 비신뢰(소형 크기) 데이터그램 프로토콜로, UDP를 사용할 수 없는 경우 옵션으로 **TCP fallback**을 사용합니다.

## ADNL 주소

각 참가자는 256비트 ADNL 주소를 가지고 있습니다.

ADNL 프로토콜을 사용하면 ADNL 주소만 사용하여 데이터그램을 보내고 받을 수 있습니다. IP 주소와 포트는 ADNL 프로토콜에 의해 숨겨집니다.

ADNL 주소는 사실상 256비트 ECC 공개 키와 동등합니다. 이 공개 키는 임의로 생성할 수 있으므로 노드가 필요한만큼 다양한 네트워크 신원을 생성할 수 있습니다.
그러나 수신자 주소로 지정된 메시지를 수신(및 해독)하려면 해당 개인 키를 알고 있어야 합니다.

실제로 ADNL 주소는 공개 키 자체가 아니라 직렬화된 TL-객체의 256비트 SHA256 해시이며, 이는 생성자에 따라 여러 유형의 공개 키와 주소를 설명할 수 있습니다.

## 암호화 및 보안

보통 보낸 사람에 의해 보낸 데이터그램은 서명되어 수신자만 메시지를 해독하고 서명을 통해 무결성을 확인할 수 있도록 암호화됩니다.

## 이웃 테이블

보통 TON ADNL 노드는 "이웃 테이블"이라는 몇 가지 다른 알려진 노드에 대한 정보를 포함하는 테이블을 가지고 있으며, 이 정보에는 추상 주소, 공개 키, IP 주소 및 UDP 포트가 포함됩니다. 시간이 지남에 따라 이러한 알려진 노드에서 수집한 정보를 사용하여이 테이블을 점진적으로 확장할 것입니다. 새로운 정보는 특수 쿼리에 대한 답변 또는 때로는 오래된 레코드를 제거함으로써 제공될 수 있습니다.

ADNL을 사용하면 point-to-point 채널과 터널(프록시 체인)을 설정할 수 있습니다.

ADNL 상에서 TCP와 유사한 스트림 프로토콜을 구축할 수 있습니다.

## 다음 단계

- [Low-Level ADNL 기사](/learn/networking/low-level-adnl)에서 ADNL에 대해 더 읽어보세요.
- [TON Whitepaper](https://docs.ton.org/ton.pdf)의 3.1장을 참조하세요.