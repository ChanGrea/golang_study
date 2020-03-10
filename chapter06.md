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



## 코드 리팩토링

## 추가 주제