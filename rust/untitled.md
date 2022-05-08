---
description: 러스트에 대해 간단히 알아보자.
---

# Start

## Pros

1. Memory management
   * Strict compiler.
     * 오류가 있으면 컴파일이 안됨.
     * 굉장히 안전.
   * **안전한 메모리 관리 - 가장 핵심인 듯 하다.**
     * C 는 개발자가 malloc, free 등을 활용해 직접 관리 -> 실수할 위험이 큼.
     * Java 는 Garbage Collector 사용 -> 성능이 저하됨.
     * Rust 는 오너십이라는 새로운 방식으로 메모리를 관리한다.&#x20;
       * 3가지 원칙
         * 각 값은 오너(Owner)라고 불리는 변수를 갖는다.
         * 한 번에 하나의 오너만 가질 수 있다.
         * 오너가 스코프를 벗어나면 값이 버려진다.
   * Cargo, package manager
     * 튼튼한 라이브러리 생태계
   * Multi core processing
   * WebAssembly
   * C, C++ Java 를 알고 있는 개발자
   * High assurance SW 개발자 (절대 실패해서는 안 되는 코딩을 하는 개발자)
     * ex) 금융
   * 시스템 프로그래밍, 임베디드 프로그래밍

## Cons

1. 배우기 어렵다.&#x20;
2. 자료가 많지 않다.&#x20;



공부하면서 단점에 대해서 추가해보도록 하겠습니다...

## 사용하는 회사

1. Libra - Facebook 에서 만든 암호화폐
2. npm
3. Cloudflare
4. Dropbox
5. AWS Lambda

## Reference

Rust Crash Course | Rustlang\
아래 동영상과 함께 러스트를 공부해보겠습니다.

{% embed url="https://www.youtube.com/watch?v=zF34dRivLOw" %}

