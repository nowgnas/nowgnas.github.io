---
title: 간단하게 jekyll 블로그 빌드하기
date: 2021-12-02 12:38 +0900
lastmod: 2021-12-02 12:38 +0900
categories: [blog]
tags: [jekyll, blog]
mermaid: true
math: true
---

이 포스트는 jekyll 블로그를 처음 시작하거나 개발환경이 변경되어 jekyll 블로그를 빌드할 환경을 다시 만들어야 하는 경우 빠르게 환경을 구성할 수 있도록 적었습니다.  
문서 편집기는 visual studio code를 사용하였습니다.

## jekyll 시작하기

jekyll을 시작하기에 앞서 본인이 좋아하는 jekyll 테마를 하나 선택합니다.

WSL2 환경을 기준으로 작성되었으며, 윈도우 사용자들은 WSL2를 사용하는것을 권장합니다.

### ruby 설치

ruby와 ruby-dev kit을 함께 설치해 줍니다.

```terminal
sudo apt-get install ruby-full build-essential zlib1g-dev
```

### jekyll bundler 설치

ruby 설치 후 jekyll과 bundler를 설치해 줍니다.

```terminal
sudo gem install jekyll bundler
```

### gem과 bundle update

다양한 의존성과 패키지를 설치할 수 있는 gem과 bundle을 업데이트 시켜줍니다.

```terminal
# gem update
sudo gem update
```

```terminal
# bundle update
bundle update
```

### jekyll 페이지 localhost에서 테스트하기

이제 아래 명령어로 localhost에서 서버를 열어 블로그를 테스트할 수 있습니다.

```terminal
bundle exec jekyll serve
```
