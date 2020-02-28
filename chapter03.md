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
- :exclamation: 실제 **"앞"**에서 잘라낸 것이랑 **"뒤"**에서 잘라낸 것의 용량이 다르다.
  - **"뒤"**에서 잘라냈다면 뒤의 공간이 남아 있기 때문에 용량은 그대로다.

```go
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
  
  sliced3 := sliced1[:4]
  fmt.Println(sliced3)
  fmt.Println("len:", len(sliced3))
  fmt.Println("cap:", cap(sliced3))
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

### 슬라이스의 내부 구현

- 슬라이스는 배열을 가리키고 있는 구조체
- `시작주소`, `길이`, `용량`으로 구성
- 복사, 이동 발생 시, 다른 배열을 보게 된다. (Immutable 개념인듯..)
  - 따라서 슬라이스에 element 추가/삭제 시, 작업이 끝나고 원래 변수에 재할당 해줘야 함

### 슬라이스 복사

```go
func Example_sliceCopy() {
	src := []int[30, 20, 50, 10, 40]
	dest := make([]int, len(src))
	for i := range src {
		dest[i] = src[i]
	}
	fmt.Println(dest)
	// Output: [30 20 50 10 40]
}
```

### 슬라이스 삽입 및 삭제

#### 슬라이스 i번째에 x 삽입 (1)

```go
if i < len(a) {
	a = append(a[:i], a[i+1:]...)
	a[i] = x
} else {
	a = append(a, x)
}
```

#### 슬라이스 i번째에 x 삽입 (2)

```go
a = append(a, x)
copy(a[i+1:], a[i:])
a[i] = x
```

- 길이 하나를 늘려주기 위해 뒤에 x 붙여주고
- 집어넣을 공간 뒤에 있는 부분을 한 칸씩 뒤로 밀어서 복사
- 해당 부분에 x 삽입

```go
// x := []int[7, 8, 9]
a = append(a, x...)
copy(a[i+len(x):], a[i:])
copy(a[i:], x)
```

- 위 예제는 **여러 개** 넣을 경우이다.

#### 슬라이스 i번째 요소(부터 k개) 삭제 (1)

- 이 방법은 O(n)의 시간 복잡도가 걸림

```go
a = append(a[:i], a[i+1:]...)
```

```go
a = append(a[:i], a[i+k:]...)
```

#### 슬라이스 i번째 요소(부터 k개) 삭제 (2)

- 이 방법은 O(1)의 시간 복잡도가 걸림
- 슬라이스 뒷 부분을 삭제할 요소에 복사

```go
a[i] = a[len(a)-1]
a = a[:len(a)-1]
```

```go
start := len(a)-k
if i+k > start {
	start = i+k
}
copy(a[i:i+k], a[start:])
a = a[:len(a)-k]
```

#### 슬라이스 삭제 시 유의점

- 삭제되는 슬랑이스 내부에 **포인터가 있는 경우**
  - 가비지 컬렉션이 일어나지 않기 때문에 메모리 누수 발생
- 해당 포인터를 `nil` 로 삭제해줘야 한다.

```go
copy(a[i:], a[i+i:])
a[len(a)-1] = nil	// 생략 시 메모리 누수 위험
a = a[:len(a)-1]
```

```go
copy(a[i:], a[i+k:])
for i := 0; i < k; i++ {
	a[len(a)-1-i] = nil
}
a = a[:len(a)-k]
```

- 포인터를 포함한 구조체의 경우에는 nil 대신에 **T{}** (T는 구조체 이름)를 넣어준다.

## 맵

### 정의

- **해시테이블**로 구현

#### 형태

```go
var m map[keyType]valueType
```

#### 초기화

```go
m := make(map[keyType]valueType)

// or

m := map[keyType]valueType{}
```

#### 읽기 / 쓰기 / 삭제

```go
value, ok := m[key]
```

- m[key]를 이용하면, key에 대한 value를 얻을 수 있다.
- 하지만 위와 같이 value와 ok 두 변수를 쓰면
  - **value** : key에 대한 값
  - **ok** : key에 해당하는 값이 있는지의 여부

```go
map[key] = value
```

- key에 값을 쓸 때에는 위와 같이 한다.

```go
delete(m, key)
```

- 삭제

#### 맵 테스트 방법

- **reflect.DeepEqual** 이용
- 가장 간단한 방법

```go
func count(s string, codeCount map[rune]int) {
	for _, r := range s {
		codeCount[r]++
	}
}
```

```go
import "reflect"

func TestCount(t *testing.T) {
	codeCount := map[rune]int{}
	
	count("가나다나", codeCount)
	if !reflect.DeepEqual(
		map[rune]int{'가': 1, '나': 2, '다': 1},
		codeCount,
	) {
		t.Error("codeCount mismatch:", codeCount)
	}
}
```

- 맵의 크기와 각각의 키와 값들을 모두 비교하는 방법

```go
func TestCount(t *testing.T) {
	codeCount := map[rune]int{}
	count("가나다나", codeCount)
	if len(codeCount) != 3 {
		t.Error("codeCount:", codeCount)
		t.Fatal("count should be 3 but:", len(codeCount))
	}
	if codeCount['가'] != 1 || codeCount['나'] != 2 || codeCount['다'] != 1 {
		t.Error("codeCount mismatch:", codeCount)
	}
}
```

- 순서가 자주 변경되는 map의 특성에 따라 순서대로 맵에 접근하는 방법

```go
func ExampleCount() {
	codeCount := map[rune]int{}
	count("가나다나", codeCount)
  var keys sort.IntSlice
  for key := range codeCount {
    keys = append(keys, int(key))
  }
  sort.Sort(keys)
	for _, key := range keys {
		fmt.Println(string(key), codeCount[key])
	}
	// Output:
	// 가 1
	// 나 2
	// 다 1
}
```

### 집합

#### 구현

- Go에서 따로 집합을 제공하지 않음

```go
func hasDupeRune(s sstring) bool {
	runeSet := map[rune]bool{}
	for _, r range s {
		if runeSet[r] {
			return true
		}
		runeSet[r] = true
	}
	return false
}
```

- 위 방법은 불필요한 bool 값을 저장하고 있어 메모리를 차지한다는 단점
- **빈 구조체** 를 값으로 사용

```go
func hasDupeRune(s string) bool {
	runeSet := map[rune]struct{}{}
	for _, r range s {
		if _, exists := runeSet[r]; exists {
			return true
		}
		runeSet[r] = strict{}{}
	}
	return false
}
```

### 맵의 한계

- 같은 키가 여러 번 들어갈 수 있는 맵은 제공하지 않는다.
- 맵을 여러 고루틴에서 동시에 구조를 변경할 수 있다.
  - 스레드 안전하지 않다

