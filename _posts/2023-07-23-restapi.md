---
title: REST API의 특징과 설계
date: 2023-07-23 16:31:07
tags: [rest api, http]
categories: java
---

## API란 무엇인가 ?

API는 Application Programming Interface(애플리케이션 프로그램 인터페이스)의 줄임말입니다. API의 맥락에서 애플리케이션이라는 단어는 고유한 기능을 가진 모든 소프트웨어를 나타냅니다. 인터페이스는 두 애플리케이션 간의 서비스 계약이라고 할 수 있습니다. 이 계약은 요청과 응답을 사용하여 두 애플리케이션이 서로 통신하는 방법을 정의합니다. API 문서에는 개발자가 이러한 요청과 응답을 구성하는 방법에 대한 정보가 들어 있습니다.  
<sub>[aws 문서](https://aws.amazon.com/ko/what-is/api/)<sub>

## REST API(Representational State Transfer)

REST는 자원을 이름으로 구분해서 자원의 상태를 주고 받는 것을 의미한다. Client에서 Server로 요청할 때 Http 프로토콜 메소드를 사용해 요청이 이뤄진다.

GET, PUT, DELETE 등의 함수 집합을 정의한다. 이런 함수들을 이용해 Client는 Server로부터 데이터를 교환하게 된다.

RESTful한 API를 구축하기 위해 6가지 조건이 있다.

### Client-Server architecture

정보를 가지고 있는 Server와 정보를 요청하는 Client로 구성된다. Server는 API제공, 비즈니스 로직 처리를 담당하고, Client는 사용자 인증이나 세션, 로그인 정보 등을 직접 관리하고 책임진다. Server와 Client 사이의 의존성이 줄어들게 된다.

### Stateless

Stateless(무상태)는 REST API의 대표적 특징이다. Http의 특성을 이용하며 Server에서 작업을 하기 위해 상태 정보를 기억할 필요가 없고 Client의 요청에 대해서만 처리하면 된다. Server는 Client의 요청을 독립적으로 인식하고 처리하기 때문에 Server 처리의 일관성을 부여하게되고 부담이 줄어들게 된다.

### Cashable

캐시 처리 기능은 Http의 특징인 캐시를 사용할 수 있다. 캐시는 대량의 요청을 효율적으로 관리 가능하고 빠른 응답 시간을 가진다.

### Layerd System

계층화 특징은 Client와 Server의 통신 사이에 프록시 서버나 암호화 계층 등 중간 매체를 둘 수 있다. 또한, 중간 매체와 통신 여부를 Client나 Server는 알 수 없도록 설계 되어야 한다.

### Uniform Interface

인터페이스 일관성은 URI로 지정한 자원(정보)에 대한 동작이 통일되어야 하고, 사용자의 정보가 하나의 URI에 속함을 보장하는 특징이다. 특정 기술이나 언어에 종속적이지 않고 Http 프로토콜을 따르는 플랫폼에서 모두 사용 가능하다.

### Code On Demand(Optional)

REST API는 일반적으로 정적 자원(정보)을 전송하지만, 응답에 실행 코드가 포함될 수 있다. 이 경우에 코드는 요청시에만 실행 되어야 한다.

## REST API 설계

REST API는 다음 규칙에 따라 설계한다.

```bash
http://example.com/post   (O)

http://example.com/post/  (X)
```

- URI는 명사를 사용하고 소문자로 작성되어야 한다.
- URI의 마지막에는 `/`를 포함하지 않는다.

```bash
http://example.com/post-list  (O)

http://example.com/post_list  (X)
```

- URI에는 언더바가 아닌 하이픈을 사용한다.

```bash
http://example.com/post/assets/example  (O)

http://example.com/post/assets/example.png  (X)
```

- URI에는 파일의 확장자를 표시하지 않는다.

```bash
http://example.com/post/:post-id

http://example.com/post/{post-id}  (O)
```

- resource 간에 연관 관계가 있는경우 위와 같이 사용한다.
