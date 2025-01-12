# 저수준 ADNL

Abstract Datagram Network Layer (ADNL)은 TON의 핵심 프로토콜로, 네트워크 피어 간의 통신을 돕는 역할을 합니다.

## 피어 식별

각 피어는 하나 이상의 식별 정보를 가지고 있어야 하며, 여러 식별 정보를 사용할 수는 있지만 필수는 아닙니다. 각 식별 정보는 피어 간의 Diffie-Hellman 연산을 수행하는 데 사용되며, 공개 키로부터 추상적인 네트워크 주소가 다음과 같이 파생됩니다: `address = SHA-256(type_id || public_key)`입니다. 여기서 `type_id`는 little-endian uint32로 직렬화되어야 합니다.

## 공개 키 암호화 시스템 목록

| type_id    | 암호화 시스템       |
| ---------- | ------------------- |
| 0x4813b4c6 | ed25519<sup>1</sup> |

_1. x25519을 수행하려면 키 쌍을 x25519 형식으로 생성해야 합니다. 그러나 공개 키는 네트워크를 통해 ed25519 형식으로 전송되므로 x25519에서 ed25519으로 공개 키를 변환해야 합니다. 이러한 변환 예제는 [Rust](https://github.com/tonstack/adnl-rs/blob/master/src/integrations/dalek.rs#L10)와 [Kotlin](https://github.com/andreypfau/curve25519-kotlin/blob/f008dbc2c0ebc3ed6ca5d3251ffb7cf48edc91e2/src/commonMain/kotlin/curve25519/MontgomeryPoint.kt#L39)에서 찾을 수 있습니다._

## 클라이언트-서버 프로토콜 (ADNL over TCP)

클라이언트는 TCP를 사용하여 서버에 연결하고 ADNL 핸드셰이크 패킷을 보냅니다. 이 패킷에는 서버의 추상적인 주소, 클라이언트의 공개 키 및 클라이언트가 결정한 암호화된 AES-CTR 세션 매개변수가 포함되어 있습니다.

### 핸드셰이크

먼저 클라이언트는 자신의 개인 키와 서버의 공개 키를 사용하여 키 합의 프로토콜 (예: x25519)을 수행해야 합니다. 결과적으로 클라이언트는 `secret`을 얻게 되며, 이는 향후 단계에서 세션 키를 암호화하는 데 사용됩니다.

그런 다음 클라이언트는 AES-CTR 세션 매개변수를 생성해야 합니다. 이는 TX (클라이언트에서 서버로) 및 RX (서버에서 클라이언트로) 방향을 위한 16바이트의 무작위 nonce와 32바이트의 키로 구성되며, 다음과 같이 160바이트 버퍼로 직렬화됩니다:

| 매개변수 | 크기      |
| -------- | --------- |
| rx_key   | 32 바이트 |
| tx_key   | 32 바이트 |
| rx_nonce | 16 바이트 |
| tx_nonce | 16 바이트 |
| padding  | 64 바이트 |

패딩의 목적은 알려지지 않았으며, 서버 구현에서는 사용되지 않습니다. 이 160바이트 버퍼를 무작위 바이트로 채우는 것이 권장되며, 그렇지 않으면 공격자가 손상된 AES-CTR 세션 매개변수를 사용하여 적극적인 MitM(중간자 공격) 공격을 수행할 수 있습니다.

다음 단계는 `secret`를 사용하여 세션 매개변수를 키 합의 프로토콜 위에서 암호화하는 것입니다. 이를 위해 AES-256을 big-endian 카운터를 사용하여 CTR 모드로 초기화해야 하며, (키, nonce) 쌍을 사용하여 다음과 같이 계산해야 합니다 (`aes_params`는 위에서 생성된 160바이트 버퍼입니다):

```cpp
hash = SHA-256(aes_params)
key = secret[0..16] || hash[16..32]
nonce = hash[0..4] || secret[20..32]
```

`aes_params`의 암호화인 `E(aes_params)` 후에는 더 이상 AES가 필요하지 않으므로 제거해야 합니다.

이제 우리는 이 모든 정보를 256바이트 핸드셰이크 패킷으로 직렬화하고 서버에게 보내야 합니다:

| 매개변수            | 크기       | 설명                                       |
| ------------------- | ---------- | ------------------------------------------ |
| receiver_address    | 32 바이트  | 해당 섹션에서 설명한 대로 서버 피어 식별자 |
| sender_public       | 32 바이트  | 클라이언트 공개 키                         |
| SHA-256(aes_params) | 32 바이트  | 세션 파라미터의 무결성 증명                |
| E(aes_params)       | 160 바이트 | 암호화된 세션 파라미터                     |

서버는 세션 파라미터를 복호화해야 합니다. 이때, 서버도 클라이언트와 동일한 키 합의 프로토콜을 사용하여 비밀 정보를 추출해야 합니다. 그런 다음, 서버는 다음과 같은 보안 속성을 확인하기 위한 다음 점검을 수행해야 합니다:

1. 서버는 `receiver_address`에 해당하는 개인 키를 가져야 합니다. 그렇지 않으면 키 합의 프로토콜을 수행할 방법이 없습니다.
2. `SHA-256(aes_params) == SHA-256(D(E(aes_params)))` 여부를 확인해야 합니다. 그렇지 않으면 키 합의 프로토콜이 실패한 것이며, `secret`가 양쪽 모두 동일하지 않습니다.

이러한 점검 중 하나라도 실패하면 서버는 즉시 클라이언트에게 응답 없이 연결을 종료해야 합니다. 모든 점검이 통과하면 서버는 클라이언트에게 빈 데이터그램(데이터그램 섹션 참조)을 전송하여 지정된 `receiver_address`의 개인 키를 소유함을 증명해야 합니다.

### 데이터그램

클라이언트와 서버 모두 TX 및 RX 방향을 위해 각각 두 개의 AES-CTR 인스턴스를 초기화해야 합니다. AES-256은 빅엔디언 128비트 카운터를 사용하는 CTR 모드로 사용되어야 합니다. 각 AES 인스턴스는 그에 속하는 (키, 논스) 쌍을 사용하여 초기화됩니다. 이 정보는 핸드셰이크에서 `aes_params`에서 가져올 수 있습니다.

데이터그램을 보내려면 한 피어(클라이언트 또는 서버)는 다음 구조를 작성하고 암호화한 다음 다른 피어로 전송해야 합니다:

| 매개변수 | 크기               | 설명                                                    |
| -------- | ------------------ | ------------------------------------------------------- |
| 길이     | 4바이트 (LE)       | `길이` 필드를 제외한 데이터그램 전체의 길이             |
| 논스     | 32바이트           | 임의의 값                                               |
| 버퍼     | `길이 - 64` 바이트 | 다른 피어로 보낼 실제 데이터                            |
| 해시     | 32바이트           | 무결성을 보장하기 위해 `SHA-256(none \|\| buffer)` 사용 |

전체 구조는 해당 AES 인스턴스(TX는 클라이언트 -> 서버, RX는 서버 -> 클라이언트)를 사용하여 암호화되어야 합니다.

수신 피어는 먼저 처음 4바이트를 가져와 `길이` 필드로 복호화한 다음 정확히 `길이` 바이트를 읽어 전체 데이터그램을 가져와야 합니다. 수신 피어는 `버퍼`를 먼저 복호화하고 처리를 시작할 수 있지만, 데이터가 훼손될 수 있으므로 유의해야 합니다. 데이터그램 `해시`는 `버퍼`의 무결성을 보장하기 위해 확인해야 합니다. 실패한 경우 새로운 데이터그램을 발행하지 못하고 연결을 종료해야 합니다.

세션에서 첫 번째 데이터그램은 핸드셰이크 패킷이 서버에서 성공적으로 수락된 후 서버에서 클라이언트로 이동합니다. 클라이언트는 이를 복호화하고 실패한 경우 서버가 프로토콜을 올바르게 따르지 않았으며 실제 세션 키가 서버와 클라이언트 측에서 다르다는 것을 의미하므로 연결을 끊어야 합니다.

### 통신 세부 정보

통신 세부 정보에 대해 자세히 알아보려면 [ADNL TCP - Liteserver](/develop/network/adnl-tcp) 문서를 확인하십시오.

### 보안 고려 사항

#### 핸드셰이크 패딩

왜 초기 TON 팀이 핸드셰이크에 이 필드를 포함하려고 결정했는지 알 수 없습니다. `aes_params`의 무결성은 SHA-256 해시에 의해 보호되며 기밀성은 `secret` 매개변수에서 파생된 키에 의해 보호됩니다. 아마도 언젠가 AES-CTR에서 마이그레이션하기 위해 이러한 확장 사항을 포함하려고 했을 것입니다. 이를 위해 사양은 `aes_params`에 특수한 매직 값이 포함되도록 확장될 수 있으며, 이 값은 피어가 업데이트된 원시를 사용할 준비가 되었음을 나타냅니다. 이러한 핸드셰이크에 대한 응답은 새로운 및 이전 스키마로 두 번 복호화되어 다른 피어가 실제로 어떤 스키마를 사용하는지 명확히하는 데 사용될 수 있습니다.

#### 세션 파라미터 암호화 키 파생 프로세스

암호화 키가 `secret` 매개변수만으로 파생되면 정적인 키가 됩니다. 개발자는 세션마다 새로운 암호화 키를 파생하기 위해 `SHA-256(aes_params)`도 사용합니다. 이 값은 `aes_params`가 무작위일 경우 무작위이지만, 다른 하위 배열을 연결하는 실제 키 파생 알고리즘은 유해하다고 여겨집니다.

#### 데이터그램 논스

데이터그램의 `논스` 필드가 있는 이유는 명확하지 않습니다. 실제로, 없어도 두 개의 암호문이 세션에 제한된 키로 인해 다릅니다. 그러나 `논스`가 없거나 예측 가능한 경우 다음과 같은 공격이 수행될 수 있습니다. CTR 암호화 모드는 AES와 같은 블록 암호를 비트 뒤집기 공격을 수행할 수 있게 만들기 위해 스트림 암호로 바꿉니다. 공격자가 암호문에 속하는 평문을 알고 있다면 순수한 키 스트림을 얻을 수 있으며, 이를 자신의 평문과 XOR하여 다른 피어가 보낸 메시지를 효율적으로 대체할 수 있습니다. 버퍼 무결성은 SHA-256 해시에 의해 보호되지만, 공격자는 무결성의 해시를 알고 있으므로 이를 대체할 수 있습니다. `논스` 필드는 이러한 공격을 방지하기 위해 존재하며, 공격자가 논스를 알지 못하면 SHA-256을 대체할 수 없습니다.

## P2P 프로토콜 (ADNL over UDP)

자세한 설명은 [ADNL UDP - Internode](/develop/network/adnl-udp) 문서에서 확인할 수 있습니다.

## 참고 자료

- [The Open Network, p. 80](https://ton.org/ton.pdf)
- [TON에서의 ADNL 구현](https://github.com/ton-blockchain/ton/tree/master/adnl)

_커뮤니티에 기여한 [hacker-volodya](https://github.com/hacker-volodya)에게 감사드립니다!_  
_원문은 [GitHub의 원본 문서](https://github.com/tonstack/ton-docs/tree/main/ADNL)에서 확인하실 수 있습니다._
