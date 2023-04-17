---
title: MYSQL event scheduler로 데이터 조작하기
categories: database
date: 2022-12-29 12:55 +0900
description: mysql에서 event scheduler를 사용해보자
lastmod: 2022-12-29 12:55 +0900
tags: mysql
---

# Event Scheduler

mysql의 event scheduler는 주기적으로 데이터베이스에 작업을 해야 할 경우 사용한다. 지속적으로 쌓이는 temporary 데이터가 있을 때 해당 테이블을 자주 비워줌으로써 용량 차지가 되지 않게끔 해야한다.

매번 개발자가 테이블을 비울 필요 없이 데이터베이스 자체에 어떤 이벤트를 걸어주고 주기적으로 반복되게끔 할 수 있는 것이 event scheduler이다.

## 회원 테이블 생성하기

```sql
create table member
(
    id    varchar(20),
    name  varchar(20),
    email varchar(20)
);
```

Event Scheduler 를 테스트하기 위해 간단한 회원 테이블을 생성한다

## Event Scheduler 사용 확인하기

![스크린샷 2022-12-29 오후 1.10.36.png](/assets/posting/database/event_scheduler/pic1.png)

event scheduler 사용여부를 확인한다. Value가 ON으로 되어 있다.

## 1분마다 데이터가 등록되는 Event Scheduler

```sql
create event if not exists insert_event_minute
    on schedule
        every 1 minute
    on completion not preserve
    enable
    comment 'insert member'
    DO
    insert into member (id, name, email)
    VALUES ('1', 'name', 'email');
```

![스크린샷 2022-12-29 오후 2.10.31.png](/assets/posting/database/event_scheduler/pic2.png)

1분마다 동일한 회원 정보를 추가하였다.

![스크린샷 2022-12-29 오후 2.11.29.png](/assets/posting/database/event_scheduler/pic3.png)

이벤트 스키마의 테이블이다. 3번째 이벤트가 1분마다 실행되는 이벤트의 데이터이다. 시작 시간은 05:07이고 마지막으로 실행된 시간이 05:10이다. 1분 마다 정상적으로 실행 중인것을 확인할 수 있다.

## 현재 시각으로부터 5분 후 모든 데이터를 삭제하는 Event Scheduler

```sql
create event if not exists del_data
    on schedule
        AT DATE_ADD(now(), interval 5 minute)
    on completion not preserve
    enable
    comment 'delete table'
    do
    truncate member;
```

![스크린샷 2022-12-29 오후 2.21.52.png](/assets/posting/database/event_scheduler/pic4.png)

현재 시각으로부터 5분 후 member의 데이터를 모두 삭제하는 쿼리이다. 정확한 5분을 위해 `DATE_ADD()`를 사용해 `now()`로부터 5분 후에 삭제하도록 구현하였다. member의 모든 데이터가 삭제된 것을 확인할 수 있다. `completion not preserve` 옵션으로 이벤트 수행 후 이벤트를 삭제하도록 했다.

## 특정 기간 동안만 반복 실행되는 Event Scheduler

```sql
create event if not exists insert_event_term
    on schedule
        every 1 minute starts now() ends DATE_ADD(now(), interval 5 minute)
    on completion preserve
    enable
    comment 'insert member'
    DO
    insert into member (id, name, email)
    VALUES ('1', 'name', 'email');
```

![스크린샷 2022-12-29 오후 2.28.52.png](/assets/posting/database/event_scheduler/pic5.png)

이벤트 정보 테이블이다. 4번째 이벤트가 실행 중이고, 특정 기간 동안만 반복하는 이벤트이다. 5분 동안 1분마다 회원 정보를 추가하는 쿼리로 실행하였다. 05:27에 시작하여 05:32에 종료되는 이벤트이다.

![스크린샷 2022-12-29 오후 2.32.53.png](/assets/posting/database/event_scheduler/pic6.png)

5분이 지난 후 이벤트가 DISABLED 상태로 되었다.

![스크린샷 2022-12-29 오후 2.33.26.png](/assets/posting/database/event_scheduler/pic7.png)

데이터 또한 정상적으로 추가된 것을 확인할 수 있다.

# 이벤트 테이블 정보

![스크린샷 2022-12-29 오후 2.36.45.png](/assets/posting/database/event_scheduler/pic8.png)

위에서 실행시켰던 이벤트들의 정보이다.
