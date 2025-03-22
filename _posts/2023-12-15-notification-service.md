---
title: "[MSA 2] AWS SQS를 이용한 알림 시스템 설계 및 구현"
description: SQS를 이용해 알림 시스템을 설계 한다.
date: 2023-12-15 01:10 +0900
lastmod: 2023-12-15 01:10 +0900
categories: msa
tags: sqs, notification, sse, sms
---

프로젝트를 설계할 때는 알림 시스템을 간단하게 생각하고 설계했었다. 하지만 알림을 구현하기 시작하면서 재 설계의 필요성을 느꼈다.

## 알림 시스템의 요구사항

### 알림 템플릿

먼저 알림 시스템의 템플릿을 알아보자.

```plaintext
배송이 시작되었습니다. [알림 메세지]
https://mypage   [연관된 페이지]
```

### 알림 형태

| 수신자        | 전송 방법 |
| :------------ | :-------- |
| 시스템 관리자 | SSE       |
| 가게 사장     | SSE       |
| 구매자        | SSE, SMS  |

알림 전송 방법은 SSE와 문자 메세지를 이용한다. SSE와 문자 메세지의 템플릿을 통일해서 알림 데이터를 다루기 쉽도록 구성했다.

### 알림 기본 프로세스

알림이 발생하는 프로세스는 다음 그림과 같다.
![pic1](https://github.com/lotteon2/lotteon2.github.io/assets/55802893/9efb7c0e-b78b-401b-ab04-9caa6f9ddbc7)

1. 액터(구매자, 가게 사장, 시스템 관리자)에 의해 micro service에서 특정 동작이 수행된다.
2. micro service에서 동작 완료 후 특정 조건을 만족하면 이벤트를 발생 시킨다.
3. AWS SQS 로 메세지를 publish 하게 된다.
4. notification service 의 특정 SQS listener가 메세지를 polling 한다.
5. 알림을 전달할 사용자의 Role에 따라 SSE, SMS를 전송하고 알림 정보를 mysql 데이터베이스에 저장한다.
   전체적인 프로세스이다. 특정 이벤트에 대해서는 SNS를 사용하여 여러 SQS가 이벤트를 받는 경우가 발생할 수 있다.

### 알림 이벤트 전달 객체 설계

![Pasted image 20231215005648](https://github.com/lotteon2/lotteon2.github.io/assets/55802893/9d9e413d-46af-4a90-8a10-3b190d18531a)  
알림 서비스를 설계하기 위해 그린 그림이다.

알림은 여러 micro service에서 발생한 이벤트에 의해 전송된다. 주문 생성, 배송 시작, 문의 등록, 답변 등록 등 사용자에게 즉시 상태 변화를 알려줄 수 있어야 한다. 또한 여러 micro service에서 알림 정보를 notification service에 전달하기 때문에 알림 정보를 전달하는 코드는 통일성을 가지면 좋다.

#### notification data

```java
...
public class NotificationData<T> {
  @JsonProperty private T whoToNotify;
  private PublishNotificationInformation publishInformation;

  public static <T> NotificationData<T> notifyData(
      T data, PublishNotificationInformation publishInformation) {
    return NotificationData.<T>builder()
        .whoToNotify(data)
        .publishInformation(publishInformation)
        .build();
  }
```

위 코드는 micro service에서 SQS에 publish 할 데이터를 받는 클래스이다. `whoToNotify`는 타입 T를 부여해 데이터의 형태에 제약을 두지 않았다.
알림 전달을 한번에 여러명에게 할 수도 있고, 한명에게만 전달할 수도 있다. 또한 알림 형태에 따라 요구되는 속성이 다를 수 있다.
각 micro service 에서 `notify()`메서드를 호출하여 누구에게 전달할지, 알림 정보는 어떤 것인지를 넣어 SQS에 메세지를 전송하게 된다.

#### SQS Listener

```java
@SqsListener(
    value = "${cloud.aws.sqs.question-register-notification-queue.name}",
    deletionPolicy = SqsMessageDeletionPolicy.NEVER)
public void consumeQuestionRegisterNotificationQueue(
    @Payload String message, @Headers Map<String, String> headers, Acknowledgment ack)
    throws JsonProcessingException {
  NotificationData<ClassName> questionRegisterNotification =
      objectMapper.readValue(message, ClassName.class);
...

  notificationActionHelper.publishQuestionRegisterNotification(notification);
  ack.acknowledge();
}
```

sqs listener는 위와 같다. `spring-cloud-aws-messaging` 패키지에 있는 SqsListener를 사용해 SQS 메세지를 polling하게 된다. message 파라미터로 publish된 메세지를 받아오고 objectMapper를 이용해 객체를 변환한다.
메세지를 받은 후 SSE, SMS 전송과 알림 정보를 데이터베이스에 저장하는 동작을 진행한다.

#### 메세지 전송하기

이 프로젝트에서는 구매자에게는 SSE, SMS를 전송하고 가게 사장과 시스템 관리자에게는 SSE를 전송한다. SSE와 SMS로 전달되는 데이터는 동일했기 때문에 `publish` 메서드를 가진 인터페이스를 두고 SendSMS, SendSSE 클래스가 상속받는 방법을 사용했다.

![Screenshot 2023-12-15 at 00 00 22](https://github.com/lotteon2/lotteon2.github.io/assets/55802893/bb4fdf22-976a-4a2d-b569-d506c02b7516)

```java
// interface
public interface InfrastructureActionHandler<T extends NotificationCommand.NotificationInformation> {
  void publishCustomer(T notifyData);
}
```

```java
// SendSMS
public class SendSMS implements InfrastructureActionHandler<NotificationInformation> {
...
@Override
public void publishCustomer(NotificationInformation notifyData) {
  SnsClient snsClient = awsConfiguration.snsClient();
  try {
    setSMSAttribute(snsClient);
    PublishRequest publishRequest = makePublishRequest(notifyData);
    PublishResponse response = snsClient.publish(publishRequest);
    log.info(response.messageId() + " message sent " + response.sdkHttpResponse().statusCode());
  } catch (SnsException e) {
    log.error("재입고 알림 메세지 전송 실패");
  }
}
```

```java
// SendSSE
public class SendSSE implements InfrastructureActionHandler<NotificationInformation> {
  private final SseService sseService;

  @Override
  public void publishCustomer(NotificationInformation notifyData) {
    sseService.notify(notifyData);
  }
}
```

SSE와 SMS 모두 NotificationInformation 객체를 받아 알림을 전송한다. SQS에서 받아오는 객체 형태는 달라도 동일한 알림 객체를 만들 수 있도록 구성했다.

#### SQS에서 메세지 받은 후 동작 구성하기

SQS에서 메세지를 polling 하게되면 SMS, SSE 전송 후 데이터베이스에 알림 정보를 저장하게된다. 구매자에게 배송 상태 정보를 알려주는 동작을 알아본다.

```java
public void publishDeliveryStartNotification(
    NotificationData<DeliveryNotification> notificationData) {

  NotificationInformation notifyData =
      NotificationInformation.getDeliveryNotificationData(notificationData);
  sse.publishCustomer(notifyData);
  sms.publishCustomer(notifyData);

  // save notification
  notificationCommandService.saveSingleNotification(
      notificationData.getPublishInformation(), notificationData.getWhoToNotify().getUserId());
}
```

메세지로부터 받은 데이터를 `NotificationInformation` 객체로 변환하여 SSE와 SMS를 보내고 알림을 저장한다. `notificationActionHelper`를 사용한 이유는 보통 service 계층에서 데이터 저장과 업데이트 시 `@Transactional`을 사용하여 하나의 트랜잭션으로 묶고 실패 시 롤백을 한다.
service계층에서 SSE, SMS 전송은 트랜잭션에 묶고 싶지 않아 Helper 클래스를 사용하게 되었다.

알림 정보를 저장하는 메서드 또한 `saveSingleNotification()` 을 통해 저장할 수 있도록 구성했다.

```java
public void saveSingleNotification(PublishNotificationInformation publishInformation, Long id) {
  notificationJpaRepository.save(getNotification(publishInformation, id));
}
```

해당 메서드를 가지고 있는 `NotificationCommandService` 클래스에 @Transactional을 붙여 저장시 트랜잭션 처리를 한다.

### 배송 상태 변경 알림 서비스

![Pasted image 20231215005410](https://github.com/lotteon2/lotteon2.github.io/assets/55802893/2278b768-cf6f-4e20-99c6-293532ac8835)

이 프로젝트의 알림 중 배송 상태 변경 알림 프로세스이다. 가게 사장이 배송 상태 변경(시작, 완료) 버튼을 누르는 것을 트리거로 구매자에게 알림이 전송되는 시점 까지의 프로세스이다.

알림 시스템을 설계하며 SQS를 사용하고, 이벤트의 중복 처리에 대해 고민하여 메세지 polling 후 특정 동작을 실패하여 ack 처리를 하지 못해 다시 메세지를 받는 경우 문자나 SSE를 한번만 전송할 수 있는 멱등한 알림 시스템을 구성하는 것이 좋을것 같다.
