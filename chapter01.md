# Chapter01. 시작하기

## Go 언어 소개

- 범용 프로그래밍 언어로, 깔끔하고 간결하게 생산성 높은 프로그래밍이 가능
- 빠른 컴파일 & 가비지 컬렉션
- 정적 자료형 언어
- 동시성 지원 코드 작성 가능

### 생산성 높은 이유

- 부분적이지만, **편리한 자료형 추론** 제공 :point_right: 반복해서 자료형 이름을 쓰지 않아도 된다.
- 소스 코드 형식을 자동으로 맞춰주는 도구와 편리한 도구가 기본으로 제공
- 쉽게 **테스트 코드**를 작성하면서 코드 문서화까지 할 수 있다.
- **함수 리터럴** 및 **클로저** 자유자재 사용 가능
- 명시적으로 인터페이스를 지정하지 않아도 **인터페이스** 구현 가능, 기존에 있던 코드를 고치지 않고도 유연한 구현이 가능하다.
- **채널**을 이용하여 동시성 구현을 락 등을 이용하지 않고 간편하게 할 수 있고, 언어 고유의 자원으로 교착 상태나 경쟁 상태를 파악하기 쉽다.
- **컴파일 속도가 빨라서** 컴파일 및 테스트를 반복적으로 수행하면서 코드를 작성하기 용이하다.
- **가비지 컬렉션 지원**으로 메모리 관리에 대한 부담을 덜 수 있다.
- 자료형 리터럴을 쉽게 쓸 수 있다.

[공식 홈페이지 http://golang.org](http://golang.org), [A Tour of Go](https://tour.golang.org) - Go 언어의 기본을 쉽게 배울 수 있는 곳

## Hello world

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, playground")
}
```

- Go 컴파일러는 세미콜론으로 구분된 코드를 해석
- 하지만 위에서는 세미콜론이 안 붙었는데, 구문 분석기가 소스 코드를 스캔하는 과정에서 단순한 규칙을 적용하여 자동으로 붙인다.
- :exclamation:줄이 끝난 것처럼 보이기만 해도 세미콜론이 붙는다. (여러 줄 나눠 쓸 때 주의해야 함)

따라서 아래와 같이 쓸 수 없다.

```go
// ERROR!
func main()
{
    fmt.Println("hello, playground")
}
```

## 자료형 및 변수

- 정적 자료형 지원
- 자료형 추론 기능

### 변수 선언

#### 일반 변수

```go
var x int
```

- "변수 x는 int형이다."

#### 배열

```go
var arr [5]int
```

#### 함수

```go
func(int, int)  // 인자로 두 정수를 받는 함수
func(int) int   // 인자로 정수 하나를 받고, 정수를 리턴하는 함수
func(int, func(int, int)) func(int) int
```

#### 포인터

```go
var p *int
```

#### 변수 선언과 동시에 값 할당

```go
var x int = 10
```

### 자료형 추론

```go
var i = 10
var p = &i  // i의 주소값을 p에 저장
```

> = 대신에 := 쓰면 var 생략 가능

```go
i := 10
p := &i
```

```go
i := 10         // 가능, i는 새로운 정수형 변수
s := "hello"    // 가능, s는 새로운 문자열 변수
i = 20          // 가능, i의 값을 변경
j = 30          // 불가능, j는 선언되지 않은 변수
i = "hello"     // 불가능, 이미 i는 정수형임
i := 30         // 불가능, 이미 i는 선언되었음
i := "hi"       // 불가능, 마찬가지 이유
```

## 함수와 간단한 제어 구조

### 팩토리얼

```go
// Package main implements factorial.
package main

import "fmt"

//fac returns n!.
func fac(n int) int {
    if n <= 0 {
        return 1
    }
    return n * fac(n-1)
}

func main() {
    fmt.Println(fac(5))
}
```

이것을 반복문으로 구현하면,

```go
// Package main implements factorial.
package main

import "fmt"

// facItr return n!.
func facItr(n int) int {
    result := 1
    for n > 0 {
        result *= n
        n--
    }
    return result
}

func main() {
    fmt.Println(facItr(5))
}
```

다른 언어의 for문처럼 쓸 수도 있다.

```go
// facItr2 returns n!.
func facItr2(n int) int {
    result := 1
    for i := 2; i <= n; i++ {
        result *= i
    }
    return result
}
```
