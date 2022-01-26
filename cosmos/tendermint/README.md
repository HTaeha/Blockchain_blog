---
description: Cosmos에서 합의 알고리즘과 네트워킹 부분을 담당하고 있는 텐더민트에 대해 알아보자.
---

# Tendermint

텐더민트는 Cosmos 블록체인의 합의 엔진이다. Cosmos가 블록체인으로 동작할 수 있도록 해준다. Cosmos 블록체인의 특징, 성능, 한계는 텐더민트 합의 엔진의 특징, 성능, 한계를 따라간다.

텐더민트 코어는 두 개의 프로토콜, 즉 합의 알고리즘과 P2P 네트워킹 프로토콜로 구성된 low-level 프로토콜이다.

![](https://miro.medium.com/proxy/1\*TK54bxWDennWamJqPZW2xQ.jpeg)

## 합의

### Tendermint BFT DPOS

#### 부분 동기 모델

텐더민트를 설명할 때 부분 동기 모델이라는 단어가 나온다. 텐더민트를 비롯한 많은 BFT 계열 알고리즘이 부분 동기 모델을 사용한다. 부분 동기 모델에 대한 설명은 아래 페이지에서 살펴보자.&#x20;

{% content-ref url="../../blockchain/network-model.md" %}
[network-model.md](../../blockchain/network-model.md)
{% endcontent-ref %}

#### Termination

Nakamoto consensus에서는 프로세스의 안전성을 위해 알려진 고정 상한선(비트코인의 10분, 이더리움의 15초)을 가진다. 그리고 상한선이 유지되지 않는 경우 합의 알고리즘이 깨지고 체인이 포크된다.&#x20;

텐더민트는 1/3 이하의 프로세스가 결함이 있는 경우 절대 포크하지 않는다. 검증인 집합의 2/3 이상이 합의에 도달하기 전까지 포크가 일어나지 않고 일시적으로 중단된다.&#x20;

### 리더 선출

#### 검증인

투표권을 갖고 있는 노드들을 검증인이라고 부른다. 검증인들은 다음 블록에 동의하는 암호서명, 즉 투표를 전파함으로써 합의 프로토콜에 참여한다.&#x20;

검증인은 staking한 토큰의 양의 의해 결정된다.&#x20;

코스모스는 Atom이라고 부르는 Staking 토큰을 임의의 검증인에게 위임하여 일정한 블록 수수료와 Atom 보상을 얻을 수 있지만 위임 검증인이 해킹 당하거나 프로토콜을 위반하면 함께 처벌 받는다.&#x20;

텐더민트는 100명의 고정된 검증인을 가지고 있다. 이들은 지분을 기반으로 투표하여 결정된다. &#x20;

#### 라운드 리더

검증인들은 한번에 하나의 블록에 대해 합의를 시도한다. 블록에 대한 합의는 라운드를 통해 진행되는데 각 라운드에는 블록을 제안하는 라운드 리더가 있다.&#x20;

검증인들은 블록을 받아들일 것인지 또는 다음 라운드로 넘어갈 것인지에 대한 단계별 투표를 진행한다. 각 라운드의 리더는 검증인 리스트에서 가중 round-robin 방식으로 번갈아 가면서 선출된다. 투표권이 많을수록 더 많은 비중을 갖게 되고 이와 비례하여 리더가 될 가능성이 높아진다. 동일한 투표권을 가지고 있는 검증인들이 있으면 프로토콜에 의해 동일한 횟수만큼 리더로 선출된다(결정론적이다).

#### 리더 선출 과정

1. 검증인마다 가중치가 설정되어 있다.
2. 검증인이 리더로 선출되고 새로운 블록을 제안한다.
3. 가중치가 다시 계산되고 라운드가 완료된 후 약간의 양이 감소한다.
4. 각 라운드가 진행됨에 따라 가중치는 투표권에 비례하여 증가한다.&#x20;
5. 검증인들 중 리더가 다시 선출된다.&#x20;

리더는 결정론적으로 선택되기 때문에 검증인 집합과 각 검증인의 투표 권한을 알고 있으면 라운드 리더가 누구인지 정확하게 계산할 수 있다. 리더를 예측할 수 있는 경우 리더를 DDoS 공격하여 블록체인의 진행을 중단시킬 수 있는데 이런 공격에 대해 텐더민트는 Sentry Architecture를 구현하여 대응한다. &#x20;

### 합의 과정

전체 합의 과정은 3단계(Propose, Prevote, Precommit)으로 구성된다.&#x20;

1. 리더가 블록을 Propose하면 검증자들은 블록을 검증한 후 Prevote 투표를 한다.
2. 전체 검증자의 2/3 이상이 Prevote 투표를 했으면 다음 단계인 Precommit으로 넘어간다.
3. 전체 검증자의 2/3 이상이 Precommit을 했으면 해당 블록을 Commit한다.&#x20;

전체 합의 과정을 3단계로 나눠 2번의 2/3 이상 검증을 받아야 safety를 보장할 수 있다. 또 텐더민트 lock이라는 것을 적용해야 safety를 보장할 수 있다.

텐더민트 lock은 노드가 한 번 precommit을 하고 나면 이후 precommit한 블록과 같은 블록에 대해서만 prevote를 해야 하고 리더가 되었을 때도 precommit한 블록과 같은 블록을 제안해야 한다는 규칙이다.&#x20;

자세한 내용은 아래의 블로그에 잘 설명되어 있기 때문에 꼭 참고하기 바란다.&#x20;

{% embed url="https://kwangyulseo.com/2019/08/15/%ED%85%90%EB%8D%94%EB%AF%BC%ED%8A%B8-%ED%95%A9%EC%9D%98%EC%97%90-3%EB%8B%A8%EA%B3%84%EA%B0%80-%ED%95%84%EC%9A%94%ED%95%9C-%EC%9D%B4%EC%9C%A0" %}

{% embed url="https://kwangyulseo.com/2019/08/22/%ED%85%90%EB%8D%94%EB%AF%BC%ED%8A%B8-%ED%95%A9%EC%9D%98%EC%97%90-%EB%9D%BD%EC%9D%B4-%ED%95%84%EC%9A%94%ED%95%9C-%EC%9D%B4%EC%9C%A0" %}

투표 진행시에 참여한 지분을 네트워크에 동결시킨다. 프로토콜은 임의의 검증인이 안정성을 저해하거나 시도하면 이를 식별할 수 있고 지분을 빼았는다. 이를 통해 Nothing of Stake 문제를 해결할 수 있다.&#x20;

![](https://miro.medium.com/max/1400/0\*qOqpBMjxvVl5ZgVL.)

## P2P networking protocol

### 센트리 노드 아키텍처

검증인은 검증인의 IP 주소를 노출하거나 타 노드와의 연결을 허용하지 않아야 한다. 검증인은 실제 위치를 찾지 못하게 풀노드의 proxy 역할을 하는 센트리 노드를 능동적으로 생성할 수 있다. 센트리 노드를 사용하면 P2P 단에서 검증인의 IP주소가 노출되지 않는다.&#x20;

센트리 노드 아키텍처를 이용하는 것은 검증인의 선택사항이다. 검증인이 네트워크 안전에 필요한 모든 사전 조치를 하지 않는다면 해당 검증인은 검증인 집합에서 퇴장 당하고 staking한 지분이 날아가기 때문에 대부분의 검증인이 센틔 노드 아키텍처를 사용할 것이다.&#x20;

## 특징

### One block finality

텐더민트는 선 합의 후 블록 생성 프로세스를 거치기 때문에 모든 블록이 생성되는 순간 확정된다.&#x20;

### 라이트 클라이언트

텐더민트가 one block finality 를 보장하기 때문에 텐더민트 합의 엔진 기반의 블록체인 사용자들은 블록체인에 저장된 모든 데이터를 전부 내려받지 않고 가장 최근에 생성된 블록 정보를 가져와 사용해도 안전하다.&#x20;

비트코인의 경우 블록헤더 체인들을 동기화하여 가장 많은 작업증명을 가진 체인을 찾아야 하지만 텐더민트를 사용한 블록체인의 경우 그럴 필요가 없다.&#x20;

### 성능

텐더민트 합의는 5개 대륙 7개 데이터센터에 분산되어 있는 64개 노도의 클라우드를 기준으로 약 1\~2초의 커밋 지연속도와 함께 초당 수천 개의 transaction을 처리한다.&#x20;

검증인들이 실패하거나 악의적으로 조작된 투표를 전파하는 가혹한 공격 상황에서도 초당 1,000번의 transaction 성능은 가볍게 능가한다.&#x20;

![](https://raw.githubusercontent.com/gnuclear/atom-whitepaper/master/images/tendermint\_throughput\_blocksize.png)

## Reference

{% embed url="https://v1.cosmos.network/resources/whitepaper/ko" %}

{% embed url="https://medium.com/cosmos-korea/%ED%85%90%EB%8D%94%EB%AF%BC%ED%8A%B8-tendermint-%EC%84%A4%EB%AA%85-%ED%8D%BC%EB%B8%94%EB%A6%AD-%EB%B8%94%EB%A1%9D%EC%B2%B4%EC%9D%B8-public-blockchain-%EC%84%B8%EA%B3%84%EC%97%90%EC%84%9C-%EB%B9%84%EC%9E%94%ED%8B%B4-%EA%B2%B0%ED%97%98-%EA%B0%90%EB%82%B4-bft-%EA%B8%B0%EB%B0%98-%EC%A7%80%EB%B6%84%EC%A6%9D%EB%AA%85-pos-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0-d195944b984b" %}

{% embed url="https://blog.naver.com/lunagram" %}

{% embed url="http://wiki.hash.kr/index.php/%ED%85%90%EB%8D%94%EB%AF%BC%ED%8A%B8_%EB%B9%84%EC%9E%94%ED%8B%B4_%EC%9E%A5%EC%95%A0_%ED%97%88%EC%9A%A9" %}

{% embed url="https://blog.cosmos.network/technology-choices-when-building-your-own-blockchain-a15385cf59bd" %}

{% embed url="https://kwangyulseo.com/2019/08/13/%EB%B6%80%EB%B6%84-%EB%8F%99%EA%B8%B0-%EB%AA%A8%EB%8D%B8-partial-synchrony-model" %}

{% embed url="https://medium.com/@matthewminseokkim/%EC%BD%94%EC%8A%A4%EB%AA%A8%EC%8A%A4%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90-a4549b541658" %}
