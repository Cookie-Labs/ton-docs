# 오버레이 서브네트워크

구현:

- https://github.com/ton-blockchain/ton/tree/master/overlay

## 개요

TON과 같은 다중 블록체인 시스템에서 심지어 전체 노드들도 보통 몇 개의 샤드체인 주변에서 업데이트(예: 새로운 블록)를 얻기를 원할 것입니다. 이를 위해 TON 네트워크 내부에는 각 샤드체인을 위한 특수한 오버레이 서브네트워크가 구축되었습니다. 이 서브네트워크는 ADNL 프로토콜 위에 구축되었습니다.

또한 오버레이 서브네트워크는 TON 스토리지, TON 프록시 등의 운영에도 사용됩니다.

## ADNL vs 오버레이 네트워크

ADNL과 대조적으로, TON 오버레이 네트워크는 일반적으로 다른 임의의 노드로 데이터그램을 보내는 것을 지원하지 않습니다. 대신, 특정 노드 간에 "반 영구적 링크" 또는 "반영구적 연결"이 설정되며 (해당 오버레이 네트워크를 고려할 때 "이웃"이라고 함), 메시지는 일반적으로 이러한 링크를 따라 전달됩니다 (즉, 노드에서 그 중 하나의 이웃 노드로).

각 오버레이 서브네트워크는 보통 오버레이 네트워크에 대한 설명의 SHA256과 같은 256비트 네트워크 식별자를 가지고 있습니다. 이 식별자는 일반적으로 TL-직렬화된 객체입니다.

오버레이 서브네트워크는 공개 또는 비공개일 수 있습니다.

오버레이 서브네트워크는 특수한 [gossip](https://en.wikipedia.org/wiki/Gossip_protocol) 프로토콜에 따라 작동합니다.

:::info
더 자세한 내용은 [Overlay subnetworks](/develop/network/overlay) 문서에서 더 읽어보거나 [TON 화이트페이퍼](https://ton.org/docs/ton.pdf)의 3.3 장에서 확인할 수 있습니다.
:::
