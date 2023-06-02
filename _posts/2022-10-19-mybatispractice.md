---
title: mybatis를 이용한 spring legacy project 구조를 알아보자
categories: spring
date: 2022-10-18 18:37 +0900
description: mybatis를 사용한 spring legacy project 구조를 알아본다
lastmod: 2022-10-18 18:37 +0900
tags: java mvc mybatis spring sts
---

이번 포스팅에서는 sts3에서 Spring Legacy Project를 생성하고 웹 페이지에서 사용자 회원가입, 로그인을 mybatis를 이용해 구현한다.

## 🛠기술 스택

| position | stack                     | version |
| -------- | ------------------------- | ------- |
| server   | Java                      | jdk 17  |
| database | mysql                     | 8.x     |
| client   | html<br>javascript<br>css | -       |

## Project structure

#### Server 폴더 구조

```
.
└── nowgnas
    ├── HomeController.java
    ├── dao
    │   └── MemberDAO.java
    ├── dto
    │   └── Member.java
    └── service
        └── MemberService.java
```

#### Client 폴더 구조

```
.
├── resources
│   └── mybatis
│       ├── mappers
│       │   └── member.xml
│       └── model
│           └── modelConfig.xml
└── webapp
    ├── WEB-INF
    │   ├── config
    │   │   └── jdbc
    │   ├── spring
    │   │   ├── appServlet
    │   │   └── root-context.xml
    │   ├── views
    │   │   └── home.jsp
    │   └── web.xml
    ├── css
    │   └── styles.css
    ├── html
    │   └── memberInsertForm.html
    ├── index.html
    ├── js
    │   ├── my.js
    │   └── scripts.js
    └── resources
```

상단의 resources는 데이터베이스 관련 설정 폴더와 Logger에 대한 설정이 있다.

## mybatis로 데이터베이스 정의하기

mybatis를 사용하기 위해 의존성 정의와 데이터베이스 정의를 위한 xml파일을 작성해 준다.

### 의존성 정의

mybatis를 이용해 데이터베이스를 제어하려면 mysql과 mybatis관련 의존성을 pom.xml에 정의해야 한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.hello</groupId>
    <artifactId>cafe</artifactId>
    <name>1018_cafe2</name>
    <packaging>war</packaging>
    <version>1.0.0-BUILD-SNAPSHOT</version>
    <!-- 이 부분 아래로 버전 설정 -->
    <properties>
        <java-version>1.8</java-version>
        <org.springframework-version>5.3.23</org.springframework-version>
        <org.aspectj-version>1.9.9.1</org.aspectj-version>
        <org.slf4j-version>1.7.36</org.slf4j-version>
    </properties>
    <dependencies>
        ...
        <!--spring jdbc, mysql connector, mybatis 관련 추가된 의존성 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.3.23</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.30</version>
        </dependency>

        <dependency>
            <groupId>commons-beanutils</groupId>
            <artifactId>commons-beanutils</artifactId>
            <version>1.8.0</version>
        </dependency>

        <dependency>
            <groupId>commons-dbcp</groupId>
            <artifactId>commons-dbcp</artifactId>
            <version>1.2.2</version>
        </dependency>

        <dependency>
            <groupId>cglib</groupId>
            <artifactId>cglib</artifactId>
            <version>2.2</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.1.0</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>1.1.0</version>
        </dependency>
    </dependencies>
    <!--의존성 정의 끝 -->

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-eclipse-plugin</artifactId>
                <version>2.9</version>
                <configuration>
                    <additionalProjectnatures>
                        <projectnature>org.springframework.ide.eclipse.core.springnature</projectnature>
                    </additionalProjectnatures>
                    <additionalBuildcommands>
                        <buildcommand>org.springframework.ide.eclipse.core.springbuilder</buildcommand>
                    </additionalBuildcommands>
                    <downloadSources>true</downloadSources>
                    <downloadJavadocs>true</downloadJavadocs>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.5.1</version>
                <configuration>
                    <source>1.6</source>
                    <target>1.6</target>
                    <compilerArgument>-Xlint:all</compilerArgument>
                    <showWarnings>true</showWarnings>
                    <showDeprecation>true</showDeprecation>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>1.2.1</version>
                <configuration>
                    <mainClass>org.test.int1.Main</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

spring-jdbc와 mysql-connector, mybatis 관련 의존성들이다. Intellij에서는 reload all maven project를 해주면 되고, Eclipse에서는 프로젝트 우클릭 후 Maven→update project를 해주면 된다.

의존성을 적용시켰으니 mybatis를 사용하기 위한 설정을 해보자

### context.xml에 데이터베이스 정의하기

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.3.xsd">

    <bean id="propertyPlaceholderConfigurer"
          class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer"
    >
        <property name="locations">
            <list>
                <value>/WEB-INF/config/jdbc/jdbc.properties</value>
            </list>
        </property>
    </bean>

    <bean id="dataSource" class="org.apache.ibatis.datasource.pooled.PooledDataSource">
        <property name="driver" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
		...
</beans>
```

spring의 bean factory를 사용해 연결할 데이터베이스 정보를 jdbc.properties에 정의한다.

```
jdbc.driverClassName=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:<port>/<schema name>
jdbc.username=username
jdbc.password=password
```

데이터베이스 정보를 위와 같이 `jdbc.properties` 파일에 추가한다. root-context.xml에 datasource를 EL태그로 지정해 준다. ${jdbc.~}형태로 사용한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.3.xsd">
		...
    <bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="classpath:mybatis/model/modelConfig.xml"/>
        <property name="mapperLocations" value="classpath:mybatis/mappers/*.xml"/>
    </bean>

    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactoryBean"/>
    </bean>

    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <tx:annotation-driven transaction-manager="txManager"/>
</beans>
```

sqlSessionFactory로 mybatis의 configuration과 mapper 경로를 정의한다. TransactionManager와 annotation을 사용한다는 annotation-driven을 정의해 준다.

## 회원가입하기

이제 사용자에게 회원가입을 할 수 있도록 해줄 것이다. 기본 템플릿 페이지에 회원가입 폼을 만들어 회원가입이 가능하게 만들어 본다.

### 사용자 등록 form 생성

```html
<form role="form">
  <div class="form-group">
    <label for="inputName">성명</label>
    <input
      type="text"
      class="form-control"
      name="name"
      id="name"
      placeholder="이름을 입력해 주세요"
    />
  </div>
  <div class="form-group">
    <label for="InputEmail">ID</label>
    <input
      type="text"
      class="form-control"
      name="id"
      id="id"
      placeholder="ID를 입력해주세요"
    />
  </div>
  <div class="form-group">
    <label for="inputPassword">비밀번호</label>
    <input
      type="password"
      class="form-control"
      name="pw"
      id="pw"
      placeholder="비밀번호를 입력해주세요"
    />
  </div>
  ...
  <div class="form-group text-center">
    <input
      type="button"
      id="memberInsertBtn"
      class="btn btn-primary"
      value="회원가입"
    />
    <button type="submit" class="btn btn-warning">
      가입취소<i class="fa fa-times spaceLeft"></i>
    </button>
  </div>
</form>
```

먼저 webapp/html에 memberInsertForm.html을 생성해 사용자 회원가입 폼을 만들어 준다. 회원가입 버튼을 눌러 요청을 하기 위해 스트립트로 동작을 하게 해준다.

```jsx
$(document).ready(() => {
  $("#memberInsertBtn").click(() => {
    let name = $("#name").val();
    let id = $("#id").val();
    let pw = $("#pw").val();

    $.post("../memberInsert.shop", { name, id, pw }, (data) => {
      alert(data);
      window.close();
    });
  });
});
```

버튼에 클릭 이벤트를 준다. Form에 name, id, pw 값을 가져와 controller로 post요청을 한다. post요청 경로는 ../memberInsert.shop으로 요청한다.

#### Controller

```java
@Controller
public class HomeController {
    @Autowired
    MemberService memberService;

    @RequestMapping(
            value = "memberInsert.shop",
            method = RequestMethod.POST,
            produces = "application/text; charset=utf8")
    @ResponseBody
    public String memberInsert(Member member) {
        System.out.println("home controller");
        memberService.memberInsert(member);
        return member.getName() + "님 환영합니다";
    }
}
```

클라이언트의 회원가입 요청이 발생하면 동작하는 controller이다. `@RequestMapping`으로 요청 경로, 프로토콜 타입, 인코딩을 정의해 준다. `@ResponseBody`를 사용하게 되면 반환 시 클라이언트로 body가 넘어가게 된다. MemberService를 `@Autowired`로 객체 의존성을 주입한다.

#### Service

```java
@Service
public class MemberService {

    @Autowired
    MemberDAO memberDAO;
    public void memberInsert(Member member) {
        memberDAO.memberInsert(member);
        System.out.println("MemberService" + member.getName());
    }
}
```

`@Service` 어노테이션으로 Service임을 명시하고 MemberDAO 객체 의존성을 주입한다. Service에서는 DAO에 memberInsert를 호출한다.

#### MemberDAO

```java
@Repository
public class MemberDAO {

    @Autowired
    SqlSession sqlSession;

    public void memberInsert(Member member) {
        // jdbc 6단계
        System.out.println("MemberDAO " + member.getName());
        System.out.println("member dao " + sqlSession);
        /*mapper의 namespace + id => mapper.member.memberInsert*/
        sqlSession.insert("mapper.member.memberInsert", member);
    }
}
```

DAO는 `@Repository` 어노테이션을 사용한다. DAO에서는 위 내용에서 mybatis로 정의된 SqlSession을 사용해 데이터베이스에 접근한다.

sqlSession으로 쿼리를 사용하려면 main/resources/mybatis/mappers에 정의된 xml파일에 접근해야 한다. insert()의 첫번째 인자는 mapper 요청 경로이고, 두 번째 인자는 mapper에서 지정한 데이터 타입이다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="mapper.member">
    <!--dto의 클래스 - modelConfig.xml에서 정의한 것 -->
    <insert id="memberInsert" parameterType="member">
        insert into member(name, id, pw)
        values (#{name}, #{id}, #{pw})
    </insert>
</mapper>
```

mapper의 namespace를 mapper.member로 지정한다. insert 쿼리를 사용하기 위해 <insert>태그를 사용한다. insert 태그의 id는 DAO에서 mapper요청 경로인 memberInsert로 지정한다. parameterType은 member객체로 설정한다.

sql쿼리에서 입력될 값은 #{}을 사용하며, 변수명은 DTO에 정의된 변수명과 같아야 한다.

## 사용자 로그인 하기

회원가입한 사용자에게 로그인을 하게 해주어야 한다. 로그인 요청을 구현해 본다.

```html
<div id="msgDiv">
  ID<input size="5" id="loginId" /> PW<input
    size="5"
    id="loginPw"
    type="password"
  />
  <input type="button" id="loginBtn" value="login" />
</div>
```

로그인을 위한 필드를 만들어 준다. ID와 PW를 입력 받아 login 버튼으로 로그인을 시도한다.

```jsx
$(document).ready(() => {
    $("#loginBtn").click(() => {
        let id = $("#loginId").val();
        let pw = $("#loginPw").val();

        $.post("login.shop", {id, pw}, (data) => {
            let obj = JSON.parse(data);
            if (obj.name) {
                data = obj.name + "<input type='button' value='logout' id='logoutBtn' class='btn btn-primary'>";
                $.cookie("logined", data);
                $("#msgDiv").html(data);
            } else {
                alert(obj.msg);
                location.reload();
            }
        })
    })
	...
})
```

loginBtn에 클릭 이벤트를 연결한다. login 필드에 사용되는id와 pw를 가져와 `login.shop`으로 post요청을 한다. 로그인 성공 시 사용자의 이름이 data에 반환되고 logout 버튼으로 로그인 필드를 변경해 준다. post 요청 후 동작을 알아본다.

#### Controller

```java
@Controller
public class HomeController {
    @Autowired
    MemberService memberService;

    @RequestMapping(value = "login.shop",
            method = {RequestMethod.POST},
            produces = "application/text; charset=utf8")
    @ResponseBody
    public String login(HttpServletRequest request) {
        String id = request.getParameter("id");
        String pw = request.getParameter("pw");

        JsonObject json = new JsonObject();

        try {
            Member member = new Member(id, pw);
            String name = memberService.login(member);
            if (name != null) {
                HttpSession session = request.getSession();
                session.setAttribute("member", member);

                json.addProperty("name", name);
            } else {
                json.addProperty("msg", "로그인 실패");
            }
        } catch (Exception e) {
            json.addProperty("msg", e.getMessage());
        }
        return json.toString();
    }
}
```

Controller에 login 메서드를 추가한다. RequestMapping 경로는 `login.shop`, post 방식, 한글 깨짐 방지를 위한 인코딩을 설정해 주고, ResponseBody 어노테이션을 추가한다.

로그인 요청은 request로 id와 pw를 받아오게된다. 클라이언트에 넘겨주기 위해 gson의 JsonObject를 사용했다. Member dto에 id와pw를 받는 생성자를 만들어 주었기 때문에 id와 pw으로만 Member 객체를 생성할 수 있다. 새로 생성된 Member 객체를 MemberService의 login 메서드로 넘겨준다.

#### Service

```java
@Service
public class MemberService {

    @Autowired
    MemberDAO memberDAO;

    public String login(Member member) {
        return memberDAO.login(member);
    }
}
```

Service는 간단하게 DAO의 login을 호출한다.

#### MemberDAO

```java
@Repository
public class MemberDAO {

    @Autowired
    SqlSession sqlSession;

    public String login(Member member) {
        return sqlSession.selectOne("mapper.member.login", member);
    }
}
```

DAO에서 sqlSession의 selectOne으로 로그인 요청을 한다. 요청 경로는 mapper.member.login이고 id와 pw가 포함되어 있는 member를 넘겨준다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="mapper.member">
    <!--dto의 클래스 - modelConfig.xml에서 정의한 것 -->
    <insert id="memberInsert" parameterType="member">
        insert into member(name, id, pw)
        values (#{name}, #{id}, #{pw})
    </insert>

    <select id="login" resultType="String" parameterType="member">
        <![CDATA[
        select name
        from member
        where id = #{id}
          and pw = #{pw}
        ]]>
    </select>
</mapper>
```

sqlSession의 selectOne으로 요청하면 member.xml의 id가 login인 select태그로 들어온다. 쿼리문의 반환값인 resultType은 String이며, DAO에서 넘어온 parameterType은 member객체이다.

쿼리문의 CDATA는 문자열 비교연산이나 부등호를 구분하기 위해 사용한다. CDATA 태그 안에서는 모두 문자열로 처리하게 된다.

where 조건에 id와 pw를 #{}로 값을 받아온다. 변수명은 회원가입 예시와 동일하게 Member dto의 변수명과 같아야 한다.
