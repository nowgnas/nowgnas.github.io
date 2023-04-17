---
title: JDK 설치와 Sprint Tool Suite 시작하기
categories: java
date: 2022-07-16 16:13 +0900
description: m1에서 JDK 설치와 STS 3에서 프로젝트 실행하기
lastmod: 2022-07-16 16:13 +0900
tags: eclipse java spring
---

## STS 3 설치하기

sprint tool suite는 Eclipse에서 spring 프로젝트를 동작할 수 있는 툴 같다. 현재 spring tool suite 4 버전까지 나와 있으며, 4버전은 spring 프로젝트만 가능하다고 한다.

이번 프로젝트에서는 3버전이 필요하다. 3버전은 [spring.io](https://spring.io/tools) 아래에 `Sprint Tool Suite 3 wiki`에서 확인할 수 있다.

![스크린샷 2022-07-16 오후 3.33.16.png](/assets/posting/ssafy/startcamp/pic1.png)

macos를 사용하고 있으므로 `.dmg`확장자를 설치해준다.

![스크린샷 2022-07-16 오후 3.38.27.png](/assets/posting/ssafy/startcamp/pic2.png)

`.dmg`파일을 더블클릭해서 3개의 파일을 모두 Applications에 넣어준다. 넣어준 후 새 폴더를 생성해서 3개의 파일을 모두 넣어준다.

![스크린샷 2022-07-16 오후 3.36.53.png](/assets/posting/ssafy/startcamp/pic3.png)

STS를 설치하기 이전에는 zulu jdk를 m1용으로 설치했었다. m1용 JDK로 STS를 열려고하면 오류가 난다. STS 3 버전은 `x86 64-bit` architecture가 필요하다고 해서 zulu jdk 페이지에서 새로 받아주었다.

## JDK 환경변수 설정하기

jdk를 설치하고 환경변수를 잡아줘야 한다. 설정파일에 환경변수를 잡아줘야 하는데 인텔 macos는 .bash_profile, m1은 .zshrc일 것이다. vi나 nano를 사용해도 좋고 vscode에서 편하게 열어서 설정하면 된다.

```bash
export JAVA_HOME=/Library/Java/JavaVirtualMachines/{jdk version}/Contents/Home
export PATH=$JAVA_HOME/bin:$PATH
```

설정파일을 열어 위 두 줄을 넣어준 후 `source ~/.zshrc` 명령을 터미널에 타이핑해 주면 터미널에 적용된다.

![스크린샷 2022-07-16 오후 3.46.42.png](/assets/posting/ssafy/startcamp/pic4.png)

터미널에서 `java -version` 명령으로 설치된 jdk의 버전을 확인할 수 있다. 위와 유사하게 나온다면 정상적으로 환경변수가 잡힌 것이다.

## STS에서 프로젝트 실행해 보기

![스크린샷 2022-07-16 오후 3.50.51.png](/assets/posting/ssafy/startcamp/pic5.png)

STS를 실행 시키고 상단 바에서 `FIle→Open Project from File System`을 클릭하면 위와 같은 창이 뜨게 된다. `Archive`를 눌러 원하는 프로젝트 폴더나 zip파일을 선택하면 된다.

현재 프로젝트는 MYSQL 연결이 되어 있는 프로젝트다. 도커로 MYSQL 컨테이너를 실행하고 데이터베이스와 테이블을 생성한 후 STS 프로젝트를 실행시켰다.

![스크린샷 2022-07-16 오후 4.03.31.png](/assets/posting/ssafy/startcamp/pic6.png)

Package Explorer에서 프로젝트 우클릭한 후 `Run As → Sprint Boot App`으로 실행시키면 STS 터미널에 아래와 같은 로그가 뜬다.

![스크린샷 2022-07-16 오후 4.07.27.png](/assets/posting/ssafy/startcamp/pic7.png)

JDK 설치와 STS 프로젝트 실행에 대해 알아보았다.
