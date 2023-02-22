---
title: Go Routine, Channel
categories: [Go]
tags: [go, routine, channel]     # TAG names should always be lowercase
---

# Go routine

Go routine은 경량 쓰레드(Lightweight thread)로서, OS 쓰레드와는 다르게 작은 메모리를 사용하며 매우 적은 시스템 자원을 필요로 합니다. 이를 통해 Go 프로그램은 매우 빠른 속도로 동시성 작업을 처리할 수 있습니다.

아래 예제와 같이 사용할 수 있습니다.

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	fmt.Println("Start main()")

	go func() {
		fmt.Println("Start goroutine")

		time.Sleep(time.Second * 2)

		fmt.Println("End goroutine")
	}()

	time.Sleep(time.Second * 3)

	fmt.Println("End main()")
}
```

**중요한 것은** Go routine에서 발생하는 패닉은 기본적으로 프로그램을 종료시키지 않으며, 해당 Go routine만 종료됩니다. 따라서, Go routine에서 발생하는 패닉을 처리하기 위해서는 별도의 방법을 사용해야 합니다.



# 채널(Channel)

채널(Channel)을 사용하여 Go routine간 통신을 할 수 있습니다. 채널은 고루틴 간에 데이터를 안전하게 전달하기 위한 도구로 사용됩니다. 채널은 다른 언어에서 사용하는 큐(Queue)나 파이프(Pipe)와 비슷한 역할을 합니다. 채널은 다음과 같은 방식으로 생성됩니다.

```go
func calculateSquare(c chan int) {
	for i := 0; i <= 10; i++ {
		c <- i * i
	}
	close(c)
}

func main() {
	c := make(chan int)

	go calculateSquare(c)

	for i := range c {
		fmt.Println(i)
	}
}
```

## 예제

더 구체적인 예제로서, [노마드 코더](https://nomadcoders.co/go-for-beginners/lectures/1526) 니콜라스 강좌의 `FAST URLChecker` 예제를 참고하면 좋습니다.

```go
package main

import (
	"errors"
	"fmt"
	"net/http"
)

type requestResult struct {
	url    string
	status string
}

var errRequestFailed = errors.New("Request failed")

func main() {
	results := make(map[string]string)
	c := make(chan requestResult)
	urls := []string{
		"https://www.airbnb.com/",
		"https://www.google.com/",
		"https://www.amazon.com/",
		"https://www.reddit.com/",
		"https://www.google.com/",
		"https://soundcloud.com/",
		"https://www.facebook.com/",
		"https://www.instagram.com/",
		"https://academy.nomadcoders.co/",
	}
	for _, url := range urls {
		go hitURL(url, c)
	}

	for i := 0; i < len(urls); i++ {
		result := <-c
		results[result.url] = result.status
	}

	for url, status := range results {
		fmt.Println(url, status)
	}

}

func hitURL(url string, c chan<- requestResult) {
	resp, err := http.Get(url)
	status := "OK"
	if err != nil || resp.StatusCode >= 400 {
		status = "FAILED"
	}
	c <- requestResult{url: url, status: status}
}

```