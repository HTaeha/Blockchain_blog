# Variables and Mutability

## 변수

### let

기본적으로 Rust의 모든 변수는 불변하다. 하지만 mut 키워드를 붙임으로써 가변한 변수를 만들 수 있다.

아래 코드에서 mut를 붙이지 않으면 에러가 발생한다.

```rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

대규모 데이터 구조체를 다루는 경우 가변한 인스턴스를 사용하는 것이 새로 인스턴스를 할당하고 반환하는 것보다 빠를 수 있다. 데이터 규모가 작을수록 새 인스턴스를 생성하고 함수적 프로그래밍 스타일로 작성하는 것이 더 합리적이고, 그렇기에 약간의 성능 하락을 통해 가독성을 확보할 수 있다면 더 가치있는 선택이다.

### const

상수를 사용할 때는 const 키워드를 사용한다.

값의 유형을 선언해야 한다.

상수는 불변성 그 자체이므로 mut이 허용되지 않는다.

## Shadowing

앞서 선언된 변수와 같은 이름의 변수를 새로 선언함으로써 앞의 변수를 지우고 새로운 값을 가진 변수를 선언할 수 있다. Rustacean들은 이를 첫 변수가 두번째에 의해 shadowed 됐다고 표현한다.

```rust
fn main() {
    let x = 5;
    let x = x + 1;
    let x = x * 2;

    println!("The value of x is: {}", x);
}
```

#### mut으로 선언하는 것과의 차이

shadow를 사용하게 되면 몇 번 값을 변경할 수는 있지만 결국 불변한 변수를 얻을 수 있다.

값의 유형을 변경하면서도 동일 이름을 사용할 수 있다.

```rust
ex)
let spaces = "   ";
let spaces = spaces.len();
```

첫 spaces는 문자열이고 두번째는 숫자이다. 이렇게 값의 유형이 변경되면서도 실제로 저장하고자 했던 공백의 개수를 표현할 수 있다. (space\_str, space\_num 과 같이 대체된 이름을 사용할 필요가 없다.)

```rust
let mut spaces = "   ";
spaces = spaces.len();
```

위 코드와 같은 형태로 mut을 사용했다면 타입의 변화로 에러가 발생한다.
