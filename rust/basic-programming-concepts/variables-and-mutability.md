# Variables and Mutability

## 변수

### let

기본적으로 Rust의 모든 변수는 불변하다. 하지만 mut 키워드를 붙임으로써 가변한 변수를 만들 수 있다.&#x20;

```
let mut x = 5;
println!("The value of x is: {}", x);
x = 6;
println!("The value of x is: {}", x);e
```



### const
