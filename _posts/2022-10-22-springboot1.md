---
title: spring boot와 react를 사용해 웹 서비스 개발하기
categories: spring-boot
date: 2022-10-22 13:00 +0900
description: spring boot와 react를 사용한 웹 서비스 개발
lastmod: 2022-10-22 13:00 +0900
tags: java react spring-boot swagger web
---

Spring boot 와 react를 사용하여 웹 서비스를 개발해 보려고 한다. spring boot에 관견된 내용은 스프링 부트 핵심 가이드 책을 참고하였다.

## 기술 스택

|    name     | version |        |
| :---------: | :-----: | ------ |
|    Yarn     | 1.22.18 |        |
| Spring Boot |  2.5.6  |        |
|    React    | 18.2.0  |        |
|    MYSQL    |   8.x   |        |
|  Intellij   |         | server |
|   Vscode    |         | client |

이번 포스팅에서는 client와 server의 간단한 연결을 알아본다.

## server 구성하기

![스크린샷 2022-10-22 오후 1.15.15.png](/assets/posting/spring/springboot1/pic1.png)

server는 spring boot로 개발한다. spring initializer로 프로젝트를 만들어 준다. java version은 11, 빌드 타입은 maven, packaging은 War을 선택해 줬다.

![스크린샷 2022-10-22 오후 1.31.22.png](/assets/posting/spring/springboot1/pic2.png)

의존성은 책 내용에 따라 3가지를 선택해 프로젝트를 생성해 준다.

```xml
<!--pom.xml-->

<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.6</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

pom.xml에 parent 태그에 spring boot 버전을 2.5.6으로 변경한 후 maven을 업데이트해 준다. 서버의 기본 프로젝트를 만들어 봤다. 이제 client를 만들어보자

## client 구성하기

client는 react를 사용하기 때문에 폴더 위치는 어디에 둬도 문제가 없다. Jsp를 사용한다면 resources에 넣어두지만, react는 root 폴더에서 생성한다.

### create react app

```bash
yarn create react-app client
```

root 폴더에서 터미널에 위 명령을 실행해 준다.

### 필요한 패키지 설치

```bash
yarn add react-router-dom@6
yarn add axios
```

Routing을 위한 react-router-dom과 sever로 요청을 위한 axios를 설치해 준다. 이제 client에서 server로 요청 테스트를 해보자

## client에서 server로 요청 보내기

### CORS error

spring boot의 포트는 8080이고 react의 포트는 3000이다. client에서 server로 요청을 할 경우 도메인이 다르면 CORS 에러가 발생하게 된다. CORS 에러를 방지하기 위한 설정을 해준다.

#### client 설정

```jsx
// api.js

import axios from "axios";

const backendPortNumber = "8080";
const serverUrl =
  "http://" + window.location.hostname + ":" + backendPortNumber + "/api/v1/";

async function get(endpoint, params = "") {
  console.log(
    `%cGET 요청 ${serverUrl + endpoint + "/" + params}`,
    "color: #a25cd1;"
  );

  return axios.get(serverUrl + endpoint + "/" + params, {
    // JWT 토큰을 헤더에 담아 백엔드 서버에 보냄.
    headers: {
      Authorization: `Bearer ${sessionStorage.getItem("userToken")}`,
    },
  });
}
```

client에서는 axios를 사용해 server 포트인 8080으로 요청하는 api.js를 만들어 줬다. api.js를 import해서 get요청을 하게 되면 요청 url을 변경해 줄 수 있다.

#### server 설정

```java
// MemberController.java

@RestController
@RequestMapping("/api/v1/user")
@CrossOrigin(origins = "*", allowedHeaders = "*")
public class MemberController {

    @GetMapping("/login")
    public String login() {
        System.out.println("hello spring");
        return "index";
    }
}
```

spring boot에서는 client에서 8080번 포트로 보내는 요청에 대해 허용을 해줘야 한다. controller에 @CrossOrigin 으로 모든 경로에 대해 허용을 해준다. 위 코드처럼 controller 전체에 정의해 줘도 되고 각 메소드 별로 설정해 줘도 된다.

### client에서 요청 하기

```jsx
// index.js

import React from "react";
import ReactDOM from "react-dom";
import App from "./App";
import { BrowserRouter } from "react-router-dom";

const rootElement = document.getElementById("root");
ReactDOM.render(
  <BrowserRouter>
    <App />
  </BrowserRouter>,
  rootElement
);
```

index.js를 위 코드와 같이 구현해 준다.

```jsx
// App.js

import { Route, Routes } from "react-router-dom";
import About from "./pages/About";
import Home from "./pages/Home";

const App = () => {
  return (
    <Routes>
      <Route path="/" element={<Home />}></Route>
      <Route path="/about" element={<About />}></Route>
    </Routes>
  );
};
export default App;
```

App.js에 페이지에 대한 routing 경로를 지정해 준다.

```jsx
// Home.js

import Menu from "./Menu";

function Home() {
  return (
    <>
      <Menu></Menu>
      <div>
        <h1>hello</h1>
      </div>
    </>
  );
}
export default Home;
```

```jsx
// Menu.js

import { Link } from "react-router-dom";
import * as API from "../api";
export default function Menu() {
  const getAction = async () => {
    try {
      const res = await API.get("user/login");
      alert("hello");
    } catch (error) {}
  };
  return (
    <div>
      <nav>
        <Link to="/">Home</Link>
        <Link onClick={getAction} to="/about">
          About
        </Link>
      </nav>
    </div>
  );
}
```

Home.js에 nav 버튼을 가지고 있는 Menu를 넣어주었다. Menu.js에 Link 태그로 넘어갈 라우팅 경로와 onClick을 설정해 주었다. About 버튼을 누르면 getAction 함수가 동작하며 `user/login` 경로로 GET 요청을 보낸 후 요청이 끝나면 hello 메세지를 가지는 alert 창을 띄우게 된다. 위의 api.js에서 변경해준 경로로 요쳥하게 되면 http://localhost:8080/api/v1/user/login/으로 요청하게 된다.

### server에서 요청 받기

```java
// MemberController.java

@Api(tags = {"user"})
@RestController
@RequestMapping("/api/v1/user")
@CrossOrigin(origins = "*", allowedHeaders = "*")
public class MemberController {

    @GetMapping("/login")
    public String login() {
        System.out.println("hello spring");
        return "index";
    }
}
```

server에서 controller 전체 url 을 /api/v1/user로 설정해 주었고 login메서드를 GET방식의 /login 경로로 설정했다. client에서 요청한 url에 따라 spring boot 터미널에서는 hello spring이 출력된다. 위에서 설명했던 CORS 설정을 해 주었기 때문에 server로 정상적인 요청이 가능한 것이다.

#### 결과 확인하기

![스크린샷 2022-10-22 오후 2.32.27.png](/assets/posting/spring/springboot1/pic2_1.png)

About 버튼을 누르게되면 server 터미널에 hello spring이 찍히고 화면에서는 hello 메세지를 가진 alert창이 뜬다.

### swagger api 문서 생성

위 MemberController.java에서 @Api 어노테이션이 있다. Swagger api 문서를 자동으로 생성하기 위한 것이다. swagger api를 설정하는 방법을 알아본다.

#### swagger 사용을 위한 의존성

```xml
<!-- pom.xml -->

<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>3.0.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>3.0.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```

swagger 3을 사용하기 위한 의존성이다. pom.xml에 추가해 준 후 maven을 업데이트해준다.

#### swagger config

```java
// SwaggerConfiguration.java

@Configuration
@EnableSwagger2
public class SwaggerConfiguration {
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                .apis(RequestHandlerSelectors.any())
                .paths(PathSelectors.any())
                .build();
    }
}
```

`SwaggerConfiguration` 파일에 위와 같이 swagger를 정의해준다.

```java
@SpringBootApplication
@EnableSwagger2
public class TechblogApplication {

    public static void main(String[] args) {
        SpringApplication.run(TechblogApplication.class, args);
    }
}
```

spring application 파일에서 @EnableSwagger2 어노테이션을 추가해 준다. 이제 /swagger-ui/로 접속하면 api 문서를 확인할 수 있다.

![스크린샷 2022-10-22 오후 2.30.49.png](/assets/posting/spring/springboot1/pic3.png)

경로와 응답 타입 등 따로 설정하지 않아도 어노테이션만 붙여주면 잘 나오게 된다.

client에서 기본적인 요청과 api 문서까지 확인해 보았다.
