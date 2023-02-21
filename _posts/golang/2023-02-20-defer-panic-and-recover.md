---
title: Go의 defer, panic, recover
categories: [Go]
tags: [go,panic,recover,defer,chatgpt_used]     # TAG names should always be lowercase
---
*본 문서의 예제 코드는 ChatGPT가 만들었습니다.*

# defer

해당 함수가 종료된 뒤 수행되는 기능. 예제 참고.

```go
package main

import "fmt"

func main() {
    defer fmt.Println("World")
    fmt.Println("Hello")
}
```

위 코드의 경우 main.go를 실행 시, Hello를 출력하는 부분이 수행되고 난 후, main이 종료될 때 마지막으로 defer 내의 fmt.Println("World")가 실행됩니다.


# panic

`panic`은 Go 언어에서 예기치 않은 오류가 발생하였을 때 프로그램 실행을 멈추고, 프로그램을 강제로 종료하는 방법 중 하나입니다. `panic` 함수를 호출하면, 해당 함수를 즉시 종료하고, 현재 함수에서 발생한 패닉 메시지를 포함한 오류 정보를 출력합니다.

`panic` 함수가 호출되면, 프로그램은 패닉이 발생한 곳에서부터 호출 스택을 거슬러 올라가며, 해당 패닉을 처리하는 [recover](#recover) 함수를 찾습니다. 만약, [recover](#recover) 함수를 찾지 못하면, 패닉이 발생한 함수를 종료하고, 그 함수를 호출한 상위 함수에서 패닉 처리를 시도합니다. 이러한 과정을 반복하면서, 패닉이 발생한 함수의 호출 스택에 있는 모든 함수들이 종료되고, 프로그램은 강제로 종료됩니다.

Go 언어에서는 `panic` 함수를 가능한 적게 사용하도록 권장합니다. 

`panic` 함수는 예상치 못한 오류 상황에서 프로그램의 실행을 중지시키는 것이므로, 대부분의 상황에서는 **error** 값을 반환하는 것이 더 나은 방법입니다. 

이를테면, `fmt.Errorf` 함수를 사용하여 오류 메시지를 생성하고, 이를 반환하는 방법을 사용할 수 있습니다. **error** 값을 반환하는 방법은 [recover](#recover) 함수를 사용하지 않아도 되므로, 프로그램의 안정성과 가독성을 향상시키는 데 도움이 됩니다.

예를 들어, `panic`에 대한 가장 간단한 Go 예제를 작성해 본다면,

```go
func divide(a, b int) int {
    if b == 0 {
        panic("division by zero")
    }
    return a / b
}

```

위와 같이 작성할 수 있습니다. **그러나** 안전한 방법은, 아래 코드와 같이 Error을 정의하고 문제가 발생하면 해당 Error을 return하여 오류를 처리하는 것입니다.

```go
// private divisionError struct 정의
type divisionError struct {
    dividend int
    divisor  int
}

// divisionError를 string형태 정의
func (e divisionError) Error() string {
    return fmt.Sprintf("division by zero: %d / %d", e.dividend, e.divisor)
}

// divide 함수 정의
func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, divisionError{a, b}
    }
    return a / b, nil
}

// 메인 함수
func main() {
    fmt.Println(divide(10, 2)) // 5, <nil>
    result, err := divide(10, 0)
    if err != nil {
        fmt.Println(err)
    } else {
        fmt.Println(result)
    }
}

```

이제 main 함수에서 divide 함수를 호출하여, 10을 2로 나눈 결과와 10을 0으로 나눈 결과를 출력합니다. 

10을 2로 나눈 경우에는 5가 출력되고, nil 값이 반환됩니다. 

10을 0으로 나눈 경우에는 divisionError 타입의 값이 반환되며, 이를 사용하여 "division by zero: 10 / 0"라는 메시지를 출력합니다.





# recover

물론 권장되진 않지만 위 [panic](#panic)의 간단한 예제,

```go
func divide(a, b int) int {
    if b == 0 {
        panic("division by zero")
    }
    return a / b
}

```

에서 panic이 발생했을 때 이를 처리하는 방법은, `recover`을 이용하는 것입니다.

```go
func divide(a, b int) int {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered from panic:", r)
        }
    }()
    if b == 0 {
        panic("division by zero")
    }
    return a / b
}

```

`recover`는 Go 언어에서 `panic` 함수로 인해 발생한 패닉 상황을 복구할 때 사용하는 함수입니다. `recover` 함수를 사용하면, 프로그램은 패닉이 발생한 곳에서부터 호출 스택을 거슬러 올라가며, 패닉을 처리하는 `recover` 함수를 찾습니다.

`recover` 함수는 반드시 `defer` 구문 안에서 호출해야 합니다. 이는 `defer` 구문을 사용하여 패닉이 발생한 함수의 스택 프레임을 제거하고, 제어를 상위 함수로 다시 전달하는 방식으로 패닉을 복구하기 때문입니다.

`recover` 함수는 패닉이 복구될 때, `panic` 함수에서 전달된 메시지를 반환합니다. 만약, 패닉이 발생하지 않았거나, 패닉이 복구되지 않았을 경우, `recover` 함수는 `nil`을 반환합니다. 따라서, `recover` 함수의 반환값을 사용하여, 패닉이 발생했는지 여부를 확인할 수 있습니다.

Go 언어에서는 `recover` 함수를 가능한 적게 사용하도록 권장합니다. `recover` 함수는 프로그램의 실행을 재개하는 것이므로, 패닉이 발생한 원인을 해결하지 않는 한, 프로그램의 안정성에 영향을 미칠 수 있습니다. 따라서, 대부분의 경우에는 `panic` 함수를 호출하지 않거나, `error` 값을 반환하는 것이 더 나은 방법입니다.




