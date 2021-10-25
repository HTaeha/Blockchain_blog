---
description: 기본적인 Go struct 연습.
---

# 1. Banking

nomadCoder 강의 실습 #1

1. main.go

```go
package main

import (
	"fmt"
	accounts "go_tutorial/nomadCoder/practiceStruct/banking"
)

type person struct {
	name    string
	age     int
	favFood []string
}

func main() {
	account := accounts.NewAccount("taeha")
	account.Deposit(10)
	fmt.Println(account.Balance())
	err := account.Withdraw(20)
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println(account.Balance(), account.Owner())
	fmt.Println(account)

}
```

2\. banking.go

* struct
* private, public
  * 첫 글자가 소문자이면 private, 대문자이면 public.&#x20;
  * function에서도 마찬가지이다. export 가능 여부가 달라짐.
* method
  * func 옆에 receiver 를 설정함으로써 method 작성이 가능.
  * pointer 사용에 주의하자. (deepCopy, shallowCopy)
* exception handling
  * try-catch 등의 기능이 없다.&#x20;
  * error에 대해 개발자가 직접 신경써서 처리해줘야 함.&#x20;
  * error 가 아닌 경우 nil을 return 한다.
* String()
  * String() method를 정의함으로써 해당 객체를 print 했을 때 내용을 설정할 수 있다.&#x20;

```go
package accounts

import (
	"errors"
	"fmt"
)

var errNoMoney = errors.New("No money error")

// Account struct
type Account struct {
	owner   string
	balance int
}

// NewAccount creates Account
func NewAccount(owner string) *Account {
	account := Account{owner: owner, balance: 0}
	return &account
}

// Deposit x amount on your account
func (a *Account) Deposit(amount int) {
	a.balance += amount
}

// Balance of your account
func (a *Account) Balance() int {
	return a.balance
}

// Withdraw x amount from your account
func (a *Account) Withdraw(amount int) error {
	if a.balance < amount {
		return errNoMoney
	}
	a.balance -= amount
	return nil
}

// ChangeOwner of the account
func (a *Account) ChangeOwner(newOwner string) {
	a.owner = newOwner
}

// Owner of account
func (a Account) Owner() string {
	return a.owner
}

func (a *Account) String() string {
	return fmt.Sprint(a.Owner(), "'s account.\nHas: ", a.Balance())
}
```

