---
title: Go의 http 통신
categories: [Go]
tags: [go,http]     # TAG names should always be lowercase
published: false

---

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