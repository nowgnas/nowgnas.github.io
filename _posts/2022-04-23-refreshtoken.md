---
title: Refresh Token이란??
description: access token과 refresh token을 사용해보자
date: 2022-04-23 02:51 +0900
lastmod: 2022-04-23 02:51 +0900
categories: [web]
tags: [web, token, refresh token, access token]
mermaid: true
---

# Access Token

로그인이 필요한 모든 웹 페이지에서는 사용자가 인증되었는지 확인해야 한다. 일반적으로 access token은 JWT(jsonwebtoken)을 많이 사용하고, 헤더의 authorization에 저장해서 페이지마다 token을 검증하게 된다. Token 탈취의 위험이 있어 짧은 시간의 유효 기간을 갖는다. 유효 기간은 30분에서 1시간 정도다.

JWT는 json 형태를 사용해서 사용자에 대한 속성을 저장하는 `Claim`기반의 웹 토큰이다. 구조는 `header`, `payload`, `verify signature`로 이뤄진다. `header`는 token의 타입이나 사용될 암호화 알고리즘을 정한다. `payload`는 저장하려고 하는 데이터가 담겨져 있다. `signature`는 `header`에서 정의된 알고리즘과 개발자가 지정한 `secret key`를 이용해 생성한다.

# Refresh token

![ppt 이미지 저장](/assets/posting/backend/refreshtoken/refresh4.png)

ppt 이미지 저장

Refresh token은 새로운 access token을 재발급 받을 수 있는 유효 기간이 긴 token이다. Access token과 함께 사용된다.

Access token과 Refresh token의 동작은 위 그림과 같다.

1. 사용자가 로그인을 시도한다.
2. 서버는 DB에 사용자가 저장되어 있는지 확인한다.
3. 사용자가 존재하면 access token과 refresh token을 발급한다.
4. 발급된 refresh token은 사용자의 id와 `Token` 테이블에 저장된다.
5. 서버는 client에 access token과 refresh token을 응답한다.
6. 사용자가 서버에 요청을 하면, 서버는 access token을 검증 후 요청한 데이터를 응답한다.
7. Access Token Expired
8. access token이 만료된 후 사용자가 서버에 요청을 하게되면, access token의 만료됨을 응답한다.
9. access token의 만료를 확인한 client는 서버에 access token의 refresh를 요청한다.
10. 서버는 만료된 access token에서 사용자의 id를 얻은 후 id로 DB에 refresh token이 유효한지 확인한다.
11. refresh token이 유효하다면 access token을 재발급, client에 전송하고, refresh token도 만료되었다면 새로 로그인하도록 한다.

Refresh token은 access token보다 긴 2주 정도의 유효 기간을 가지게 한다. refresh token이 만료되기 전에 사용자가 사이트를 방문하게 되면 로그인 없이 바로 서비스를 이용할 수 있다.

현재 진행중인 프로젝트에서는 메인 페이지에서 이메일을 입력 후 refresh token이 유효하면 바로 서비스를 사용하게 되고, 유효하지 않으면 비밀번호를 입력해 로그인을 진행하게 한다.
