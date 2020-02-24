# Gin-framework

## 공식 문서

[https://github.com/gin-gonic/gin](https://github.com/gin-gonic/gin)

## Install

```go
import "github.com/gin-gonic/gin"
```

위 코드를 입력 후 `option` + `enter` 를 누른다.(패키지 추가, GoLand 기준)

## Quick Start

> 디렉토리 구성이나 환경 셋팅은 [Chapter02.md]("./chapter02.md") 를 참고

이 부분은 실제 Gin Framework를 이용해서 간단한 JSON 서버를 실행하는 코드이다.

```go
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})
	r.Run() // listen and serve on 0.0.0.0:8080 (for windows "localhost:8080")
}
```

이후 실행해본다. (참고로 GoLand에서는 그냥 **초록색 삼각형** 을 누르면 된다.)

```shell
$ go run example.go
```

"http://localhost:8080/ping"으로 요청을 보내면, 아래와 같은 응답이 온다.

<img src="./img/Gin-framework/quickstart-postman.png" />

