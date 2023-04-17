---
title: Jekyll Blog 테마 적용하기(Chirpy theme)
date: 2021-07-05
categories: [blog]
tags: [jekyll, blog]
mermaid: true
---

# jekyll blog Chirpy 테마 적용하기

이전에 사용하던 테마는 커스텀 하기 어려워 테마를 찾아보다가 Chirpy 테마를 찾았다

다른 테마와는 다르게 gh-pages를 사용한다.

처음 적용하려고 했을때 계속 오류가 발생해 애를 먹었다

테마 변경하는 순서를 정리해 보려고 한다

## Repository 생성

![/assets/img/posts/startblog/2021_07_05_20_19_13.png](/assets/img/posts/startblog/2021_07_05_20_19_13.png)

- username.github.io로 repository를 생성한다.

## git repository 연결하기

![/assets/img/posts/startblog/2021_07_05_20_23_35.png](/assets/img/posts/startblog/2021_07_05_20_23_35.png)

```
git remote add origin <repository url>
```

- 원하는 위치에 생성한 repository를 연결해준다.

## 테마 다운로드 및 초기화

![/assets/img/posts/startblog/2021_07_05_20_10_15.png](/assets/img/posts/startblog/2021_07_05_20_10_15.png)

- Download ZIP을 이용해서 테마를 다운로드 받아준 후 위에서 repository를 연결한 곳에 풀어준다.

![/assets/img/posts/startblog/2021_07_05_20_27_36.png](/assets/img/posts/startblog/2021_07_05_20_27_36.png)

```
bundle
```

- bundle 명령을 사용하여 필요한 종속성들을 설치해 준다

![/assets/img/posts/startblog/2021_07_05_20_31_11.png](/assets/img/posts/startblog/2021_07_05_20_31_11.png)

```
bash tools/init.sh
```

- init 명령으로 초기화 해 준다
- init 명령은 \_post 폴더에 있는 post들과 .travis.yml, docs 폴더를 삭제해 준다.

## \_config.yml 설정하기

![/assets/img/posts/startblog/2021_07_05_20_39_44.png](/assets/img/posts/startblog/2021_07_05_20_39_44.png)

- 배포하기 전 \_config.yml 파일에 url부분을 변경해줘야 한다.
- "https://<username>.github.io "으로 설정해 주면 된다.

## 배포하기

tools 폴더에 `deploy.sh` 파일이 있어 실행해 보았지만 직접 실행 시키는 것이 아니었다. 그냥 repository에 push만 해주면 알아서 해준다.

![/assets/img/posts/startblog/2021_07_05_20_33_24.png](/assets/img/posts/startblog/2021_07_05_20_33_24.png)

```
bash tools/run.sh
```

- run 명령은 로컬에서 블로그를 컴파일해 준다.

```
git add .
git commit -m "initial commit"
git push -u origin master
```

- master branch에 push해주면 끝이다.

![/assets/img/posts/startblog/2021_07_05_20_44_36.png](/assets/img/posts/startblog/2021_07_05_20_44_36.png)

- 몇분 기다리고나면 gh-pages branch가 생성된다
- settings → Github Pages 에서 gh-pages로 선택해줘야 블로그 주소로 들어갔을 때 정상적으로 보인다.
