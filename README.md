# [Svelte](https://svelte.dev/)
프레임워크가 없는 프레임 워크 혹은 컴파일러
- 2016년 11월 `v0.0.2` 처음 release 이후 현재 (19/11/29 기준) `v3.15.0`
- [svelte REPL](https://svelte.dev/repl/hello-world?version=3.6.10)

## 기본 문법
기본 문법은 html과 비슷
``` html
// App.svelte
<script>
const name = 'ashnamuh'
</script>

<style>
  h1 {
    color: blue;
  }
</style>

<h1>Hello {name}!</h1>
```
### props
``` html
<script>
import Person from './components/Person.svelte'

const ashnamuh = {
  age: 23
}

const jaemok = {
  name: 'Jaemok',
  age: 23
}

</script>

<Person name="ashnamuh" age={ashnamuh.age}></Person>
<Person {...jaemok}></Person>
```

### 이벤트
`:on` 디렉티브로 돔 이벤트 리스닝 (vue와 비슷)
- `|`를 이용하여 이벤트 수식어 붙일 수 있음 ex) `on:click|capture`

### 반응성
script 영역에 선언한 것은 반응성을 지원
- 값이 바뀌면 자동으로 DOM을 갱신한다는 의미

#### $
특정한 값이 변함에 따라 반응해서 변하는 값 선언 가능 (vue의 computed와 비슷)
``` html
// Count.svelte
<script>
let count = 0

$: doubled = count * 2

setInterval(() => {
  count++
}, 1000)
</script>

<p>count: {count}</p>
<p>doubled: {doubled}</p>
```

### await
비동기 작업을 처리하는 `await` 제어문 제공
- `then`, `catch`등 Promise와 비슷한 문법으로 비동기 작업 결과에 따라 분기 가능
``` html
<script>
let promise = getRandomNumber()

const getRandomNumber = async () => {
  const res = await fetch(`https://svelte.dev/tutorial/random-number`)
  const text = await res.text()

  if (res.ok) {
    return text
  } else {
    throw new Error(text)
  }
}

const handleClick = () => {
  promise = getRandomNumber()
}
</script>

<button on:click={handleClick}>
  generate random number
</button>

{#await promise}
  <p>...waiting</p>
{:then number}
  <p>The number is {number}</p>
{:catch error}
  <p style="color: red">{error.message}</p>
{/await}
```

## 특징

### Write less code

#### state update
``` js
// svelte
let count = 0;

function increment() {
    counte += 1;
}
```

``` js
// react
const [count, setCount] = useState(0);

function increment() {
  setCount(count + 1);
}
```

#### basic example
``` js
// React

import React, { useState } from 'react';

export default () => {
  const [a, setA] = useState(1);
  const [b, setB] = useState(2);

  function handleChangeA(event) {
    setA(+event.target.value);
  }

  function handleChangeB(event) {
    setB(+event.target.value);
  }

  return (
    <div>
      <input type="number" value={a} onChange={handleChangeA}/>
      <input type="number" value={b} onChange={handleChangeB}/>

      <p>{a} + {b} = {a + b}</p>
    </div>
  );
};
```

``` html
<!-- Vue -->

<template>
  <div>
    <input type="number" v-model.number="a">
    <input type="number" v-model.number="b">

    <p>{{a}} + {{b}} = {{a + b}}</p>
  </div>
</template>

<script>
  export default {
    data: function() {
      return {
        a: 1,
        b: 2
      };
    }
  };
</script>
```

``` html
<!-- Svelte -->

<script>
  let a = 1;
  let b = 2;
</script>

<!-- root가 꼭 1개가 아니여도 됨 -->
<input type="number" bind:value={a}>
<input type="number" bind:value={b}>

<p>{a} + {b} = {a + b}</p>
```

#### 특별한 svelte: 엘리먼트
##### svelte:self
컴포넌트 자기 자신을 재귀적으로 사용
``` html
<script>
export let count = 5
</script>

{#if count > 0}
  <p>카운트가 내려갑니다 {count}</p>
  <svelte:self count="{count - 1}"/>
{:else}
  <p>카운트가 {count}이 되었습니다!</p>
{/if}
```

##### svelte:component
동적으로 컴포넌트 렌더링
``` html
<script>
import Cat from './Cat.svelte'
import Dog from './Dog.svelte'

const options = [
  { animal: 'cat', component: Cat },
  { animal: 'dog', component: Dog }
]

let selected = options[0]
</script>

<select bind:value={selected}>
  <option value={options[0]}>고양이</option>
  <option value={options[1]}>개</option>
</select>

<svelte:component this={selected.component}/>
```

##### svelte:window
브라우저 글로벌 객체 window를 일부 참조
- addEventListener 이벤트 등록하지 않고 사용 할 수 있음 !!
``` html
<script>
const handleScroll = () => console.log('scrolled!')
</script>

<svelte:window on:scroll={handleScroll} />
```

- `bind`로 window 객체 속성인 `innerWidth`, `innerHeight` 값들을 얻을 수 있음
``` html
<script>
import { onMount } from 'svelte'

let innerWidth

onMount(() => {
  console.log(innerWidth) // 실제 innerWidth 값이 출력됨
})
</script>

<svelte:window bind:innerWidth={innerWidth} />
```

##### svelte:body
body에 대한 참조를 얻을 수 있음
- body에 이벤트 등록 가능
``` html
<script>
const handleClick = () => console.log('clicked!')
</script>

<svelte:body on:click={handleClick} />
```


##### svelte:head
head 영역에 컨텐츠 삽입 가능
- 메타 태그 등 조작 시 유용 !!
``` html
<svelte:head>
  <title>ashnamuh 개발 블로그</title>
  <meta name="description" content="ashnamuh의 개발 블로그입니다.">
</svelte:head>
```

### No virtual DOM
Svelte는 Virtual DOM을 사용하지 않음
- Vue와 React는 Virtual DOM을 사용하여 빠른 렌더링 지원
  - Diff 알고리즘 수행이 필요함
  - 최종적으로는 실제 DOM을 업데이트 해야함

### Truly reactive
변경된 값이 Virtual DOM이 아닌 DOM에 자동으로 반영됨을 의미
- 별도의 Setter 없이 data의 할당 만으로 업데이트를 트리거할 수 있음
- 개발 시 data 변경 시 업데이트 되는 DOM을 지정하는 형태
  - 빌드 될 때 app에서 변경 사항이 어떻게 발생하는지 알고 있는 컴파일러


##### 출처
###### https://velog.io/@ashnamuh/hello-svelte
###### https://velog.io/@ashnamuh/hello-svelte
###### https://heropy.blog/2019/09/29/svelte/
###### https://novemberde.github.io/javascript/2019/10/11/Svelte-revealjs.html
