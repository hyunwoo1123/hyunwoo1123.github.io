---
title: Go의 http 통신
categories: [Go]
tags: [go,http,chatgpt_used]     # TAG names should always be lowercase
---
*본 문서의 예제 코드는 ChatGPT가 만들었습니다.*

# GO의 http 통신

```go
package main

import (
    "fmt"
    "net/http"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintln(w, "Hello, world!")
    })

    err := http.ListenAndServe("localhost:8080", nil)
    if err != nil {
        panic(err)
    }
}
```