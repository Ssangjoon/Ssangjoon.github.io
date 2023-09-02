---
title: Vue ref에 value를 붙이는 이유
date: 2023-09-02 00:00:30 +0900
categories: [Vue]
tags: [study, Vue]     # TAG names should always be lowercase
published: true
---
Vue는 상태의 변경을 proxy로 감시한다. Proxy는 객체. (자바스크립트에는 `value(값)`과 `reference(참조)`가 있습니다.) 참조가 없는 원시타입(string,number 등)은 Proxy로 만들 수가 없다. 

## reactive

만약 관리하고자 하는 상태가 object라면 참조를 가지고 있기 때문에 그냥 reactive로도 proxy를 만들어 변화를 추적할 수 있다. 

```java
<script setup>
import { reactive } from 'vue'
const msg = reactive("")
</script>

<template>
  <h1>msg = {{ msg }}</h1>
  <input v-model="msg">
</template>
```

원시타입은 변화가 일어나도 반응하지 않는다. proxy는 프로퍼티의 변화를 추적하는데 원시타입은 프로퍼티를 가질 수 없기 때문. 

object로 감싸준다면, 참조를 가지기 때문에 , `value`라는 프로퍼티의 변경을 추적하기 시작한다.

```java
<script setup>
import { reactive } from 'vue'
const msg = reactive({value:""})
</script>

<template>
  <h1>msg = {{ msg.value }}</h1>
  <input v-model="msg.value">
</template>
```

## ref

그래서 원시타입을 상태로 추적하기 위해 제공되는 것이 ref다. 

ref가 하는 일은 원시타입을 감싸준 것처럼 .value안에 값을 넣어주고 object로 변경해주는 것. 

Proxy를 사용하기 위해 value에 reference를 부여해주는 것이다. 

```java
<script setup>
import { ref } from 'vue'
const msg = ref("")
</script>

<template>
  <h1>msg = {{ msg }}</h1>
  <input v-model="msg">
</template>
```

### toRef

원시타입의 식별자 부재는 object의 프로퍼티를 복사할 때도 문제가 된다. 만약 복사하려는 값이 원시타입이면 자바스크립트는 바로 value 값만 복사하기 때문에 Proxy의 반응성을 잊어버린다. 

```java
<script setup>
import { reactive, toRef } from 'vue'

const words = reactive({a:"a",b:"b"})
const msg = words.a
</script>

<template>
  <h1>msg = {{ msg }}</h1>
  <input v-model="msg">
</template>
```

이를 해결하기 위해 toRef를 사용할 수 있는데, toRef는 하나의 property에 대해 부모 object와의 연결성을 유지한채로 reactive를 반환한다. 

```java
<script setup>
import { reactive, toRef } from 'vue'

const words = reactive({a:"a",b:"b"})
const msg = toRef(words, 'a')
</script>

<template>
  <h1>msg = {{ msg}}</h1>
  <input v-model="msg">
</template>
```

이 복사본의 변화는 부모에게도 반영이 되어 추적된다. 

## toRefs

마지막으로 toRefs는 reactive의 모든 프로퍼티 대해 toRef를 적용해 반환한다. 

그래서 destructing을 쓸 수가 있다. 

```java
<script setup>
import { reactive, toRefs } from 'vue'

const words = reactive({a:"a",b:"b"})
const {a,b} = toRefs(words)
</script>

<template>
  <h1>msg = {{ a }}</h1>
  <input v-model="a">
</template>
```

---

[Vue의 reactive, ref, toRef, toRefs](https://velog.io/@chosh/Vue의-reactive-ref-toRef-toRefs)