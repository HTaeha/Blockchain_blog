# Introduction

Interchain 거래소는 블록체인 사이에 사고 파는 오더를 생성하는 모듈이다. 이 튜토리얼에서는 어떻게 Cosmos SDK를 사용하여 사고 파는 order pair를 생성하는지 배울 것이다. 한 블록체인에서 다른 블록체인으로 토큰을 스왑할 수 있고 블록체인 사이에서 구매, 판매 order를 하는 order book을 생성할 것이다.

### You will learn how to

* Starport로 블록체인 생성
* Cosmos SDK IBC module 생성
* 호스트가 사고 파는 order를 포함하는 order book 생성
* 한 블록체인에서 다른 블록체인으로 IBC packet 보내기
* IBC packet의 timout과 acknowledgements 다루기

## How the module works

2개 또는 그 이상의 블록체인에서 작동하는 거래소를 어떻게 생성하는지 배울 것이다. Module은 ibcdex라고 부른다. 그 모듈은 서로 다른 블록체인 상의 pair token 사이에서 order book 거래소를 여는 것을 허용한다. 블록체인은 ibcdex 모듈을 이용가능하게 해야 한다. 토큰들은 간단한 order book에서 지정가 주문으로 구매 또는 판매 될 수 있으며 유동성 풀 또는 AMM 개념은 없다.

Market은 단방향이다. Source chain에서 판매된 토큰은 그대로 되돌릴 수 없으며 target chain에서 구매한 토큰은 동일한 pair를 사용해서 되팔 수 없다. Source chain의 토큰이 판매되면 order book에 새로운 pair를 생성해야 되살릴 수 있다. 이것은 target chain에 바우처 토큰을 생성하는 IBC의 특성 때문이다. 이 튜토리얼에서는 native 블록체인 토큰과 다른 블록체인에서 발행되는 바우처 토큰의 차이점을 배운다. Native 토큰을 다시 받기 위해 두번째 order book pair를 생성하는 법을 배울 것이다.
