---
title: Spring Framework 파헤쳐보기
categories: spring
date: 2022-10-17 23:20 +0900
description: spring에서 DI와 AOP에 대해 알아본다
lastmod: 2022-10-17 23:20 +0900
tags: aop di java spring
---

## Spring Framework

Spring 프레임워크는 자바를 사용해 Enterprise Application을 만들 때 포괄적으로 사용하는 프로그래밍 및 Configuration Model을 제공해 주는 프레임워크로 Application 수준의 인프라를 제공한다.

Spring 프레임워크는 개발자가 복잡하고 실수하기 쉬운 low level에 신경쓰지 않고 business logic 구현에만 집중할 수 있게 해준다.

Spring 컨테이너는 자바 객체의 생성, 소멸과 같은 라이프사이클을 관리한다. 언제든지 스프링 컨테이너로부터 필요한 객체를 가져와 사용할 수 있다.

스프링은 JDBC와 Mybatis, Hibernate, JPA 등 DB 처리를 위해 사용되는 라이브러리와의 연동을 지원한다. 또한 JMS, 메일, 스케쥴링 등 다양한 API를 설정파일과 Annotation을 통해 쉽게 사용할 수 있도록 지원한다.

## Spring Framework의 구조

### POJO(Plain Old Java Object)

특정 환경이나 기술에 종속적이지 않은 객체지향 원리에 충실한 자바 객체이다. 테스트에 용이하며 객체지향 설계를 자유롭게 적용할 수 있는 특징이 있다.

### PSA(Portable Service Abstraction)

환경과 세부기술의 변경과 관계없이 일관된 방식으로 기술에 접근할 수 있게 해주는 설계원칙으로 트랜잭션 추상화, OXM 추상화, 데이터 엑세스의 Exception 변환기능 등 기술적인 복잡함은 추상화하여 low level의 기술 구현 부분과 기술을 사용하는 인터페이스로 분리한다.

### IoC(Inversion of Control)/DI(Dependency Injection)

DI는 유연하게 확장 가능한 객체를 만들어 두고 객체 간의 의존관계는 외부에서 다이나믹하게 설정한다. IoC는 Servlet container 및 EJB container에 객체 제어권이 주어지는 특징이 있다.

### AOP(Aspect Oriented Programming)

관심사의 분리를 통해서 소프트웨어의 모듈성을 향상시킨다. 공통 모듈을 여러 코드에서 쉽게 적용 가능하게 만들어 준다.

## Container

container는 라이프사이클을 관리하고 스레드 관리와 의존성 객체를 제공한다. container는 비즈니스 로직 외에 부가적인 기능에 대해서 독립적으로 관리되도록 하기 위해 존재한다. 서비스 객체를 사용하기 위해 각 Factory나 Singleton 패턴을 직접 구현하지 않아도 된다.

### DI Container

DI 컨테이너가 관리하는 객체를 Bean이라고 하고 Bean의 라이프사이클을 관리하는 것이 BeanFactory이다. BeanFactory에 여러 컨테이너 기능을 추가하여 ApplicationContext라고 한다.

Bean은 스프링이 IoC 방식으로 관리하는 object를 말한다. Bean은 스프링이 직접 생성과 제어를 담당한다.

BeanFactory는 스프링 IoC를 담당하는 핵심 컨테이너로 Bean의 등록, 생성, 조회, 반환을 담당한다. ApplicationContext는 BeanFactory를 확장한 IoC컨테이너이다. 기본 동작은 BeanFactory와 동일하며, 스프링이 제공하는 부가 서비스를 추가로 제공한다.

### IoC Container

스프링에서 IoC를 담당하는 컨테이너는 BeanFactory, ApplicationContext가 있다. Object의 생성과 관계 설정, 사용, 제거 등 작업을 애플리케이션 코드 대신 독립된 컨테이너가 담당한다.

객체 생성을 컨테이너에 위임하여 처리하므로 객체 간의 결합도를 낮출 수 있다. 프로젝트의 결합도가 높으면 유지보수 시 하나의 클래스가 변경될 때 다수의 클래스들도 변경되어야 하므로 결합도가 낮은 프로젝트가 유지보수에 용이하다.

![스크린샷 2022-10-18 오전 8.14.02.png](/assets/posting/spring/bean/pic1.png)

IoC의 호출 방식은 위와 같다. 팩토리 패턴의 장점을 더하여 어떠한 것에도 의존하지 않는 형태를 가진다. 실행 시점에 클래스 간의 관계가 형성된다.

applicationContext.xml에서 bean을 이용해 service 구현체 객체의 생성 및 관리를 하고, controller에서 ApplicationContext의 ClassPathXmlApplicationContext로 xml파일을 읽어 각 bean을 불러와 service를 불러온다.

## DI(Dependency Injection)

```java
@Component("memberService")
@Scope("singleton") // singleton으로 생성
public class MemberServiceImpl implements MemberService {
    @Override
    public void insertMember(String name) {

    }
}
```

```java
<beans:bean id="memberService" class="com.hello.myapp.MemberServiceImpl" scope="prototype"/>
```

스프링 Bean은 싱글톤으로 만들어지므로 컨테이너가 제공하는 모든 빈의 인스턴스는 항상 동일하다. 싱글톤 정의는 annotation으로 한다. 컨테이너가 항상 새로운 인스턴스를 반환하게 만들고 싶은 경우 scope를 prototype으로 설정해줘야 한다.

#### Bean의 범위 지정

##### singleton: 스프링 컨테이너당 하나의 인스턴스 빈만 생성(default)

##### prototype: 컨테이너 빈을 요청할 때마다 새로운 인스턴스 생성

##### request: HTTP Request 별로 새로운 인스턴스 생성

##### session: HTTP Session 별로 새로운 인스턴스 생성

스프링 Bean설정은 xml과 annotation 방식으로 가능하다. xml에서는 `<bean>`태그를 사용해 제어한다.

어플리케이션의 규모가 커지고 bean의 개수가 많아질 경우 xml 파일을 관리하는 것이 힘들어진다. Bean으로 사용될 클래스에 annotation을 부여해 자동으로 bean을 등록해 준다.

오브젝트 빈 스캐너로 빈 스캐닝을 통해 자동으로 등록된다. Bean 스캐너는 기본적으로 클래스 이름의 bean의 아이디로 사용한다.

```xml
<context:component-scan base-package="com.hello.cafe" />
```

annotation으로 bean을 사용할 경우 servlet-context.xml에서 context에 component-scan을 추가해 줘야 한다.

#### spring bean annotation의 종류

##### @Repository: Data Access Layer의 DAO 또는 Repository 클래스에 사용된다. Data Access Exception 자동변환과 같은 AOP의 적용 대상을 선정하기 위해 사용된다.

##### @Service: Service layer의 클래스에 사용된다.

##### @Controller: Presentation Layer의 MVC controller에 사용된다. 스프링 웹 서블릿에 의해 웹 서블릿에 의해 웹 요청을 처리하는 컨트롤러 bean으로 선정한다

##### @Component: 위의 layer 구분을 적용하기 어려운 일반적인 경우 설정한다.

## spring bean 의존관계 설정하기 (xml)

객체 또는 값을 constructor를 통해 주입 받는다. constructor-arg 태그는 `<bean>`의 하위 태그로 설정한 bean 객체 또는 값을 생성자를 통해 주입하도록 설정한다.

`<constructor-arg>`의 하위 태그로 ref와 value가 있다. ref는 객체 주입 시 사용하고 `<ref bean=”bean name” />` 으로 사용한다. value에는 문자열이 들어가며, primitive data 주입 시 사용한다. value의 속성으로 type이 들어갈 수 있다.

속성으로 사용할 수도 있다. `<constructor-arg ref=”bean name" />`으로 사용하고 `<constructor-arg value=”값” />`으로 사용한다.

```xml
<bean id="dao" class="" />

<bean id="service" class="">
		<constructor-arg ref="dao" />
</bean>
```

위 코드 처럼 주입하게 된다.

Bean 의존 관계설정에 property 태그를 사용할 수 있다. property 또한 하위 태그와 속성으로 이용할 수 있다. 하위 태그인 경우 constructor-arg와 동일하며, 속성으로 사용할 경우 `<property name=”property name” ref=”bean name” />`, `<property name=”property name” value=”값” />` 으로 사용한다.

## Annotation 방식을 사용한 bean 의존 관계 설정

Annotation은 멤버 변수에 직접 정의 하는 경우 setter method를 만들지 않아도 된다. 앞으로 많이 사용하게 될 annotation 방식 bean 의존 관계를 알아보자.

| annotation | 설명                                                                                                                                                            |
| :--------: | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| @Resource  | Spring 2.5부터 지원<br>멤버 변수, setter method에 사용 가능<br>타입에 맞춰 연결                                                                                 |
| @Autowired | Spring 2.5 부터 지원<br>Spring에서만 사용 가능<br>Required 속성을 통해 DI여부 조정<br>멤버 변수, setter, constructor, 일반 method 사용 가능<br>타입에 맞춰 연결 |
|  @Inject   | Spring 3.0부터 지원<br>Framework에 종속적이지 않다<br>Javax.Inject.x.jar가 필요<br>멤버 변수, setter, constructor, 일반 method 사용 가능<br>이름으로 연결       |

## Bean 객체의 생성 단위

|   scope   | 설명                                               |
| :-------: | -------------------------------------------------- |
| singleton | 스프링 컨테이너당 하나의 인스턴스 빈만 생성        |
| prototype | 컨테이너에 빈을 요청할 때마다 새로운 인스턴스 생성 |
|  request  | HTTP Request 별로 새로운 인스턴스 생성             |
|  session  | HTTP Session 별로 새로운 인스턴스 생성             |

## Bean의 life cycle

지금까지 봤던 bean의 생명주기이다.

![beanlifecycle.png](/assets/posting/spring/bean/beanlifecycle.png)
