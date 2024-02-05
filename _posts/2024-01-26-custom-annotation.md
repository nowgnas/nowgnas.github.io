---
title: "[MSA 3] Custom annotation과 AOP를 이용한 멱등성을 보장하는 알림 전송하기"
description: custom annotation과 aop를 사용한 알림 중복 전송 방지 구현
date: 2024-01-26 11:25 +0900
lastmod: 2024-01-26 11:25 +0900
categories: msa
tags:
  - eks
  - aop
  - annotation
  - event
  - notification
  - sse
  - sms
  - sqs
  - lambda
  - sns
  - idempotent
---

## 알림 시스템 구조

> [[MSA 2] AWS SQS를 이용한 알림 시스템 설계 및 구현](https://nowgnas.github.io/posts/notification-service/)

![Pasted image 20240124150932](https://github.com/nowgnas/nowgnas.github.io/assets/55802893/81bdd10f-5723-4f5b-80d4-cbe4feccdf3d)

알림 시스템은 위와 같이 구성되어 있다.
여러 micro service에서 이벤트가 발생하게 되면 SNS나 SQS에 메세지를 publish하게 된다. Notification service는 SQS에 담겨 있는 메세지를 polling 받아 알림을 전송하고 알림 정보를 데이터베이스에 저장한다.

```java
// NotificationSQSListener.java
@SqsListener(
    value = "${cloud.aws.sqs.settlement-notification-queue.name}",
    deletionPolicy = SqsMessageDeletionPolicy.NEVER)
public void consumeSettlementNotificationQueue(
    @Payload String message, @Headers Map<String, String> headers, Acknowledgment ack)
    throws JsonProcessingException {

...

  notificationActionHelper.publishNotification(notification);
  ack.acknowledge(); // SQS 데이터 ack 처리
}
```

```java
// NotificatinActionHandler.java
public void publishNotification(
    NotificationData<InqueryResponseNotification> notification) {
  ...
  sse.publish(sseData); // sse 알림 전송
  sms.publish(smsData); // sms 알림 전송

  // save notification
  notificationCommandService.saveSingleNotification(notification); // save data
}
```

알림 정보를 polling 후 이뤄지는 프로세스는 다음과 같다.

- SSE, SMS 알림 전송
- notification 데이터 Mysql에 저장
  SQS Listener 부분 메서드를 간략화한 코드에서도 알 수 있듯이 알림 정보를 SSE, SMS를 통해 전달하고 데이터베이스에 저장한다.

SQS를 이용해 알림을 유실하는 것은 방지했지만, 일시적으로 알림 정보를 저장해야하는 시점에 서비스나 데이터베이스에 이상이 생기면 SQS 데이터를 acknowledge 처리하지 못하게 되어 다시 polling 하게 된다.
SQS 메세지를 다시 polling하게 되면 SSE, SMS는 다시 전송될 것이고 사용자는 2번 이상의 중복된 알림을 받을 가능성이 커진다.

> 배송 시작 알림이 두 번 온다면??  
> "나는 하나만 주문했는데 왜 두개가 오지?!!"

사용자의 혼란과 SMS 비용을 낭비하지 않고 멱등성을 보장하기 위해 알림 중복 전송 방지 처리가 필요하다. 이를 custom annotation과 AOP를 이용해 해결했다.

## AOP (Aspect Oriented Programming)

"관점 지향 프로그래밍" 은 관심사를 분리하여 핵심 관점, 부가 관점으로 분리하여 관점을 기준으로 모듈화 하는 것을 말한다.
AOP를 이용해 알림이 전송된 적이 있다면 전송하지 않고 진행하도록 구성할 것이다. AOP의 Point cut으로 annotation을 이용하여 SSE와 SMS가 중복으로 전송되지 않도록 한다.

멱등성 보장을 위해 이벤트가 발생하는 시점에 eventId를 발급하여 알림 정보를 큐에 전달한다. Notification service는 알림 정보를 받아 eventId와 알림 정보를 가지고 Redis에 알림 정보가 존재하는지 확인한다.
알림 정보가 존재한다면 해당 알림은 전송하지 않고, 존재하지 않으면 알림 정보를 Redis에 저장하고 알림을 전송한다.
![Pasted image 20240124170524](https://github.com/nowgnas/nowgnas.github.io/assets/55802893/6a3a3f0b-ad9f-43a0-bc20-c316e4383346)

위 그림과 같이 SSE나 SMS를 전송하기 전에 알림 정보를 Redis에 확인하는 과정이 진행된다. Custom annotation 을 사용해 간단하게 구현했다.

### Custom annotation

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface DuplicateEventCheck {

    String getEventType();
}
```

알림 정보가 존재하는지 확인하기 위한 custom annotation이다. Custom annotation을 구성하기 위해 어떤 곳에 annotation을 적용할지 결정하는 `Target`과 annotation이 유지되는 시점을 명시하기 위한 `Retention`이 필요하다.

`Target`은 TYPE, FIELD, METHOD 외에도 여러가지로 설정할 수 있고, `Retention`은 SOURCE, CLASS, RUNTIME이 있다.
Annotation을 알림 정보를 전송하는 SSE와 SMS 메서드에 붙여 처리하고 싶어 Target은 METHOD로 정했다. 또한 서비스 실행 중에 유지되길 원해 Retention은 RUNTIME으로 지정했다.
custom annotation을 적용해본다.

```groovy
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-aop:2.7.17'
}
```

```java
@Aspect
@Slf4j
@Component
@RequiredArgsConstructor
public class DuplicateEventHandlerAop {
...
}
```

AOP 의존성을 추가하고 `@Aspect` annotation을 클래스에 붙여준다. `@Aspect`는 AOP 기본 모듈을 가지고 있는 싱글톤 형태의 객체로 구성되어 있다.

```java
@Pointcut("@annotation(duplicateEventHandleAnnotation)")
public void duplicateEvent(DuplicateEventCheck duplicateEventHandleAnnotation) {}
```

해당 AOP가 적용될 PointCut을 정의한다. parameter로 custom annotation 클래스를 받고 Pointcut을 위와 같이 정의하면 된다.

### Advice

```java
@Around(
    value = "duplicateEvent(duplicateEventHandleAnnotation) && args(notifyData)",
    argNames = "joinPoint,notifyData,duplicateEventHandleAnnotation")
public Object duplicateEventHandlerAop(ProceedingJoinPoint joinPoint,
    NotificationInformation notifyData,
    DuplicateEventCheck duplicateEventHandleAnnotation)
    throws Throwable {}
```

advice는 횡단 관심사에 해당하는 공통기능 코드이며, advice로 구현된 메서드가 어느 시점에 동작할지 결정한다. Advice의 종류는 before, after, after-returning, after-throwing, around 가 존재한다.

이번 aop 구성은 Around를 사용했다. `@Around`는 `value`와 `argNames` 두 가지 파라미터를 사용할 수 있다. `value`는 advice 메서드가 사용할 변수들을 정의하고 `argNames`는 advice 메서드의 파라미터 이름을 정의한다.

```java
@DuplicateEventCheck(getEventType = "sse")
@Override
public void publish(NotificationInformation notifyData) {
  sseService.notify(notifyData);
}
```

Pointcut으로 지정한 custom annotation과 핵심 기능 메서드의 파라미터를 args로 받는다.

```java
@Around(
    value = "duplicateEvent(duplicateEventHandleAnnotation) && args(notifyData)",
    argNames = "joinPoint,notifyData,duplicateEventHandleAnnotation")
public Object duplicateEventHandlerAop(
    ProceedingJoinPoint joinPoint,
    NotificationInformation notifyData,
    DuplicateEventCheck duplicateEventHandleAnnotation)
    throws Throwable {

  // 파라미터에서 받는 값
  String eventId = notifyData.getEventId();
  String id = String.valueOf(notifyData.getId());
  Role role = notifyData.getRole();
  // annotation value
  String eventType = duplicateEventHandleAnnotation.getEventType();

  String generateEventId = eventId + ":" + id + ":" + role + ":" + eventType;
  Object event = redisTemplate.opsForValue().get(generateEventId);
  log.info("event id : " + generateEventId);

  if (event == null) {
    redisTemplate.opsForValue().set(generateEventId, eventType);
    redisTemplate.expire(generateEventId, expire, TimeUnit.MINUTES);

    Object[] args = joinPoint.getArgs(); // 핵심 기능 파라미터

    return joinPoint.proceed(args); // 핵심 기능 실행
  }
  return null;
}
```

advice 메서드 전체 로직은 위와 같다. 이벤트 id를 정의하고 이벤트 id가 redis에 저장되어 있으면 핵심 기능을 실행 시키지 않고 이벤트 id가 저장되어 있지 않으면 핵심 기능을 실행하는 방식으로 구현했다.

`eventType` 은 custom annotation에서 정의한 값을 사용한다.

```java
@DuplicateEventCheck(getEventType = "sms")
```

annotation에 값을 정의할 수 있고 custom annotation을 정의할 때 default 값도 지정할 수 있다.

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface OutOfStockNotificationDuplicateCheck {

    String getEventType() default "out-of-stock";
}
```

default 값은 위와 같이 지정할 수 있다.

```java
// SendSMS.java
@DuplicateEventCheck(getEventType = "sms")
@Override
public void publish(NotificationInformation notifyData) {
...
}
```

```java
// SendSSE.java
@DuplicateEventCheck(getEventType = "sse")
@Override
public void publish(NotificationInformation notifyData) {
  sseService.notify(notifyData);
}
```

SSE와 SMS를 전송하는 메서드에 custom annotation 을 붙여 알림 중복 전송을 방지할 수 있었다.
