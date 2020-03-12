# chpater06. 웹 어플리케이션 작성하기

## Hello, World

```go
func main() {
  http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "Hello, world!")
  })
  log.Fatal(http.ListenAndServe(":8080", nil))
}
```



## Todo app 만들기

- **Taskman** 이라는 이름으로 만든다.

### RESTful API

`GET`, `PUT`, `POST`, `DELETE`

### Data Access Object(DAO)

- 데이터 베이스에 필요한 연산들을 추상 인터페이스로 만들어서 사용하는 것

#### DataAccess 인터페이스

```go
// task를 식별하게 위한 식별자
type ID string

// task 리소스에 접근하기 위한 인터페이스
type DataAccess interface {
  Get(id ID) (task.Task, error)
  Put(id ID, t task.Task) error
  Post(t task.Task) (ID, error)
  Delete(id ID) error
}
```

#### 인터페이스 구현

```go
// 간단한 in-memory 데이터베이스
type MemoryDataAccess struct {
  tasks map[ID]task.Task
  nextID int64
}

func NewMemoryDataAccess() DataAccess {
  return &MemoryDataAccess{
    tasks: map[ID]task.Task{},
    nextID: int64(1),
  }
}
```

- DataAccess 인터페이스를 반환함으로써 받는 쪽에서 유연한 처리 가능
- 현재 상태에서는 컴파일 오류 발생
  - MemoryDataAccess가 DataAccess인터페이스를 구현하고 있지 않기 때문

```go
// ID를 찾지 못햇을 때 발생
var ErrTaskNotExist = errors.New("task does not exists")

// Get
func (m *MemoryDataAccess) Get(id ID) (task.Task, error) {
  t, exists := m.tasks[id]
  if !eixsts {
    return task.Task{}, ErrTaskNotExist
  }
  return t, nil
}

// Put
func (m *MemoryDataAccess) Put(id ID, t task.Task) error {
  if _, exists := m.tasks[id]; !exists {
    return ErrTaskNotExist
  }
  m.tasks[id] = t
  return nil
}

// Post
func (m *MemoryDataAccess) Post(t task.Task) (ID, error) {
  id := ID(fmt.Sprint(m.nextID))
  m.nextID++
  m.tasks[id] = t
  return id, nil
}

// Delete
func (m *MemoryDataAccess) Delete(id ID) error {
  if _, exists := m.tasks[id]; !exists {
    return ErrTaskNotExist
  }
  delete(m.tasks, id)
  return nil
}
```

### RESTful API 핸들러 구현

#### RESTful API 설계(?)

```bash
GET /api/v1/task/{id}

PUT /api/v1/task/{id}
task: {task}

POST /api/v1/task/
task: {task}

DELETE /api/v1/task/{id}
```

#### 요청에 대한 응답

- ID, Task, Error를 묶는 구조체

```go
// JSON 응답을 위한 error
type ResponseError struct {
  Err error
}

// error의 JSON 형태를 리턴
func (err ResponseError) MarshalJSON() ([]byte, error) {
  if err.Err == nil {
    return []byte("null"), nil
  }
  return []byte(fmt.Sprintf("\"%v\"", err.Err)), nil
}

// JSOn 형태의 error를 파싱
func (err *ResponseError) UnmarshalJSON(b []byte) error {
  var v interface{}
  if err := json.Unmarshal(b, v); err != nil {
    return err
  }
  if v == nil {
    err.Err = nil
    return nil
  }
  switch tv := v.(type) {
  case string:
    if tv == ErrTaskNotExist.Error() {
      err.Err = ErrTaskNotExist
      return nil
    }
    err.Err = errors.New(tv)
    return nil
  default:
    return errors.New("ResponseError unmarshal failed")
  }
}
```



- JSON response

```go
type Response struct {
  ID		ID						`json:"id,omitempty"`
  Task	task.Task			`json:"task"`
  Error	ResponseError	`json:"error"`
}
```

### 메인 함수

```go
// FIXME: m is NOT thread-safe.
var m = NewMemoryDataAccess()

const pathPrefix = "/api/v1/task/"

func apiHandler(w http ResponseWriter, r *http Request) {
  ...
}

func main() {
  http.HandleFunc(pathPrefix, apiHandler)
  log.Fatal(http.ListenAndServe(":8887", nil))
}
```

- 전역으로 m을 선언한 것은 좋지 않다.
  - 이 전역 변수에 여러 고루틴이 접근
  - 맵으로 구현되어 있고 스레드 안전하지 않다.
  - 동시성을 다루는 7장에서 다룰 예정

### API Handler 작성

```go
switch r.Method {
case "GET":
  id, err := getID()
  if err != nil {
    logPrintln(err)
    return
  }
  t, err := m.Get(id)
  err = json.NewEncoder(w).Encode(Response{
    ID:		id,
    Task:	t,
    Error:ResponseError{err},
  })
  if err != nil {
    log.Println(err)
  }
case "PUT":
  id, err := getID()
  if err != nil {
    log.Println(err)
    return
  }
  tasks, err := getTasks()
  if err != nil {
    log.Println(err)
    return
  }
  for _, t := range tasks {
    err = m.Put(id, t)
    err = json.NewEncoder(w).Encode(Response{
      ID:		id,
      Task:	t,
      Error:ResponseError{err},
    })
    if err != nil {
      log.Println(err)
      return
    }
  }
case "POST":
  tasks, err := getTasks()
  if err != nil {
    log.Println(err)
    return
  }
  for _, t := range tasks {
    id, err := m.Post(t)
    err = json.NewEncoder(w).Encode(Response{
      ID:		id,
      Task:	t,
      Error:ResponseError{err},
    })
    if err != nil {
      log.Println(err)
      return
    }
  }
case "DELETE":
  id, err := getID()
  if err != nil {
    log.Println(err)
    return
  }
  err = m.Delete(id)
  err = json.NewEncoder(w).Encode(Response{
    ID:		id,
    Error:ResponseError{err},
  })
  if err != nil {
    log.Println(err)
    return
  }
}
```

### HTML 템플릿 작성

- [https://golang.org/pkg/html/template/](https://golang.org/pkg/html/template/) 에서 제공하는 템플릿 엔진 사용

#### htmlHandler

```go
const htmlPrefix = "/task/"

var tmpl = template.Must(template.ParseGlob("html/*.html"))

func htmlHandler(w http.ResponseWriter, r *http.Request) {
  if r.Method != "GET" {
    log.Println(r.Method, "method is not supported")
    return
  }
  getID := func() (ID, error) {
    id := task.ID(r.URL.Path[len(htmlPrefix):])		// prefix path 뒷 부분 추출
    if id == "" {
      return id, erros.New("htmlHandler: ID is empty")
    }
    return id, nil
  }
  id, err := getID()
  if err != nil {
    log.Println(err)
    return
  }
  t, err := m.Get(id)
  err = tmpl.ExecuteTemplate(w, "task.html", &Response{
    ID:		id,
    Task:	t,
    Error:ResponseError{err},
  })
  if err != nil {
    log.Println(err)
    return
  }
}
```



## 코드 리팩토링

## 추가 주제