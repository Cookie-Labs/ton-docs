# Get 메서드

:::note
진행하기 전에 [FunC 프로그래밍 언어](/develop/func/overview)와 TON 블록체인에서의 [스마트 계약 개발](/develop/smart-contracts)에 대한 기본적인 이해가 있으면 여기에서 제공된 정보를 더 효과적으로 이해하는 데 도움이 될 것입니다.
:::

## 소개

Get 메서드는 스마트 계약에서 특정 데이터를 조회하기 위해 만들어진 특별한 함수입니다. 이러한 함수의 실행에는 어떠한 수수료도 들지 않으며 블록체인 외부에서 실행됩니다.

이러한 함수들은 대부분의 스마트 계약에서 매우 일반적입니다. 예를 들어, 기본 [지갑 계약](/participate/wallets/contracts)은 `seqno()`, `get_subwallet_id()`, `get_public_key()`와 같은 여러 개의 get 메서드를 가지고 있습니다. 이러한 메서드들은 지갑에 대한 데이터를 가져오기 위해 지갑, SDK 및 API에서 사용됩니다.

## Get 메서드 디자인 패턴

### 기본 Get 메서드 디자인 패턴

1. **단일 데이터 포인트 조회**: 기본 디자인 패턴 중 하나는 계약 상태에서 개별 데이터 포인트를 반환하는 메서드를 만드는 것입니다. 이러한 메서드는 매개변수가 없으며 단일 값을 반환합니다.

   예시:

   ```func
   int get_balance() method_id {
       return get_data().begin_parse().preload_uint(64);
   }
   ```

2. **집계 데이터 조회**: 다른 일반적인 패턴은 일부 데이터 포인트가 자주 함께 사용될 때 계약 상태에서 여러 데이터 포인트를 하나의 호출에서 반환하는 메서드를 만드는 것입니다. 이는 주로 일부 데이터 포인트가 함께 자주 사용될 때 사용됩니다. 이러한 패턴은 주로 [Jetton](#jettons) 및 [NFT](#nfts) 계약에서 사용됩니다.

   예시:

   ```func
   (int, slice, slice, cell) get_wallet_data() method_id {
       return load_data();
   }
   ```

### 고급 Get 메서드 디자인 패턴

1. **계산된 데이터 조회**: 경우에 따라 조회해야 하는 데이터가 계약 상태에 직접 저장되지 않고 상태와 입력 인수를 기반으로 계산되는 경우가 있습니다.

   예시:

   ```func
   slice get_wallet_address(slice owner_address) method_id {
       (int total_supply, slice admin_address, cell content, cell jetton_wallet_code) = load_data();
       return calculate_user_jetton_wallet_address(owner_address, my_address(), jetton_wallet_code);
   }
   ```

2. **조건부 데이터 조회**: 때로는 조회해야 하는 데이터가 현재 시간과 같은 특정 조건에 의존하는 경우가 있습니다.

   예시:

   ```func
   (int) get_ready_to_be_used() method_id {
       int ready? = now() >= 1686459600;
       return ready?;
   }
   ```

## 가장 일반적인 Get 메서드

### 표준 지갑

#### seqno()

```func
int seqno() method_id {
    return get_data().begin_parse().preload_uint(32);
}
```

특정 지갑 내에서 거래의 순서 번호를 반환합니다. 이 메서드는 주로 [재생 보호](/develop/smart-contracts/tutorials/wallet#replay-protection---seqno)를 위해 사용됩니다.

#### get_subwallet_id()

```func
int get_subwallet_id() method_id {
    return get_data().begin_parse().skip_bits(32).preload_uint(32);
}
```

- [서브지갑 ID란 무엇인가요?](/develop/smart-contracts/tutorials/wallet#what-is-subwallet-id)

#### get_public_key()

```func
int get_public_key() method_id {
    var cs = get_data().begin_parse().skip_bits(64);
    return cs.preload_uint(256);
}
```

지갑과 연결된 공개 키를 검색합니다.

### Jettons

#### get_wallet_data()

```func
(int, slice, slice, cell) get_wallet_data() method_id {
    return load_data();
}
```

이 메서드는 jetton 지갑과 관련된 전체 데이터 세트를 반환합니다.

- (int) 잔액
- (slice) 소유자 주소
- (slice) jetton 마스터 주소
- (cell) jetton 지갑 코드

#### get_jetton_data()

```func
(int, int, slice, cell, cell) get_jetton_data() method_id {
    (int total_supply, slice admin_address, cell content, cell jetton_wallet_code) = load_data();
    return (total_supply, -1, admin_address, content, jetton_wallet_code);
}
```

jetton 마스터에 대한 데이터를 반환하며, 이 데이터에는 총 공급량, 관리자 주소, jetton 내용 및 지갑 코드가 포함됩니다.

#### get_wallet_address(slice owner_address)

```func
slice get_wallet_address(slice owner_address) method_id {
    (int total_supply, slice admin_address, cell content, cell jetton_wallet_code) = load_data();
    return calculate_user_jetton_wallet_address(owner_address, my_address(), jetton_wallet_code);
}
```

소유자의 주소를 입력하면 이 메서드는 소유자의 jetton 지갑 계약 주소를 계산하여 반환합니다.

### NFTs

#### get_nft_data()

```func
(int, int, slice, slice, cell) get_nft_data() method_id {
    (int init?, int index, slice collection_address, slice owner_address, cell content) = load_data();
    return (init?, index, collection_address, owner_address, content);
}
```

Non-fungible 토큰에 관련된 데이터를 반환하며, 이 데이터에는 초기화 여부, 컬렉션 내의 인덱스, 컬렉션 주소, 소유자 주소 및 개별 컨텐츠가 포함됩니다.

#### get_collection_data()

```func
(int, cell, slice) get_collection_data() method_id {
    var (owner_address, next_item_index, content, _, _) = load_data();
    slice cs = content.begin_parse();
    return (next_item_index, cs~load_ref(), owner_address);
}
```

NFT 컬렉션에 대한 데이터를 반환하며, 이 데이터에는 다음으로 발행할 항목의 인덱스, 컬렉션의 컨텐츠 및 소유자 주소가 포함됩니다.

#### get_nft_address_by_index(int index)

```func
slice get_nft_address_by_index(int index) method_id {
    var (_, _, _, nft_item_code, _) = load_data();
    cell state_init = calculate_nft_item_state_init(index, nft_item_code);
    return calculate_nft_item_address(workchain(), state_init);
}
```

인덱스를 입력하면 이 메서드는 해당 컬렉션의 해당 NFT 항목 계약 주소를 계산하여 반환합니다.

#### royalty_params()

```func
(int, int, slice) royalty_params() method_id {
    var (_, _, _, _, royalty) = load_data();
    slice rs = royalty.begin_parse();
    return (rs~load_uint(16), rs~load_uint(16), rs~load_msg_addr());
}
```

NFT에 대한 로열티 매개변수를 가져옵니다. 이러한 매개변수에는 NFT가 판매될 때 원래 생성자에게 지불되는 로열티 비율이 포함됩니다.

#### get_nft_content(int index, cell individual_nft_content)

```func
cell get_nft_content(int index, cell individual_nft_content) method_id {
    var (_, _, content, _, _) = load_data();
    slice cs = content.begin_parse();
    cs~load_ref();
    slice common_content = cs~load_ref().begin_parse();
    return (begin_cell()
            .store_uint(1, 8) ;; 오프체인 태그
            .store_slice(common_content)
            .store_ref(individual_nft_content)
            .end_cell());
}
```

인덱스와 [개별 NFT 내용](#get_nft_data)을 입력하면 이 메서드는 NFT의 공통 및 개별 내용을 가져와 반환합니다.

## Get 메서드 사용 방법

### 인기 있는 탐색기에서 Get 메서드 호출

#### Tonviewer

"메서드" 탭에서 Get 메서드를 호출할 수 있습니다.

- [Tonviewer에서 Get 메서드 호출](https://tonviewer.com/EQAWrNGl875lXA6Fff7nIOwTIYuwiJMq0SmtJ5Txhgnz4tXI?section=Methods)

#### Ton.cx

"Get 메서드" 탭에서 Get 메서드를 호출할 수 있습니다.

- [Ton.cx에서 Get 메서드 호출](https://ton.cx/address/EQAWrNGl875lXA6Fff7nIOwTIYuwiJMq0SmtJ5Txhgnz4tXI)

### 코드에서 Get 메서드 호출

아래 예제에서는 JavaScript 라이브러리와 도구를 사용합니다.

- [ton 라이브러리](https://github.com/ton-core/ton)
- [Blueprint](/develop/smart-contracts/sdk/javascript) SDK

가정해봅시다. 다음과 같은 Get 메서드를 가진 어떤 스마트 계약이 있다고 합시다:

```func
(int) get_total() method_id {
    return get_data().begin_parse().preload_uint(32); ;; 계약 데이터에서 32비트 숫자를 로드하고 반환합니다.
}
```

이 메서드는 계약 데이터에서 로드한 단일 숫자를 반환합니다.

아래 코드 스니펫은 이 Get 메서드를 알려진 주소에 배포된 어떤 계약에서 호출하는 방법을 보여줍니다:

```ts
import { Address, TonClient } from "ton";

async function main() {
  // 클라이언트 생성
  const client = new TonClient({
    endpoint: "https://toncenter.com/api/v2/jsonRPC",
  });

  // Get 메서드 호출
  const result = await client.runMethod(
    Address.parse("EQD4eA1SdQOivBbTczzElFmfiKu4SXNL4S29TReQwzzr_70k"),
    "get_total"
  );
  const total = result.stack.readNumber();
  console.log("Total:", total);
}

main();
```

이 코드는 `Total: 123` 출력을 생성합니다. 숫자는 다를 수 있으며 이것은 단순한 예제입니다.

### Get 메서드 테스트

스마트 계약을 테스트하기 위해 [Sandbox](https://github.com/ton-community/sandbox)를 사용할 수 있습니다. Sandbox는 새로운 Blueprint 프로젝트에 기본으로 설치되어 있습니다.

먼저, get 메서드를 실행하고 결과를 반환하는 특수한 메서드를 계약 래퍼에 추가해야 합니다. 예를 들어, 계약이 *Counter*이고 이미 저장된 숫자를 업데이트하는 메서드를 구현했다고 가정합니다. `wrappers/Counter.ts`를 열고 다음 메서드를 추가합니다.

```ts
async getTotal(provider: ContractProvider) {
    const result = (await provider.get('get_total', [])).stack;
    return result.readNumber();
}
```

이 메서드는 get 메서드를 실행하고 결과 스택을 가져옵니다. get 메서드의 경우 스택은 반환한 것과 동일합니다. 이 코드에서는 스택에서 숫자를 읽습니다. 여러 값을 동시에 반환하는 복잡한 경우에는 여러 번 `readSomething` 유형의 메서드를 호출하여 스택에서 전체 실행 결과를 구문 분석할 수 있습니다.

마지막으로 이 메서드를 테스트에서 사용할 수 있습니다. `tests/Counter.spec.ts`로 이동하고 다음 테스트를 추가합니다.

```ts
it("should return correct number from get method", async () => {
  const caller = await blockchain.treasury("caller");
  await counter.sendNumber(caller.getSender(), toNano("0.01"), 123);
  expect(await counter.getTotal()).toEqual(123);
});
```

터미널에서 `npx blueprint test`를 실행하여 확인하고 모든 것을 올바르게 수행하면 이 테스트가 통과됩니다!

## 다른 계약에서 Get 메서드 호출

Get 메서드를 다른 계약에서 직접 호출하는 것은 가능하지 않습니다. 이는 주로 블록체인 기술의 본질과 합의의 필요성 때문입니다.

첫째, 다른 샤드체인에서 데이터를 가져오는 데는 시간이 소요될 수 있습니다. 이러한 지연은 블록체인 작업이 결정적이고 시기 적절하게 실행되어야 하는 환경에서 실행 흐름을 쉽게 방해할 수 있습니다.

둘째, 검증자 간에 합의를 이루는 것은 문제가 될 수 있습니다. 검증자가 트랜잭션의 정확성을 검증하려면 대상 계약에서 동일한 get 메서드를 호출해야 합니다. 그러나 이러한 다중 호출 사이에서 대상 계약의 상태가 변경되는 경우, 검증자는 트랜잭션 결과의 다른 버전을 가질 수 있습니다.

마지막으로, TON 블록체인의 스마트 계약은 같은 입력에 대해 항상 같은 출력을 생성하는 순수한 함수로 설계되었습니다. 이 원칙은 메시지 처리 중 합의를 간단하게 하므로 런타임에서 임의로 변경되는 데이터를 가져오는 것은 결정적인 속성을 깨뜨릴 것입니다.

### 개발자에게 미치는 영향

이러한 제한 사항은 한 계약이 다른 계약의 Get 메서드를 통해 다른 계약의 상태에 직접 접근할 수 없음을 의미합니다. 계약의 결정론적 흐름에 실시간 외부 데이터를 통합할 수 없는 무능력은 제한적으로 보일 수 있습니다. 그러나 이러한 제한 조건들이 블록체인 기술의 무결성과 신뢰성을 보장하는 것입니다.

### 솔루션 및 해결 방법

TON 블록체인에서 스마트 계약은 다른 계약으로부터 직접적으로 상태를 조회하기 위한 호출이 아닌 메시지를 통해 통신합니다. 특정 메서드를 실행하도록 요청하는 메시지는 대상 계약에 전송됩니다. 이러한 요청은 일반적으로 특별한 [운영 코드](/develop/smart-contracts/guidelines/internal-messages)로 시작합니다.

이러한 요청을 수용하도록 설계된 계약은 요청된 메서드를 실행하고 결과를 별도의 메시지로 반환합니다. 이러한 방식은 계약 간 통신을 간소화하고 블록체인 네트워크의 확장성과 성능을 향상시키는 데 기여합니다.

이 메시지 전달 메커니즘은 TON 블록체인의 작동에 필수적이며 샤드 간의 범용적인 동기화 없이 확장 가능한 네트워크 성장을 가능하게 합니다.

효율적인 계약 간 통신을 위해 귀하의 계약이 요청을 올바르게 수락하고 응답할 수 있도록 설계되어야 합니다. 이는 응답을 반환하기 위해 체인 상에서 호출할 수 있는 메서드를 지정하는 것을 포함합니다.

간단한 예를 살펴보겠습니다.

```func
#include "imports/stdlib.fc";

int get_total() method_id {
    return get_data().begin_parse().preload_uint(32);
}

() recv_internal(int my_balance, int msg_value, cell in_msg_full, slice in_msg_body) impure {
    if (in_msg_body.slice_bits() < 32) {
        return ();
    }

    slice cs = in_msg_full.begin_parse();
    cs~skip_bits(4);
    slice sender = cs~load_msg_addr();

    int op = in_msg_body~load_uint(32); ;; 작업 코드 로드
    if (op == 1) { ;; 숫자를 증가시키고 업데이트합니다.
        int number = in_msg_body~load_uint(32);
        int total = get_total();
        total += number;
        set_data(begin_cell().store_uint(total, 32).end_cell());
    }
    elseif (op == 2) { ;; 숫자를 조회합니다.
        int total = get_total();
        send_raw_message(begin_cell()
            .store_uint(0x18, 6)
            .store_slice(sender)
            .store_coins(0)
            .store_uint(0, 107)
            .store_uint(3, 32) ;; 응답 작업 코드
            .store_uint(total, 32) ;; 결과 데이터
            .end_cell(), 64);
    }
}
```

이 예에서 `recv_internal` 메서드는 입력 메시지를 분석하고 두 가지 작업을 수행할 수 있습니다. 첫 번째 작업은 숫자를 업데이트하고 두 번째 작업은 현재 숫자를 조회하는 것입니다. `get_total` 메서드는 현재 숫자를 조회합니다.

입력 메시지에서 작업 코드를 가져와 작업을 분기 처리합니다. 작업 코드가 1인 경우 숫자를 업데이트하고 작업 코드가 2인 경우 숫자를 조회하고 결과를 반환하는 메시지를 생성하여 송신자에게 보냅니다.

이러한 방식으로 다른 계약에서 Get 메서드를 직접 호출하는 대신 요청을 통해 필요한 데이터를 가져올 수 있습니다. 이렇게 함으로써 블록체인 네트워크의 정합성과 합의를 유지하면서 계약 간 효율적인 통신을 가능하게 합니다.
