---
description: 윤주운 개발자님이 코스모스 아카데미에서 코스모스 SDK에 대해 설명하신 내용을 정리해보려고 한다.
---

# 코스모스 SDK

## Why the SDK?

* 이더리움, 이오스 등의 스마트 컨트랙트 플랫폼들은 하나의 체인에서 모든 컨트랙트들이 돌아가는 특성을 가진다. 내가 스마트 컨트랙트를 메인넷에 올리고 다른 사람이 또 다른 스마트 컨트렉트를 올리면 같은 체인에서 돌아간다. SDK는 이와는 다르다. &#x20;
* 기본적으로 하나의 블록체인 어플리케이션이 하나의 블록체인 위에서 돌아가는 것을 원칙으로 갖는다. 컨트랙트는 서로 통신을 필요로 하는 경우가 많다. 예를 들면 Dex 컨트랙트는 돈을 가져오기 위해 은행과 통신할 경우가 많고 토큰 컨트랙트에 은행이 요청을 보낼 수도 있다. 이를 원활하게 하기 위해 기존에는 하나의 블록체인 위에서 동작하던 것인데 텐더민트 합의 알고리즘에 의해서 체인들끼리 소통할 수 있게 되었다.&#x20;
* 성능 향상, 최적화라는 장점을 가져오게 되었다.&#x20;
* 솔리디티의 가장 큰 문제는 불편함이다. 낯선 환경, 낯선 개발자 풀에서 자원을 끌어와야 한다. Go-lang을 통해서 블록체인을 개발할 수 있게 되면서 기존의 모듈들을 사용할 수 있고 개발자 풀도 넓은 장점이 있다.&#x20;

### 모듈러 Development

* 솔리디티를 예로 들면 다른 모듈을 가져와서 사용하기가 매우 어렵다.&#x20;
* 스토리지 접근 권한을 그대로 라이브러리에 넘겨줘야 사용할 수 있는 경우가 많기 때문이다.&#x20;
* 기존에 존재하던 컨트랙트를 한번 더 작성하게 되는 비효율적임이 있다.&#x20;
* SDK는 modulity capability securty 모듈을 통해서 각 모듈이 서로간에 독립된 공간을 가지고 제한된 방식으로 통신할 수 있도록 했다.
* 낯선 제 3자가 작성한 모듈이어도 안전하게 사용 가능하다.&#x20;

## KVStore

* 모든 블록체인은 기본적으로 상태 변조를 위해 존재한다.
* 여러 분산된 기계들간의 공통된 상태를 복제하고 그것이 어긋나지 않도록 유지하는 것이 BFT(Byzantine fault-tolerant algorithms) state machine이다. &#x20;
* SDK에서도 똑같이 KVStore를 사용한다.
* 매우 간단한 메소드만 가지고 있다.&#x20;

```go
type KVStore interface {
    Store
    Get(key []byte) []byte
    Has(key []byte) bool
    Set(key, value []byte)
    Delete(key []byte)
    Iterator(start, end []byte) Iterator
    ReverseIterator(start, end []byte) Iterator
}
```

KVStore 간단한 예제

```go
// store를 컨텍스트로부터 가져온다. 
store := ctx.KVStore(storeKey)
// hello라는 키에 world라는 밸류를 연결시킨다. 
store.Set([]byte("hello"), []byte("world"))
// hello를 가져오면 world가 나옴.
bz := store.Get([]byte("hello")) // == []byte("world")
```

## Keepers

이더리움과 이오스의 컨트랙트라고 볼 수 있는 것.

Go-lang struct

### What is a Keeper?

상태에 의존하지 않는 방식으로 모듈의 기능을 정의하고 저장한다.&#x20;

ex) Bank Keeper

* 토큰 컨트랙트와 비슷
* 누가 얼마만큼의 돈을 가지고 있는지
* 돈을 누구한테서 누구한테 보낼 수 있는지 메소드를 정의하고 저장한다.&#x20;

#### StoreKeys

* Keeper안의 값들을 가져올 수 있는 권한, 열쇠이다.&#x20;

#### Methods

* Keeper는 Go-lang structure이기 때문에 메소드를 가질 수 있다. 일반적인 구조체.

#### Keepers from other modules

* 다른 Keeper도 가질 수 있음.
* 나중에 추가로 설명.

### Nameservice example

앞으로 nameservice 예제를 이용해서 계속 설명 진행.

#### Stores a mapping from domain to value

* mydomain.com이 127.0.0.1에 대응한다라는 테이블을 가지고 있는 것이 nameservice이다.&#x20;
* KVStore와 잘 맞는 예제.
* 도메인에 경매를 해서 값을 치루고 사올 수 있는 기능을 추가로 구현 할 것.

### StoreKeys

#### Immutable Keeper

Keeper를 불변하게 유지하는데 중요한 역할을 담당한다.

Keeper는 상태에 의존하지 않음.\
KVStore 자체를 Keeper의 상태변수로 가지고 있을 수도 있음. 그럴 경우의 문제점은 transaction이 실패했을 때 수수료가 부족하거나 에러가 생겼을 때 tx안의 모든 상태변화가 revert되어야 한다. 만약에 Keeper가 store를 그대로 들고 있었다면 모든 상태 revert작업을 메뉴얼하게 해주어야 하고 모듈러 방식을 헤친다. 그렇게되면 Keeper가 상태 변화를 완전히 담당하고 있기 때문에 권한이 너무 막강해진다.

따라서 Keeper에 오직 key만을 저장한다. Key는 Store를 가지고 올 수 있는 권한을 의미한다. Keeper는 오직 권한만을 소유한다. 그 자체로는 어떠한 store도 들고 있지 않다.&#x20;

```go
type Keeper struct{
    key sdk.StoreKey // The (unexposed) keys used to access the store from the Context.
}

// 특정 도메인이 어떤 IP주소로 연결되는지 가져옴. 
func (k Keeper) GetValue(ctx sdk.Context, domain string) string {
    // Keeper의 key를 통해서 KVStore를 가져옴. 
    // 자기가 가지고 있는 영역 내의 store에만 접근할 수 있음.
    // 서로간의 영역을 침범하지 않음으로써 안전한 환경 제공.
    store := ctx.KVStore(k.Key)
    bz := store.Get([]byte(domain))
    return string(bz)
}

// Value를 setting.
func (k Keeper) setValue(ctx sdk.Context, domain string, val string) {
    store := ctx.KVStore(k.key)
    // domain이 val에 대응되도록 setting.
    store.Set([]byte(domain), []byte(val))
}
```

```go
k.setValue(ctx, "mydomain.com", "127.0.0.1")
val := k.GetValue(ctx, "mydomain.com") // == "127.0.0.1"
```

![](<../.gitbook/assets/image (13).png>)

Keeper는 오직 키만을 가지고 있고 이는 context의 KVStore로 전달이 된다.\
이 리턴값은 하나의 sdk.KVStorer가 되며 Get, Set 메소드를 가지고 있다.&#x20;

![](<../.gitbook/assets/image (14).png>)

상태가 존재하는 영역과 상태가 존재하지 않는 영역을 명확하게 구분할 수 있다.&#x20;

Keeper는 불변한다. struct이지만 블록체인이 쓰이는 동안 절대로 변경되지 않는다. 가능한 상태변경의 방법만을 기술한다.&#x20;

모든 가능한 상태는 context서 가져올 수 있다. Tx, 메세지가 실행될 때마다 context는 바뀌게 된다. Keeper의 영역 밖에서 우리가 core에서 정의한 방법대로 상태는 변경된다.&#x20;

#### Private Store

StoreKey는 독립된 store를 유지하는데 도움을 준다.&#x20;

각 모듈은  자신이 가지고 있는 키를 통해서만 모듈을 가지고 올 수 있기 때문에 서로간의 영역이 침범되지 않는다. 이는 object capability security라고 불린다. 키를 소유하고 있어야만 해당 스토어에 접근할 수 있다.&#x20;

다른 Keeper가 자신의 store에 접근하고 싶어하면 자신의 key를 그 Keeper에게 넘겨줄 수 있다. 넘겨주지 않는 이상은 접근이 불가하다.&#x20;

```go
type Keeper struct{
    key sdk.StoreKey // The (unexposed) keys used to access the store from the Context.
}

// 특정 도메인이 어떤 IP주소로 연결되는지 가져옴. 
func (k Keeper) GetValue(ctx sdk.Context, domain string) string {
    // Keeper의 key를 통해서 KVStore를 가져옴. 
    // 자기가 가지고 있는 영역 내의 store에만 접근할 수 있음.
    // 서로간의 영역을 침범하지 않음으로써 안전한 환경 제공.
    store := ctx.KVStore(k.Key)
    bz := store.Get([]byte(domain))
    return string(bz)
}

// Reputation Keeper
// 예시를 위한 Keeper.
// 하나의 domain이 갖고 있는 평점 또는 접속자 수를 뜻한다고 하자. 
type RepKeeper struct{
    key sdk.StoreKey
    // string, byte[]는 쉽게 Get과 Set에 쓰여있는대로 형변환 할 수 있지만,
    // 다른 타입(int, boolean) 등은 그렇지 않다. 
    // 우리가 별도로 byte[]로 인코딩, 디코딩해주어야 한다. 
    cdc *codec.Codec
}

// domain과 reputation (정수)가 연결되어 있다. 
func (k RepKeeper) GetRep(ctx sdk.Context, domain string)(rep uint64) {
    store := ctx.KVStore(k.key)
    bz := store.Get([]byte(domain))
    // byte[]로부터 실제 값을 해독을 함. 해독한 값을 rep로 옮김.
    k.cdc.UnmarshalBinary(bz, &rep)
    return
}
```

![](<../.gitbook/assets/image (15).png>)

Keeper와 RepKeeper의 storeKey가 다르기 때문에 Context는 서로 다른 KVStore를 반환해준다. 이는 명확하게 공간을 나누어주면 KVStore가 StoreKey에만 dependent하게 만들어준다. 이로 서로간의 영역이 침범될 수 없는 상황이 된다.&#x20;

### Methods

```go
// 어떤 domain이 갖고 있는 값들을 가져와서 모두 대문자로 바꾸어 저장하는 메소드. 
// Keeper의 메소드들은 상태를 변조할수도 있고 하지 않을 수도 있다. 
func (k Keeper) Capitalize(ctx sdk.Context, domain string) {
    val := k.GetValue(ctx, domain)
    newVal := capitalize(val)
    k.SetValue(ctx, domain, newVal)
}

// 특정 domain의 value를 삭제해버림. 
// name이 아니라 domain이 들어가야 하는 것 아닌가??
func (k Keeper) delete(ctx sdk.Context, name string){
    k.SetValue(ctx, name, []byte{})
}
```

### Keepers from Other Modules

Keeper는 다른 모듈로부터 Keeper를 가져올 수 있음.&#x20;

```go
// 가장 많이 쓰이게 될 Keeper
type BankKeeper interface {
    // 전송
    SendKeeper
    // 토큰 추가
    AddCoins(ctx sdk.Context, addr sdk.AccAddress, amt sdk.Coins)(sdk.Coins, sdk.Tags, sdk.Error)
    // 토큰 차감
    SubtractCoins(ctx sdk.Context, addr sdk.AccAddress, amt sdk.Coins)(sdk.Coins, sdk.Tags, sdk.Error)
    // 토큰 정
    SetCoins(ctx sdk.Context, addr sdk.AccAddress, amt sdk.Coins) sdk.Error
}

type Keeper struct{
    key sdk.StoreKey
    bk BankKeeper
}

// buyer가 domain을 구매하고 buyer의 계좌에서 토큰을 차감한다.
func (k Keeper) BuyDomain(ctx sdk.Context, domain string, val string, buyer sdk.AccAddress, bid sdk.Coins){
    k.SetValue(ctx, name, val)
    // bankKeeper가 우리에게 접근하도록 허용한 method들을 통해서만 context의 상태를 변조할 수 있다. 
    k.bankKeeper.SubtractCoins(ctx, buyer, bid)
}
```

## Transactions

Transaction이 블록체인에 들어와서 실행되는 방식으로 유저들은 interaction한다.&#x20;

이더리움이나 다른 transaction과는 약간 다른 점이 있다.&#x20;

### Users interacting with the chain

* StdTx
  * Standard Transaction을 먼저 정의함.
  * 곧 AnteHandler로 넘어감.
* &#x20;AnteHandler
  * Ante는 포커용어에서 나온 것. 게임 시작 전에 의무적으로 넣는 돈을 의미한다. 판돈. 취소할 수 없음.
  * Transaction을 날릴 때 최소한의 노, 최소한의 수수료는 한번 차감되고 다시는 회복되지 않는다.&#x20;
  * Transaction 실행에 가장 기본적인 operation들.
    * 서명확인
    * 수수료 차감
    * 시퀀스 확인
* Msgs
  * AnteHandler가 올바르다고 신호를 보내면 Message들이 차례차례 실행된다.&#x20;
* Handlers
  * Msgs는 Handler들에 의해 실행된다.&#x20;

#### StdTx

본질적으로는 메세지의 나열이다.&#x20;

이더리움에서는 contract에 ERC20토큰을 보내는 것이 굉장히 불편하다. 토큰 contract에 이 contract이 토큰을 가져갈 수 있다는 것을 허용해주어야 하고 그 다음 실제로 그 contract가 내 계좌의 토큰을 꺼내가도록 해야 함.  즉 2번의 transaction이 연관된다. &#x20;

이러한 귀찮음을 방지하기 위해 한 transaction에 여러가지 Message가 포함될 수 있도록 했다.&#x20;

그 외에는 당연한 수수료, 서명, 추가적인 메모 같은 것들이 존재.

```go
// A standard way to wrap a Msg
// with a Fee and Signatures.
type StdTx struct {
    Msgs          []sdk.Msg         `json:"msgs"`
    Fee           StdFee            `json:"fee"`
    Signatures    []StdSignature    `json:"signatures"`
    Memo          string            `json:"memo"`
}
```

#### AnteHandler

Transaction의 기본적인 적합성을 검증한다.&#x20;

* 서명
* Sequence Numbers
* Gas에 충분한 수수료가 첨부가 되어 있는지 (코스모스는 이더리움과 같은 가스 모델을 사용한다.)

적합성을 검증하면 Msg들을 적절한 Handler에 넘겨준다.&#x20;

#### Msgs

Type, ValidateBasic, GetSignBytes, GetSigners는 메세지가 실제로 실행되기 이전 적합성 검증에 도움을 주는 필드들이다.&#x20;

* Type
  * 이 메세지가 어느 handler에 의해 실행되어야 하는지 알려준다. &#x20;
* ValidateBasic
  * &#x20;기본적인 검증을 진행.&#x20;
* GetSignBytes
  * Msg를 byte\[]로 바꾼다.
* GetSigners&#x20;
  * 메세지가 실행되기 위해 누구의 사인이 필요한가를 검증한다. \
    Ex) A가 B에게 토큰을 전송하는 메세지를 첨부했다. -> A의 사인을 요구함.

```go
// Transactions messages must fulfill the Msg
type Msg interface {
    // Return the message type.
    // Must be alphanumeric or empty.
    Type() string
    
    // validateBasic does a simple validating check that
    // doesn't require access to any other information.
    ValidateBasic() Error
    
    // Get the Canonical byte representation of the Msg.
    GetSignBytes() []byte
    
    // Signers returns the addrs of signers that must sign.
    // CONTRACT: All signatures must be present to be valid.
    // CONTRACT: Returns addrs in some deterministic order.
    GetSigners() []AccAddress
}
```

#### Example Msg

```go
// Domain을 얼마(Bid)에 누가(Buyer) 사겠다.
// Domain이 가리키는 값은 무엇(Value)으로 설정하겠다.
type MsgBuyDomain struct {
    Domain string
    Value string
    Bid sdk.Coins
    Buyer sdk.AccAddress
}

// nameservice라는 type.
func (msg MsgBuyDomain) Type() string {
    return "nameservice"
}

func (msg MsgBuyDomain) GetSignBytes() []byte {
    b, err := json.Marshal(msg)
    if err != nil {
        panic(err)
    }
    return sdk.MustSort.JSON(b)
}

// 정말 기본적인 것만 검증함. 
// 빈 문자열인가?
func (msg MsgBuyDomain) ValidateBasic() sdk.Error {
    msg.Buyer.Empty() {
        return sdk.ErrInvalidAddress(msg.Buyer.String())
    }
    return nil
}

func (msg MsgBuyDomain) GetSigners() []sdk.AccAddress {
    return []sdk.AccAddress(msg.Buyer)
}
```

#### Handler

* 메세지의 type에 의거하여 메세지가 어떤 식으로 상태를 변환해야 하는지를 정의한다.&#x20;
* 같은 type의 메세지이면 같은 handler로 온다.&#x20;
* Keeper를 가지고 Msg와 조합하여 context에 상태를 변환하는 명령을 내린다.&#x20;
* 최종적으로 돌려주는 것은 sdk result다.&#x20;

```go
// Result is the union of ResponseDeliverTx and ResponseCheckTx.
type Result struct {
    // Code is the response code, is stored back on the chain.
    Code ABCICodeType
    
    // 특정한 값을 돌려줘야 하는 Tx라면 어떤 값을 돌려주겠다. 
    // Data is any data returned from the app.
    Data []byte
    
    // 에러가 있다면 어떤 에러가 만들어졌다. 
    // Log is just debug information. NOTE: nondeterministic.
    Log string
    
    // Gas를 얼마나 원했다. 
    // GasWanted is the maximum units of work we allow this tx to perform.
    GasWanted int64
    
    // Gas를 얼마나 썼다. 
    // GasUsed is the amount of gas actually consumed. NOTE: unimplemented
    GasUsed int64
    
    // 수수료를 얼마나 썼다. 
    // Tx fee amount and denom.
    FeeAmount int64
    FeeDenom string
    
    // Tags are used for transaction indexing and pubsub.
    Tags Tags
}
```

Transaction이 실행되고 나서 사용자들에게 돌려줘야 할 값들을 정의하고 있다.&#x20;

#### Example Handler

```go
func NewHandler(k Keeper) sdk.Handler {
    return func(ctx sdk.Context, msg sdk.Msg)
    sdk.Result{
        // 지금 우리는 하나의 메세지만 보고 있지만 실제로는 여러 타입의 메세지가 있을 수 있다. 
        // 보통 switch를 이용해서 각 type의 메세지들을 처리한다. 
        switch msg := msg(type){
            case MsgBuyDomain:
                return handleMsgBuyDomain(ctx, k, msg)
        }
    }
}

// 아까는 Keeper에 값을 어떻게 가져올 것인가, 값을 어떻게 setting할 것인가 그런 아주 기본적인 것들만 정의했다.
// Handler는 그것을 실제로 어떻게 활용을 해서 이 메세지를 해석할 것인가를 정의한다. 
func handleMsgBuyDomain(ctx sdk.Context, k Keeper, msg MsgBuyDomain) sdk.Result{
    // 돈을 적게 냈다면 에러를 리턴한다. 
    if(msg.Bid < k.GetPrice(ctx, msg.Domain)){
        return ErrInsufficientCoins("Not enough").Result()
    }
    // 기존에 이 domain을 누가 가지고 있었는
    currentOwner := k.GetOwner(ctx, msg.Domain)
    k.bk.SendCoins(ctx, msg.Buyer, currentOwner.msg.Bid)
    k.SetValue(ctx, msg.Domain, msg.Value)
    k.SetPrice(ctx, msg.Domain, msg.Bid)
    return sdk.Result{}
}
```

### Transaction의 흐름

1. &#x20;StdTx는 사용자의 수수료 또는 Signature 같은 정보와 여러 메세지로 이루어져있다.&#x20;
2. AnteHandler에서 검증을 거침.
3. 검증이 실패하면 Tx는 폐기되고 실행되지 않은 상태로 돌아감. 물론 수수료는 차감.
4. 검증 성공하면 라우터로 들어감.
5. 라우터는 Type에 따라 메세지를 handler에 매칭시켜줌.
6. Handler는 코드에 따라 해야 할 일을 함.

## Module review

### Keeper

* 앱의 기능을 정의하기 위해 필요하다.
* Store를 기초적으로 어떻게 변조할 수 있을지를 나타낸다.&#x20;
* 그것은 Keeper가 상태를 변조할 수 있는 방법을 제약한다.&#x20;
* 안정성을 확보할 수 있고 다른 Keeper들과 상호작용했을 때 예측하지 못한 경우를 상당히 줄일 수 있다.

### Transaction

* 유저가 블록체인과 어떻게 상호작용하는지 정의한다.&#x20;

## Reference

{% embed url="https://www.youtube.com/watch?v=ZD7xk1SfdBw&t=3s" %}

