# 블록체인의 블록체인

:::tip
이 문서에서 '**스마트 계약**', '**계정**' 및 '**액터**'라는 용어는 블록체인 엔터티를 설명하는 데 사용됩니다.
:::

## 단일 액터

하나의 스마트 계약을 고려해 봅시다.

TON에서, 이것은 `주소`, `코드`, `데이터`, `잔액` 등과 같은 속성을 가진 것입니다. 다시 말해, 이것은 어떤 _저장_ 및 *동작*을 가진 객체입니다.
이 동작은 다음과 같은 패턴을 가집니다:

- 어떤 일이 발생합니다 (가장 일반적인 상황은 계약이 메시지를 받는 것입니다).
- 계약은 TON 가상 머신에서 자체 `코드`를 실행하여 자체 속성에 따라 이벤트를 처리합니다.
- 계약은 자체 속성을 수정합니다 (`코드`, `데이터` 등).
- 계약은 선택적으로 발신 메시지를 생성합니다.
- 계약은 다음 이벤트가 발생할 때까지 대기 모드로 들어갑니다.

이러한 단계의 조합을 **트랜잭션**이라고 합니다. 이벤트가 하나씩 처리되므로 *트랜잭션*은 엄격하게 정렬되어 있으며 서로 중단시킬 수 없습니다.

이 동작 패턴은 '액터'라고 알려져 있으며 잘 알려진 것입니다.

### 최하위 수준: 계정 체인

_트랜잭션_ `Tx1 -> Tx2 -> Tx3 -> ....`의 연속을 **체인**이라고 할 수 있습니다. 그리고 고려된 경우 **계정 체인**이라고 합니다. 하나의 계정의 트랜잭션 체인입니다.

이제, 트랜잭션을 처리하는 노드가 상태를 조율해야 하는 경우 (상태에 대한 *일치*를 도출하려면) 이러한 *트랜잭션*들은 배치됩니다:
`[Tx1 -> Tx2] -> [Tx3 -> Tx4 -> Tx5] -> [] -> [Tx6]`.
배치는 순서에 개입하지 않으며, 각 트랜잭션에는 여전히 하나의 '이전 트랜잭션'과 최대 하나의 '다음 트랜잭션'만 있습니다. 그러나 이제 이 순서는 **블록**으로 나누어집니다.

또한 *블록*에는 들어오는 및 나가는 메시지의 대기열을 포함하는 것이 유용합니다. 이 경우 *블록*에는 스마트 계약에 대한 발생 내용을 결정하고 설명하는 정보의 완전한 세트가 포함됩니다.

## 여러 계정 체인: 샤드

이제 여러 계정을 고려해 봅시다. 몇 가지 *계정 체인*을 얻을 수 있으며, 이러한 _계정 체인_ 세트를 **샤드 체인**이라고 합니다. 마찬가지로 **샤드 체인**을 **샤드 블록**으로 나눌 수 있으며, 이는 개별 *계정 블록*의 집합입니다.

### 샤드 체인의 동적 분할 및 병합

*샤드 체인*은 쉽게 식별 가능한 *계정 체인*으로 구성되므로 쉽게 분할할 수 있습니다. 따라서 100만 개의 계정에 발생하는 트랜잭션 수가 한 노드에서 처리하고 저장하기에 너무 많다면 그 체인을 두 개의 작은 *샤드 체인*으로 나눌 수 있습니다. 각 체인은 50만 개의 계정을 담당하며 각 체인은 별도의 노드 하위 집합에서 처리됩니다.

비슷하게, 어떤 샤드가 너무 미사용 상태가 되면 더 큰 샤드로 **병합**할 수 있습니다.

물론 두 가지 극단적인 경우가 있습니다. 한 계정만 포함된 경우 (그래서 더 나눌 수 없는 경우)와 한 샤드에 모든 계정이 포함된 경우입니다.

계정은 서로 메시지를 보내면서 상호 작용할 수 있습니다. 메시지를 보내기 위한 특별한 라우팅 메커니즘이 있어서 메시지를 발신 대기열에서 해당 수신 대기열로 이동하고 1) 모든 메시지가 전달되며 2) 메시지가 연속적으로 전달됩니다.

:::info 부가 정보
분할과 병합을 결정하기 위해 계정 체인을 샤드에 집합화하는 것은 계정 주소의 비트 표현을 기반으로 합니다. 예를 들어, 주소는 `(샤드 접두사, 주소)`와 같이 생겼습니다. 이렇게 하면 샤드 체인의 모든 계정이 정확히 동일한 이진 접두사를 가집니다 (예: 모든 주소가 `0b00101`로 시작).
:::

## 블록체인

하나의 규칙 집합에 따라 작동하는 모든 샤드를 집합으로 하는 것을 **블록체인**이라고 합니다.

TON에서는 많은 규칙 집합이 가능하며, 따라서 동시에 작동하고 서로 메시지를 보내는 여러 블록체인이 존재할 수 있습니다. 한 블록체인의 계정이 서로 상호 작용할 수 있는 것처럼 메시지를 교차 체인으로 보낼 수 있습니다.

### 워크체인: 사용자 정의 규칙을 가진 블록체인

Shardchains의 규칙을 사용자 정의하려면 **워크체인**을 만들 수 있습니다. 예를 들어 EVM 기반으로 작동하고 Solidity 스마트 계약을 실행하는 워크체인을 만드는 것이 좋은 예입니다.

이론적으로 커뮤니티의 모든 사람들은 자신만의 워크체인을 만들 수 있습니다. 실제로는 워크체인을 구축하고 생성 비용을 지불하고 해당 워크체인의 생성을 승인하기 위해 검증자의 2/3의 투표를 받는 것이 꽤 복잡한 작업입니다.

TON은 최대 `2^30`개의 워크체인을 생성할 수 있으며, 각각을 최대 `2^60`개의 샤드로 세분화할 수 있습니다.

현재 TON에는 MasterChain과 BaseChain 두 가지 워크체인만 있습니다.

BaseChain은 주로 액터 간의 일상 거래에 사용되며 비용이 비교적 저렴합니다. 반면 MasterChain은 TON에 중요한 역할을 하는데, 그 역할에 대해 알아봅시다!

### 마스터체인: 블록체인의 블록체인

메시지 라우팅과 트랜잭션 실행을 동기화해야 하는 필요가 있습니다. 다시 말해, 네트워크의 노드들은 멀티체인 상태에서 어떤 '점'을 고치고 해당 상태에 대한 일치를 도출하기 위한 방법이 필요합니다. TON에서는 이러한 목적으로 **마스터체인**이라는 특수한 체인을 사용합니다. *마스터체인*의 블록에는 시스템의 다른 모든 체인에 대한 추가 정보 (최신 블록 해시)가 포함되어 있으므로 관찰자는 어떤 마스터체인 블록에서도 멀티체인 시스템의 상태를 명확하게 결정할 수 있습니다.
