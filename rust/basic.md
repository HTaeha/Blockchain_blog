# Basic

## Getting started

1. installation

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

So simple!

2\. build

* &#x20;\> rustc \[filename]
* rustc로 임의로 빌드할 수 있지만 보통 cargo를 이용한다.

3\. cargo

* Rust의 빌드 시스템 및 패키지 매니저이다.
* cargo init\
  Cargo.toml 파일은 package.js 와 비슷한 역할을 한다. (패키지, 디펜던시 관리)
* cargo run\
  빌드와 실행을 모두 한다.&#x20;
* cargo build\
  실행은 하지 않고 빌드만 한다.\
  cargo build --release 명령어를 실행하면 release 폴더가 생성되고 optimize된 파일이 생긴다.
*

{% embed url="https://www.rust-lang.org/learn/get-started" %}

