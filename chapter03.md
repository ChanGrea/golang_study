# chapter03. 문자열 및 자료구조

## 문자열

### 유니코드 처리

#### Go 언어의 소스 코드는 **UTF-8**로 되어 있다.

- 한글로 코딩 가능

  ```go
  for i, r := range "가나다" {
  	fmt.Println(i, r)
  }
  fmt.Println(len("가나다"))
  
  // 0 44032
  // 3 45208
  // 6 45796
  // 9
  ```

- 한글은 3byte

- 위 코드에서 한글을 출력하고 싶으면 아래와 같이 수정

  ```go
  for _, r := range "가갛힣" {
      fmt.Println(string(r), r)
  }
  
  // 가 44032
  // 갛 44059
  // 힣 55203
  ```

- **44059 - 44032 = 27**

  - 이것이 의미하는 것은 "가"에서 "갛"까지 이  두 글자를 모두 포함하여 28자가 된다.

#### 한글 받침 유무 판별 코드 (hangul.go)

```go
package hangul

var (
	start = rune(44032) // "가"의 유니코드 포인트
	end   = rune(55204) // "힣"의 유니코드 포인트
)

func HasConsonantSuffix(s string) bool {
	numEnds := 28
	result := false
	for _, r := range s {
		if start <= r && r < end {
			index := int(r-start)
			result = index%numEnds != 0
		}
	}
	return result
}
```

### Example 테스트

위에서 만든 한글 받침 유무 판별 코드를 테스트한다.

#### 테스트 코드 작성 (hangul_test.go)

```go
package hangul

import "fmt"

func ExampleHasConsonantSuffix() {
	fmt.Println(HasConsonantSuffix("Go 언어"))
	fmt.Println(HasConsonantSuffix("그럼"))
	fmt.Println(HasConsonantSuffix("우리 밥 먹고 합시다"))
	// Output:
	// .
}
```

- :exclamation: 함수 이름은 반드시 **Example**로 시작해야 한다.
- **Output:** 아래는 출력된 결과를 쓰는 부분
  - 점(.) 하나만 찍었으니 당연히 테스트 결과는 fail

#### 테스트 실행

```bash
$ go test github.com/ChanGrea/test/hangul
```

### 패키지 문서

- [https://golang.org/pkg/](https://golang.org/pkg/) 를 열고 작업하는 것이 좋다.
- **godoc** 거쳐 위 사이트 같은 문서가 됨
- 패키지 주석의 경우 `Package`라는 단어로 시작하는 것이 관습
  - "Package regex implements ..."
- 한글보다는 **영어**로 작성

### 문자열 잇기

- 문자열은 읽기 전용
- 플러스(+) 연산을 통해 붙일 수 있긴 한데, 이것은 새로운 문자열을 생성하는 것
- fmt.Sprint(s, "def")
- s = fmt.Sprintf("%sdef", s)
- s = strings.Join([]string{s, "def"}, " ")

## 배열과 슬라이스

### 배열

- 연속된 메모리 공간을 순차적으로 이용하는 자료구조

```go
func Example_array() {
	fruits := [3]string{"사과", "바나나", "토마토"}
	for _, fruit := range fruits {
		fmt.Printf("%s는 맛있다,\n", fruit)
	}
	// Output:
	// 사과는 맛있다.
	// 바나나는 맛있다.
	// 토마토는 맛있다.
}
```

- 컴파일러가 배열의 개수를 알아내어서 넣게 만들고 싶으면 **[3] 대신에 [...]**을 이용하면 된다.
  - fruits := [...]string["사과", "바나나", "토마토"]

### 슬라이스

- 배열은 자주 쓰이지 않음
- 슬라이스는 배열보다 유연한 구조
- 배열은 크기가 자료형에 고정, 반면에  슬라이스는 **길이와 용량을 갖고 있고 길이가 변할 수 있는 구조**



```go
// 빈 문자열 슬라이스
var fruits string[]

// 빈 스트링 n개 갖고 있는 슬라이스
fruits := make([]string, n)
```

- 빈 슬라이스에는 nil 값이 들어간다.
  - C++의 nullptr, 자바의 null과 비슷

### 슬라이스 자르기

```go
func Example_slicing() {
	nums := []int{1, 2, 3, 4, 5}
	fmt.Println(nums)
	fmt.Println(nums[1:3])
	fmt.Println(nums[2:])
	fmt.Println(nums[:3])
	// Output:
	// [1, 2, 3, 4, 5]
	// [2, 3]
	// [3, 4, 5]
	// [1, 2, 3]
}
```

### 슬라이스 덧붙이기

#### 하나 덧붙이기

```go
fruits = append(fruits, "포도")
```

#### 여러개 덧붙이기

```go
fruits = append(fruits, "포도", "딸기")
```

```go
func Example_append() {
	f1 := []string{"사과", "바나나", "토마토"}
	f2 := []string{"포도", "딸기"}
	f3 := append(f1, f2...)
	f4 := append(f1[:2], f2...)
	fmt.Println(f1)
	fmt.Println(f2)
	fmt.Println(f3)
	fmt.Println(f4)
	// Output:
	// [사과 바나나 토마토]
	// [포도 딸기]
	// [사과 바나나 토마토 포도 딸기]
	// [사과 바나나 포도 딸기]
}
```

### 슬라이스 용량

- 실제 슬라이스에 **얼마나 넣을 수 있느냐**에 대한 것
- **cap(x)** 이용

```
func Example_sliceCap() {
	nums := []int{1, 2, 3, 4, 5}
	
	fmt.Println(nums)
	fmt.Println("len:", len(nums))
	fmt.Println("cap:", cap(nums))
	fmt.Println()
	
	sliced1 := nums[:3]
	fmt.Println(sliced1)
	fmt.Println("len:", len(sliced1))
	fmt.Println("cap:", cap(sliced1))
	fmt.Println()
	
	sliced2 := nums[2:]
	fmt.Println(sliced2)
	fmt.Println("len:", len(sliced2))
	fmt.Println("cap:", cap(sliced2))
	fmt.Println()
	
	nums[2] = 100
	fmt.Println(nums, sliced1, sliced2, sliced3)
	// Output:
	// [1 2 3 4 5]
	// len: 5
	// cap: 5
	//
	// [1 2 3]
	// len: 3
	// cap: 5
	//
	// [3 4 5]
	// len: 3
	// cap: 3
	//
	// [1 2 3 4]
	// len: 4
	// cap: 5
	//
	// [1 2 100 4 5] [1 2 100] [100 4 5] [1 2 100 4]
}
```

