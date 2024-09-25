---
title: "Reactor Kafka를 이용한 비동기 메시지 처리"
description: "reactor kafka produce와 consumer에 대해 간단한 예시를 알아본다."
date: 2024-09-25 23:55 +0900
lastmod: 2024-09-25 23:55 +0900
categories: reactor
tags:
  - reactor kafka
  - project reactor
  - retry
---

> Kafka는 메시지 브로커로 실시간 데이터 처리를 위한 높은 처리량, 낮은 지연 시간을 지닌 플랫폼을 제공하는 것이 목표이다. 확장 가능한 pub/sub 메시지 큐로 정의할 수 있으며, 스트리밍 데이터를 처리하기 위한 기업 인프라를 위한 고부가 가치 기능이다. <sub>위키백과</sub>

시스템에서 Kafka를 이용하면 실시간 데이터 스트리밍, 확장성, 내구성 등의 이점을 이용한 분산 시스템 구축이 가능하다. 분산 트랜잭션을 구성하거나 이벤트 기반 시스템을 구성하기 용이하다.
이번 글에서는 Reactor Kafka를 이용한 비동기 메시지 처리를 알아본다.

## Reactor Kafka란?

Reactor Kafka는 Project Reactor의 리액티브 프로그래밍 모델을 이용해서 Apache Kafka와 통합된 라이브러리이다. Kafka에서 제공하는 데이터 스트리밍을 Non-Blocking 방식으로 처리할 수 있게 도와준다. Reactor 프레임워크를 기반으로 한 producer와 consumer를 제공한다.

Reactive Streams 프로토콜을 기반으로 Backpressure 기능을 통해 소비자가 처리할 수 있는 양만큼 데이터를 요청하며, 생산자가 과도한 데이터를 보내지 않도록 조절할 수 있다. 예를 들어, consumer가 처리할 수 있는 속도보다 더 많은 데이터를 producer가 보내는 경우, Backpressure가 이를 제어하여 메모리 과부하나 처리 지연을 방지할 수 있다.

Reactor Kafka는 Flux와 Mono를 이용해서 리액티브 방식으로 메시지를 처리한다. 스레드가 블로킹되지 않고, 병렬 처리가 가능하여 고성능 실시간 데이터 처리에 유리하다. 리액티브 파이프라인 내에서 `onErrorResume`, `onErrorContinue` 등을 이용한 예외 처리 및 재시도 전략을 구성할 수 있다.

Reactor Kafka를 이용한 producer와 consumer를 간단하게 구현해 본다.

## Reactive Producer

```java
@Slf4j
@Service
public class ReactiveKafkaProducer {
  private final KafkaSender<String, String> sender;

  public ReactiveKafkaProducer() {
    Map<String, Object> props = new HashMap<>();
    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:29092");
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);

    SenderOptions<String, String> senderOptions = SenderOptions.create(props);
    this.sender = KafkaSender.create(senderOptions);
  }

  public Mono<Void> sendMessage(String topic, String key, String message) {
    return sender
        .send(Flux.just(SenderRecord.create(topic, null, null, key, message, null)))
        .doOnError(
            error -> {
              log.error("produce error");
              log.error(error.toString());
              error.printStackTrace();
            })
        .doOnNext(
            result -> {
              log.info("produce success");
            })
        .then();
  }
}
```

Producer는 간단하게 구현했다. Producer 설정 값으로 SenderOptions를 생성하고 Sender를 이용해 topic으로 메시지를 생성한다. Flux.just로 Flux 스트림을 생성해서 토픽 정보를 담는다. `SenderRecord.create`에 topic 정보, 파티션, timestamp, 키값, 메시지, meta data를 담아서 생성할 수 있다.

Flux 파이프라인을 사용해서 많은 동작을 제어할 수 있다. 이번 예시에서는 간단하게 `doOnError`, `doOnNext를` 사용해서 구현했다. 메시지 발행 후 에러가 발생하면 로그를 찍고, send에 성공한 다음 메시지 발행 성공했다는 로그를 찍는다. Flux 파이프라인 연산으로 다양한 경우를 제어할 수 있다.

## Reactive Consumer

Reactive consumer는 config 부분과 topic을 consume하는 부분을 나눠서 구현해 보았다.

```java
@Configuration
public class ReactiveKafkaConsumer {

  public ReceiverOptions<String, String> getConsumerConfig() {
    Map<String, Object> props = new HashMap<>();
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:29092");
    props.put(ConsumerConfig.GROUP_ID_CONFIG, "reactive-group");
    props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
    props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
    props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
    return ReceiverOptions.create(props);
  }

  public Flux<ReceiverRecord<String, String>> receiver(String topics) {
    return KafkaReceiver.create(getConsumerConfig().subscription(Collections.singleton(topics)))
        .receive();
  }
}
```

getConsumerConfig로 consumer의 설정을 정의하고 receiver 메서드로 여러 토픽에 대한 listener Flux를 생성할 수 있도록 구현했다. Topic 명을 매개변수로 전달 받아 Flux ReceiverRecord를 반환한다. 토픽의 메시지를 수동으로 읽음처리 할 수 있도록 ReceiverRecord를 사용했다. ReceiverRecord를 사용하면 offset acknowledge를 직접 제어할 수 있다.

```java
@RequiredArgsConstructor
public class ReactiveConsumerListener {
  private final ReactiveKafkaConsumer reactiveKafkaConsumer;
  private final ObjectMapper objectMapper;

  @PostConstruct
  public void reactiveKafkaConsumerListener() {
    reactiveKafkaConsumer
        .receiver("reactive-kafka-topic")
        .doOnError(
            error -> {
              log.error("receiver error");
            })
        .subscribe(
            success -> {
              MemberInformation memberInformation = null;
              try {
                memberInformation =
                    objectMapper.readValue(success.value(), MemberInformation.class);
              } catch (IOException e) {
                throw new RuntimeException(e);
              }
              log.info("consumer message: {}", memberInformation.toString());
              log.info("Message successfully processed and committed.");
              success.receiverOffset().acknowledge();
            },
            error ->
                log.error(
                    "All retries failed, message will not be reprocessed: {}", error.getMessage()));
  }
}
```

`reactiveKafkaConsumer.receiver()` 를 사용해서 Flux 객체를 얻고 Flux의 처리를 담당하는 subscribe()에서 메시지를 가져와서 로그를 찍도록 했다.

```java
.subscribe(
    success -> {
      doMyLogic(success); // 필요한 로직을 메서드로 실행
      success.receiverOffset().acknowledge();
    },
    error ->
        log.error(
            "All retries failed, message will not be reprocessed: {}", error.getMessage()));
```

이렇게 메세지를 메서드에 담아 필요한 로직처리는 따로 할 수 있다. 또한 Flux 파이프라인 연산에서 retry(), retryBackoff(), retryWhen()을 이용한 재시도 전략도 설정할 수 있다.

```java
.receiver("reactive-kafka-topic")
.flatMap(
    receiverRecord -> {
      return Mono.fromCallable(
              () -> {
                String message = receiverRecord.value();
                MemberInformation memberInformation;

                try {
                  memberInformation =
                      objectMapper.readValue(message, MemberInformation.class);

				  ...(check something)

                } catch (IOException e) {
                  log.error("Failed to deserialize message: {}", message, e);
                  throw new RuntimeException(e);
                }

                return receiverRecord;
              })
          .doOnSuccess(
              record -> {
                log.info("Message processed successfully: {}", record.value());
                record.receiverOffset().acknowledge();
              })
          .doOnError(
              error -> {
                log.error("Processing failed: {}", error.getMessage());
              });
    })
.retryWhen(
    errors ->
        errors
            .zipWith(Flux.range(1, 3), (error, attempt) -> attempt)
            .flatMap(
                attempt -> {
                  log.warn("Retrying attempt #{}", attempt);
                  return Mono.delay(Duration.ofSeconds(1));
                }))
```

receiver로부터 ReceiverRecord를 받아온 후 레코드로부터 메시지를 확인하고 데이터 처리 중 에러가 발생하면 retryWhen을 이용해 재시도 전략을 사용할 수 있다.
위 코드는 로직에서 에러가 발생하면 3번까지 시도하게 된다. 1초 간격으로 시도하게 되고 Flux.range에서 attempt로 시도 횟수를 로그로 찍게 했다.

Reactive kafka를 이용해서 메시지를 발행하고 소비하는 예시를 간단하게 알아보았다. Flux와 Mono를 이용해서 비동기 처리를 할 수 있는 특징과 비동기에 적합한 부분에 사용할 수 있다.
