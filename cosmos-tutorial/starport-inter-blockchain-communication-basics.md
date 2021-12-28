---
description: Starport 에서 진행한 IBC 튜토리얼
---

# \[Starport] Inter-Blockchain Communication: Basics

이 튜토리얼에서는 어떻게 blockchain간에 packet을 생성하고 보내는지 이해할 것이다.

#### 배울 점

* IBC를 사용해서 blockchain간에 packet을 생성하고 보낸다
* Cosmos SDK와 Starport Relayer를 사용해서 blockchain 사이를 이동한다
* 다른 blockchain에 기초적인 블로그를 게시하고 저장한다

## What is IBC?

IBC module은 두 blockchain 사이에서 기초적인 상호작용이다. IBC module은 packet과 message들이 blockchain에서 수신과 발신이 어떻게 구성되는지 정의한다.

IBC relayer는 IBC가 있는 체인 사이를 연결해준다. 이 튜토리얼에서는 두 blockchain에서 relayer를 어떻게 사용하는지 배운다.

이 튜토리얼은 module, IBC packets, relayer와 IBC에 라우팅된 packet들의 lifecycle을 다룬다.

## Create a Blockchain

다른 blockchain에 포스트를 작성하는 blog module을 가진 blockchain app을 생성한다. Hello Mars, Hello Cosmos, Hello Earth message를 작성할 것이다.

* 체인들은 IBC를 이용해서 서로 post를 보낼 수 있다
* Sending chain에서는 acknowledged와 timed out 포스트를 저장한다.

Transaction을 받는 체인에게 승인이 된 후에 양쪽 blockchain에서 post가 저장되었다는 것을 알 수 있다.

* 발신 chain은 추가적으로 postID 데이터를 가진다.

![](<../.gitbook/assets/image (31) (1).png>)

1. 제목과 내용을 담은 post packet을 보낸다.
2. 만약에 Mars에 제 시간에 도착하지 못하면 TimedoutPosts에 추가된다.
3. 포스트를 받는다.
4. postID를 포함한 Ack를 보내준다.
5. Ack를 받으면 SentPosts에 추가한다. (제대로 보낸 것 확인 완료)

## Build your Blockchain App

```bash
starport scaffold chain github.com/cosmonaut/planet --no-module
```

Blockchain 생성.

## Scaffold the blog module inside your blockchain

```bash
starport scaffold module blog --ibc
```

blog module을 생성한다.

IBC module logic도 함께 생성된다.

## Generate CRUD actions for types

* blog post 생성

```bash
starport scaffold list post title content --module blog
```

* 인증된 sent posts 생성

```bash
starport scaffold list sentPost postID title chain --module blog
```

* post timeout 관리

```bash
starport scaffold list timedoutPost title chain --module blog
```

## Starport Scaffold List Command Overview

```bash
starport scaffold list [typeName] [field1] [field2] ... [flags]
```

첫 매개변수는 type의 이름을 말한다. blog 에서는 post, sentPost, timedoutPost 3개의 type을 생성했다.

다음 매개변수는 type 과 관련된 field들이다.

—module flag는 어떤 module에 새로운 transaction type이 추가될 지 지정한다. module이 여러개일 때 사용하고 지정하지 않으면 repo의 이름과 매칭되는 module에 생성된다.

새로운 type이 생성되면 기본적으로 user에 의해 생성될 수 있는 CRUD작업의 message들이 생성된다. —no-message flag를 설정하면 message가 생성되지 않는다.

## Scaffold a sendable and interpretable IBC packet

제목과 내용을 담고 있는 packet을 생성해야 한다.

`starport packet` command가 다른 blockchain으로 보내질 수 있는 IBC packet logic을 생성한다.

* 제목과 내용이 타겟 체인에 저장되어 있다.
* sending chain에 postID가 인증된다.

```bash
starport scaffold packet ibcPost title content --ack postID --module blog
```

ibcPost의 필드가 post type 과 일치한다.

* —ack flag는 어떤 identifier가 리턴될지 정의한다.
* —module flag는 packet이 어떤 module에 생성될 지 지정한다.

## Modify the Source Code

### Add creator to the blog post packet

proto file에서 IBC packet 구조를 정의해보자.

포스트의 creator를 정의하기 위해 packet에 creator field를 추가한다.

```protobuf
// planet/proto/blog/packet.proto
message IbcPostPacketData {
    string title = 1;
    string content = 2;
    string creator = 3; // < ---
}
```

수신 체인에 블로그 post creator에 대한 콘텐츠가 있는지 확인하려면 이 값을 IBC 패킷에 추가하라(packet.Creator). sender의 내용은 SendIbcPost 메시지에 자동으로 포함된다. sender가 메시지의 서명자로 확인되므로 IBC를 통해 보내기 전에 새 패킷의 작성자로 msg.Sender를 추가할 수 있다.

```go
// x/blog/keeper/msg_server_ibc_post.go
// Construct the packet
    var packet types.IbcPostPacketData
    packet.Title = msg.Title
    packet.Content = msg.Content
    packet.Creator = msg.Creator // < ---
    // Transmit the packet
    err := k.TransmitIbcPostPacket(
        ctx,
        packet,
        msg.Port,
        msg.ChannelID,
        clienttypes.ZeroHeight(),
        msg.TimeoutTimestamp,
    )
```

### Receive the post

주요한 transaction logic은 planet/x/blog/keeper/ibcPost.go 파일에 들어있다.

* TransmitIbcPostPacket
  * IBC로 packcet을 보낼 때 불린다.
  * 이 메소드는 packet이 다른 블록체인 앱에 IBC로 보내지기 전 logic을 정의한다.
* OnRecvIbcPostPacket
  * 체인에서 packet을 받을 때 자동으로 불린다.
  * 이 메소드는 packet 수신 logic을 정의한다.
* OnAcknowledgementIbcPostPacket
  * 보낸 packet이 승인됐을 때 불린다.
  * 이 메소드는 packet이 수신 됐을 때 logic을 정의한다.
* OnTimeoutIbcPostPacket
  * 보낸 packet이 시간초과 됐을 때 불린다.
  * 이 메소드는 packet이 목표 체인에 전달되지 못 했을 때 logic을 정의한다.

Post 메시지를 받으면 받은 제목과 내용으로 새로운 post를 생성해야 한다.

누가 메시지를 생성했는지 알기 위해 아래 포맷으로 identifier를 설정한다.

$\<portID>-\<channelID>-\<creatorAddress>$

마지막으로 새로 추가된 post의 ID를 리턴하는 AppendPost 함수를 생성한다. Acknowledgment를 통해 이 값을 source chain으로 리턴할 수 있다.

* ctx는 immutable data structure로 transaction의 header data를 가지고 있다. See [how the context is initiated](https://github.com/cosmos/cosmos-sdk/blob/master/types/context.go#L71)
* Identifier 포맷을 미리 정의
* title은 blog post의 Title
* content 는 blog post의 Content

```go
// x/blog/keeper/ibc_post.go
import (
  //...
  "strconv"
)

func (k Keeper) OnRecvIbcPostPacket(ctx sdk.Context, packet channeltypes.Packet, data types.IbcPostPacketData) (packetAck types.IbcPostPacketAck, err error) {
    // validate packet data upon receiving
    if err := data.ValidateBasic(); err != nil {
        return packetAck, err
    }
    id := k.AppendPost(
        ctx,
        types.Post{
            Creator: packet.SourcePort+"-"+packet.SourceChannel+"-"+data.Creator,
            Title: data.Title,
            Content: data.Content,
        },
    )
    packetAck.PostID = strconv.FormatUint(id, 10)
    return packetAck, nil
}
```

### Receive the post acknowledgement

타겟 체인이 post를 받았는지 알기 위해 sentPost를 저장한다.

title과 target을 저장한다.

OnRecvIbcPostPacket이 error를 리텉하면 Acknowledgement\_Error 타입이 set된다.

```go
// x/blog/keeper/ibc_post.go
func (k Keeper) OnAcknowledgementIbcPostPacket(ctx sdk.Context, packet channeltypes.Packet, data types.IbcPostPacketData, ack channeltypes.Acknowledgement) error {
	switch dispatchedAck := ack.Response.(type) {
	case *channeltypes.Acknowledgement_Error:
		// We will not treat acknowledgment error in this tutorial
		return nil
	case *channeltypes.Acknowledgement_Result:
		// Decode the packet acknowledgment
		var packetAck types.IbcPostPacketAck

		if err := types.ModuleCdc.UnmarshalJSON(dispatchedAck.Result, &packetAck); err != nil {
			// The counter-party module doesn't implement the correct acknowledgment format
			return errors.New("cannot unmarshal acknowledgment")
		}
		k.AppendSentPost(
			ctx,
			types.SentPost{
				Creator: data.Creator,
				PostID:  packetAck.PostID,
				Title:   data.Title,
				Chain:   packet.DestinationPort + "-" + packet.DestinationChannel,
			},
		)
		return nil
	default:
		return errors.New("the counter-party module does not implement the correct acknowledgment format")
	}
}
```

### Store information about the timed-out packet

타겟 체인이 받지 못한 post를 timedoutPost에 저장한다. 이 logic은 sentPost와 동일한 포멧이다.

```go
// x/blog/keeper/ibc_post.go
func (k Keeper) OnTimeoutIbcPostPacket(ctx sdk.Context, packet channeltypes.Packet, data types.IbcPostPacketData) error {
	k.AppendTimedoutPost(
		ctx,
		types.TimedoutPost{
			Creator: data.Creator,
			Title:   data.Title,
			Chain:   packet.DestinationPort + "-" + packet.DestinationChannel,
		},
	)
	return nil
}
```

## Use the IBC Modules

이제 다른 블록체인 앱으로 blog post를 보낼 수 있다. 여러 터미널을 켜서 다음 step을 진행해보자.

### Test the IBC modules

한 기기에서 2개의 블록체인 네트워크를 실행시킨다. 두 블록체인은 같은 코드를 사용한다. 각 블록체인은 유니크한 chain ID를 가지고 있다.

하나는 earth이고 하나는 mars이다.

아래의 파일이 필요하다.

```yaml
# earth.yml
accounts:
  - name: alice
    coins: ["1000token", "100000000stake"]
  - name: bob
    coins: ["500token", "100000000stake"]
validator:
  name: alice
  staked: "100000000stake"
faucet:
  name: bob
  coins: ["5token", "100000stake"]
genesis:
  chain_id: "earth"
init:
  home: "$HOME/.earth"
```

```yaml
# mars.yml
accounts:
  - name: alice
    coins: ["1000token", "1000000000stake"]
  - name: bob
    coins: ["500token", "100000000stake"]
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
  grpc: ":9092"
  grpc-web: ":9093"
  api: ":1318"
  frontend: ":8081"
  dev-ui: ":12346"
genesis:
  chain_id: "mars"
init:
  home: "$HOME/.mars"
```

earth와 mars를 실행한다.

```bash
starport chain serve -c earth.yml
```

```bash
starport chain serve -c mars.yml
```

### Remove Existing Relayer and Starport Configurations

사용하고 있던 relayer가 있으면 제거한다.

```bash
rm -rf ~/.starport/relayer
```

### Configure and start the relayer

Relayer를 설계한다.

```bash
starport relayer configure -a \\
--source-rpc "<http://0.0.0.0:26657>" \\
--source-faucet "<http://0.0.0.0:4500>" \\
--source-port "blog" \\
--source-version "blog-1" \\
--source-gasprice "0.0000025stake" \\
--source-prefix "cosmos" \\
--source-gaslimit 300000 \\
--target-rpc "<http://0.0.0.0:26659>" \\
--target-faucet "<http://0.0.0.0:4501>" \\
--target-port "blog" \\
--target-version "blog-1" \\
--target-gasprice "0.0000025stake" \\
--target-prefix "cosmos" \\
--target-gaslimit 300000
```

earth blockchain이 source가 되고 mars가 target이 되는 relayer가 설계되었다.

output

```bash
---------------------------------------------
Setting up chains
---------------------------------------------

🔐  Account on "source" is "cosmos1xcxgzq75yrxzd0tu2kwmwajv7j550dkj7m00za"

 |· received coins from a faucet
 |· (balance: 100000stake,5token)

🔐  Account on "target" is "cosmos1nxg8e4mfp5v7sea6ez23a65rvy0j59kayqr8cx"

 |· received coins from a faucet
 |· (balance: 100000stake,5token)

⛓  Configured chains: earth-mars
```

relayer를 시작한다.

```bash
starport relayer connect
```

Result

```bash
🔌  Linked chains with 1 paths.

---------------------------------------------
Chains by paths
---------------------------------------------

earth-mars:
    earth > (port: blog) (channel: channel-0)
    mars  > (port: blog) (channel: channel-0)

---------------------------------------------
Listening and relaying packets between chains...
---------------------------------------------
```

### Send packets

earth의 Alice가 mars에게 post를 보낸다.

```bash
planetd tx blog send-ibc-post blog channel-0 "Hello" "Hello Mars, I'm Alice from Earth" --from alice --chain-id earth --home ~/.earth
```

mars의 post list를 본다.

```bash
planetd q blog list-post --node tcp://localhost:26659
```

timeout을 테스트하기 위해 timeout을 1 nanosecond로 설정하고 packet을 보낸다.

```bash
planetd tx blog send-ibc-post blog channel-0 "Sorry" "Sorry Mars, you will never see this post" --from alice --chain-id earth --home ~/.earth --packet-timeout-timestamp 1
```

Mars에서 Earth로 보내기

```bash
planetd tx blog send-ibc-post blog channel-0 "Hello" "Hello Earth, I'm Alice from Mars" --from alice --chain-id mars --home ~/.mars --node tcp://localhost:26659
```

## Congratulations

이 튜토리얼을 끝내면 IBC module과 블록체인 앱을 만들고 IBC를 사용하는 법을 배운 것이다.

이 튜토리얼에서 달성한 것:

* IBC module을 포함한 Hello World blockchain app을 만들기
* 생성된 CRUD action logic을 수정하기
* 두 블록체인을 연결하는 relayer를 사용하기
* 서로 다른 블록체인간에 IBC packet을 전송하기

## Reference

[Inter-Blockchain Communication: Basics | Starport](https://docs.starport.network/guide/ibc.html)

[https://github.com/HTaeha/planet](https://github.com/HTaeha/planet)
