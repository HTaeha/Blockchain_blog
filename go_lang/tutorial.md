---
description: Start Go lang tutorial!!
---

# Tutorial

Go Documentation 의 튜토리얼 따라해보며 Go 구조를 경험해보자. &#x20;

{% embed url="https://github.com/HTaeha/go_tutorial" %}

### Documentation - Getting Started

1. 모듈 관리
   * Go lang 은 언어 자체에서 모듈 관리를 한다. 따라서 GOROOT 경로 안에서 코드를 작성해야 새로운 모듈을 추가할 수 있고 Go lang이 해당 코드, 모듈의 위치를 알 수 있다. 하지만 Go Modules를 추가하면 GOROOT 경로 외에서도 모듈을 import 할 수 있다.&#x20;
   * Go Modules
     * `go mod init` creates a new module, initializing the `go.mod` file that describes it.
     * `go build`, `go test`, and other package-building commands add new dependencies to `go.mod` as needed.
     * `go list -m all` prints the current module’s dependencies.
     * `go get` changes the required version of a dependency (or adds a new dependency).
     * `go mod tidy` removes unused dependencies.

{% embed url="https://golang.org/doc/" %}
Documentation - Getting Started를 해보자.
{% endembed %}

{% embed url="https://tour.golang.org/list" %}
Go lang의 여러 문법과 구조를 학습할 수 있다.&#x20;
{% endembed %}

{% embed url="https://go.dev/" %}
Go package를 검색할 수 있다.&#x20;
{% endembed %}

{% embed url="https://nomadcoders.co/go-for-beginners/lobby" %}
노마드 코더 강의를 보며 간단한 Go project를 실습했다.&#x20;
{% endembed %}

