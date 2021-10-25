---
description: Go lang에서 쓸만한 메소드 정리.
---

# Useful Methods - Slice

### Slice 내부에 특정 값이 존재하는지 판단하는 메소드

```go
func numberInSlice(a int, list []int) bool {
	for _, b := range list {
		if b == a {
			return true
		}
	}
	return false
}
```

### Slice 내부에 특정 위치의 값을 제거하는 메소드

#### 순서가 상관없을 때

* 제거할 값 자리에 slice의 마지막 값을 대입 후 마지막 값을 제거

```go
// i index의 값을 제거한다. (순서 바)
func remove(s []int, i int) []int {
    s[i] = s[len(s)-1]
    return s[:len(s)-1]
}
```

#### 순서가 유지되어야 할 때

* 값을 제거하고 그 뒤의 모든 값을 shifting한다.

```go
// i index의 값을 제거한다. (순서 유)
func remove(slice []int, s int) []int {
    return append(slice[:s], slice[s+1:]...)
}
```

### Slice 내부에 특정 값을 제거하는 메소드

위의 특정 index의 값을 제거하는 메소드와 연계하여 사용

```go
// s slice에서 가장 먼저 나온 v값을 제거한다. 
func removeVal(s []int, v int) []int {
    for i, val := range s{
        if val == v{
            s = remove(s, i)
            break
        }
    }
    return s
}
```

&#x20;
