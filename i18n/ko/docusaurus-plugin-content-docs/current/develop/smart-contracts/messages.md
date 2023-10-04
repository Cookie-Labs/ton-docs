# 메시지 전송

메시지의 구성, 구문 분석 및 전송은 [TL-B 스키마](/learn/overviews/TL-B), [트랜잭션 단계 및 TVM](/learn/tvm-instructions/tvm-overview.md)의 교차점에 있습니다.

실제로 FunC는 [send_raw_message](/develop/func/stdlib#send_raw_message) 함수를 노출시키는데, 이 함수는 직렬화된 메시지를 인수로 받습니다.

TON은 넓은 기능을 갖춘 포괄적인 시스템이기 때문에 이러한 기능을 지원해야 하는 메시지는 상당히 복잡할 수 있습니다. 그럼에도 불구하고 이러한 기능 중 대부분은 일반적인 시나리오에서 사용되지 않으며, 대부분의 경우 메시지 직렬화는 다음과 같이 간소화될 수 있습니다:

```func
  cell msg = begin_cell()
    .store_uint(0x18, 6)
    .store_slice(addr)
    .store_coins(amount)
    .store_uint(0, 1 + 4 + 4 + 64 + 32 + 1 + 1)
    .store_slice(message_body)
  .end_cell();
```

따라서 개발자는 이 문서에서 처음 읽을 때 이해하기 어려워서 두려워하지 않아도 됩니다. 전반적인 개념을 이해하면 됩니다.

자세히 알아봅시다!

## 메시지 유형

세 가지 유형의 메시지가 있습니다:

- 외부 - 블록체인 외부에서 블록체인 내의 스마트 계약으로 전송되는 메시지입니다. 이러한 메시지는 스마트 계약이 `크레딧 가스`라고 하는 과정에서 명시적으로 수락해야 합니다. 메시지가 수락되지 않으면 노드는 해당 메시지를 블록에 허용하거나 다른 노드로 중계해서는 안 됩니다.
- 내부 - 블록체인 엔터티에서 다른 블록체인 엔터티로 전송되는 메시지입니다. 외부와는 달리 이러한 메시지는 일부 TON을 전달할 수 있으며, 이 경우 이 메시지를 수신하는 스마트 계약이 수락하지 않을 수 있습니다. 이 경우 가스가 메시지 값에서 차감됩니다.
- 로그 - 블록체인 엔터티에서 외부 세계로 전송되는 메시지입니다. 일반적으로 이러한 메시지를 블록체인 밖으로 보내는 메커니즘이 없습니다. 실제로 네트워크의 모든 노드가 메시지가 생성되었는지 여부에 대해 합의하더라도 처리 방법에 대한 규칙은 없습니다. 로그는 직접 `/dev/null`로 보내거나 디스크에 기록하거나 인덱싱된 데이터베이스에 저장하거나 비 블록체인 수단(이메일/텔레그램/문자 메시지)으로 보낼 수 있으며, 이러한 모든 것은 주어진 노드의 재량에 따라 결정됩니다.

## 메시지 레이아웃

우리는 내부 메시지 레이아웃부터 시작하겠습니다.

스마트 계약에서 전송할 수 있는 메시지를 설명하는 TL-B 스키마는 다음과 같습니다:

```tlb
message$_ {X:Type} info:CommonMsgInfoRelaxed
  init:(Maybe (Either StateInit ^StateInit))
  body:(Either X ^X) = MessageRelaxed X;
```

이것을 말로 설명하겠습니다. 모든 메시지의 직렬화는 세 가지 필드로 구성됩니다: 정보(소스, 대상 및 기타 메타데이터를 설명하는 일종의 헤더), 초기화를 위한 init(메시지 초기화에만 필요한 필드) 및 본문(메시지 페이로드).

`Maybe` 및 `Either` 및 다른 유형의 표현은 다음과 같이 해석됩니다:

- `info:CommonMsgInfoRelaxed` 필드가 있으면 `CommonMsgInfoRelaxed`의 직렬화가 직접 직렬화 셀에 주입됩니다.
- `body:(Either X ^X)` 필드가 있으면 유형 `X`를 (역) 직렬화할 때 먼저 하나의 ` either` 비트를 넣습니다. 이 비트는 `X`가 동일한 셀에 직렬화되는 경우 `0`이거나 별도의 셀에 직렬화되는 경우 `1`입니다.
- `init:(Maybe (Either StateInit ^StateInit))` 필드가 있으면 먼저 이 필드가 비어 있는지 여부에 따라 `0` 또는 `1`을 넣습니다. 그리고 비어 있지 않은 경우 `Either StateInit ^StateInit`를 직렬화합니다(다시 말해 `StateInit`이 동일한 셀에 직렬화되는 경우 `0`이거나 별도의 셀에 직렬화되는 경우 `1`을 넣습니다).

`CommonMsgInfoRelaxed` 레이아웃은 다음과 같습니다:

```tlb
int_msg_info$0 ihr_disabled:Bool bounce:Bool bounced:Bool
  src:MsgAddress dest:MsgAddressInt
  value:CurrencyCollection ihr_fee:Grams fwd_fee:Grams
  created_lt:uint64 created_at:uint32 = CommonMsgInfoRelaxed;

ext_out_msg_info$11 src:MsgAddress dest:MsgAddressExt
  created_lt:uint64 created_at:uint32 = CommonMsgInfoRelaxed;
```

우선 `int_msg_info`에 중점을 두겠습니다.
`0`으로 시작하며, 여기에는 1비트 접두사 `0`이 있습니다. 이 비트는 `int_msg_info`임을 나타냅니다.

그런 다음 3개의 1비트 플래그가 있으며, 현재 항상 true인 인스턴트 하이퍼큐브 라우팅 비활성화 여부, 처리 중에 오류가 발생하면 메시지가 바운스되어야 하는지 여부, 메시지 자체가 바운스의 결과인지 여부를 나타냅니다. 그런 다음 소스 및 대상 주소가 직렬화되며, 메시지 값 및 메시지 전달 수수료 및 시간과 관련된 네 개의 정수가 이어집니다.

메시지가 스마트 계약에서 보내진 경우 이러한 필드 중 일부는 올바른 값으로 다시 쓰여집니다. 특히 검증자는 `bounced`, `src`, `ihr_fee`, `fwd_fee`, `created_lt` 및 `created_at`를 다시 작성합니다. 이것은 두 가지 의미를 가지고 있습니다. 첫째, 메시지 처리 중에 다른 스마트 계약은 이러한 필드를 신뢰할 수 있습니다(송신자는 소스 주소, `bounced` 플래그 등을 변조할 수 없음). 둘째, 직렬화하는 동안 이러한 필드에 유효한 값이 넣어질 수 있습니다(어쨌든 이러한 값은 덮어쓰일 것이기 때문에).

메시지를 단순히 직렬화하는 방법은 다음과 같습니다:

```func
  var msg = begin_cell()
    .store_uint(0, 1) ;; tag
    .store_uint(1, 1) ;; ihr_disabled
    .store_uint(1, 1) ;; allow bounces
    .store_uint(0, 1) ;; not bounced itself
    .store_slice(source)
    .store_slice(destination)
    ;; serialize CurrencyCollection (see below)
    .store_coins(amount)
    .store_dict(extra_currencies)
    .store_coins(0) ;; ihr_fee
    .store_coins(fwd_value) ;; fwd_fee
    .store_uint(cur_lt(), 64) ;; lt of transaction
    .store_uint(now(), 32) ;; unixtime of transaction
    .store_uint(0,  1) ;; no init-field flag (Maybe)
    .store_uint(0,  1) ;; inplace message body flag (Either)
    .store_slice(msg_body)
  .end_cell();
```

그러나 모든 필드를 단계별로 직렬화하는 대신 개발자들은 일반적으로 단축키를 사용합니다. 따라서 스마트 계약에서 메시지를 보내는 방법을 살펴보겠습니다. [elector-code](https://github.com/ton-blockchain/ton/blob/master/crypto/smartcont/elector-code.fc#L153)에서 가져온 예제를 사용하겠습니다.

```func
() send_message_back(addr, ans_tag, query_id, body, grams, mode) impure inline_ref {
  ;; int_msg_info$0 ihr_disabled:Bool bounce:Bool bounced:Bool src:MsgAddress -> 011000
  var msg = begin_cell()
    .store_uint(0x18, 6)
    .store_slice(addr)
    .store_coins(grams)
    .store_uint(0, 1 + 4 + 4 + 64 + 32 + 1 + 1)
    .store_uint(ans_tag, 32)
    .store_uint(query_id, 64);
  if (body >= 0) {
    msg~store_uint(body, 32);
  }
  send_raw_message(msg.end_cell(), mode);
}
```

첫 번째로 6비트에 `0x18` 값을 넣어 6비트로 `0b011000`을 넣습니다. 이게 무슨 의미일까요?

- 첫 번째 비트는 `0`으로 `int_msg_info`임을 나타내는 1비트 접두사입니다.
- 그런 다음 인스턴트 하이퍼큐브 라우팅이 비활성화되었음, 메시지에 오류가 발생하면 바운스될 수 있으며 메시지 자체가 바운스의 결과가 아님을 나타내는 3비트가 있습니다.
- 그런 다음에는 소스 주소를 직렬화합니다. 그러나 어차피 이 주소는 동일한 효과를 나타내므로 여기에는 유효한 주소를 저장할 수 있습니다. 가장 짧은 유효한 주소 직렬화는 `addr_none`의 경우이며 `00`이라는 두 비트 문자열로 직렬화됩니다.

따라서 `.store_uint(0x18, 6)`은 태그와 첫 4개 필드를 최적화된 방식으로 직렬화하는 방법입니다.

다음 줄에서는 대상 주소를 직렬화합니다.

그런 다음 값을 직렬화해야 합니다. 일반적으로 메시지 값은 다음과 같은 스키마를 가진 `CurrencyCollection` 객체입니다:

```tlb
nanograms$_ amount:(VarUInteger 16) = Grams;

extra_currencies$_ dict:(HashmapE 32 (VarUInteger 32))
                 = ExtraCurrencyCollection;

currencies$_ grams:Grams other:ExtraCurrencyCollection
           = CurrencyCollection;
```

이 스키마는 TON 값 외에도 추가 "extra-currencies" 딕셔너리를 함께 전달할 수 있다는 것을 의미합니다. 그러나 현재로서는 무시하고 메시지 값은 "number of nanotons as variable integer" 그리고 "`0` - empty dictionary bit"로 가정하고 메시지 값을 직렬화할 수 있습니다.

실제로 위의 elector 코드에서 `.store_coins(grams)`을 통해 동전 금액을 직렬화하지만 길이가 `1 + 4 + 4 + 64 + 32 + 1 + 1`인 제로 문자열을 넣습니다. 이것은 무엇인가요?

- 첫 번째 비트는 빈 `extra-currencies` 딕셔너리를 나타냅니다.
- 그런 다음 두 개의 4비트 길이 필드가 있습니다. 이것들은 `VarUInteger 16`을 인코딩한 0을 나타냅니다. 실제로 `ihr_fee`와 `fwd_fee`는 덮어쓰여질 것이므로 여기에 0을 넣어도 됩니다.
- 그런 다음 `created_lt` 및 `created_at` 필드에 0을 넣습니다. 이러한 필드는 덮어쓰여질 것이지만 수수료와 달리 고정된 길이를 갖고 있으므로 64비트 및 32비트 긴 문자열로 인코딩됩니다.
- (이미 메시지 헤더를 직렬화하고 init/body에 전달했습니다)
- 다음 0 비트는 `init` 필드가 없음을 나타냅니다.
- 마지막 0 비트는 msg_body가 인라인으로 직렬화될 것임을 나타냅니다.
- 이후에는 메시지 본문(임의의 레이아웃)이 인코딩됩니다.

이 방법으로 14개 매개변수의 개별 직렬화 대신 4개의 직렬화 기본 작업을 수행합니다.

## 전체 스키마

메시지 레이아웃 및 모든 구성 필드의 레이아웃(그리고 TON의 모든 객체의 스키마)의 전체 스키마는 [block.tlb](https://github.com/ton-blockchain/ton/blob/master/crypto/block/block.tlb)에 제시되어 있습니다.

## 메시지 크기

:::info 셀 크기
[Cell](/learn/overviews/cells)이 최대 `1023` 비트까지 포함할 수 있음을 참고하십시오. 더 많은 데이터를 저장해야 하는 경우, 데이터를 조각내고 참조 셀에 저장해야 합니다.
:::

예를 들어, 메시지 본문 크기가 900 비트인 경우, 메시지 헤더와 동일한 셀에 저장할 수 없습니다.
실제로, 메시지 헤더 필드 외에도 셀의 총 크기가 1023 비트를 초과하며 직렬화 중에 `셀 오버플로우` 예외가 발생할 것입니다. 이 경우에는 "인플레이스 메시지 본문 플래그 (Either)"로 표시되는 `0` 대신에 `1`이 있어야 하며 메시지 본문은 참조 셀에 저장되어야 합니다.

이러한 사항은 일부 필드가 가변 크기를 가지고 있기 때문에 주의 깊게 처리해야 합니다.

예를 들어, `MsgAddress`는 네 가지 생성자로 표현될 수 있으며 `addr_none`, `addr_std`, `addr_extern`, `addr_var`로 2 비트에서 ( `addr_none`의 경우) 가장 큰 형태의 `addr_var`로 586 비트까지의 길이를 가질 수 있습니다. 또한, 나노톤 금액은 `VarUInteger 16`으로 직렬화되며, 이는 정수의 바이트 길이를 나타내는 4 비트를 나타내고 이전에 언급한 바이트를 나타냅니다. 이렇게하면 0 나노톤은 `0b0000` (0 길이 바이트 문자열을 인코딩하는 4 비트 및 그 다음에 0 바이트)로 직렬화되고, 100,000,000 TON(또는 100000000000000000 나노톤)은 `0b10000000000101100011010001010111100001011101100010100000000000000000` (`0b1000`은 8 바이트를 나타내며 그 다음에 8 바이트 자체)로 직렬화됩니다.

## 메시지 모드

아마도 `send_raw_message`를 사용하여 메시지를 보내는 동안 모드를 수락하는 방법에 대해 알아보았을 것입니다. 모든 경우에서 메시지 자체를 처리할 뿐만 아니라 모드도 수락합니다. 필요에 가장 적합한 모드를 파악하려면 다음 표를 참조하십시오.

| 모드  | 설명                                                                                 |
| :---- | :----------------------------------------------------------------------------------- |
| `0`   | 보통 메시지                                                                          |
| `64`  | 새로운 메시지에서 초기에 지정된 값 외에 들어오는 메시지의 모든 남은 값을 가져옵니다. |
| `128` | 메시지에서 원래 지정된 값 대신 현재 스마트 계약의 잔액을 모두 가져옵니다.            |

| 플래그 | 설명                                                                               |
| :----- | :--------------------------------------------------------------------------------- |
| `+1`   | 메시지 값에서 전송 수수료를 별도로 지불합니다.                                     |
| `+2`   | 작업 단계 중에 이 메시지를 처리하는 동안 발생하는 모든 오류를 무시합니다.          |
| `+32`  | 결과 잔액이 제로인 경우 현재 계정을 파괴해야 합니다 (Mode 128과 함께 자주 사용됨). |

`send_raw_message`를 위한 모드를 작성하려면 모드와 플래그를 더하여 결합하기만 하면 됩니다. 예를 들어, 일반 메시지를 보내고 전송 수수료를 별도로 지불하려면 모드 `0`과 플래그 `+1`을 사용하여 `모드 = 1`을 얻을 수 있습니다. 전체 계약 잔액을 보내고 즉시 파괴하려면 모드 `128`과 플래그 `+32`를 사용하여 `모드 = 160`을 얻을 수 있습니다.
