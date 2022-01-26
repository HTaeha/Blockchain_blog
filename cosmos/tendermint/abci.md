# ABCI

## Connections

ABCI application은 Tendermint state-machine과 같은 process로 실행할 수도 있고 다른 process로 실행할 수도 있다. 같은 프로세스일 때 Tendermint는 ABCI application method를 직접 불러서 실행시킨다.

Tendermint와 ABCI application이 다른 프로세스로 실행될 때 Tendermint는 4가지 connection을 열어놓는다. 각 connection은 ABCI method의 일부를 다룬다.

#### Consensus connection

* 합의 프로토콜에 의해 실행되며 블록 실행을 담당한다.
* InitChain, BeginBlock, DeliverTx, EndBlock, Commit method를 다룬다.

#### Mempool connection

* 새로운 transaction이 공유되거나 블록에 포함되기 전에 검증한다.
* CheckTx 를 다룬다.

#### Info connection

* 초기화와 사용자를 위한 쿼리를 처리한다.
* Info, Query 를 다룬다.

#### Snapshot connection

* 노드간에 상태를 동기화시키고 복원한다.
* ListSnapshots, LoadSnapshotChunk, OfferSnapshot, ApplySnapshotChunk 를 다룬다.

추가적으로 Flush method는 모든 connection에서 부르고 Echo method는 디버깅을 위해 사용된다.

## Errors

몇몇 method들(Echo, Info, InitChain, BeginBlock, EndBlock, Commit)은 에러를 리턴하지 않는다. Application의 심각한 오류를 나타내고 Tendermint가 할 수 있는 일이 없기 때문이다.

다른 모든 method들(Query, CheckTx, DeliverTx)은 application마다 0이 OK 사인으로 예약되어 있는 `Code uint32`를 반환한다.

Query, CheckTx, DeliverTx는 다른 도메인의 Code를 명확히 하는 Codespace가 포함되어 있다. Codespace는 Code의 namespace이다.

## Events

CheckTx, BeginBlock, DeliverTx, EndBlock method들은 Events 필드를 그들의 Response에 포함하고 있다. Event는 타입과 method 실행 중 발생한 일을 나타내는 key-value 쌍인 attribute list로 구성된다.

Event는 실행 중에 발생한 일에 따라 transaction 및 블록을 인덱싱하는 데 사용할 수 있다. BeginBlock 및 EndBlock에서 블록에 대해 반환된 Event 집합이 병합된다. 두 method 모두 동일한 태그를 반환하는 경우 EndBlock에 정의된 값만 사용된다.

Type 필드는 Event를 카테고리화한다. Response나 tx는 중복된 type 값을 가진 여러 Event를 포함할 수 있다.

```go
message Event {
  string                  type       = 1;
  repeated EventAttribute attributes = 2;
}
```

```go
message EventAttribute {
  bytes key   = 1;
  bytes value = 2;
  bool  index = 3;  // nondeterministic
}
```

```go
abci.ResponseDeliverTx{
  // ...
 Events: []abci.Event{
  {
   Type: "validator.provisions",
   Attributes: []abci.EventAttribute{
    abci.EventAttribute{Key: []byte("address"), Value: []byte("..."), Index: true},
    abci.EventAttribute{Key: []byte("amount"), Value: []byte("..."), Index: true},
    abci.EventAttribute{Key: []byte("balance"), Value: []byte("..."), Index: true},
   },
  },
  {
   Type: "validator.provisions",
   Attributes: []abci.EventAttribute{
    abci.EventAttribute{Key: []byte("address"), Value: []byte("..."), Index: true},
    abci.EventAttribute{Key: []byte("amount"), Value: []byte("..."), Index: false},
    abci.EventAttribute{Key: []byte("balance"), Value: []byte("..."), Index: false},
   },
  },
  {
   Type: "validator.slashed",
   Attributes: []abci.EventAttribute{
    abci.EventAttribute{Key: []byte("address"), Value: []byte("..."), Index: false},
    abci.EventAttribute{Key: []byte("amount"), Value: []byte("..."), Index: true},
    abci.EventAttribute{Key: []byte("reason"), Value: []byte("..."), Index: true},
   },
  },
  // ...
 },
}
```

## EvidenceType

Tendermint 보안 모델은 일부 네트워크 참가자의 악의적인 행동을 증거로 남긴다. 악의적인 행동을 감지하고 체인에 commit하고 모든 검증자가 확인한 후 ABCI를 통해 application에 넘긴다. 증거를 처리하고 처벌하는 것은 application의 책임이다.

```protobuf
enum EvidenceType {
  UNKNOWN               = 0;
  DUPLICATE_VOTE        = 1;
  LIGHT_CLIENT_ATTACK   = 2;
}
```

중복 투표와 라이트 클라이언트 공격 2가지의 증거 형태가 있다.

## Determinism

ABCI application은 Tendermint 합의에 의해 결정론적인 finite-state machine을 구현해야 한다. 동일한 순서의 요청이 주어지면 모든 노드가 BeginBlock, DeliverTx, EndBlock, Commit 에 대해 동일한 응답을 계산한다. 다음 블록의 헤더에 응답에 포함되므로 이것은 매우 중요하다.

이러한 이유로 애플리케이션은 Tendermint Core와 같은 합의 엔진에 대한 ABCI 연결을 제외하고는 외부 사용자 또는 프로세스에 노출되지 않는 것이 좋다. 애플리케이션은 다른 종류의 요청이 아닌 블록 실행(BeginBlock, DeliverTx, EndBlock, Commit)의 입력을 기반으로 상태를 변경해야 한다. 이것은 모든 노드가 동일한 transaction을 보고 동일한 결과를 계산하도록 하는 유일한 방법이다.

State machine에 비결정성이 있는 경우 노드가 블록 헤더 값에 동의하지 않기 때문에 합의가 실패하게 된다. 비결정성을 수정하고 노드를 다시 시작해야 한다.

#### 비결정성의 원인들

하드웨어 오류

* Cosmic rays, 과열

노드 종속 상태

* 랜덤 숫자
* 시간

Underspecification

* 라이브러리 버전 변경
* Race conditions
* 부동 소수점 숫자
* JSON 직렬화
* hash-tables/maps/dictionaries를 통한 순회

외부 소수

* 파일 시스템
* 네트워크 콜(외주의 REST API service)

더 자세한 설명은 [https://github.com/tendermint/abci/issues/56](https://github.com/tendermint/abci/issues/56) 에서 확인하길 바란다.

몇몇 method들(Query, CheckTx, DeliverTx)는 명시적으로 Info, Log 필드로 비결정적 데이터를 반환한다. 이것들은 블록 헤더 계산에 포함되지 않는 필드들이기 때문에 괜찮다. Response의 모든 다른 필드들은 엄격하게 결정적이어야 한다.

## Block Execution

새로운 블록체인이 처음 시작하면 Tendermint는 InitChain을 실행시킨다. 그 다음, 각 블록에서 다음의 method들이 실행된다: BeginBlock, \[DeliverTx], EndBlock, Commit

DeliverTx는 블록 안의 각각의 transaction에서 실행된다. 그 결과는 application 상태를 업데이트한다. DeliverTx, EndBlock, Commit의 결과는 암호화되어 다음 블록의 헤더에 포함된다.

## State Sync

State sync는 새로운 노드가 과거의 블록들을 모두 재실행하는 것이 아니라 state machine snapshot을 이용해서 빠르게 동기화하도록 한다.

새로운 노드가 P2P 네트워크에서 snapshot을 발견하면 존재하고 있던 노드는 ListSnapshots를 불러서 local state snapshot을 검색한다. 새 노드는 이 snapshot을 OfferSnapshot을 통해 로컬 application에 제공한다.

Application이 snapshot을 수락하고 복원을 시작하면 Tendermint는 LoadSnapshotChunk를 통해 기존 노드에서 스냅샷 청크를 가져와 ApplySnapshotChunk를 사용하여 로컬 애플리케이션에 순차적으로 적용한다. 모든 청크가 적용되면 애플리케이션 AppHash가 Info 쿼리를 통해 검색되고 라이트 클라이언트를 통해 확인된 블록체인의 AppHash와 비교된다.

## Reference

[A Blockchain App Architecture | Cosmos Developer Portal](https://tutorials.cosmos.network/academy/2-main-concepts/architecture.html#checktx)

[spec/abci.md at c939e155a6aab1bfab36479a652d2d8a34a2ba29 · tendermint/spec](https://github.com/tendermint/spec/blob/c939e15/spec/abci/abci.md)
