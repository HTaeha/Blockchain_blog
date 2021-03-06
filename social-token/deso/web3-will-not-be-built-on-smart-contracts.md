---
description: Deso blog의 글을 읽고 번역해보았다.
---

# Web3 Will Not Be Built on Smart Contracts

## 높은 저장 비용

이더리움, 솔라나, 아발란체, 카르다노 등 기존의 범용 체인은 가스비가 너무 높게 책정되어 있다. 이것은 수많은 Tx가 필요한 소셜 네트워크 구현에 어려움을 겪는다. 좋아요 한번 누르는데 1달러 이상이 든다면 누가 그 플랫폼을 사용하려고 하겠는가?

아래의 표는 1 Gb의 온체인 상태를 저장하는 비용을 블록체인에 따라 나타낸 것이다. 이 비용은 블록체인이 더 대중화되고 스토리지가 더 부족해질수록 증가할 것이다.

![](<../../.gitbook/assets/Untitled (8).png>)

이러한 높은 비용으로 인해 대다수의 Web 2.0 어플리케이션을 구현할 수 없다. Arweave나 Filecoin과 같은 스토리지 중심의 블록체인을 이용하여 링크만 저장한다고 하더라도 0.1달러 \~ 1달러 사이의 비용이 든다. 또한 이 비용은 체인이 대중화됨에 따라 훨씬 더 올라갈 것이다.

## From Finite State to Infinite State

현재 시장에 나와 있는 범용 블록체인들은 유한 상태 어플리케이션을 구동하기 위해 구축되었다. 이들은 각 사용자가 보유해야 하는 데이터 또는 상태의 양이 유한한 어플리케이션이다. 예를 들면 금융 앱에서 거래를 확인하기 위해 가장 중요한 것은 사용자의 계정 잔액이다. 자금을 수만번 이체할 수 있지만 결국 저장해야 하는 것은 각 사용자의 최종 잔액을 나타내는 숫자이다.

무한 상태 응용 프로그램은 각 사용자가 수행하는 작업에 따라 저장해야 하는 데이터 양이 무한하게 증가하는 응용 프로그램이다. 일반적인 소셜 앱을 보면 사용자는 상태를 추가하는 프로플이르 만들고 게시물을 만들고, 다른 사용자를 팔로우한다. 소셜 앱을 사용하면 무한한 양의 데이터를 저장할 수 있어야 하고 이 상태를 자주 쿼리해야 한다. 현재 시장에 나와있는 범용 블록체인들은 이러한 유형의 어플리케이션을 처리할 수 없다.

## The Inevitable Congestion of General-Purpose Chains

현재 대부분의 최신 범용 블록체인은 모든 계정 상태를 메모리에 저장하여 높은 TPS를 유지한다. 이것은 DeFi와 같은 유한 상태 응용 프로그램에 적합하다. 그러나 누군가가 체인에 무한 상태 앱을 빌드하려고 하면 각 사용자에 대해 단일 숫자를 저장하는 것에서 메가바이트 이상을 저장하는 것으로 바뀐다. 또한 범용 블록체인에서는 인메모리에 필요한 계정 상태와 그렇지 않은 계정 상태에 대해 지능적인 결정을 내릴 수 없으며 쿼리 가능하도록 데이터를 인덱싱할 수도 없다. 최종적으로 모든 범용 블록체인은 실행 가능한 상태를 유지하기 위해 저장 제한을 부과해야 한다. 이로 인해 스토리지 요금은 치솟았고 이것은 체인의 인기가 높아지면 더욱 악화될 것이다.

Web 2 어플리케이션의 대다수는 무한 상태이다. 현재 블록체인에서 대부분의 Web2를 구현할 수 없다면 Web3가 어떻게 도약할 수 있겠는가?

## Scaling Infinite State Applications with Deso

저장될 데이터 유형(스키마)에 대한 가정을 할 수 없으면 데이터 저장, 인덱싱 및 쿼리 비용이 급증한다. 그래서 무한 상태 응용 프로그램에 내재된 저장 및 인덱싱 요구 사항을 처리하기 위한 맞춤화된 블록체인이 구축되어야 한다.

Deso가 그 예 중 하나이다. Deso는 소셜 어플리케이션을 위해 처음부터 맞춤으로 제작되었으며, Deso가 저장하고 인덱싱하는 모든 데이터가 정해진 스키마를 따른다. 프로필, 게시물, 팔로우 등 정보를 각각 다르게 저장하고 인덱싱 한다. 이러한 수준의 커스터마이징은 저장 비용을 Avalanche, Solana보다 10000배 저렴하게 만들 뿐만 아니라 즉각적인 쿼리를 제공할 수 있도록 한다.

## A Note on Storage-Focused Blockchains

Filecoin이나 Arweave와 같이 스토리지에 집중한 블록체인과 범용 블록체인을 함께 사용하면 무한 상태 응용 프로그램을 구현할 수 있다고 제안하기도 한다. 하지만 실제로는 범용 블록체인의 저장 비용이 너무 비싸서 Filecoin 또는 Arweave에 대한 간단한 링크를 저장하는 것조차 0.1 \~ 1달러 이상이다. 또 Filecoin이나 Arweave에 저장된 데이터는 적절하게 인덱싱되지 않으므로 대규모로 사용하려면 별도의 인덱싱 레이어를 만들어야 한다.

Deso 블록체인에서는 사용자가 원하는 경우 이미지와 비디오를 Arweave에 저장할 수 있다. 그 후 Deso는 Arweave에 대한 링크를 저장한다. Deso에 링크를 저장하는 비용은 거의 무료이다(0.0000184달러).

## Reference

[Web3 Will Not Be Built on Smart Contracts](https://www.deso.org/blog/web-3-will-not-be-built-on-smart-contracts)
