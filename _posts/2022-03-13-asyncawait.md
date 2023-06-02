---
title: async와 await
description: async와 await, promise
date: 2022-03-13 18:25 +0900
lastmod: 2022-03-13 18:25 +0900
categories: [web]
tags: [elice, web, node js, backend, async, promise]
mermaid: true
---

# 비동기(asynchronous)

Javascript는 single thread 이므로 서버 요청을 기다려야 하는 경우 사용자는 멈춰있는 브라우저를 보게 된다. 그러므로 동기 처리가 아닌 비동기 처리가 필요하다. 비동기는 코드의 순서와 다르게 실행되는 것을 말한다. 비동기 처리 코드를 감싸고 있는 블럭은 `task queue`에 넣어진다. Main thread가 동기 코드를 실행한 후 `event loop`가 `task queue`에 있는 비동기 코드를 실행한다.

# Promise

Promise는 비동기 실행이 완료된 후에 `then`, `catch`, `finally`등의 핸들러를 붙여서 각각 데이터 처리, 에러, 클린업 로직을 실행할 수 있다. `then`을 이어 붙이면 동기 실행처럼 구현 가능하다.

```jsx
const promise = () => {
  return new Promise((resolve, reject) => {
    if (condition) {
      first;
    }
  })
    .then((num) => {
      console.log();
    })
    .catch((error) => {
      console.log(err.message);
    })
    .finally(() => {
      console.log("finally");
    });
};
```

promise가 정상적으로 종료되면 `then`으로 넘어가게 되고, 에러가 발생하면 `catch`로 넘어간다. 마지막에 있는 `finally`는 어떤 인자도 받을 수 없으며 promise가 반환된다.

## Promise의 state

Promise 객체는 pending, fulfilled, rejected 3가지의 상태를 가진다. fulfilled와 rejected 두 가지 상태를 `settled`라고 한다. pending은 비동기 실행이 끝나기를 기다리는 상태이며 fulfilled는 비동기 실행이 성공한 상태이다. rejected는 비동기 실행이 실패한 상태다.

## Multiple Promise handling

하나 이상의 promise를 다루는 방법이 몇 가지 있다. 하나씩 알아보자

### Promise.all()

Promise.all() 내에 존재하는 모든 promise가 완료되어야 다음 단계로 넘어갈 수 있다. 모든 promise가 정상 동작하게 되면 `then`으로 넘어가게 되고 하나의 promise라도 실패하게 되면 `catch`로 넘어가게 된다.

### Promise.allSettled()

모든 promise가 `settled`되기를 기다린다. `settled`는 위에서도 설명되어 있듯이 `fulfilled`와 `rejected`상태를 말한다.

### Promise.race()

넘겨진 promise 중 하나라도 `settled`가 되기를 기다린다.

### Promise.any()

넘겨진 promise 중에서 하나라도 `fulfilled`가 되기를 기다린다.

# async, await

```jsx
async function name() {
  const vari = await action();
}
```

async와 await은 promise 체인을 구현하지 않고 promise를 사용할 수 있게 해준다. promise를 기다려야 하는 동작 앞에 await을 붙이게 되고 await은 항상 async function에서만 사용할 수 있다. 여러 개의 await을 순서대로 나열해서 then chain을 구현할 수 있다.

## 여러개의 await

```jsx
const asyncFunc = async () => {
  try {
    const post = await action();
    const userList = await action();
    return { post, userList };
  } catch (error) {
    console.log(e.message);
  }
};
```

`try catch`문을 사용해서 두 가지의 `await`을 사용해 보았다. `post`와 `userList` 둘 중 하나라도 에러가 발생하면 `catch`가 실행된다. await이 순차적으로 사용되어도 병렬적으로 실행된다.
