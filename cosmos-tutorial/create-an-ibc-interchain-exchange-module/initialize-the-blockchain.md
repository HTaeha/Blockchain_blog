# Initialize the Blockchain

## Install Starport

```bash
curl <https://get.starport.network/starport@v0.16.2>! | bash
```

## Create the Blockchain

```bash
starport scaffold chain github.com/username/interchange --no-module
cd interchange
```

## Create the Module

```bash
starport scaffold module ibcdex --ibc --ordering unordered
```

나는 여기서 the module name can't be prefixed with ibc because of potential store key collision 라는 에러가 발생했다.

모듈 이름을 ibc로 시작하면 안 된다고 해서 myibcdex로 모듈 이름을 바꿨다.

starport 버젼은 0.18.4이다.

## Create the Transaction Types

```bash
starport scaffold map sellOrderBook amountDenom priceDenom --no-message --module ibcdex
starport scaffold map buyOrderBook amountDenom priceDenom --no-message --module ibcdex
```

sellOrderBook, buyOrderBook 2개의 transaction type을 생성한다.

* `amountDenom` 은 어떤 토큰이 팔리고 얼만큼 팔리는지 나타낸다.
* `priceDenom` 토큰의 판매 가격

## Create the IBC Packets

* `createPair` : order book pair
* `sellOrder` : 판매 주문
* `buyOrder` : 구매 주문

`—ack` flag는 target 체인에서 패킷을 받은 후 반환하는 필드 이름과 acknowledgment의 타입을 정의한다. `—ack`의 값은 콤마로 구분되고 : 뒤에 타입을 지정할 수 있다.

## Cancel messages

```bash
starport scaffold message cancel-sell-order port channel amountDenom priceDenom orderID:int --desc "Cancel a sell order" --module ibcdex
starport scaffold message cancel-buy-order port channel amountDenom priceDenom orderID:int --desc "Cancel a buy order" --module ibcdex
```

취소 주문은 packet을 보낼 필요가 없고 local에서 처리하면 된다.

`—desc` flag는 CLI command의 설명을 정의한다.

## Trace the Denom

Token denom은 `ibc-transfer` 모듈과 같은 동작을 해야 한다.

* 체인에서 받은 외부 토큰에는 `voucher`라고 하는 고유한 `denom`이 있다.
* 토큰이 블록체인으로 전송된 다음 다시 전송 및 수신되면 체인은 바우처를 해결하고 이를 다시 원래 토큰 액면가로 변환할 수 있다.

`Voucher` 토큰은 해시로 표시되므로 바우처와 관련된 원래 액면가를 저장해야 하며 색인화된 유형으로 이를 수행할 수 있다. `voucher`에 저장할 것 : source port ID, source channel ID, 원래 가격

```bash
starport scaffold map denomTrace port channel origin --no-message --module ibcdex
```

## Create the Configuration

두 블록체인 네트워크를 테스트 하기 위해 `mars.yml`, `venus.yml` 을 추가한다. `interchange` 폴더에 config 파일을 추가해라.

Mars : `mcx`, `marscoin`

Venux : `vcx`, `venuxcoin`

```bash
# mars.yml
accounts:
  - name: alice
    coins: ["1000token", "100000000stake", "1000mcx"]
  - name: bob
    coins: ["500token", "1000mcx", "100000000stake"]
validator:
  name: alice
  staked: "100000000stake"
faucet:
  name: bob
  coins: ["5token", "100000stake"]
genesis:
  chain_id: "mars"
init:
  home: "$HOME/.mars"
```

```bash
# venus.yml
accounts:
  - name: alice
    coins: ["1000token", "1000000000stake", "1000vcx"]
  - name: bob
    coins: ["500token", "1000vcx", "100000000stake"]
validator:
  name: alice
  staked: "100000000stake"
faucet:
  host: ":4501"
  name: bob
  coins: ["5token", "100000stake"]
host:
  rpc: ":26659"
  p2p: ":26658"
  prof: ":6061"
  grpc: ":9091"
  api: ":1318"
  frontend: ":8081"
  dev-ui: ":12346"
genesis:
  chain_id: "venus"
init:
  home: "$HOME/.venus"
```
