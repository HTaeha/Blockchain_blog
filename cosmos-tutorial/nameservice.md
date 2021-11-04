---
description: Cosmos SDK Tutorials Nameservice
---

# Nameservice

## Getting Started

이 튜토리얼에서  Cosmos SDK application을 만들고 SDK의 기본적 컨셉과 구조를 배울 것이다. 얼마나 쉽고 빠르게 블록체인을 구성할 수 있을지 보여줄 것이다.&#x20;

이 튜토리얼이 끝나면 nameservice application이 만들어질 것이다. 이것은 DNS system (map\[domain]zonefile)인 Namecoin, ENS, Handshake 와 비슷하다. 유저들은 사용하지 않는 도메인 이름을 사거나 팔고 거래할 것이다.&#x20;

### Requirements

* golang > 1.15
* Github account and either Github CLI or Github Desktop

이 튜토리얼은 Starport 0.13.1 버전을 사용한다. 더 높은 버전을 사용하면 명령어가 맞지 않는다.&#x20;

```
curl https://get.starport.network/starport@v0.13.1! | bash
```

## Application Goals

이 application의 목표는 유저들이 도메인 이름을 사고 그 이름에 가치를 부여하는 것이다. 그 이름의 주인은 현재 가장 높은 가격을 제시한 응찰자이다. 이 섹션에서 이러한 간단한 요구사항이 application design으로 어떻게 전환되는지 배울 것이다. &#x20;

Tendermint 가 블록체인의 networking과 consensus layer를 구성해준다. 따라서 개발자는 해당 부분에 지식이 없어도 블록체인을 구성할 수 있고 그 위에 application layer만 구성하면 된다.&#x20;

Cosmos SDK가 state machine을 디자인하는 것을 도와준다. SDK는 modular framework인데 각각의 모듈은 그들의 message/transaction processor가 존재하고 SDK는 각각의 message를 적절하게 routing 해준다.

nameservice application에는 다음의 7가지 모듈이 필요하다.&#x20;

* auth : 계정과 수수료를 정의하고 이 기능들에 대해 다른 어플리케이션의 접근 권한을 부여한다.&#x20;
* bank :  application이 토큰을 생성하고 관리할 수 있게 한다.&#x20;
* staking : application이 사람들이 자신의 자산을 위임한 검증인을 가질 수 있게 한다.&#x20;
* distribution : 검증인과 위임자 사이에 기본적으로 보상을 나누는 방법을 준다. &#x20;
* slashing : 네트워크에 지분을 staking한 사람들(검증인)의 지분을 삭감한다.&#x20;
* supply : 이 모듈은 체인의 전체 공급을 가지고 있다.&#x20;
* nameservice : 아직 존재하지 않는다. nameservice application의 핵심 logic을 처리할 것이다.&#x20;

### State



