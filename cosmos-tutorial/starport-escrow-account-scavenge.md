---
description: Starport 에서 진행한 Scavenge 튜토리얼
---

# \[Starport] Escrow Account: Scavenge

## Introduction

이 튜토리얼에서 scavenger hunt game을 위한 blockchain을 만든다.

배울 점

* CLI command에서 custom logic을 구현한다.
* 토큰을 저장하는 escrow 계좌를 사용한다.

## The Scavenger Hunt Game

* 누구나 문제를 암호화 된 답과 함께 올릴 수 있다.
* 문제는 coin과 함께 등록된다.
* 누군가 그 문제에 대한 답을 올려서 맞으면 함께 등록된 coin을 받는다.

### Front Running

1. 당신이 문제에 대한 답을 올렸다.
2. 누군가 당신의 답을 보고 당신 바로 앞에 답을 올렸다.
3. 당신보다 당신의 답을 보고 올린 사람이 먼제 게시했기 때문에 보상을 받는다.

대기 시간이 있는 공용 네트워크에서는 front running 문제가 일어날 수 있다. 이를 방지하기 위해 commit-reveal 방식을 구현한다.

### Commit-reveal

단일 상호작용을 2개의 안전한 상호 작용으로 바꾼다.

1. 첫 번째 상호작용은 commit이다. 답변은 다음 상호작용에서 게시하기로 약속하고 이 commit에서는 답변과 이름을 결합한 암호화 hash를 등록한다.
2. 두 번째 상호작용에서 답변을 게시한다(reveal). Application은 이름과 답변을 가져와서 이전 상호작용에서 commit한 값과 일치하는지 확인한다.

## Scaffolding

starport v0.18.3

```jsx
starport scaffold chain github.com/coreators/scavenge --no-module
```

* scavenge 비어있는 Cosmos SDK가 있는 blockchain 을 생성.
* —no-module flag 를 적용하면 module scaffolding을 스킵한다.

```jsx
starport scaffold module scavenge --dep bank
```

* scavenge라는 module을 생성한다.
* scavenge module이 참여자간에 token을 전송해야하기 때문에 bank module을 dependency로 사용한다.
* —dep flag를 적용하면서 —dep 뒤에 원하는 module이름을 적으면 해당 module의 keeper가 scaffold하는 module keeper에 등록된다.

## Messages

Message는 application이 할 수 있는 행동을 정의하기 때문에 module 구성을 시작할 때 좋다. 유저가 application의 상태를 변경할 수 있는 모든 시나리오를 생각해보자.

Scavenge module은 3가지 메시지를 포함한다.

* Submit scavenge
* Commit solution
* Reveal solution

### Submit Scavenge Message

Submit scavenge message는 scavenge를 생성할 때 필요한 정보를 포함하고 있어야 한다.

* Description
  * 문제에 대한 설명
* Solution hash
  * 문제의 답
* Reward
  * 문제를 맞출 때 가져갈 수 있는 reward

```jsx
starport scaffold message submit-scavenge solutionHash description reward
```

* proto/scavenge/tx.proto
  * MsgSubmitScavenge, MsgSubmitScavengeResponse proto message를 추가. SubmitScavenge RPC를 Msg service에 등록.
* x/scavenge/types/message\_submit\_scavenge.go
  * Msg interface의 method들이 정의됨.
* x/scavenge/handler.go
  * MsgSubmitScavenge message가 등록됨.
  * 해당 메세지가 들어오면 handler에서 msgServer, keeper에 정의된 함수를 실행함.
* x/scavenge/keeper/msg\_server\_submit\_scavenge.go
  * SubmitScavenge keeper method가 정의됨.
  * 실제 작동하는 내용이 정의되는 부분.
* x/scavenge/client/cli/tx\_submit\_scavenge.go
  * CLI command가 추가됨.
  * submit-scavenge command를 실행해서 transaction을 broadcast하는 부분이 적힘.
* x/scavenge/client/cli/tx.go
  * CLI command가 등록됨.
  * 위에서 정의한 tx\_submit\_scavenge의 내용을 등록.
* x/scavenge/types/codec.go
  * codec이 등록됨.

x/scavenge/types/message\_submit\_scavenge.go 를 보면 message는 sdk.Msg interface를 따른다는 것을 알 수 있다.

message는 처음 scaffold할 때 지정한 Description, SolutionHash, Reward와 자동으로 생성된 Creator의 정보를 포함하고 있다.

msg는 Creator에 의해 signed and submitted 된다.

### Commit Solution Message

* Solution hash
  * 정답 hash
* Solution scavenger hash
  * 정답과 이름을 섞은 hash

### Reveal Solution Message

* Solution
  * 정답

scaffold message를 했을 때 생성되는 것들은 submit scavenge message와 같다.

## Types

Message를 정의했으니 type과 method를 구현해야 한다.

Keeper method로 CRUD function이 정의된다.

Scavenge blockchain에는 scavenge와 commit type이 필요하다.

map 형태의 자료구조로 scavenge와 commit type을 정의할 것이다.

### Scavenge

첫 매개변수가 type의 이름이다. 여기서는 scavenge.

나머지가 필드들이다.

기본적으로 CRUD 메시지들이 생성되는데 메시지가 이미 있을 경우 —no-message flag를 적용하면 message 생성을 스킵한다.

```bash
starport scaffold map scavenge solutionHash solution description reward scavenger --no-message
```

* proto/scavenge/scavenge.proto
  * Scavenge type이 정의된다.
* proto/scavenge/query.proto
  * Query service에 등록되고 proto message가 정의된다.
* proto/scavenge/genesis.proto
  * blockchain state를 export하기 위한 type.
  * 예를 들면 software 업그레이드할 때
* x/scavenge/keeper/grpc\_query\_scavenge.go
  * query를 위한 keeper method
* x/scavenge/keeper/scavenge.go
  * store로부터 scavenge를 get, set, remove하는 keeper method를 정의
* x/scavenge/client/cli/query\_scavenge.go
  * blockchain을 query하기 위한 CLI command
  * list-scavenge, show-scavenge 가 정의되어 있다.
* x/scavenge/client/cli/query.go
  * CLI command 등록.
* x/scavenge/types/keys.go
  * state에 scavenge를 저장할 때 쓰이는 prefix key
* x/scavenge/genesis.go
  * exporting을 위한 logic
* x/scavenge/types/genesis.go
  * genesis file을 검증하기 위한 logic
* x/scavenge/module.go
  * gRPC gateway route를 등록

### Commit

```bash
starport scaffold map commit solutionHash solutionScavengerHash --no-messagee
```

## Handlers

Message가 Keeper에 도달하려면 Handler를 통해 이동한다.

MVC architecture에 비유하면 Keeper가 Model, Handler가 Controller이다.

React/Redux에 비유하면 Keeper가 Reducer/Store이고 Handler가 Action이다.

handler는 x/scavenge/handler.go에 정의된다.

이번의 경우 3개의 message type이 handler에 추가되었다.

* MsgSubmitScavenge
* MsgCommitSolution
* MsgRevealSolution

## Keeper

### Create Scavenge

* 정답 hash가 존재하지 않는지 체크
* 토큰을 scavenge creator 계좌에서 module 계좌로 보내기
* scavenge를 store에 쓰기

moduleAcct은 public key pair에 의해 컨트롤 되는 것이 아닌 module자체의 계좌이다. 문제가 풀릴 때까지 reward를 보관하고 있는다.

SubmitScavenge는 bank module로부터 SendCoins method를 사용한다.

### Commit Solution

* commit의 hash가 store에 존재하지 않는지 체크
* 새로운 commit 을 store에 작성

### Reveal Solution

* commit의 hash가 store에 존재하는지 체크
* scavenge의 solution hash가 store에 존재하는지 체크
* scavenge가 이미 풀리지 않았는지 체크
* module account에서 문제를 푼 계좌로 토큰 전송
* 업데이트된 scavenge를 store에 작성

## CLI

* x/scavenge/client/cli/tx.go
  * state를 업데이트할 message를 포함하는 transaction을 만든다
* x/scavenge/client/cli/query.go
  * state로부터 정보를 읽는 능력을 제공하는 query를 만든다.

### Commit Solution

sha256 라이브러리로 plain text를 해싱한다. 그로 인해 정답 유출을 막을 수 있다.

## Play With Your Blockchain

### Starting the Blockchain

```bash
starport chain serve
```

### Creating a Scavenge

```bash
scavenged tx scavenge submit-scavenge "A stick" "What's brown and sticky?" 100token --from alice
```

문제 : What's brown and sticky?

정답 : A stick

리워드 : 100토큰

출제자 : alice

### Querying For a List of Scavenges

```bash
scavenged q scavenge list-scavenge --output json
```

scavenge list 출력

### Committing a solution

```bash
scavenged tx scavenge commit-scavenge "A stick" --from bob
```

### Querying For a List of Commits

```bash
scavenged q scavenge list-commit --output json
```

commit list 출력

### Revealing a Solution

```bash
scavenged tx scavenge reveal-solution "A stick" --from bob
```

## Reference

[Introduction | Starport](https://docs.starport.network/guide/scavenge/)

[https://github.com/HTaeha/scavenge](https://github.com/HTaeha/scavenge)
