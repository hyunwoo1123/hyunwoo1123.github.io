---
title: Go의 defer, panic, recover
categories: [Go]
tags: [go,panic,recover,defer,chatgpt_used]     # TAG names should always be lowercase
---
*본 문서의 예제 코드는 ChatGPT가 만들었습니다.*

### GO의 defer

해당 함수가 종료된 뒤 수행되는 기능. 예제 참고.

```
package main

import "fmt"

func main() {
    defer fmt.Println("World")
    fmt.Println("Hello")
}
```

위 코드의 경우 main.go를 실행 시, Hello를 출력하는 부분이 수행되고 난 후, main이 종료될 때 마지막으로 defer 내의 fmt.Println("World")가 실행된다.
