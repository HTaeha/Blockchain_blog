---
description: 윤주운 개발자님이 코스모스 아카데미에서 코스모스 SDK 실습에 대해 설명하신 내용을 정리해보려고 한다.
---

# 코스모스 SDK 실습 - nameservice

## 준비

1. Docker 설치
2. docker run -t -i mossid/sdk-example:0.2
3. cd root/
4. source .profile

## 시작&#x20;

nsd와 nscli 명령어를 사용할 수 있는데 각각은 gaiad와 gaiacli에 대응한다.&#x20;

* nohup nsd start &
  * 위 명령어를 통해 백그라운드에서 블록체인을 돌린다.&#x20;
  * 텐더민트는 PoS로 동작하는데 지금은 staking 기능을 전부 뺐기 때문에 블록체인을 돌리고 있는 로컬 기기가 100%의 투표권을 가지고 있는 1인 노드이다.&#x20;

![](<../.gitbook/assets/image (17).png>)

* ChainID는 각 체인별로 존재하는 유니크한 아이디이다.&#x20;
* 여러 체인이 접근할 수 있을 때 ChainID를 통해서 구별한다.&#x20;
* 이더리움의 테스트넷들의 이름간의 차이라고 보면 된다.&#x20;



* nscli 는 데몬에 접속해서 tx을 날리거나 필요한 정보를 얻어오는 역할을 한다.&#x20;
* nscli keys list
  * 현재 가지고 있는 key, 계정들의 리스트를 보여준다.&#x20;
  * 생성을 하지 않았기 때문에 아직은 아무것도 뜨지 않는다.&#x20;
* nscli keys add mykey
  * mykey라는 이름의 key를 생성.
  * 복구 키워드들이 뜬다.&#x20;
  * Account의 비밀번호 또는 데이터를 잃어버리면 아래 사진에 뜨는 영문자들이 계정을 복구하기 위한 유일한 방법이다.&#x20;

![](<../.gitbook/assets/image (18).png>)

* nscli keys list
  * 키를 추가했기 때문에 키의 정보가 나타난다.&#x20;

![](<../.gitbook/assets/image (19).png>)

* nscli query account cosmos1053vky7zpurj8697n9djhpmr8hptrdmvtr67q8 --chain-id test-chain-jADW4u
  * 현재 계정에 대한 정보가 없어서  Internal error 가 뜬다.&#x20;
  * 우리가 계좌를 만든 것은 client side이기 때문에 블록체인에는 이 계정에 대한 정보가 전혀 기록되어 있지 않다.&#x20;
* nscli tx request-coins --amount 10atom --from mykey --chain-id test-chain-jADW4u
  * 원래는 수수료를 내야만 tx을 포함시킬 수 있기 때문에 누군가가 돈을 보내주기 전까지는 이 계정을 사용해서 블록체인에 접근할 수가 없다. (수수료 낼 돈이 없기 때문에)
  * 현재는 수수료 로직을 제거했기 때문에 가능.&#x20;
  * 다양한 수수료 적용 방식을 직접 코딩해서 적용할 수 있다.&#x20;
  * 위 명령어를 실행했을 때 첫 시도에서 비밀번호 입력후 freeze되고 다음으로 넘어가지 않는 문제가 있었다. 하지만 새로 터미널을 켜서 다시 시작했더니 제대로 동작했다.&#x20;

![](<../.gitbook/assets/image (20).png>)

* nscli query account cosmos1xmjv33tc33sm8z04kk9dmhkavmsjdpsd54ysj9 --chain-id test-chain-Q94s4Z
  * Internal error 떴던 쿼리 다시 한번 실행
  * 한번 새로 실행했기 때문에 위의 명령어와는 account와 chain-id가 다르다.
  * 이번에는 제대로 데이터를 받아온다.&#x20;

![](<../.gitbook/assets/image (21).png>)

* nscli tx buy-domain --domain mydomain.com --value 127.0.0.1 --amount 5atom --from mykey --chain-id test-chain-Q94s4Z
  * 5atom으로 mydomain.com을 구매.
  * 92번째 블록에 기록됨.

![](<../.gitbook/assets/image (22).png>)

* nscli query domain mydomain.com --chain-id test-chain-Q94s4Z
  * 구매한 mydomain.com에 대한 정보를 받아옴.&#x20;

![](<../.gitbook/assets/image (23).png>)

* nscli tx buy-domain --domain mydomain.com --value 127.0.0.1 --amount 2atom --from mykey --chain-id test-chain-Q94s4Z
  * 방금 구매한 mydomain.com을 다시 2atom만 주고 구매한다고 했을 때 가격이 맞지 않기 때문에 에러가 발생한다.&#x20;
  * 더 높은 금액으로 구매를 시도했을 때는 정상적으로 작동한다.&#x20;
  * 더 높은 금액으로 구매를 시도하기 전에 request-coins로 atom을 더 받아와야 한다. 하지만 아까와 같은 명령어로는 Tx already exists in cache라는 에러가 뜰 것이다.&#x20;
  * 끝에 --sequence 1 을 추가해주면 해결할 수 있다.&#x20;
  * AnteHandler가 sequence number를 체크하기 때문에 sequence는 항상 증가되어야 한다.&#x20;
  * 이러한 모든 로직들은 Keeper 안에 정의된다.&#x20;
  * Handler가 Keeper를 어떻게 다루느냐에 따라서 예외상황들을 규정할 수 있다.&#x20;

![](<../.gitbook/assets/image (24).png>)

## Question

### Logic에 업그레이드가 있을 경우에는 어떻게 대처하는가?

* 지금 예제처럼 validator가 1명일 경우에는 node를 잠시 닫고 software를 업데이트 한 후 다시 실행시키면 된다.&#x20;
* 하지만 validator가 100명일 때 50명은 업데이트를 하고 50명은 업데이트를 하지 않았으면 합의 실패가 일어난다. 서로 다른 logic을 가지고 있기 때문이다.&#x20;
* 그렇기 때문에 governance를 통해 정확히 validator들이 업데이트에 동의하는 것을 투표하고 업데이트를 진행해야 한다.&#x20;

### 하나의 앱당 하나의 블록체인이라면 다른 블록체인과 소통하는 로직은 어디에 있는가?

* 코스모스는 텐더민트 합의 알고리즘을 쓰는데 텐더민트의 큰 장점은 완결성이다. 1block finality가 보장된다.&#x20;
* 비트코인의 경우 10, 20블록을 기다려야만 내가 제출한 tx가 올바르다는 것을 확신할 수 있었다.&#x20;
* 텐더민트는 합의 알고리즘 상 해시 파워에 상관없이 무조건 1block만에 finality가 결정된다.&#x20;
* 여러개의 텐더민트 기반 체인이 있을 때 한 체인이 다른 체인을 무조건적으로 읽을 수 있다.&#x20;
* 기본적으로 라이트 클라이언트로 작동한다.
* IBC protocol이 그것을 규정하고 있다.&#x20;
* 텐더민트 합의 알고리즘 위에 만들어진 여러 앱들이 서로 어떻게 통신을 하는지 규정한 것이 IBC protocol이다. &#x20;

## Reference

{% embed url="https://www.youtube.com/watch?v=ZD7xk1SfdBw&t=3s" %}
