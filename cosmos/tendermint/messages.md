---
description: >-
  Tendermint의 내용과 주요한 method들만 정리해보았다. 여러 Type들이나 Snapshot관련 method들은 tendermint
  github를 참고하기 바란다.
---

# Messages

## Query

Application이 query를 보내 데이터를 요청한다.

선택적으로 Merkle proof를 반환한다.

Merkle proof는 다양한 타입의 Merkle tree와 인코딩 포맷을 지원하기 위해 type 필드를 포함한다.

## InitChain

genesis에서 한번 불린다.

ResponseInitChain.Validators가 비어있으면 초기 검증자 집합은 RequestInitChain.Validators로 정해진다.

ResponseInitChain.Validators가 비어 있지 않으면 RequestInitChain.Validators에 어떤 것이 있는지 상관없이 초기 검증자 집합이 된다.

이를 통해서 application은 tendermint가 제안한 초기 검증자 집합을 수락할지 다른 것을 사용할 지 정할 수 있다.

#### The initial chain state

초기 체인 상태를 지정.

Tendermint는 app\_state\_bytes를 application에 보내서 블록체인의 초기(genesis) 상태를 구성한다.

app\_hash에서 genesis 상태에 해당하는 Merkle root hash byte를 반환한다.

또한 application은 Tendermint로부터 보내진 검증자들의 리스트를 관리한다. Cosmos SDK의 BaseApp이 그 리스트를 관리한다.

## BeginBlock

새로운 블록의 시작을 알린다. 모든 DeliverTx보다 먼저 실행된다.

헤더는 height, timestamp 등을 포함한다. 이것은 Tendermint 블록 헤더와 정확히 일치한다.

LastCommitInfo와 ByzantineValidators는 검증자들을 보상 또는 처벌하는 데 사용할 수 있다.

#### A new block is about to be created

이 명령은 application이 올바른 위치에 **상태를 로드**하도록 지시한다.

* AppHash: \[]byte
  * 이전 블록을 실행하고 커밋한 후 애플리케이션에서 반환한 임의의 바이트 배열이다. 이는 ABCI 애플리케이션에서 나온 Merkle 증명을 검증하는 기초 역할을 하며 블록체인 자체의 상태가 아닌 실제 애플리케이션의 상태를 나타낸다. 첫 번째 블록의 block.Header.AppHash는 ResponseInitChain.app\_hash에 의해 제공된다.

BeginBlock으로 상태를 로드한 후 CheckTx와 DeliverTx를 받을 준비가 된다.

BeginBlock과 EndBlock은 블록에 transaction이 포함되지 않은 경우에도 ABCI를 통해 전송된다. 이 것은 연결이 되었다는 것을 확인시키고 작업이 없는 기간을 계산하는 데 도움이 된다.

## CheckTx

기술적으로 선택 사항 - 블록 처리에 관여하지 않는다. 모든 노드는 transaction을 로컬 mempool로 보내기 전에 CheckTx를 실행한다. Transaction은 외부 사용자 또는 다른 노드에서 올 수 있다. CheckTx는 transaction을 완전히 실행할 필요가 없지만 서명 및 계정 잔액 확인과 같이 가벼운 검사를 수행해야 한다. 가상 머신에서 코드를 실행하지는 않는다. ResponseCheckTx.Code != 0인 transaction은 거부된다. 다른 노드로 브로드캐스트되거나 제안 블록에 포함되지 않는다. Tendermint는 응답 코드에 다른 값을 부여하지 않는다.

#### A new transaction appears in the transaction pool

Transaction이 유효한지 application에서 검증한다.

Tendermint는 transaction이 유효한지 판단하지 않는다. ABCI(Application Blockchain Interface)를 통해 CheckTx 함수를 받고 이것을 이용해서 transaction을 체크한다. CheckTx는 application에서 구현된다.

2개의 transaction이 차례로 전송되고 둘 다 유효한 경우 두번째 transaction을 먼저 확인하면 첫번째 transaction의 상태가 반영되지 않았기 때문에 유효하지 않은 것으로 나타날 수 있다. 블록체인 상태에 대한 판단은 CheckTx에서 하지 않는 것이 좋다.

CheckTx에서는 transaction의 유효성을 확인하고 잘못된 형식, 입력 등이 포함된 transaction을 거부한다. 그러나 context에 의존하여 성공 여부를 판별하지 않아야 한다.

## DeliverTx

Application에서 필수 요소이다.

전체 transaction을 실행한다.

#### A transaction is added and needs to be processed

Tendermint는 블록의 정보를 application layer에 보내기 위해 DeliverTx 함수를 부른다.

가장 최신의 블록 상태에 적용한다.

필요한 경우에 오류를 처리한다.

ABCI를 통해 transaction을 되돌려보내야 할지 아닐지 판단한다. Transaction이 성공하면 다음 transaction을 위해 변경된 state를 메모리에 유지한다. 이 시점에서 실제로 state를 바꾸진 않는다.

또한 `events: repeated Event` 를 통해 어떤 정보가 인덱싱 될지 정의할 수 있다. 인덱싱된 데이터는 블록 전체에서 빠르게 검색할 수 있다.

## EndBlock

블록의 마지막이라는 신호이다.

Commit 전, 모든 transaction후에 실행된다.

#### The block is being wrapped up

## Commit

애플리케이션 상태를 유지한다. 나중에 쿼리를 호출하면 이 머클 루트 해시에 고정된 애플리케이션 상태에 대한 증명을 반환할 수 있다. 개발자는 원하는 모든 것을 반환할 수 있다(아무것도 아니거나 상수 문자열 등이 될 수 있다). 단, BeginBlock/DeliverTx/EndBlock method에서 가져온 결정적인 함수여야 한다.

RetainHeight는 해당 블록 height 아래의 블록을 제거하는데 네트워크의 모든 노드가 과거 블록을 제거하면 이 데이터는 영구적으로 손실되고 새 노드는 네트워크 및 부트스트랩에 참여할 수 없다.

## Reference

[A Blockchain App Architecture | Cosmos Developer Portal](https://tutorials.cosmos.network/academy/2-main-concepts/architecture.html#checktx)

[spec/abci.md at c939e155a6aab1bfab36479a652d2d8a34a2ba29 · tendermint/spec](https://github.com/tendermint/spec/blob/c939e15/spec/abci/abci.md)
