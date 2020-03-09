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

## 코드 리팩토링

## 추가 주제