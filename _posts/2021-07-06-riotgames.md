---
title: Riot games coding webinar
date: 2021-07-06
categories: [교육]
tags: [webinar, git, code review]
mermaid: true
---

> 2021.07.06 라이엇 게임즈 코딩 웨비나 내용을 정리하였습니다.

# Git과 Code Review

- 버전 관리 시스템의 필요성
- git 과 github
- 개념과 기본 이해

# Code Review

- 코드리뷰와 깃허브
- git flow와 github flow
- git pages

---

## 버전 관리 시스템의 필요성

- 작업을 되돌리고 싶다
- 다수의 인원이 협업을 할때 파일 수정 시 충돌이 잦다

## RCS

- 하나의 프로젝트를 협업할 때 RCS를 이용하기 어렵다 (로컬에서 버전 관리)

## CVCS

- 중앙 서버 사용
- 중앙 서버 날라가면 코드 다날라감

## DVCS

- 서버와 로컬 컴퓨터가 동일한 프로젝트 가지고 있음
- 프로젝트 복사 저장

# 정리

![/assets/img/posts/riotWebinar/pic1.png](/assets/img/posts/riotWebinar/pic1.png)

---

# Git

- 리누스 토발즈가 리눅스 커널을 관리하기 위해 만든 도구
- 2주만에 만들었다

### 특징

- 빠르고 효율적
- 단순한 구조
- 완벽한 분산
- 동시에 여러개의 기능 개발
- 리눅스 같은 대형 프로젝트에도 유용
- 배우기 쉽다(?)

# Github

- git을 웹으로 호스팅 해주는 웹서비스
- 웹에서 git 기반의 소스 이력관리를 직접 하거나 모니터링 가능
- 웹 환경에서 여러 사람들과 같이 협업 가능
- automation CD & CI 가능
- 수많은 오픈 소스가 등록되어 있다
- 가능하다면 평소에 github를 관리하고 이력서에 github 주소를 등록하기를 권장

# Git 이해하기

![/assets/img/posts/riotWebinar/pic2.png](/assets/img/posts/riotWebinar/pic2.png)

1. 워킹 디렉토리에서 파일을 수정한다
2. Staging Area에 파일을 추가한다
3. Staging Area에 있는 파일을 커밋해서 git 디렉토리에 영구적인 스냅샷을 만든다

---

# Git config

```
git config --global user.name "name"
git config --global user.email@ex.com
```

- 사용자 정의하는 코드이다

# .git

- `git init`을 햐면 생성되는 폴더
- git으로 관리하기 위한 모든 정보를 포함하고 있다

# Untracked files

- 생성한 깃 폴더에 추가된 파일
- git이 현재 관리하고 있지 않은 파일을 의미

# git commit

```
git add .
git commit -m "init"
git commit -am "init"
```

- 해당 명령어를 통해 깃 디렉토리에 파일을 등록할 수 있음
- 등록한 파일은 삭제하더라도 복구 가능
- 메세지와 함께 커밋할 수 있다

### commit message 규칙

- 제목과 본문을 한 줄 띄워 분리
- 제목은 영문 기준 50자
- 제목 끝에 .금지
- 제목이나 본문에 이슈 번호 추가
- 제목은 명령조로
- 본문은 영문 기준 72자
- 본문은 어떻게보다는 무엇을, 왜에 맞춰 작성

# Git branch

- git 장점 중 하나
- 작업의 단위
- 맨 처음에 git에 커밋하면 main 브랜치가 자동으로 생성됨(master →main)

# Branch 생성하기

```
git branch hotfix
git checkout hotfix

# branch를 생성과 동시에 checkout
git checkout -b hotfix
git branch
git branch -r
git branch -a
```

![/assets/img/posts/riotWebinar/pic3.png](/assets/img/posts/riotWebinar/pic3.png)

# Git remote

- git에는 remote 환경이 존재
- 협업을 위해 각 로컬 환경에서 공유하고 있는 외부 기준점 필요
- 인터넷 혹은 외부 네트워크에서 호스팅 되는 깃 레포지토리를 remote라고 한다

```
git clone "url"
git push

# remote에서 가져옴
git pull
git fetch
```

---

# Code Review

- 개발자들끼리 서로의 코드를 리뷰해주는 행위
- 코드를 보고 수정할 점 에러 등 이야기 해줌
- 배포전 기능에 대해 서로 교차 검증이 가능하다
- 팀원 수준의 비슷한 레벨로 맞출 수 있다

# Open Source

- 소스코드 공개해 누구나 특별 제한 없이 코드를 보고 수정 사용할 수 있는 코드
- github에 공개
- 누구나 소스를 확인할 수 있고 수정 요청/기능추가를 할 수 있다

# Branch strategy

- 일반적으로 회사에서는 작업 단위로 branch 를 생성후 작업한다
- 어떻게 브랜치를 생성하는지에 대한 다양한 전략이 존재
  - git flow
  - github flow

# 좋은 개발자가 되기위한 습관

- 배우는것
- 정리하는것
- 공유하는것
- 꾸준히 공부하는 습관
