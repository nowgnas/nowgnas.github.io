---
title: AOP를 이용한 예외처리 분리하기
categories: spring-boot
date: 2022-10-26 20:23 +0900
description: AOP를 이용해서 예외처리를 사용자가 정의하여 분리한다
lastmod: 2022-10-26 20:23 +0900
tags: spring-boot aop exception java
---

## AOP (Aspect Oriented Programming)

AOP는 관점지향 프로그래밍으로 로직을 기준으로 핵심 관점과 부과적인 관점으로 나눠서 보고 그 관점을 기준으로 각각을 모듈화 하는 것이다.

핵심 기능은 비즈니스 로직을 구현하는 과정에서 비즈니스 로직이 처리하려는 목적 기능을 말한다. 클라이언트로부터 상품 정보 등록 요청을 받아 DB에 저장하고, 상품 정보를 조회하는 비즈니스 로직을 구현할 경우, 정보를 저장하는 것과 정보 데이터를 보여주는 부분이 핵심이다.

클래스들은 Aspect를 재활용하여 사용할 수 있다. Service 비즈니스 로직에서 트랜잭션이라는 부가 기능 관심사를 분리할 수 있다.

이번 포스팅에서는 AOP를 이용해 웹 서비스에서 발생하는 예외를 처리해 본다.

## 사용자 정의 예외처리

```java
// GlobalExceptionHandler.java

@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(NullPointerException.class)
    public ResponseEntity<ErrorResponse> NullPointerException(NullPointerException n) {
        ErrorResponse response = new ErrorResponse(404, n.getMessage());
        return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
    }

    @ExceptionHandler(SQLSyntaxErrorException.class)
    public ResponseEntity<ErrorResponse> SqlSyntaxErrorException(SQLSyntaxErrorException s) {
        System.out.println("sql syntax error");
        ErrorResponse response = new ErrorResponse(404, s.getMessage());
        return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
    }

    @ExceptionHandler(BadSqlGrammarException.class)
    public ResponseEntity<ErrorResponse> BadSqlGrammarException(BadSqlGrammarException b) {
        System.out.println("bad grammar");
        ErrorResponse response = new ErrorResponse(404, b.getMessage());
        return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
    }

    @ExceptionHandler(Exception.class)
    public String Exception(Exception e) {
        System.out.println("Exception");
        return "sql grammar error";
    }

    @ExceptionHandler(RuntimeException.class)
    public String RuntimeException(RuntimeException r) {
        System.out.println("run time exception");
        return "run time exception";
    }

}
```

`GlobalExceptionHandler.java`이다. 발생할 수 있는 예외들을 정의해 놓은 파일이다. ControllerAdvice 어노테이션으로 모든 controller에 대한 예외를 처리해 준다. 각 메서드에는 ExceptionHandler 어노테이션이 붙어 있다. ExceptionHandler는 어떤 예외 클래스에 대한 처리를 할지 정의해 주는 것이다. ExceptionHandler로 예외 클래스를 지정해 주기 때문에 각 메서드들의 이름은 큰 의미가 없다. 웹 서비스에서 일부러 예외를 발생시켜 확인해 보자.

```java
// ErrorResponse.java

@Getter
public class ErrorResponse {
    private String message;
    private int status;

    public ErrorResponse(int i, String message) {
        this.status = i;
        this.message = message;
    }
}
```

위 코드는 `GlobalExceptionHandler.java`에서 사용하는 `ErrorResponse` 객체이다.

### BadSqlGrammarException

```xml
<select id="login" resultType="Member" parameterType="map">
    select *
    from member_
    where id = #{id}
      and pw = #{pw};
</select>
```

위 코드는 mybatis(mybatis 설명은 생략한다)로 정의한 사용자 로그인 쿼리이다. `from member`가 올바른 쿼리이며, `member_`로 쿼리에서 오류를 발생시켜보았다. 로그인 요청을 하게되면 BadSqlGrammarException이 발생하게 된다.

![스크린샷 2022-10-26 오후 8.46.10.png](/assets/posting/spring/aop/pic1.png)

```java
@ExceptionHandler(BadSqlGrammarException.class)
public ResponseEntity<ErrorResponse> BadSqlGrammarException(BadSqlGrammarException b) {
    System.out.println("bad grammar");
    ErrorResponse response = new ErrorResponse(404, b.getMessage());
    return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
}
```

ExceptionHandler에 예외 클래스를 정의해 준다. ErrorResponse 객체로 오류 발생 시 반환할 객체를 생성하고 ResponseEntity로 반환해 준다.

![스크린샷 2022-10-26 오후 8.40.22.png](/assets/posting/spring/aop/pic2.png)
postman으로 로그인 요청을 보낸 후 받은 응답 body이다. 요청에 대한 응답은 ErrorResponse 객체를 받은 것을 확인할 수 있다. errors와 code는 정의하지 않아 null로 반환되었다. 코드 위쪽의 이미지에서 BadSqlGrammarException이 콘솔에 찍혀 있는것을 확인할 수 있다.

### NullPointerException

다시 사용자 로그인을 사용해 NullPointerException을 발생시켜 본다. 위에서 로그인 쿼리는 다시 되돌려 놓는다. 이번에는 데이터베이스에 존재하지 않는 정보로 로그인을 시도한다.

![스크린샷 2022-10-26 오후 8.54.17.png](/assets/posting/spring/aop/pic3.png)
id가 p이고 pw가 1234인 사용자는 데이터에 존재하지 않는다. 이 데이터를 가지고 로그인 요청을 하게되면 NullPointerException이 발생한다.

![스크린샷 2022-10-26 오후 8.56.41.png](/assets/posting/spring/aop/pic4.png)

```java
@ExceptionHandler(NullPointerException.class)
public ResponseEntity<ErrorResponse> NullPointerException(NullPointerException n) {
    n.printStackTrace();
    ErrorResponse response = new ErrorResponse(404, n.getMessage());
    return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
}
```

ExceptionHandler에 NullPointerException으로 예외 클래스를 정의해 준다. 데이터베이스에 존재하지 않는 사용자이므로 NullPointerException가 발생하는 것을 볼 수 있다. ErrorResponse로 응답 객체를 만들고 ResponseEntity로 json 형태로 반환하게 되면 다음과 같은 응답을 얻을 수 있다.

![스크린샷 2022-10-26 오후 8.58.56.png](/assets/posting/spring/aop/pic5.png)
에러의 메세지와 지정해준 상태 코드가 담겨 응답으로 반환된다. errors와 code 데이터는 추가하지 않아 null을 반환하게 된다.

AOP를 이용해서 controller에 클래스에 따른 예외처리를 적용해 보았다. 핵심 기능과 부가 기능으로 구분해서 각각을 하나의 관점으로 본다는 AOP의 관점을 조금 이해할 수 있었던 예제였다.
