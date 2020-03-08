# Chapter05. 구조체

## 구조체

- 서로 다른 자료형의 자료들을 묶어 놓은 것

### 사용법

```go
// 정의
var Task = struct {
  title string
  done bool
  due *time,Time
}

// 선언
var myTask Task
```

> :exclamation: 보통은 아래와 같이 사용

```go
var myTask = Task{
  title: "laundry",
  done: true,
}
```

### const와 iota

#### 확장성을 위해 bool형을 쓸 곳에 enum형을 쓰자

```go
const UNKNOWN status = 0
const TODO status = 1
const DONE status = 2
```

위와 같이 상수형으로 정의해서 쓰는 것을 묶는다.

```go
const (
	UNKNOWN	status = 0
  TODO		status = 1
  DONE		status = 2
)
```

`iota`를 이용하여 0,1,2...를 생략 할 수 있다.

```go
const (
	UNKNOWN	status = iota
  TODO		status = iota
  DONE		status = iota
)
```

> 한 번만 써 줘도 됨

```go
const (
	UNKNOWN	status = iota
  TODO
  DONE
)
```

#### Example

- [https://golang.org/doc/effective_go.html#constants](https://golang.org/doc/effective_go.html#constants)

```go
type ByteSize float64

const (
    _           = iota // ignore first value by assigning to blank identifier
    KB ByteSize = 1 << (10 * iota)
    MB
    GB
    TB
    PB
    EB
    ZB
    YB
)
```

### 테이블 기반 테스트

- 여러 사례를 테스트할 때
- 구조체와 배열을 이용하여 테이블 기반 테스트

```go
func TestFib(t *testing T) {
  cases := []struct {
    in, want int
  }{
    {0, 0},
    {5, 5},
    {6, 8}
  }
  
  for _, c := range cases {
    got := seq.Fib(c.in)
    if got != c.want {
      t.Errorf("Fib(%d) == %d, want %d", c.in, got, c.want)
    }
  }
}
```

- cases라는 구조체 배열 선언
  - (input, output) 형태로 사례를 대입
- **if문을 이용한 테스트**가 관습

### 구조체 내장

- 다른 언어에 비해 go의 구조체가 가지는 내장 기능은 별거 없음

#### Example

```go
type Address struct {
  City string
  State string
}

type Telephone struct {
  Mobile string
  Direct string
}

type Contact struct {
  Address
  Telephone
}

func ExampleContact() {
  var c contact
  c.Mobile = "123-456-789"		//c.Mobile에 접근했지만, c.Telephone.Mobile에 접근한 것과 동일
  fmt.Println(c.Telephone.Mobile)
  c.Address.City = "San Francisco"
  c.State = "CA"
  c.Direct = "N/A"
  fmt.Println(c)
  // Output:
  // 123-456-789
  // {{San Francisco CA} {123-456-789 N/A}}
}
```

- 상속과 같은 개념과는 달리 실제로는 **내부에 필드로 내장하고 있으면서 편의를 제공하는 것뿐**

## 직렬화(Serialization)와 역직렬화(Deserialization)

#### 직렬화(Serialization)?

- 객체의 상태를 보관이나 전송 가능한 상태로 변환하는 것

#### 역직렬화(Deserialization)?

- 직렬화의 반대로 보관되거나 전송받은 것을 다시 객체로 복원하는 것

### JSON(JavaScript Object Notation)

- 자료 고환 형식 중에 하나
- 자바스크립트에서 객체를 표현하는 형식과 비슷
- XML에 비해 사람이 읽기 쉽고 간단하기 때문에 널리 사용

#### JSON 직렬화 및 역직렬화

##### 직렬화

```go
func Example_marshalJSON() {
  t := Task{
    "Laundry",
    DONE,
    NewDeadline(time.Date(2015, time.August, 16, 15, 43, 0, 0, time.UTC)),
  }
  b, err := json.Marshal(t)
  if err != nil {
    log.Println(err)
    return
  }
  fmt.Println(string(b))
  // Output:
  // {"Title":"Laundry","Status":2,"Deadline":"2015-08-16T15:43:00Z"}
}
```

- `json.Marshal` 함수 호출 (역직렬화는 `json.Unmarshal` 함수 호출)
- **대문자로 시작하는 필드**들만 JSON으로 직렬화
  - 직렬화 하기 싫은 필드들은 **소문자**로 시작하게 하면 된다.

##### 역직렬화

```go
func Example_unMarshalJSON() {
  b := []byte(`{"Title":"Buy Milk","Status":2,"Deadline":"2015-08-16T15:43:00Z"}`)
  t := Task{}
  err := json.Unmarshal(b, &t)
  if err != nil {
    log.Println(err)
    return
  }
  fmt.Println(t.Title)
  fmt.Println(t.Status)
  fmt.Println(t.Deadline.UTC())
  // Output:
  // Buy Milk
  // 2
  // 2015-08-16 15:43:00 +0000 UTC
}
```

- 포인터를 이용해야 수정된 것이 반영되므로 t 대신에 &t 사용
- 문자열 안에 따옴표가 있기 때문에 복잡해 보일 수 있으므로  (\\") 대신에 (`) 사용

#### JSON 태그

- 기본 직렬화 필드들을 변형하고 싶을 때 사용
  - 이름 변경
  - 필드 생략

```go
type MyStruct struct {
  Title string `json:"title"`
  Internal string `json:"-"`
  Value int64 `json:",omitempty"`
  ID int64 `json:",string"`
}
```

- Title 대신에 title을 필드로 사용
- Internal 필드는 JSON에는 나타나지 않고 무시
- Value필드는 0일 경우 JSON 결과를 내지 않음
- ID는 JSON에서는 문자열로 나타냄
  - :exclamation: 참고로 웹 애플리케이션에서는 자바스크립트의 숫자형은 8바이트 실수형이기 때문에 정수값은 53 비트를 넘어서면 정확도가 떨어진다. 따라서 \`json:,string\` 을 해주는 습관을 기르는 것이 좋다.

#### JSON 직렬화 사용자 정의

```go
// MarshalJSON implements the json.Marshaler interface
func (s status) MarshalJSON() ([]byte, error) {
  switch s {
  case UNKNOWN:
    return []byte(`"UNKNOWN"`), nil
  case TODO:
    return []byte(`"TODO"`), nil
  case DONE:
    return []byte(`"DONE"`), nil
  default:
    return nil, errors.New("status.MarshalJSON: unknown value")
  }
}

// UnmarshalJSON implements the json.Unmarshaler interface
func (s *status) UnmarshalJSON(data []byte) error {
  switch string(data) {
  case `"UNKNOWN"`:
    *s = UNKNOWN
  case `"TODO"`:
    *s = TODO
  case `"DONE"`:
    *s = DONE
  default:
    return errors.New("status.UnmarshalJSON: unknown value")
  }
  return nil
}
```

#### Gob

- Go에서 기본으로 제공하는 또 다른 직렬화 방식
- Go에서만 쓸 수 있다.
- JSON과 다른 점
  - gob.NewEncoder, gob.NewDecoder로 인코더와 디코더를 생성해서 사용
  - io.Writer와 io.Reader를 넘긴다.
  - bytes.Buffer를 넘기게 되면 []byte를 뽑아낼 수 있다.

```go
func Example_gob() {
  var b bytes.Buffer
  enc := gob.NewEncoder(&b)
  data := map[string]string{"N": "J"}
  if err := enc.Encode(data); err != nil {
    fmt.Println(err)
  }
  const width = 16
  for start := 0; start < len(b.Bytes()); start += width {
    end := start + width
    if end > len(b.Bytes()) {
      end = len(b.Bytes())
    }
    fmt.Printf("%x\n", b.Bytes()[start:end])
  }
  dec := gob.NewDecoder(&b)
  var restored map[string]string
  if err := dec.Decode(&restored); err != nil {
    fmt.Println(err)
  }
  fmt.Println(restored)
  // Output:
  // 0e ff 81 04 01 02 ff 82 00 01 0c 01 0c 00 00 08
  // ff 82 00 01 01 42 01 4a
  // map[N;J]
}
```

