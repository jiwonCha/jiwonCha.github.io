---
layout: post
title:  "[Go Concurrency] Select 구문"
date:   2021-07-25 15:01:00 +0900
categories: Go Resource
---

## [Go Concurreny] Select / Timeout and Deadline

----------

### Select

golang concurrency를 control 할 수 있는 structure를 의미합니다

`select`는 여러 channel들을 다룰 수 있는 방법 중 하나로서, 얼핏 `swtich`와 비슷한 것 같지만 각각의 case문이 모두 communication이 라는 데에 차이가 있습니다

`default` 구문은 select에 명시 된 모든 channel이 ready 되지 않았을 때 실행됩니다

```go
select {
    case v1 := <-c1:
        fmt.Println("Recevied v1 : %v from c1", v1)
    case c2 <- 22:
        fmt.Println("Send to c2 : %v", 22)
    default:
        fmt.Println("Nothing Ready to Start")
}
```

### Timeout

channel에 너무 오랫동안 blocking 되는 것을 방지하기 위하여 timeout을 지정해놓을 수 있습니다

```go
timeout := time.After(10* time.Seconds)
select {
    case v1 := <-c1:
        fmt.Println("Recevied v1 : %v from c1", v1)
    case <-timeout:
        fmt.Println("So Longggggg blocking")
        return
}
```

### Quit

`select`문을 통해 더 이상 communication을 지속할 필요가 없을 때는 통신을 종료하기 위한 channel을 하나 두어 quit 기능을 구현

```go
quit := make(chan bool)
quit <- true
```

```go
select {
    case v1 := <-c1:
        fmt.Println("Recevied v1 : %v from c1", v1)
    case <-quit:
        fmt.Println("Commnication Quit")
        return
}
```
