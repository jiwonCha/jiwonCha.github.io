---
layout: post
title:  "[k8s] Finalizer란 무엇일까"
date:   2021-08-12 00:03:00 +0900
categories: k8s
---

### Background

----------

최근에 업무를 진행하면서, 쿠버네티스에서 CRD 객체를 삭제할 때에 정상적으로 진행되지 않는 경우가 있었는데요

`k8s patch` 명령을 통해서 직접 .metadata.finalizer를 먼저 제거해준 후에 삭제를 진행했더니 정상적으로 삭제됨을 확인할 수 있었습니다

동작은 확인을 했지만, 왜 이렇게 동작하는지는 잘 모르겠더라구요

이번 포스트는 쿠버네티스에서의 finalizer의 역할은 무엇인지 궁금증을 해소하기 위해서 적어보도록 하겠습니다

----------

### kubectl delete

Finalizer를 본격적으로 살펴보기 전에 `kubectl delete`의 매커니즘을 확인할 필요가 있습니다

쿠버네티스에는 객체를 `create`, `get`, `patch`, `delete` 할 수 있는 명령들이 존재합니다

configmap 객체에 대해서 명령들이 어떻게 동작하는지 확인해 보겠습니다

```bash
kubectl create configmap mymap
configmap/mymap created
```

`create`를 통해서 객체를 생성하게 되면 `get` 명령으로 아래와 같이 확인이 가능합니다

```bash
kubectl get configmap/mymap
NAME    DATA   AGE
mymap   0      12s
```

하지만 `delete`를 통해 객체를 삭제한 뒤에는 `get` 명령을 통해 객체의 정보를 얻을 수 없습니다

```bash
kubectl delete configmap/mymap
configmap "mymap" deleted
```

```bash
kubectl get configmap/mymap
Error from server (NotFound): configmaps "mymap" not found
```



