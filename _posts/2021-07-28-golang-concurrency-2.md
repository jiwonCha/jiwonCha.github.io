---
layout: post
title:  "[Go] 동시성 접근을 위한 Concurrent Map"
date:   2021-07-28 00:20:00 +0900
categories: Go Resource
---

### Concurrent Map

----------

다양한 go routine에서 동시적으로 `map`에 접근해야 하는 경우가 생기기 때문에, 이때에 발생하는 concurrent access를 safe하게 처리하기 위해 `concurrent map`을 구성할 필요가 있습니다

여기서는 `uint64`형의 key를 가지고, value는 다양한 type으로 받을 수 있는 `ConcurrentMap` struct를 생성해 보겠습니다

```go
    // ConcurrentMap struct
    type ConcurrentMap struct {
        sync.RWMutex
        items map[uint64]interface{}
    }

    // ConcurrentMapItem struct
    type ConcurrentMapItem struct {
        Key   uint64
        Value interface{}
    }
```

동시적인 접근을 제어해야 하기 때문에 내부적으로 `sync.RWMutex`를 선언했습니다

`sync.RWMutex`는 golang의 Mutex struct로서 내부에는 Lock(), Unlock() 의 두가지 function을 제공합니다

기본적인 CRUD 기능은 아래처럼 구현이 가능합니다

공통적인 패턴은 Lock() 함수를 호출하고 Unlock() 함수 앞에 `defer`를 호출하여 원하는 동작을 수행한 이후 안전하게 Unlock()이 호출되도록 하는 것입니다

```go
    // Set is for setting map item
    func (cm *ConcurrentMap) Set(key uint64, value interface{}) {
        cm.Lock()
        defer cm.Unlock()

        cm.items[key] = value
    }

    // Get is for getting map item
    func (cm *ConcurrentMap) Get(key uint64) (interface{}, bool) {
        cm.Lock()
        defer cm.Unlock()

        value, ok := cm.items[key]

        return value, ok
    }

    // Remove is for removing map item
    func (cm *ConcurrentMap) Remove(key uint64) {
        cm.Lock()
        defer cm.Unlock()

        delete(cm.items, key)
    }
```

map item을 순회하는 기능은 아래처럼 구현할 수 있습니다

Iter() 함수를 호출하면 map item을 channel 형태로 리턴합니다

```go
    // Iter is for iterating map item
    func (cm *ConcurrentMap) Iter() <-chan ConcurrentMapItem {
        c := make(chan ConcurrentMapItem)

        go func() {
            cm.Lock()
            defer cm.Unlock()

            for k, v := range cm.items {
                c <- ConcurrentMapItem{k, v}
            }
            close(c)
        }()

        return c
    }
```

Iter()를 사용하려면 recv channel을 이용할 때 처럼 사용하면 됩니다

모든 item을 다 순회한 이후에는 `close(c)`로 channel이 닫히게 되면서 순회가 종료됩니다

```go

itemChan := Iter()

for {
    item := <-itemChan
    if item {
        fmt.Println(item.Key, item.Value)
    }
}
```
