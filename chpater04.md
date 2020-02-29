# Chapter04. 함수

## Go의 함수 특징

- 내부적으로 **스택**으로 구현
  - 아마 다른 언어들도 마찬가지일 것 같음..
- Call by value 만 지원
  - Call by reference는 해당 값이 들어 있는 주소값을 넘겨받아서 그 주소에 있는 값을 변경하여 비슷한 효과를 내는 식으로 구현

## 값 넘겨주고 넘겨받기

### 값 넘기기

```go
func AddOne(nums []int) {
	for i := range nums {
		nums[i]++
	}
}

func ExampleAddOne() {
	n := []int{1, 2, 3, 4}
	AddOne(n)
	fmt.Println(n)
	// Output:
	// [2 3 4 5]
}
```

- 함수의 인자로 값을 넘겨준다.
- 하지만 **슬라이스**는 배열에 대한 `포인터`, `길이`, `용량`을 넘겨준다.
  - 그렇기 때문에 포인터가 가리키고 있는 배열의 변경이 발생

### 값 반환

#### 둘 이상의 반환값

```go
func WriteTo(w io.Writer, lines []string) error
```

- 실제로 몇 바이트를 썼는지 알고 싶을 때 아래와 같이 반환값을 두 개를 써준다.

```go
func WriteTo(w io.Writer, lines []string) (int64, error)
```

### 에러 처리

#### 에러 처리 방법

- Go는 관례상 에러는 **가장 마지막 값**으로 반환
- **패닉(panic)**이라는 개념이 있어서 다른 언어들의 예외(exception)와 같은 것을 제공
  - 일반적 에러 상황 (x)
  - 심각한 에러 상황 (o)
- 에러 관련 코드 작성 시, 코드양이 매우 많아지지만, if 문에서 다음과 같이 쓸 수 있다.

```go
if err := MyFunc(); err != nil {
	...
}
```

#### 새로운 에러 생성 방법

1. errors.New

```go
return erros.New("stringlist.ReadFrom: line is too long")
```

- 이런 에러는 **ReadFrom** 말고도 같은 패키지 안에 있는 다른 읽기 함수에서도 발생하는 경우가 많다.
  - ReadFrom을 빼도 된다.

2. fmt.Errorf

```go
return fmt.Errorf("stringlist: too long line at %d", count)
```

- fmt.Errorf는 다른 부가 정보를 추가한 메시지를 돌려줄 수 있다.

### 가변인자

#### 기본 형식

```go
func f(w io.Writer, nums []int) {
	...
}

func main() {
	...
	f(w, []int{x, y, z})
}
```

#### WriteTo 변형

```go
func WriteTo(w io.Writer, lines... string) (n int64, err error) {}

func main() {
  ...
  WriteTo(w, "hello", "world", "Go language")
}
```

- 원래는 **슬라이스 자체**를 넘겨줬어야 하지만, 위와 같이 **나열**하는 식으로 넘겨주는 것도 가능
- **append**, **fmt에** 많이 쓰이는 방식
- 아래와 같이 슬라이스를 넘겨야 할 때는 점 3개를 붙여서 넘긴다.

```go
lines := []string{"hello", "world", "Go language"}
WriteTo(w, lines...)
```

## 값으로 취급되는 함수

### 함수 리터럴

#### 원래 함수의 형태

```go
func add(a, b int) int {
	return a + b
}
```

#### 순수하게 함수의 값만 표현

```go
func(a, b int) int {
	return a + b
}
```

- 이름이 없어진 순수한 값 형태의 함수를 **함수 리터럴(Function literal)**이라고 한다.

```go
func Example_funcLiteralVar() {
	printHello := func() {
		fmt.Println("Hello!")
	}
	printHello()
	// Output:
	// Hello!
}
```

### 고계 함수 (higher-order function)

- 함수의 함수, 즉 함수를 넘기고 받는 함수

```go
func ReadFrom(r io.Reader, f func(line string)) error {
	scanner := bufio.NewScanner(r)
	for scanner.Scan() {
		f(scanner.Text())
	}
	if err := scanner.Err(); err != nil {
		return err
	}
	return nil
}
```

- r에서 한 줄씩 읽어서 매 줄마다 f 함수 호출

```go
func ExampleReadFrom_Print() {
	r := strings.NewReader("bill\ntom\njane\n")
	err := ReadFrom(r, func(line string) {
		fmt.Println("(", line, ")")
	})
	if err != nil {
		fmt.Println(err)
	}
	// Output:
	// ( bill )
	// ( tom )
	// ( jane )
}
```

### 클로저(closure)

- 외부에서 선언한 변수를 함수 리터럴 내에서 마음대로 접근할 수 있는 코드

```go
func ReadFrom(r io.Reader, f func(line string)) error {
	scanner := bufio.NewScanner(r)
	for scanner.Scan() {
		f(scanner.Text())
	}
	if err := scanner.Err(); err != nil {
		return err
	}
	return nil
}

func ExampleReadFrom_append() {
	r := strings.NewReader("bill\ntom\njane\n")
	var lines []string
	err := ReadFrom(r, func(line string) {
		lines = append(lines, line)
	})
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println(lines)
	// Output:
	// [bill tom jane]
}
```

- ReadFrom 내부에서 외부에 선언된 lines 변수에 한줄씩 추가

### 생성기(generator)

- 함수를 호출할 때마다 증가된 값을 받을 수 있는 생성기 예제

```go
func NewIntGenerator() func() int {
	var next int
	return func() int {
		next++
		return next
	}
}

func ExampleNewIntGenerator() {
	gen := NewIntGenerator()
	fmt.Println(gen(), gen(), gen(), gen(), gen())
	fmt.Println(gen(), gen(), gen(), gen(), gen())
	// Output:
	// 1 2 3 4 5
	// 6 7 8 9 10
}
```

- NewIntGenerator가 반환하는 함수는 클로저
- gen은 하나의 클로저를 반환받음
- 이 클로저 안에서 next 변수가 계속 증가됨

```go
func ExampleNewIntGenerator_multiple() {
	gen1 := NewIntGenerator()
	gen2 := NewIntGenerator()
	fmt.Println(gen1(), gen1(), gen1())
	fmt.Println(gen2(), gen2(), gen2(), gen2(), gen2())
	fmt.Println(gen1(), gen1(), gen1(), gen1())
	// Output:
	// 1 2 3
	// 1 2 3 4 5
	// 4 5 6 7
}
```

### 명명된 자료형(Named Type)

```go
type rune int32
```

- rune 형은 사실 int32의 별칭
- int32는 명명된 자료형에 속한다.
- 위 방법대로 **자료형에 이름을 붙일 수 있다.**

