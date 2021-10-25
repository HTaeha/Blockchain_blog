---
description: channel, goroutine practice
---

# 3. URL Checker

nomadCoder 강의 실습 #3

1. main.go

```go
package main

import (
	"errors"
	"fmt"
	"net/http"
)

type requestResult struct {
	url    string
	status string
}

var errRequestFailed = errors.New("Request failed")

func main() {
	c := make(chan requestResult)
	results := make(map[string]string)

	urls := []string{
		"https://www.airbnb.com/",
		"https://www.google.com/",
		"https://www.amazon.com/",
		"https://www.reddit.com/",
		"https://soundcloud.com/",
		"https://www.facebook.com/",
		"https://www.instagram.com/",
		"https://academy.nomadcoders.co/",
	}

	for _, url := range urls {
		go hitURL(url, c)
	}
	for i := 0; i < len(urls); i++ {
		result := <-c
		results[result.url] = result.status
	}
	for url, status := range results {
		fmt.Println(url, status)
	}

}

func hitURL(url string, c chan<- requestResult) {
	resp, err := http.Get(url)
	status := "OK"
	if err != nil || resp.StatusCode >= 400 {
		status = "FAILED"
	}
	c <- requestResult{url: url, status: status}
}

```



#### map 초기화 하기.

* var results = make(map\[string]string)
  * make는 map을 초기화 시켜주는 함수이다.
* var results = map\[string]string{}
  * map을 정의하고 끝에 {}를 넣어 빈 값으로 초기화 시켜준다.
* var results map\[string]string
  * 이렇게 하면 map이 초기화 되지 않아서 results에 어떤 값을 넣으려고 할 때 compiler가 panic한다.&#x20;

#### goroutine

* main function이 존재하는 동안만 유효하다.&#x20;
* main function은 goroutine을 기다려주지 않는다.&#x20;

#### channel

* goroutine과 main 혹은 goroutine과 goroutine 사이에 커뮤니케이션 할 수 있게 해줌.
* 아래는 아주 간단한 goroutine, channel 예제이다.&#x20;
* 함수 안에서 channel c에 값을 넘겨주고 <-c로 main funtion에서 어떤 값이 나오는지 기다린다. \
  (blocking operation)
* channel에 2개의 값만 넣었는데 <-c를 3개 한다면 main 함수는 마지막 1개의 값이 나오지 않아서 계속 기다린다. 혹은 dead lock에 걸린다.&#x20;
* func foo(c chan<- bool){} 이런식으로 선언하면 해당 채널은 send-only 라는 뜻이다.&#x20;

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	c := make(chan bool)
	people := [2]string{"taeha", "nico"}
	for _, person := range people {
		go isSexy(person, c)
	}
	fmt.Println(<-c)
	fmt.Println(<-c)
}

func isSexy(person string, c chan bool) {
	time.Sleep(time.Second * 5)
	fmt.Println(person)
	c <- true
}

```



#### Go packages

{% embed url="https://golang.org/pkg/" %}



