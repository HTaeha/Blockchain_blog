# Create the Order Book

이 챕터에서는 order book을 구현한다. 구매, 판매 주문을 올릴 수 있는 order book을 만들 것이다. 토큰 한 쌍을 위한 order book은 등록을 먼저 해야 한다. 등록 후에 구매, 판매 주문을 등록할 수 있다. 판매 주문 예약을 구현하기 위해 `sell_order_book.go` 파일을 생성한다. 판매 주문서에는 토큰 액면가 데이터와 토큰 판매를 위해 제안하는 가격이 포함된 판매 주문이 포함되어 있다. 구매 주문서를 구현하여 buy\_order\_book.go 파일을 생성한다. 구매 주문서에는 토큰 액면가 데이터와 토큰 구매를 위해 제안한 가격이 포함된 구매 주문이 포함되어 있다. 구매 주문과 판매 주문은 서로 다른 블록체인 앱에서 실행된다. 매수 주문과 매도 주문이 일치하면 거래소가 실행된다.

## Add The Order Book

`OrderBook` 과 `Order` 메세지를 `order.proto` 파일에 추가한다.

```bash
// proto/ibcdex/order.proto
syntax = "proto3";
package username.interchange.ibcdex;

option go_package = "github.com/username/interchange/x/ibcdex/types";

message OrderBook {
  int32 idCount = 1;
  repeated Order orders = 2;
}

message Order {
  int32 id = 1;
  string creator = 2;
  int32 amount = 3;
  int32 price = 4;
}
```

types 폴더에 order\_book.go 를 생성한다. 이 파일에서는 새로운 order book을 생성하는 로직을 정의한다. 판매, 구매 주문 사이에 공통적인 로직이 담긴다.

```bash
// x/ibcdex/types/order_book.go
package types

import (
	"errors"
	"sort"
)

const (
	MaxAmount = int32(100000)
	MaxPrice  = int32(100000)
)

type Ordering int

const (
	Increasing Ordering = iota
	Decreasing
)

var (
	ErrMaxAmount     = errors.New("max amount reached")
	ErrMaxPrice      = errors.New("max price reached")
	ErrZeroAmount    = errors.New("amount is zero")
	ErrZeroPrice     = errors.New("price is zero")
	ErrOrderNotFound = errors.New("order not found")
)

// checkAmountAndPrice amount와 price를 체크한다
func checkAmountAndPrice(amount int32, price int32) error {
	if amount == int32(0) {
		return ErrZeroAmount
	}
	if amount > MaxAmount {
		return ErrMaxAmount
	}
	if price == int32(0) {
		return ErrZeroPrice
	}
	if price > MaxPrice {
		return ErrMaxPrice
	}

	return nil
}

// NewOrderBook 새로운 OrderBook 생성
func NewOrderBook() OrderBook {
	return OrderBook{
		IdCount: 0,
	}
}

// GetOrder index에 해당하는 Order를 찾아 반환한다
func (book OrderBook) GetOrder(index int) (order Order, err error) {
	if index >= len(book.Orders) {
		return order, ErrOrderNotFound
	}

	return *book.Orders[index], nil
}

// GetNextOrderID 다음에 추가될 Order의 ID를 반환한다
func (book OrderBook) GetNextOrderID() int32 {
	return book.IdCount
}

// GetOrderFromID OrderBook에서 인자의 id와 같은 Order를 찾아 반환한다
func (book OrderBook) GetOrderFromID(id int32) (Order, error) {
	for _, order := range book.Orders {
		if order.Id == id {
			return *order, nil
		}
	}
	return Order{}, ErrOrderNotFound
}

// SetOrder 해당하는 index의 Order 값을 세팅한다
func (book *OrderBook) SetOrder(index int, order Order) error {
	if index >= len(book.Orders) {
		return ErrOrderNotFound
	}

	book.Orders[index] = &order

	return nil
}

// IncrementNextOrderID OrderBook의 IdCount를 증가시킨다
func (book *OrderBook) IncrementNextOrderID() {
	// Even numbers to have different ID than buy orders
	book.IdCount++
}

// RemoveOrderFromID 인자의 id와 매칭되는 Order를 찾아 제거한다
func (book *OrderBook) RemoveOrderFromID(id int32) error {
	for i, order := range book.Orders {
		if order.Id == id {
			book.Orders = append(book.Orders[:i], book.Orders[i+1:]...)
			return nil
		}
	}
	return ErrOrderNotFound
}

// AppendOrder 새로운 Order를 OrderBook에 추가시킨다
func (book *OrderBook) appendOrder(creator string, amount int32, price int32, ordering Ordering) (int32, error) {
	if err := checkAmountAndPrice(amount, price); err != nil {
		return 0, err
	}

	// Initialize the order
	var order Order
	order.Id = book.GetNextOrderID()
	order.Creator = creator
	order.Amount = amount
	order.Price = price

	// Increment ID tracker
	book.IncrementNextOrderID()

	// Insert the order
	book.insertOrder(order, ordering)

	return order.Id, nil
}

// insertOrder Order를 오름차순 혹은 내림차순으로 OrderBook에 추가한다
func (book *OrderBook) insertOrder(order Order, ordering Ordering) {
	if len(book.Orders) > 0 {
		var i int

		// get the index of the new order depending on the provided ordering
		if ordering == Increasing {
			i = sort.Search(len(book.Orders), func(i int) bool { return book.Orders[i].Price > order.Price })
		} else {
			i = sort.Search(len(book.Orders), func(i int) bool { return book.Orders[i].Price < order.Price })
		}

		// insert order
		orders := append(book.Orders, &order)
		copy(orders[i+1:], orders[i:])
		orders[i] = &order
		book.Orders = orders
	} else {
		book.Orders = append(book.Orders, &order)
	}
}
```

## Add The Sellorder

```bash
// proto/ibcdex/sell_order_book.proto
syntax = "proto3";
package username.interchange.ibcdex;

option go_package = "github.com/username/interchange/x/ibcdex/types";

import "ibcdex/order.proto"; // <--

message SellOrderBook {
  string creator = 1;
  string index = 2;
  string amountDenom = 3;
  string priceDenom = 4;
  OrderBook book = 5; // <--
}
```

`sell_order_book.proto` 파일을 바꿔서 `SellOrderBook`에 `OrderBook`을 추가한다.

```bash
// x/ibcdex/types/sell_order_book.go
package types

// NewSellOrderBook 새로운 SellOrderBook을 생성한다
func NewSellOrderBook(AmountDenom string, PriceDenom string) SellOrderBook {
	book := NewOrderBook()
	return SellOrderBook{
		AmountDenom: AmountDenom,
		PriceDenom: PriceDenom,
		Book: &book,
	}
}

// AppendOrder SellOrderBook에 Order를 추가한다
func (s *SellOrderBook) AppendOrder(creator string, amount int32, price int32) (int32, error) {
	return s.Book.appendOrder(creator, amount, price, Decreasing)
}

// LiquidateFromBuyOrder SellOrderBook에서 BuyOrder에 맞는 Order를 찾아 매칭시킨다
// BuyOrder가 제시한 금액 이하의 price가 가장 낮은 SellOrder를 match시킨다
// match되는 Order가 없으면 match 값을 false로 리턴한다
func (s *SellOrderBook) LiquidateFromBuyOrder(order Order) (
	remainingBuyOrder Order,
	liquidatedSellOrder Order,
	purchase int32,
	match bool,
	filled bool,
) {
	remainingBuyOrder = order

	// SellOrderBook이 비어있다
	orderCount := len(s.Book.Orders)
	if orderCount == 0 {
		return order, liquidatedSellOrder, purchase, false, false
	}

	// BuyOrder가 제시한 금액에 맞는 SellOrder가 없다
	lowestAsk := s.Book.Orders[orderCount-1]
	if order.Price < lowestAsk.Price {
		return order, liquidatedSellOrder, purchase, false, false
	}

	liquidatedSellOrder = *lowestAsk

	// BuyOrder가 구매하고자 하는 양이 충족된다
	if lowestAsk.Amount >= order.Amount {
		remainingBuyOrder.Amount = 0
		liquidatedSellOrder.Amount = order.Amount
		purchase = order.Amount

		// SellOrder가 모두 팔렸으면 SellOrderBook에서 제거한다
		lowestAsk.Amount -= order.Amount
		if lowestAsk.Amount == 0 {
			s.Book.Orders = s.Book.Orders[:orderCount-1]
		} else {
			s.Book.Orders[orderCount-1] = lowestAsk
		}

		return remainingBuyOrder, liquidatedSellOrder, purchase, true, true
	}

	// SellOrder 보다 BuyOrder가 더 많은 경우
	// 사고 싶은 양이 파는 양보다 더 많은 경우
	purchase = lowestAsk.Amount
	s.Book.Orders = s.Book.Orders[:orderCount-1]
	remainingBuyOrder.Amount -= lowestAsk.Amount

	return remainingBuyOrder, liquidatedSellOrder, purchase, true, false
}

// FillBuyOrder OrderBook에서 구매량이 충족될 때까지 거래를 한다. 그 과정에서의 부작용을 핸들링한다.
func (s *SellOrderBook) FillBuyOrder(order Order) (
	remainingBuyOrder Order,
	liquidated []Order,
	purchase int32,
	filled bool,
) {
	var liquidatedList []Order
	totalPurchase := int32(0)
	remainingBuyOrder = order

	// Liquidate as long as there is match
	for {
		var match bool
		var liquidation Order
		remainingBuyOrder, liquidation, purchase, match, filled = s.LiquidateFromBuyOrder(
			remainingBuyOrder,
		)
		if !match {
			break
		}

		// Update gains
		totalPurchase += purchase

		// Update liquidated
		liquidatedList = append(liquidatedList, liquidation)

		if filled {
			break
		}
	}

	return remainingBuyOrder, liquidatedList, totalPurchase, filled
}
```

## Add The Buyorder

```bash
// proto/ibcdex/buy_order_book.proto
syntax = "proto3";
package username.interchange.ibcdex;

option go_package = "github.com/username/interchange/x/ibcdex/types";

import "ibcdex/order.proto"; // <--

message BuyOrderBook {
  string creator = 1;
  string index = 2;
  string amountDenom = 3;
  string priceDenom = 4;
  OrderBook book = 5; // <--
}
```

```bash
// x/ibcdex/types/buy_order_book.go
package types

// NewBuyOrderBook 새로운 BuyOrderBook 생성
func NewBuyOrderBook(AmountDenom string, PriceDenom string) BuyOrderBook {
	book := NewOrderBook()
	return BuyOrderBook{
        AmountDenom: AmountDenom,
        PriceDenom: PriceDenom,
        Book: &book,
    }
}

 // AppendOrder BuyOrderBook에 Order 추가
 func (b *BuyOrderBook) AppendOrder(creator string, amount int32, price int32) (int32, error) {
	 return b.Book.appendOrder(creator, amount, price, Increasing)
 }

 // LiquidateFromSellOrder SellOrder에 맞는 가장 적합한 BuyOrder를 찾아 거래한다 
 // 가장 높은 가격에 팔 수 있는 BuyOrder를 찾아 거래한다
 // 적합한 BuyOrder가 없으면 false를 반환한다
 func (b *BuyOrderBook) LiquidateFromSellOrder(order Order) (
	 remainingSellOrder Order,
	 liquidatedBuyOrder Order,
	 gain int32,
	 match bool,
	 filled bool,
 ) {
	 remainingSellOrder = order

	 // No match if no order
	 orderCount := len(b.Book.Orders)
	 if orderCount == 0 {
		 return order, liquidatedBuyOrder, gain, false, false
	 }

	 // Check if match
	 highestBid := b.Book.Orders[orderCount-1]
	 if order.Price > highestBid.Price {
		 return order, liquidatedBuyOrder, gain, false, false
	 }

	 liquidatedBuyOrder = *highestBid

	 // Check if sell order can be entirely filled
	 if highestBid.Amount >= order.Amount {
		 remainingSellOrder.Amount = 0
		 liquidatedBuyOrder.Amount = order.Amount
		 gain = order.Amount * highestBid.Price

		 // Remove highest bid if it has been entirely liquidated
		 highestBid.Amount -= order.Amount
		 if highestBid.Amount == 0 {
			 b.Book.Orders = b.Book.Orders[:orderCount-1]
		 } else {
			 b.Book.Orders[orderCount-1] = highestBid
		 }
		 return remainingSellOrder, liquidatedBuyOrder, gain, true, true
	 }

	 // Not entirely filled
	 gain = highestBid.Amount * highestBid.Price
	 b.Book.Orders = b.Book.Orders[:orderCount-1]
	 remainingSellOrder.Amount -= highestBid.Amount

	 return remainingSellOrder, liquidatedBuyOrder, gain, true, false
 }

 // FillSellOrder SellOrder에 팔려는 수량을 채울 때까지 BuyOrderBook에서 BuyOrder를 찾아 거래한다. 
func (b *BuyOrderBook) FillSellOrder(order Order) (
	remainingSellOrder Order,
	liquidated []Order,
	gain int32,
	filled bool,
) {
	var liquidatedList []Order
	totalGain := int32(0)
	remainingSellOrder = order

	// Liquidate as long as there is match
	for {
		var match bool
		var liquidation Order
		remainingSellOrder, liquidation, gain, match, filled = b.LiquidateFromSellOrder(
			remainingSellOrder,
		)
		if !match {
			break
		}

		// Update gains
		totalGain += gain

		// Update liquidated
		liquidatedList = append(liquidatedList, liquidation)

		if filled {
			break
		}
	}

	return remainingSellOrder, liquidatedList, totalGain, filled
}
```
