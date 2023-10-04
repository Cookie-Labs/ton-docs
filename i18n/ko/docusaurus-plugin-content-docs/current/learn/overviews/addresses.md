# 스마트 계약 주소

이 섹션에서는 TON 블록체인의 스마트 계약 주소에 대한 구체적인 내용과 TON에서 액터가 스마트 계약과 동의어임을 설명합니다.

## 모든 것이 스마트 계약입니다

TON에서 스마트 계약은 [액터 모델](/learn/overviews/ton-blockchain#single-actor)을 사용하여 구축됩니다. 사실, TON의 액터는 기술적으로 스마트 계약으로 표현됩니다. 이것은 즉, 당신의 지갑조차도 간단한 액터 (즉, 스마트 계약)임을 의미합니다.

일반적으로 액터는 들어오는 메시지를 처리하고 내부 상태를 변경하며 결과로 외부 메시지를 생성합니다. 이것은 TON 블록체인의 모든 액터 (즉, 스마트 계약)이 다른 액터로부터 메시지를 수신할 수 있도록 주소를 가져야 함을 의미합니다.

:::info EVM 경험
Ethereum Virtual Machine (EVM)에서는 주소가 완전히 스마트 계약과 별개입니다. Tal Kol의 ["Solidity 개발자들에게 놀라운 TON 블록체인의 6가지 독특한 측면"](https://blog.ton.org/six-unique-aspects-of-ton-blockchain-that-will-surprise-solidity-developers) 글을 읽어서 차이점에 대해 자세히 알아보세요.
:::

## 스마트 계약 주소

TON에서 작동하는 스마트 계약 주소는 일반적으로 두 가지 주요 구성 요소로 구성됩니다.

- **(workchain_id)**: 워크체인 ID를 나타냅니다 (부호 있는 32비트 정수)

- **(account_id)**: 계정의 주소를 나타냅니다 (워크체인에 따라 64-512비트)

이 문서의 원시 주소 개요 섹션에서 **(workchain_id, account_id)** 쌍이 어떻게 표현되는지 설명합니다.

### 워크체인 ID와 계정 ID

#### 워크체인 ID

[이전에 본 것처럼](/learn/overviews/ton-blockchain#workchain-blockchain-with-your-own-rules), TON 블록체인에서는 최대 `2^32`개의 워크체인을 생성할 수 있습니다. 또한 32비트 접두사 스마트 계약 주소가 다른 워크체인 내의 스마트 계약 주소와 식별되고 연결되는 방법을 설명했습니다. 이를 통해 TON 블록체인의 서로 다른 워크체인 간에 메시지를 보내고 받을 수 있게 됩니다.

현재는 Masterchain (workchain_id=-1)과 가끔 기본 워크체인 (workchain_id=0)만 TON 블록체인에서 작동 중입니다.

둘 다 256비트 주소를 가지므로 이제부터 workchain_id가 0 또는 -1이며 워크체인 내부의 주소가 정확히 256비트임을 가정합니다.

#### 계정 ID

TON의 모든 계정 ID는 MasterChain 및 BaseChain (또는 기본 워크체인)에서 256비트 주소를 사용합니다.

사실, 계정 ID는 스마트 계약 객체 (특히 SHA256)를 위한 해시 함수로 정의됩니다. TON 블록체인에서 작동하는 모든 스마트 계약은 주로 다음과 같은 두 가지 주요 구성 요소를 저장합니다.

1. _컴파일된 코드_. 스마트 계약의 로직은 바이트 코드로 컴파일됩니다.
2. _초기 상태_. 계약이 체인에 배포된 초기 시점의 계약 값입니다.

마침내 올바른 주소 계정을 수신하려면 **(초기 코드, 초기 상태)** 쌍에 해당하는 해시를 계산해야 합니다. 현재는 [TVM](/learn/tvm-instructions/tvm-overview)의 기술적 사양과 TL-B 스키마에 대한 심층적인 탐구에 대해 언급하지 않겠지만, TON의 계정 ID는 다음 수식을 사용하여 결정됩니다:

**계정 ID = 해시(초기 코드, 초기 상태)**

이 문서를 통해 **계정 ID**의 기술 사양과 TON에서의 스마트 계약 주소와의 상호 작용에 대한 심층적인 내용을 알아볼 것입니다. 이제 우리는 **Raw**와 **User-Friendly** 주소를 설명하겠습니다.

## Raw와 User-Friendly 주소

TON의 스마트 계약 주소가 워크체인 및 계정 ID(특히 MasterChain 및 BaseChain)를 활용하는 방식을 간략히 설명한 후에 이러한 주소가 두 가지 주요 형식으로 표현되는 것을 이해하는 것이 중요합니다.

- **Raw 주소**: 스마트 계약 주소의 원래 완전한 표현입니다.
- **User-Friendly 주소**: User-Friendly 주소는 더 나은 보안과 사용 편의성을 사용하는 Raw 주소의 향상된 형식입니다.

아래에서 이 두 주소 유형 간의 차이점에 대해 자세히 설명하고, User-Friendly 주소가 TON에서 사용되는 이유에 대해 심층적으로 파헤쳐보겠습니다.

### Raw 주소

Raw 스마트 계약 주소는 워크체인 ID 및 계정 ID *(workchain_id, account_id)*로 구성되며 다음과 같은 형식으로 표시됩니다.

- [10진수 workchain_id\]:[account_id로 구성된 64개의 16진수 숫자\]

아래에는 워크체인 ID와 계정 ID를 함께 사용하는 원시 스마트 계약 주소의 예가 제공됩니다. 이 주소는 **workchain_id** 및 **account_id**로 표현됩니다:

`-1:fcb91a3a3816d0f7b8c2c76108b8a9bc5a6b7a55bd79f8ab101c52db29232260`

주소 문자열의 시작 부분에 있는 `-1`은 MasterChain에 속하는 *workchain_id*를 나타냅니다.

:::note
대문자 (예: 'A', 'B', 'C', 'D' 등)는 '/' 및 '+' 대신 주소 문자열에 사용될 수 있습니다 (예: 'a', 'b', 'c', 'd' 등의 소문자 대신).
:::

#### Raw 주소 사용 시 문제점

Raw 주소 형식을 사용하면 다음과 같은 두 가지 주요 문제가 발생합니다.

1. Raw 주소 형식을 사용할 때, 거래를 보내기 전에 주소를 확인하여 오류를 없앨 수 없습니다. 즉, 거래를 보내기 전에 주소 문자열에서 문자를 실수로 추가하거나 제거하면 거래가 잘못된 대상에 보내져 자금을 잃게 됩니다.
2. Raw 주소 형식을 사용할 때, 사용자 친화적인 주소를 사용하는 거래와 같은 특수 플래그를 추가할 수 없습니다. 아래에서 사용자 친화적인 주소를 사용하는 거래에 사용되는 특수 플래그에 대해 설명하겠습니다.

### User-Friendly 주소

User-Friendly 주소는 인터넷 (예: 공개 메시징 플랫폼 및 이메일 서비스 제공업체) 및 현실 세계에서 주소를 공유하는 TON 사용자를 위해 안전하고 간편하게 만들기 위해 개발되었습니다.

#### User-Friendly 주소 구조

User-Friendly 주소는 총 36바이트로 구성되며 다음의 구성 요소를 순서대로 생성하여 얻습니다.

1. _[플래그 - 1바이트]_: 주소에 고정된 플래그는 스마트 계약이 수신한 메시지에 대한 반응 방식을 변경합니다. User-Friendly 주소 형식을 사용하는 플래그 유형은 다음과 같습니다.

   - isBounceable. 반송 가능한 또는 반송 불가능한 주소 유형을 나타냅니다. ("bounceable"에 대한 _0x11_, "non-bounceable"에 대한 _0x51_)
   - isTestnetOnly. 테스트넷 용도로만 사용되는 주소 유형을 나타냅니다. *0x80*로 시작하는 주소는 본 프로덕션 네트워크에서 실행되는 소프트웨어에서 허용되지 않아야 합니다.
   - isUrlSafe. 주소에 대한 url 안전 플래그로 정의됩니다. 모든 주소가 url 안전한 것으로 간주됩니다.

2. _\[workchain_id - 1바이트]_: 워크체인 ID(_workchain_id_)는 부호 있는 8비트 정수 *workchain_id*에 의해 정의됩니다. (*0x00*는 BaseChain, *0xff*는 MasterChain을 나타냅니다.)
3. _\[account_id - 32바이트]_: 계정 ID는 워크체인 내부의 ([big-endian](https://www.freecodecamp.org/news/what-is-endianness-big-endian-vs-little-endian/)) 256비트 주소로 구성됩니다.
4. _\[주소 확인 - 2바이트]_: User-Friendly 주소에서 주소 확인은 이전 34바이트의 CRC16-CCITT 서명으로 구성됩니다. (예: [예제](https://github.com/andreypfau/ton-kotlin/blob/ce9595ec9e2ad0eb311351c8a270ef1bd2f4363e/ton-kotlin-crypto/common/src/crc32.kt))
   실제로 User-Friendly 주소에 대한 확인은 사용자가 실수로 존재하지 않는 카드 번호를 작성하지 못하도록 모든 은행 카드에서 사용되는 [Luhn 알고리즘](https://en.wikipedia.org/wiki/Luhn_algorithm)과 유사한 아이디어입니다.

이러한 4가지 주요 구성 요소를 추가하면 총 36바이트(사용자당 User-Friendly 주소)가 됩니다.

User-Friendly 주소를 생성하려면 개발자가 모든 36바이트를 다음 중 하나를 사용하여 인코딩해야 합니다:

- _base64_ (숫자, 대문자 및 소문자 라틴 문자, '/' 및 '+' 포함)
- _base64url_ ('/' 및 '+' 대신 '\_'와 '-' 사용)

이 과정이 완료되면 48개의 공백 없는 문자로 된 User-Friendly 주소가 생성됩니다.

:::info DNS 주소 플래그
TON에서는 DNS 주소 (예: mywallet.ton)가 종종 원시 및 User-Friendly 주소 대신 사용됩니다. 실제로 DNS 주소는 TON 도메인 내에서 DNS 레코드의 모든 필요한 플래그를 액세스할 수 있도록 User-Friendly 주소로 구성되어 있습니다.
:::

#### User-Friendly 주소 인코딩 예제

예를 들어, "테스트 기부자" 스마트 계약 (테스트넷 마스터체인에 있는 특수 스마트 계약으로, 요청하는 모든 사람에게 2개의 테스트 토큰을 보내줍니다)은 다음과 같은 Raw 주소를 사용합니다:

`-1:fcb91a3a3816d0f7b8c2c76108b8a9bc5a6b7a55bd79f8ab101c52db29232260`

위의 "테스트 기부자" Raw 주소는 User-Friendly 주소 형식으로 변환되어야 합니다. 이를 위해 이전에 소개한 base64 또는 base64url 형식을 사용하여 다음과 같이 사용됩니다:

- `kf/8uRo6OBbQ97jCx2EIuKm8Wmt6Vb15+KsQHFLbKSMiYIny` (base64)
- `kf_8uRo6OBbQ97jCx2EIuKm8Wmt6Vb15-KsQHFLbKSMiYIny` (base64url)

:::info
base64 및 base64url 양식 모두 유효하며 모두 허용되어야 합니다!
:::

#### 반송 가능 주소(Bounceable) vs. 비반송 가능 주소(Non-Bounceable)

반송 가능 주소 플래그의 핵심 아이디어는 발신자의 보안입니다.

예를 들어, 목적지 스마트 계약이 존재하지 않거나 거래 처리 중에 검색 문제가 발생한 경우, 메시지는 발신자에게 "반송"되어 원래 거래의 나머지 부분(모든 이체 및 가스 수수료 제외)을 구성하게 됩니다. 이렇게 하면 거래를 수락할 수 없는 주소로 전송된 자금을 잃지 않도록 발신자의 자금이 보호됩니다.

반송 가능한 주소에 관한 추가 정보는 [반송 가능하지 않은 메시지](/develop/smart-contracts/guidelines/non-bouncable-messages)를 보다 잘 이해하기 위해 문서에서 더 읽어보세요.

#### 방어용 base64 표현

TON 블록체인과 관련된 추가 이진 데이터는 이러한 데이터의 첫 4자를 기준으로 서로 다른 "방어용" base64 User-Friendly 주소 표현을 사용합니다. 예를 들어, 256비트 Ed25519 공개 키는 일반적으로 32바이트 시퀀스로 표현되며 다음과 같은 프로세스를 사용하여 36바이트 시퀀스를 생성합니다.

- 단일 바이트 태그를 사용하여 _0x3E_ 형식은 공개 키를 나타냅니다.
- 단일 바이트 태그를 사용하여 _0xE6_ 형식은 Ed25519 공개 키를 나타냅니다.
- Ed25519 공개 키의 표준 이진 표현을 포함하는 32바이트
- 이전 34바이트의 CRC16-CCITT의 big-endian 표현을 포함하는 2바이트

그 결과 36바이트 시퀀스는 다음과 같이 표준 방식으로 48자리 base64 또는 base64url 문자열로 변환됩니다. 예를 들어, Ed25519 공개 키 `E39ECDA0A7B0C60A7107EC43967829DBE8BC356A49B9DFC6186B3EAC74B5477D` (일반적으로 `0xE3, 0x9E, ..., 0x7D`와 같은 32바이트 시퀀스로 표현됨)는 다음과 같이 "방어용" 표현으로 표시됩니다.

`Pubjns2gp7DGCnEH7EOWeCnb6Lw1akm538YYaz6sdLVHfRB2`

### User-Friendly 주소와 Raw 주소 변환

User-Friendly 및 Raw 주소를 변환하는 가장 간단한 방법은 TON API 및 다른 도구를 사용하는 것입니다. 이를 위해 다음과 같은 도구를 사용할 수 있습니다.

- [ton.org/address](https://ton.org/address)
- [mainnet에서 toncenter API 메서드](https://toncenter.com/api/v2/#/accounts/pack_address_packAddress_get)
- [testnet에서 toncenter API 메서드](https://testnet.toncenter.com/api/v2/#/accounts/pack_address_packAddress_get)

또한 JavaScript를 사용하여 지갑의 User-Friendly 및 Raw 주소를 변환하는 두 가지 방법이 있습니다.

- [ton.js를 사용하여 사용자 친화적인 또는 Raw 형식의 주소 변환](https://github.com/ton-org/ton-core/blob/main/src/address/Address.spec.ts)
- [tonweb을 사용하여 사용자 친화적인 또는 Raw 형식의 주소 변환](https://github.com/toncenter/tonweb/tree/master/src/utils#address-class)

[SDK](/develop/dapps/apis/sdk)를 사용하여 유사한 메커니즘을 사용할 수도 있습니다.
