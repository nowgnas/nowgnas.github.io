---
title: Typescript class
description: typescript의 클래스
date: 2022-03-28 15:09 +0900
lastmod: 2022-03-28 15:09 +0900
categories: [web]
tags: [typesctipt, web, class]
mermaid: true
---

# OOP(Object Oriented Programming)

Typescript는 객체 지향 프로그래밍(OOP)의 특징을 가지고 있다. OOP는 객체의 모임으로 컴퓨터 프로그램을 파악하려는 프로그래밍 방식이다. Typescript는 인터페이스, 상속, 정적 메서드 구현 등의 패턴을 지원한다.

OOP의 장점은 프로그램의 변경을 유연하게 만든다. 개발과 유지보수가 간편하며 직관적인 코드 분석이 가능하게 한다.

# class 생성

```tsx
class Person {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
  say() {
    return "hello" + this.name;
  }
}
```

`new`를 사용해서 Person 클래스의 인스턴스를 생성한다. 클래스 안에서 `this`를 사용하면 클래스의 멤버를 의미하게 된다. 클래스의 멤버는 name, constructor, say()이다.

Class의 요소는 `member`, `field`, `constructor`, `method`가 있다.

# 접근 제어자

Typescript의 접근 제어자는 Java와 다르게 package 개념이 없어 default 접근 제어자는 존재하지 않고, `public`, `protected`, `private`가 존재한다.

## public

```tsx
class Animal {
  public name: string;
  constructor(Name: string) {
    this.name = Name;
  }
}

new Animal("Cat").name;
```

클래스 내에서 선언된 멤버들이 자유롭게 접근할 수 있다. typescript에서 기본 접근 제어자는 public이다.

## private

```tsx
class Animal {
  private name: string;
  constructor(parameters) {
    this.name = parameters;
  }
}

new Animal("cat").name;
// Property 'name' is private and only accessible within class 'Animal'.
```

private 접근 제어자는 멤버가 포함되어 있는 클래스 외부에서의 접근을 막는다. 위 코드의 오류를 보면 `name`이 private이기 때문에 접근을 하지 못한다고 되어 있다.

# 상속

```tsx
class Vehicle {
  move(dist: number) {
    console.log(`Car move ${dist}m`);
  }
}

class Bus extends Vehicle {
  getPerson(nums: number) {
    console.log(`승객 ${nums}명을 태웠습니다 `);
  }
}

const bus = new Bus();
bus.move(10);
bus.getPerson(10);
```

상속을 이용해서 존재하는 클래스를 확장해 새로운 클래스를 생성할 수 있다. extends 키워드를 사용해서 `Vehicle`의 멤버에 접근할 수 있다. Bus는 Vehicle의 기능을 확장했으므로 `move()`와 `getPerson()`을 가진 인스턴스를 생성한다.

# Getters와 Setters, readonly, static

## getters & setters

```tsx
class Persons {
  // > 멤버 초기화 필요한것 같음
  private _name: string = "";

  get name() {
    return this._name;
  }
  set name(name: string) {
    if (name.length > 10) {
      throw new Error("name too long");
    }
    this._name = name;
  }
}

let persons = new Persons();

persons.name = "lee";
console.log(persons.name);
```

비공개로 설정하려는 속성은 private으로 설정하고, 속성값을 읽고 수정하는 getter/setter 함수를 사용한다.

## readonly

```tsx
class Student {
  readonly age: number = 20;
  constructor(age: number) {
    this.age = age;
  }
}
```

읽기만 가능한 속성을 선언하기 위해 사용한다. 선언될 때나 생성자에서 값을 설정한 이후에는 값을 수정할 수 없다.

## static

```tsx
class Grid {
  static Origin = { x: 2, y: 5 };

  area(point: { x: number; y: number }) {
    let xPoint = point.x * Grid.Origin.x;
    let yPoint = point.y * Grid.Origin.y;
    return xPoint * yPoint;
  }
}
```

전역 멤버를 선언할 때 사용한다. 전역멤버는 객체마다 할당되지 않고 클래스의 모든 객체가 공유하는 멤버이다. `Grid.`으로 static 멤버에 접근 가능하다.

# 추상 클래스

```tsx
abstract class Hello {
  protected name: string;

  constructor(name: string) {
    this.name = name;
  }

  abstract hello(): void;
  welcome(): void {
    console.log("hello world");
  }
}

class Hi extends Hello {
  constructor(name: string) {
    super(name);
  }
  hello(): void {
    console.log(this.name + "hello worlds!!");
  }
}

const login = new Hi("lee");
login.hello(); // leehello worlds!!
```

추상 클래스는 다른 클래스들이 파생될 수 있는 기초 클래스이다. `abstract`키워드는 추상 클래스나 추상 메서드를 정의하는데 사용된다. 추상 메소드는 클래스에 구현되어 있지 않고, 파생된 클래스에서 구현되어야 한다.

# 디자인 패턴

디자인 패턴은 프로그램의 일부분을 서브 클래스로 캡슐화 해서 전체 구조를 바꾸지 않고 특정 단계의 기능을 바꾸는 것이다. 상위 클래스에서는 전체 알고리즘을 구현하고 다른 부분들은 하위 클래스에서 구현한다. 부분적으로 다른 메소드들의 코드 중복을 최소화 할 수 있다는 장점이 있다.
