---
title: "대규모 트래픽을 고려한 장애 허용 with circuit breaker"
date: 2024-07-08 00:10 +0900
lastmod: 2024-07-08 00:10 +0900
tags:
  - msa
  - open feign
  - fault tolerance
categories: msa
description: 대규모 트래픽을 고려한 장애 허용
---

## 대규모 트래픽 발생 시 지연에 대비하는 방법

서비스에서 사용자 수가 급격히 증가하는 이벤트가 있을 때, 대규모 트래픽을 효율적으로 처리하는 것이 중요하다. MSA로 구성된 서비스에서 지연 발생 시 장애 전파를 막기 위해서는 발생한 장애가 다른 micro service에 영향을 주지 않아야 한다.
정보 조회의 경우 응답 시간이 길어지거나 응답에 실패할 수 있고, 중요한 데이터를 변경하거나 결제와 같은 요청에 실패가 발생하지 않아야 한다. 이를 해결하기 위해서는 여러가지 방법이 있는데 이번 글에서는 서비스의 안정성을 유지하기 위한 방법 중 **circuit breaker 패턴**을 소개한다.

## Open feign으로 micro service api 호출하기

MSA 환경에서는 다른 ms에 데이터를 요청해서 처리가 되어야 하는 경우가 있다. Spring cloud의 Open feign을 사용해서 요청해본다.

### 의존성 추가하기

```java
repositories {
    mavenCentral()
    maven { url 'https://jitpack.io' }
}

dependencyManagement {
    imports {
        mavenBom 'org.springframework.cloud:spring-cloud-dependencies:Greenwich.SR6'
    }
}

dependencies {
	...
    implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'
	...
}

```

Open feign의 버전을 선택할 때는 spring boot, sprig cloud의 버전을 확인하고 적합한 버전을 사용해야 한다. 이번 테스트에서는 spring boot 2.1.x를 사용하고 spring cloud는 Greenwich를 사용했다.

### Open feign 기본 설정

```java
@SpringBootApplication
@EnableFeignClients
public class CircuitApplication {

    public static void main(String[] args) {
        SpringApplication.run(CircuitApplication.class, args);
    }

}
```

@EnableFeignClients 을 추가한다.

```java
@FeignClient(name = "notificationClientService", url = "${endpoint.url}", fallback = NotificationClientFallbackService.class)
public interface NotificationClientService {

    @GetMapping(consumes = MediaType.APPLICATION_JSON_VALUE, produces = MediaType.APPLICATION_JSON_VALUE, path = "/v1/test/api")
    ClientApiResponseModel getApiClientResponse();
}
```

@FeignClient를 클래스에 추가하여 client 이름, 요청할 url, 요청 실패시 처리할 fallback 클래스를 지정할 수 있다.
GetMapping에서 consumes와 produces는 소비와 생성 시 사용할 media type을 지정한다. 어떤 형태의 데이터를 주고 받을지 정의한다.

```java
@Component
public class NotificationClientFallbackService implements NotificationClientService {

    @Override
    public ClientApiResponseModel getApiClientResponse() {
        return ClientApiResponseModel.builder()
                .resultCode("ERROR")
                .build();
    }
}
```

fallback 클래스는 feign client 클래스를 상속받아 구현한다. Feign client 요청에 실패하게되면 각 요청에 대해 정의한 fallback응답을 반환한다.

```yaml
feign:
  client:
    config:
      default:
        connectTimeout: 3000
        readTimeout: 3000
      notificationClientService:
        connectTimeout: 500
        readTimeout: 500
```

open feign의 configuration은 application.yml에서 설정이 가능하다. 기본값과 feign client마다 다른 값을 정할 수 있다.

이제 다른 ms에 api 요청을 보낼 수 있게 설정했다. 응답을 받아오지 못한 경우 fallback 처리도 되었다. 이제 장애 전파를 막을 수 있는 circuit breaker 패턴을 적용해보자.

## Circuit Breaker 패턴

![Pasted image 20240707152713](/assets/img/posts/circuit_breaker/circuit_breaker.png)
Circuit breaker 패턴은 세 가지 일반 상태와 두 가지 특수 상태를 가진다. 호출 결과를 집계하여 circuit breaker를 제어한다. 집계를 위해 sliding window를 사용하고 이는 count based와 time based가 있다.

### Count based

count based 슬라이딩 윈도우는 N개의 측정값으로 구성된 원형 배열로 구현된다. 호출 결과가 원형 배열에 계속 쌓이게 된다. 매번 새로운 호출 값이 저장되면서 가장 마지막 값은 evict된다. 이렇게 마지막 N번의 결과를 가지고 집계를 한다.

### Time based

시간 기반 슬라이딩 윈도우는 마지막 N초 동안의 호출 결과를 집계한다. 시간 기반 슬라이딩 윈도우 크기가 10초라면, 원형 배열은 10개의 부분 집계를 가진다.
각 버킷은 특정 epoch 초에 발생하는 모든 호출의 결과를 집계한다. 원형 배열의 head는 현재 epoch 초에 대한 결과를 집계한다. 새 호출 결과에 따라 점진적으로 업데이트 되며, 가장 마지막에 저장된 버킷은 evicte된다.

### Failure rate and slow call rate thresholds

Circuit breaker는 실패율이 임계값을 넘어가면 상태가 CLOSED에서 OPEN 상태로 변경된다. Circuit이 OPEN 상태이면 더 이상 호출하지 않고 정의된 fallback 메서드의 결과를 반환한다.
실패율 외에도 느린 호출에 대해 임계값을 넘어가게 되면 동일하게 상태가 CLOSED에서 OPEN 상태로 변경된다. 타 모듈에서 응답이 불가하기 전에 해당 API에 대해 부하를 줄일 수 있는 장점을 가지고 있다.

Circuit breaker의 상태가 OPEN인 경우는 `CallNotPermittedException`으로 호출이 거부된다. 일정 시간이 지나면, 상태가 HALF_OPEN으로 변경되고, API가 응답 가능한 상태인지 확인할 수 있도록 일정 호출을 허용한다. 허용된 호출이 모두 성공하기 전까지 이외의 호출은 모두 거부된다. HALF_OPEN 상태에서 다시 실패율이나 느린 호출이 임계값을 넘어가게되면, OPEN 상태가 되고, 임계값 이하이면 CLOSED 상태로 변경되어 모든 호출이 허용된다.

### Circuit breaker 설정값

| Config property                                   | Default Value | Description                                                                                           |
| ------------------------------------------------- | ------------- | ----------------------------------------------------------------------------------------------------- |
| failureRateThreshold                              | 50            | 실패율 임계값을 백분율<br><br>                                                                        |
| slowCallRateThreshold                             | 100           | 느린 호출 임계값 백분율 <br>                                                                          |
| slowCallDurationThreshold                         | 60000 [ms]    | 느린 호출로 정의할 시간 임계값                                                                        |
| permittedNumberOfCalls <br>InHalfOpenState        | 10            | HALF_OPEN 상태에서 허용할 호출 수                                                                     |
| maxWaitDurationInHalfOpenState                    | 0 [ms]        | HALF_OPEN에서 대기 시간으로 기본값은 허용된 모든 호출이 완료될 때까지 상태를 유지한다.                |
| slidingWindowType                                 | COUNT_BASED   | 슬라이딩 윈도우 타입으로 기본값은 count based이다. TIME_BASED로 하면 시간 기반 슬라이딩 윈도우로 설정 |
| slidingWindowSize                                 | 100           | 슬라이딩 윈도우 사이즈                                                                                |
| minimumNumberOfCalls                              | 100           | 실패율 집계 최소 값<br>100으로 가정하면 99 호출까지는 모두 실패해도 상태가 변경되지 않음              |
| waitDurationInOpenState                           | 60000 [ms]    | OPEN에서 HALF_OPEN으로 전환되기까지 대기할 시간                                                       |
| automaticTransition <br>FromOpenToHalfOpenEnabled | false         | OPEN 상태에서 자동으로 HALF_OPEN 상태로 전환 여부                                                     |
| recordExceptions                                  | empty         | 실패율 집계에 사용될 예외를 지정                                                                      |
| ignoreExceptions                                  | empty         | 호출의 실패나 성공으로 간주하지 않을 예외를 지정                                                      |

## Circuit breaker 적용하기

```gradle
dependencies {
	...
    implementation 'io.github.resilience4j:resilience4j-spring-boot2:1.4.0'
    implementation 'io.github.resilience4j:resilience4j-circuitbreaker:1.4.0'
    ...
}
```

Circuit breaker 의존성을 추가한다. resilience4j에는 circuit breaker 외에도 retry, bulkhead 등 여러가지 의존성이 있다. 이번 글에서는 circuit breaker만 사용한다.

### application.yml 설정

```yaml
resilience4j.circuitbreaker:
  configs:
    default:
      registerHealthIndicator: true
      slidingWindowSize: 10
      slidingWindowType: COUNT_BASED
      permittedNumberOfCallsInHalfOpenState: 3
      waitDurationInOpenState: 70000
      failureRateThreshold: 50
  instances:
    getApiClientResponse:
      baseConfig: default
```

resilience4j도 open feign과 동일하게 application.yml에서 값 설정이 가능하다. 기본 값을 설정하고 circuit breaker 마다 각각 다른 설정 값을 사용할 수 있다.

```java
@FeignClient(name = "notificationClientService", url = "${endpoint.url}")
public interface NotificationClientService {
    @GetMapping(
            consumes = MediaType.APPLICATION_JSON_VALUE,
            produces = MediaType.APPLICATION_JSON_VALUE,
            path = "/v1/channel/test/api/delay"
    )
    @CircuitBreaker(name = "getApiClientResponse", fallbackMethod = "getApiClientResponseFallback")
    ClientApiResponseModel getApiClientDelayResponse();


    default ClientApiResponseModel getApiClientResponseFallback(Throwable T) {
        return ClientApiResponseModel.builder()
                .resultCode("resilience4j ERROR")
                .build();
    }
}
```

Open feign 요청에 대해 circuit breaker를 적용한 부분이다. Circuit breaker 인스턴스 이름과 fallback 메서드를 정의했다. 기본값을 사용하도록 설정하여 호출이 10번 실패하게되면 circuit breaker의 상태가 OPEN이 되어 호출이 거부되고 fallback 메서드가 동작한다. 7초 후 HALF_OPEN으로 전환이 되고 3번의 호출 이 성공한다면 상태는 CLOSED로 변경된다.

여기서 한가지 특징은 위 코드처럼 feign client에 circuit breaker를 설정하면, open feign의 fallback은 동작하지 않는다는 것이다. Circuit breaker가 open feign을 감싸고 있기 때문에 호출에 실패하게 되면 circuit breaker의 fallback만 동작하게 되는 것이다.

## Circuit breaker 패턴 도입 기대 효과

Circuit breaker를 적용했을 때의 효과는 외부 api 호출에 대해 장애 발생 시에 유연하게 처리 가능하다고 생각한다. 특히 MSA 아키텍처에서 fault tolerance(장애 허용)은 일부 서비스의 장애에도 전체적인 서비스에는 영향이 없도록 장애 전파를 막는 역할을 한다.
트래픽이 많이 몰릴 수 있는 서비스에서 circuit breaker 패턴은 서비스를 안정적으로 운영할 수 있는 좋은 방법이다.
