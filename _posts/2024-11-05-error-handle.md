---
title: "서비스에서 실패에 대한 다양한 처리 방법"
description: "서비스에서 실패에 대한 다양한 처리 방법"
date: 2024-11-05 00:22 +0900
lastmod: 2024-11-05 00:22 +0900
categories:
  - error handle
tags:
  - logic
  - kafka
  - client
---

이번 포스팅에서는 서비스의 여러 구간에서의 실패에 대한 처리 방법에 대해 알아본다.

최근 예외 상황에 대한 처리에 관심을 가지게 되었다. 사용자에게 어색함을 주지 않고 안정적인 서비스를 제공하려면 여러 구간에서 예외처리가 적절하게 이뤄져야 한다고 생각하여 간단하게 정리해 보았다.

## 비즈니스 로직 에러 처리

#### 메서드에서 null을 반환한다..?

오래된 코드를 보면 메서드에서 null을 반환하는 경우가 있다. 메서드에서 특정 조건에만 로직을 처리하고, 조건에 만족하지 않으면 null을 반환하는 것이다. null을 메서드의 파라미터로 전달하거나 반환하는 행위는 NPE(NullPointerException)을 발생시킬 확률이 매우 높다. 또한, 운영 환경에서 로그 추적에 어려움을 줄 수 있다.

```java
public class UserService {
    public User findUserById(String userId) {
        if (userId == null || userId.isEmpty()) {
            return null; // null을 반환하고 있다.
        }

        return new User(userId, "John Doe");
    }
}
```

우선 위의 예시에서 개선할 수 있는 부분은 Optional<T>을 반환하는 것이다. Optional<T>를 사용하게 되면 메서드 사용자에게 null일 수 있다는 점을 알려주며 null인 경우에 대한 처리를 하도록 요구하게 된다.

#### 예외 상황을 세분화 하기

```java
@RestControllerAdvice
public class UserRestControllerAdvice {

    @ExceptionHandler(value = UserInfoNotMatchException.class)
    public ResponseEntity<?> userInfoNotMatchException(UserInfoNotMatchException userInfoNotMatchException) {
        // do something when UserInfoNotMatchException
    }

}
```

Spring Framework에는 controller advice라는 전역 예외 처리가 가능한 annotation이 있다. 내부 로직에서 예외 발생 시 예외를 throw 하게 되면 예외 종류에 따라 전역적으로 처리할 수 있다.

> advice이지만 aop 방식은 아니라고 한다..

비즈니스 로직에서 `UserInfoNotMatchException`이 발생하면, `UserRestControllerAdvice`에서 이를 처리하여 클라이언트에 적절한 응답을 보낼 수 있다. 공통으로 사용할 수 있는 예외를 정의하기 좋다고 생각한다.
운영 환경에서는 공통으로 `@RestControllerAdvice`를 잡고 있어 custom 예외 클래스를 던져서 사용할 수 없어 Response에 담아서 응답해 준다.
API 명세에예외 케이스를 API 명세에 정의하면, 클라이언트 개발자가 예외를 예상하고 적절히 대처할 수 있어 더 나은 사용자 경험을 제공할 수 있다고 생각한다.

이렇게 API를 개발하거나 비즈니스 로직 내에서 메서드를 구현할 때, 클라이언트(호출자)가 회복 가능한 상황인지 여부를 판단하는 것이 중요하다고 생각한다. 예를 들어, 예상 가능한 오류는 명확하게 예외를 던지고 이를 호출자가 처리하도록 하고, 복구 불가능한 오류는 서비스의 안정성을 위해 더 높은 수준에서 처리하는 것이 적합할 수 있다. 이러한 방식으로 예외를 신중하게 처리하면, 안정적인 운영을 할 수 있을 것이라고 생각한다.

## Kafka producer, consumer에서 예외 처리

Kafka는 많은 서비스에서 사용되고 있는 이벤트 기반의 강력한 메세지 처리 기술이다. 동기, 비동기를 지원하고 클러스터링을 이용해 대규모 시스템에서도 잘 동작할 수 있는 높은 처리량을 보장한다.
하지만, kafka에서 제공하는 기능을 제대로 검증하지 않고 사용한다면 서비스에 심각한 장애와 손실을 불러올 수 있다.

#### kafka producer

```java
producer.send(record, (metadata, exception) -> {
    if (exception != null) {
        // Handle the exception, log the error or send slack alert
    } else {
        // Handle successful send, maybe log success metadata
    }
});

```

메세지를 발행하는 producer에서는 메세지 발행 시 실패에 대해 처리해야 한다. 메세지를 발행할 때 직렬화 실패나 복구 불가능한 예외가 발생하게 되었을 때 해당 메세지를 무시하고 에러 로그를 남기거나 비동기에서 전송 여부에 따른 처리를 할 수 있다.

#### kafka consumer

메세지를 소비하는 consumer에서는 메세지 처리 실패 시 방안에 대해 고민해야 한다. 리밸런싱에 의한 멱등성 문제, back pressure에 의한 스레드 부족 문제도 있지만, 이번 글에서는 메세지 처리 실패의 경우만 생각해본다. (따로 정리할 예정이다)

메세지 처리에 실패하는 경우 재시도 전략을 사용하거나, DLQ(dead letter queue)로 메세지를 발행해서 후속 작업을 하게 된다. 크게 중요하지 않은 정보는 에러 로그만 남기고 처리를 하지 않고, 중요한 정보는 DLQ로 실패한 메세지를 발행하고 실패 메세지 정보를 테이블에 저장하여 수기처리를 하게 된다.

## 클라이언트에서 예외 처리

클라이언트에서는 좋은 사용성과 안정적인 서비스를 제공하기 위해서 예외 처리는 중요하다. 클라이언트에서 발생 가능한 에러는 api에서 에러를 반환하거나, 네트워크 오류, 유효성 검사 오류, 비동기 작업의 실패 등이 있다.

클라이언트에서의 에러는 사용자에게 구체적이고 명확한 상태를 알려줘야 한다. 네트워크 오류인 경우 사용자에게 상태를 알려주거나, 비동기 작업이 실패 했을 때는 재시도나 로딩 스피너를 이용해 에러 상황에 적절한 처리를 해야 한다.

클라이언트의 예외 처리는 안정적인 서비스 운영뿐만 아니라 사용성을 개선시킬 수 있다고 생각한다. UI/UX가 좋은 서비스는 사용자를 오래 머물게 하고, 서비스의 신뢰성을 높일 수 있다.

## 정리

서비스의 여러 구간에서 예외를 처리하는 방법에 대해 살펴보았다. 비즈니스 로직, 메시징 시스템, 클라이언트 등 각 영역에서 적절한 예외 처리 전략을 세우는 것은 서비스의 안정성을 높이고 사용자 경험을 개선하는 중요한 요소이다.
앞으로도 예외처리에 대해 서비스의 안정성을 고려한 설계를 할 수 있도록 노력해야겠다.
