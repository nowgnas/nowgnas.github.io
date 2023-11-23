---
title: "[MSA 1] spring cloud로 프로젝트 초기 세팅하기"
description: spring cloud로 MSA 아키텍처를 구성해본다
date: 2023-11-14 23:03 +0900
lastmod: 2023-11-14 23:03 +0900
categories: msa
tags: kafka, msa, spring cloud
---

## MSA 아키텍처로 설계하기

아키텍처에는 대표적으로 monolithic, micro service architecture 가 있다. 클라이언트와 서버가 합쳐진 하나의 거대한 아키텍처이고, msa는 작고 독립된 서비스들로 구성된 아키텍처로 독립적인 배포와 서비스별로 다른 기술을 사용해도 되는 장점을 가지고 있다.

독립적 배포와 장애 확산을 줄일 수 있는 msa 구조로 프로젝트를 설계, 개발하려고 한다.

## 프로젝트 적용 기술 스택

| name         | version  |
| :----------- | :------- |
| spring cloud | 2021.0.8 |
| spring boot  | 2.7.17   |
| jdk          | 11       |

백엔드 서버를 구성하게될 기본적인 기술과 버전이다.

## spring cloud로 msa 구축하기

### Discovery service

spring cloud는 netflix에서 만든 eureka server를 사용해 구성할 수 있다. eureka server는 micro service 들이 eureka server에 등록하게되면 host, port, service url 등 micro service의 메타데이터를 전달한다. eureka server는 각 micro service들로부터 heartbeat를 지속적으로 받게되고 heartbeat를 받지 못하게 되면 eureka server 레지스트리에서 제거한다.

![Screenshot 2023-11-20 at 11.49.18.png](https://github.com/nowgnas/nowgnas.github.io/assets/55802893/03e99d81-9b67-4b13-a2f8-841f3ec28aa8)

intellij에서 spring boot initializer를 사용해 프로젝트를 시작한다. 프로젝트 기본 정보를 입력한 후 Next를 누르면 위와 같은 화면이 나온다.

discovery service는 eureka server만 추가해주면 된다.

```java
@SpringBootApplication
@EnableEurekaServer // 추가
public class DiscoveryServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(DiscoveryServiceApplication.class, args);
    }

}
```

프로젝트를 생성한 후 \*\*Application 파일에서 `@EnableEurekaServer`를 추가해주면 된다.

```yaml
server:
  port: 8761
spring:
  application:
    name: discovery-service
  config:
    activate:
      on-profile: local, dev, prod # 실행 환경
eureka:
  client:
    register-with-eureka: false # eureka에 등록할 것인가?
    fetch-registry: false # 레지스트리에 있는 정보를 가지고 올 것인가?
```

application.yml은 서비스가 실행될 때 필요한 변수들을 정의한 것이다. 포트와 application name, 실행 환경을 기본적으로 정의하고, eureka server는 자신을 레지스트리에 등록하지 않기 위해 client 하위 속성들을 false로 설정한다. (기본값은 true이다)

![Screenshot 2023-11-20 at 12.17.52.png](https://github.com/nowgnas/nowgnas.github.io/assets/55802893/01ab8629-f6b5-4dbe-8449-36c8470e19dc)

discovery service를 실행 후 localhost:8761로 접속하면 위와 같은 화면을 확인할 수 있다. 중간에 `Instance currently registered with Eureka`에 다른 micro service의 정보들이 추가된다.

### Config service

Spring cloud config는 micro service 개발 시 application 파일을 외부에서 관리와 환경변수 암호화 등의 기능을 지원해준다.

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'org.springframework.cloud:spring-cloud-config-server'
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    implementation "org.springframework.cloud:spring-cloud-starter-bus-kafka"
}
```

config service에서는 config server, eureka client가 필요하고 이번 프로젝트에서는 micro service들의 환경설정 파일을 서버 재시작 없이 동기화할 수 있는 kafka bus refresh를 사용한다. bus kafka와 actuator를 추가하여 사용할 수 있다.

```yaml
# application.yml
server:
  port: 8888
spring:
  config:
    activate:
      on-profile: local, dev, prod
  application:
    name: config-service
  cloud:
    bus:
      enabled: true
    config:
      server:
        git:
          uri: https://github.com/lotteon2/BB-CONFIG.git # application.yml 모아둔 레포 주소
          username: ${github.username}
          password: ${github.token}
management:
  endpoints:
    web:
      exposure:
        include:
          - "refresh"
          - "bus-refresh"
encrypt:
  key: ${secret} # cloud config의 encrpt를 위한 키
---
# application-local.yml
spring:
  kafka:
    bootstrap-servers: localhost:9092

eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://127.0.0.1:8761/eureka # eureka server 주소
  instance:
    instance-id: ${spring.application.name}:${server.port}
```

config service의 환경 설정 정보들이다. spring.cloud.config.server.git에 외부에서 관리할 레포 주소와 본인의 github 정보를 환경변수로 받는다.

management는 actuator에서 사용할 수 있는 메서드를 정의한다. application.yml은 실행 환경에 관계없이 적용할 값들을 정의한 것이다. application-local.yml에는 kafka와 eureka server에 대한 설정을 정의한다.

![Screenshot 2023-11-20 at 12.44.46.png](https://github.com/nowgnas/nowgnas.github.io/assets/55802893/e180b924-c215-4327-bdf1-9685e5d134bb)

discovery service와 config service를 실행하면 등록된 config service의 정보를 확인할 수 있다.

### Config repository

![Screenshot 2023-11-20 at 13.45.09.png](https://github.com/nowgnas/nowgnas.github.io/assets/55802893/078fbb08-4d80-4a53-a2eb-4291ea3ff905)

위 그림은 환경 설정 파일만 모아놓은 private 레포이다. 서비스별, 환경별로 파일을 생성해둔다. 필요한 변수들을 추가, 수정을 한 곳에서 할 수 있어 관리하기 편리하다.

api key나 secret과 같은 민감한 정보들은 spring cloud config 의 encryption을 사용해 암호화된 형태로 저장하여 관리한다.

![Screenshot 2023-11-20 at 13.50.09.png](https://github.com/nowgnas/nowgnas.github.io/assets/55802893/36bc60ba-ff72-4b62-a136-167a28a81f42)

```bash
POST http://localhost:8888/encrypt
POST http://localhost:8888/decrypt
```

postman에서 위와 같이 요청하여 암호화된 값을 얻을 수 있다. application.yml의 `encrypt.key`에 할당된 값으로 암,복호화 할 수 있다. 암호화된 값의 사용은 다음과 같다

```yaml
cloud:
  aws:
    credentials:
      ACCESS_KEY_ID: "{cipher}2602aa08e31937387e3e585209566fc7aa"
```

### kafka bus refresh

```yaml
version: "3"
services:
  zookeeper:
    image: wurstmeister/zookeeper
    ports:
      - "2181:2181"
  kafka:
    image: wurstmeister/kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: localhost
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    depends_on:
      - zookeeper
```

이번 프로젝트에서는 kafka를 이용해 bus refresh를 해본다. local에서 테스트를 위한 `docker-compose.yml` 파일이다.

spring cloud bus 는 분산 시스템에서 노드들의 메세지 브로커와 연결하여 변경된 환경 설정을 간편하게 적용시킬 수 있다. config service의 endpoint `/busrefresh`로 요청하게되면 config repository에서 변경된 사항이 각 MA에 적용된다. bus refresh는 각 서비스들을 재시작 하지 않아도 되는 장점이 있다.

![Screenshot 2023-11-23 at 23.28.02.png](https://github.com/nowgnas/nowgnas.github.io/assets/55802893/53a2194c-db77-4d9e-bbcb-8dfb8d1c74cc)

bus refresh를 하게되면 위와 같이 Keys refreshed 배열에 config repository에서 변경된 사항을 확인할 수 있다.

운영이나 개발환경에서는 config repository에 파일이 변경되면 github action으로 `<service-domain>/busrefresh` 요청을 하는 방법으로 사용할 수 있을 것 같다.

MSA로 백엔드를 구성하기 위한 기초 세팅이 완료되었다. Api gateway를 포함한 여러 도메인 서비스들을 개발하면 된다.
