---
layout: post
title:  "Measure Golang CPU Usage & Runtime Memory"
date:   2019-11-19 23:44:27 +0900
categories: Go
---

----------

오늘의 포스팅 주제는 golang에서 Runtime 시의 CPU, 메모리 사용량을 어떻게 측정할 수 있을까입니다.

우선, Golang에서는 Testing 패키지에서 BenchMark 방법을 제공합니다.
*(BenchMark : 실존하는 비교 대상을 두고 하드웨어나 소프트웨어 성능을 비교하여 시험하고 평가하는 일 )*

흔히 Goalng에서 Test 코드를 작성할 때 TestXX라고 명시하는 것처럼, Benchmark 방법으로 Testing을 진행하기 위해서는 BenchmarkXXX라는 이름으로 Test를 작성하면 됩니다.

아래의 예시처럼요.

```Golang
func BenchmarkHello(b *testing.B) {
    for i := 0; i < b.N; i++ {
        fmt.Sprintf("hello")
    }
}
```

여기서 지켜야 할 규칙은 다음과 같습니다.

- BenchmarkXXX 이름으로 testing 함수를 정의
- testing.B 형태로 parameter 정의
- for i := 0; i < b.N; i++ {} 안에 테스트하고 싶은 함수를 위치 시킴.

Benchmark 방식으로 go test 커맨드를 실행하려면 아래와 같이 인자를 추가해줘야 합니다.

> $ go test -bench=.

그러면 아래와 같은 결과가 나옵니다. 

>``BenchmarkHello    10000000    282 ns/op``

이것은 BenchmarkHello라는 Testing method의 for roof를 10000000번 실행했으며, 한번의 operation을 실행하는데 282ns(나노초)가 걸렸다는 의미 입니다.

----------

그러면 이제 본격적으로 어떻게 CPU, 메모리 사용량을 측정할 수 있는지 알아보겠습니다.

저는 main 함수를 테스트하기 위해서 아래와 같이 Benchmark Testing Method를 구성했습니다.

```Golang
import "testing"

func BenchmarkMain(b *testing.B) {
    for i := 0; i < b.N; i++ {
        main()
    }
}
```

1. Runtime 시 CPU 사용량 측정
2. Runtime 시 Memory 사용량 측정
