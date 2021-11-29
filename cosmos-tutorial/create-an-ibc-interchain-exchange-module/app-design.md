# App Design

이 챕터에서는 interchain 거래소 모듈이 어떻게 디자인 되는지 배운다. 모듈은 구매, 판매를 하는 order book을 가지고 있다. 먼저, 한 쌍의 토큰을 위한 order book이 생성된다. Order book이 생성된 후에 한 쌍의 토큰에 대한 구매, 판매 주문을 생성할 수 있다.

모듈은 블록체인간 통신 표준 IBC를 사용한다. IBC를 사용하여 모듈은 여러 블록체인이 상호 작용하고 토큰을 교환하도록 토큰에 대한 order book을 생성할 수 있다. 한 블록체인의 토큰 하나와 다른 블록체인의 다른 토큰으로 order book 쌍을 생성할 수 있다. 이 튜토리얼에서 생성한 모듈을 ibcdex라고 부를 것이다. 두 블록체인 모두 ibcdex 모듈을 설치하고 실행해야 한다.

사용자가 ibcdex를 이용해서 토큰을 교환할 때, 당신은 다른 블록체인으로부터 그 토큰의 바우처를 받는다. 이것은 ibc-transfer가 어떻게 설계됐는지와 비슷하다. 블록체인 모듈은 존재하는 블록체인의 새로운 토큰을 발행할 권리가 없기 때문에 target 체인의 토큰은 잠기고 구매자는 해당 토큰의 바우처를 받게 된다. 바우처가 원래 토큰의 잠금을 해제하기 위해 다시 소각되면 이 프로세스를 되돌릴 수 있다. 이것은 튜토리얼 전체에서 더 자세히 설명될 것이다.

## Assumption

모든 체인 쌍 간의 토큰 교환을 위해 order book을 생성할 수 있다. 요구 사항은 ibcdex 모듈을 사용할 수 있도록 하는 것이다. 한 쌍의 토큰에 대해 동시에 하나의 order book만 있을 수 있다. 특정 체인은 기본 토큰을 새로 발행할 수 없다. 이 모듈은 ibc-transfer 모듈에서 영감을 받았으며 바우처 생성과 같은 몇 가지 유사점이 있다. 더 복잡하지만 다음을 만드는 방법이 표시된다.

* Packet을 보내는 여러 type
* Acknowledgement을 다루는 여러 type
* 수신 시, 시간 초과 시 패킷을 처리하는 방법에 대한 좀 더 복잡한 논리

## Overview

`Venus`와 `Mars`이라는 두 개의 블록체인이 있다고 가정하자. `Venus`의 기본 토큰은 `vcx`, `Mars`의 토큰은 `mcx` 이다. Mars에서 Venus로 토큰을 교환할 때 Venus 블록체인에서 `ibc/B5CB286...A7B21307F` 처럼 생긴 IBC 바우처 토큰으로 끝난다. `ibc/` 뒤의 긴 문자열은 IBC를 통해 전송된 토큰의 denom 추적 해시이다. 블록체인의 API를 사용하여 해당 해시에서 denom trace를 얻을 수 있다. Denom 추적은 `base_denom`과 `path`로 구성된다. 이 예에서 `base_denom`은 `mcx`이고 `path`에는 토큰이 전송된 포트 및 채널 쌍이 포함된다. 단일 홉 전송 `path`는 `transfer/channel-0`과 같다. 이 토큰 `ibc/Venus/mcx`는 동일한 주문서를 사용하여 되팔 수 없다. 교환을 "반전"하고 `Mars` 토큰을 다시 받으려면 `ibc/Venux/mcx`에서 `mcx`로의 새 order book을 만들어야 한다.

## Order books

일반적인 거래소에서 새로운 pair는 `MCX`를 판매하기 위한 주문 또는 `VCX`를 구매하기 위한 주문이 포함된 order book 생성을 의미한다. 여기에는 두개의 체인이 있으며 `Mars`와 `Venus` 사이의 데이터 구조는 분리되어야 한다.

`Mars`의 사용자는 `MCX`를 팔고 `Venux`의 사용자는 `MCX`를 산다. 그러므로 우리는 `Mars`에서 `MCX`를 팔기 위한 주문과 `Venus`에서 `MCX`를 사기 위한 주문을 모두 나타낸다.

이 예에서 `Mars` 블록체인은 판매 주문을 가지고 있고 `Venux` 블록체인은 구매 주문을 가지고 있다.

## Exchanging tokens back

`ibc-transfer`와 같이 각각의 블록체인은 다른 블록체인으로부터 생성된 토큰 바우처를 추적한다.

블록체인 `Mars`가 `Venus`에 `MCX`를 판매하고 `Venus`에서 `ibc/Venus/mcx`가 발행되면 `ibc/Venus/mcx`가 `Mars`에서 다시 판매되면 잠금 해제되고 받은 토큰은 `MCX`가 된다.

## Features

* 두 체인 간에 토큰 pair를 위한 교환 order book을 생성한다.
* Source 체인에서 판매 주문을 보낸다.
* Target 체인에서 구매 주문을 보낸다.
* 구매 또는 판매 주문을 취소한다.
