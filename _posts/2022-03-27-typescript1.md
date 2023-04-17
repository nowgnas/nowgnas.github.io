---
title: Typescript 시작하기
description: typescript를 시작해보자
date: 2022-03-27 23:58 +0900
lastmod: 2022-03-27 23:58 +0900
categories: [web]
tags: [typesctipt, web]
mermaid: true
---

# 타입이 필요한 이유

Javascript에서는 타입이 없기 때문에 타입에 의한 에러가 발생해도 코드를 실행시켜 보기 전까지 오류를 알기 힘들다. typescript는 타입에 대한 오류가 발생했을 때 미리 에러가 발생하는 부분을 알려준다. 타입을 정의하여 개발 단계에서 실수를 줄이고, 명시된 타입을 보고 자료형을 쉽게 파악할 수 있다.

# Typescript의 기본 type

Typescript는 javascript 코드에 변수나 함수의 type을 지정할 수 있다. Type을 나타내기 위해 annotaion을 사용한다.

## primitive type

object와 reference 형태가 아닌 실제 값을 저장하는 자료형이다. 기본 자료형은 아래와 같다.

- string
- boolean
- number
- null
- undefined
- symbol

```tsx
const str: string = "hello";
const isSucc: boolean = true;
const nums: number = 7;
```

## reference type

객체, 배열, 함수 등 Object 형식의 type이다. 메모리에 값을 주소로 저장하고, 출력 시 메모리 주소와 일치하는 값을 출력한다. 참조 자료형은 다음과 같다.

- object
- array
- function

```tsx
function funcName(params: object): void {
  console.log("hello this is function");
}

const arr: number[] = [1, 2, 3];
// generic을 사용한 타입 표기도 가능하다
const arr2: Array<number> = [1, 2, 3];
```

## 추가 제공되는 자료형

- tuple
- enum
- any
- void
- never

```tsx
const arr3: [string, number] = ["hi", 8];
enum Car {
  bus = 1,
  taxi = 2,
  sun = 3,
}
const bus: Car = Car.bus;
const bus1: String = Car[1];
```

배열 각 요소의 타입이 정해진 배열을 선언하는 것과 특정 값들의 집합을 저장하는 타입인 `enum`이다. `enum`은 인덱스로도 접근할 수 있다.

```tsx
const Any: any = "hi";

const unknown: void = undefined;
function sayHi(params: string): void {}
```

any는 컴파일 시 type 검사를 하지 않으며 모든 타입을 저장 가능하다. void는 함수에서 반환 값이 없을 때 사용하고 변수에서는 undefined와 null만 할당한다.

```tsx
function newverEnd(params: string): never {
  while (true) {}
}
```

never는 발생할 수 없는 type이며 항상 오류를 발생시키거나 절대 반환하지 않고 종료되지 않는 함수에 사용한다.

---

# Utility types

Typescript는 공통 타입 변환을 용이하게 하기 위해 utility type을 제공한다. utitity type은 전역으로 사용 가능하다. 아래는 typescript의 utility type이다

- Partial<T>, Readonly<T>
- Record<T>, Pick<T>
- Omit<T,K>, Exclude<T,U>, Extract<T,U>
- NonNullable<T>, Parameters<T>, ConstructorParameters<T>
- ReturnType<T>, Required<T>

### Partial<T>

```tsx
interface Todo {
  title: string;
  description: string;
}

function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
  return { ...todo, ...fieldsToUpdate };
}

const todo1 = {
  title: "title",
  description: "helo",
};

const todo2 = updateTodo(todo1, {
  description: "world",
});

console.log(todo2); // { title: 'title', description: 'world' }
```

property를 선택적으로 만드는 type을 구성한다. 주어진 타입의 모든 하위 type 집합을 나타내는 type을 반환한다. 위 코드처럼 일부 property의 접근이 가능하다.

### ReadOnly<T>

```tsx
const todoReadonly: Readonly<Todo> = {
  title: "hello ",
  description: "des",
};
```

ReadOnly로 지정되면 property에 재할당이 불가능하다.

### Record<T>

```tsx
interface PageInfo {
  title: string;
}

type Page = "home" | "about" | "contact";

const x: Record<Page, PageInfo> = {
  about: { title: "about" },
  contact: { title: "contact" },
  home: { title: "hone" },
};
```

property의 집합 K로 type을 구성한다. Type의 property들을 다른 type에 매핑시키는데 사용한다.

### Pick<T, K>

```tsx
interface Todos {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPreview = Pick<Todos, "title" | "completed">;

const todos: TodoPreview = {
  title: "clean room",
  completed: false,
  description: "description", // error: is not assignable to type 'TodoPreview'
};
```

property K의 집합을 선택해 type을 구성한다. 위 코드에서 todos의 description 부분에서 에러가 발생한다. `TodoPreview`에 description이 포함되어 있지 않기 때문이다. `TodoPreview`에 description을 추가해 주면 정상적으로 동작한다.

### Omit<T, K>

```tsx
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPreview = Omit<Todo, "description">;

const todo: TodoPreview = {
  title: "clean room",
  completed: false,
  description: "description", //is not assignable to type 'TodoPreview'.
};
```

모든 property를 선택한 다음 K를 제거한 type을 구성한다. `TodoPreview`에서 description이 제거 되었으므로 todo에서는 description부분에서 에러가 발생한다.

### Exclude<T, U>

```tsx
type T0 = Exclude<"a" | "b" | "c", "a">;

type T1 = Exclude<"a" | "b" | "c", "a" | "b">;

type T2 = Exclude<string | number | (() => void), Function>;
```

T 중에서 U를 제외한 type을 구성한다

### Extract<T, U>

```tsx
type T0 = Extract<"a" | "b" | "c", "a" | "f">; // a

type T1 = Extract<string | number | (() => void), Function>; // () => void
```

T에서 U에 할당할 수 있는 모든 속성을 추출해서 type을 구성한다

### Parameters<T>

```tsx
declare function f1(arg: { a: number; b: string }): void;

type T0 = Parameters<() => string>; // []
type T1 = Parameters<(s: string) => void>; // [string]
type T2 = Parameters<<T>(arg: T) => T>; // [unknown]
type T4 = Parameters<typeof f1>; // [{a:number, b: string}]
type T5 = Parameters<any>; // unknown[]
type T6 = Parameters<never>;
```

함수 type T의 매개 변수 type들의 튜플 type을 구성한다

### ConstructorParameters<T>

```tsx
type error = ConstructorParameters<ErrorConstructor>; // [(string | undefined)?]
type func = ConstructorParameters<FunctionConstructor>; // string[]
type RegEx = ConstructorParameters<RegExpConstructor>; // [string, (string | undefined)?]

interface inter1 {
  new (args: string): Function;
}
type interType = ConstructorParameters<inter1>; // [string]

function f1(a: interType) {
  a[0];
  a[1]; // Tuple type '[args: string]' of length '1' has no element at index '1'.
}
```

생성자 함수 type의 모든 매개변수 타입을 추출한다. 모든 매개변수 type을 가지는 튜플 type(T가 함수가 아닌 경우 never)을 생성한다.

### ReturnType<T>

```tsx
declare function f1(): { a: number; b: string };

type t1 = ReturnType<() => string>; //string
type t2 = ReturnType<(s: string) => void>; // void
type t3 = ReturnType<<T>() => T>; // {}
type t4 = ReturnType<<T extends U, U extends number[]>() => T>; // number[]
type t5 = ReturnType<typeof f1>; // {a:number, b:string}
```

type의 구성을 함수 T의 반환되는 type으로 지정한다.

### Required<T>

```tsx
interface Props {
  a?: number;
  b?: string;
}

const obj: Props = { a: 5 };
const obj2: Required<Props> = { a: 5 }; // Property 'b' is missing in type '{ a: number; }' but required in type
```

T의 모든 property가 필수로 설정된 type을 구성한다.

# function 사용하기

## 매개변수

```tsx
function person(name: string, age: number, gender: string) {
  return `hello ${name} age is ${age} gender is ${gender}`;
}

const lee = person("lee", 20, "male ");
```

기본적으로 함수에 주어진 인자의 수는 함수가 기대하는 매개변수의 수와 같아야 한다.

## 선택적 매개변수

```tsx
function person(name: string, age: number, gender?: string) {
  return `hello ${name} age is ${age} gender is ${gender}`;
}

const lee = person("lee", 20);
```

함수의 매개변수에 `?`를 붙여 선택적으로 매개변수를 사용할 수 있다. 값을 넣지 않아도 에러가 발생하지 않는다.

### 초기화 매개변수

```tsx
function person(name: string, age = 20, gender: string) {
  return `hello ${name} age is ${age} gender is ${gender}`;
}

const lee = person("lee", undefined, "male ");
```

매개변수에 값을 할당하게되면 함수의 인자가 `undefined`인 경우 할당된 값으로 함수가 동작하게 된다.

### 나머지 매개변수

```tsx
function person(name: string, age = 20, ...remain: string[]) {
  return `hello ${name} age is ${age} remain ${remain}`;
}

const lee = person("lee", undefined, "hello", "world");
```

spread 연산자를 사용해서 `...` 뒤의 인자 배열을 사용할 수 있다.
