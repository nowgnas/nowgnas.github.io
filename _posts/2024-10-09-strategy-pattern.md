---
title: "[디자인 패턴] 전략 패턴을 서비스에 적용하기"
description: "비 로그인, 로그인 고객을 위한 기능 설계에 전략 패턴 적용하기"
date: 2024-10-09 15:55 +0900
lastmod: 2024-10-09 15:55 +0900
categories:
  - design pattern
tags:
  - strategy pattern
---

이번 글에서는 디자인 패턴 중 하나인 전략 패턴에 대해 알아본다.

## 전략 패턴

전략 패턴은 객체 지향 설계 패턴 중 하나로, 여러 알고리즘을 서로 교체할 수 있도록 설계하는 방법이다. 런타임 시점에 동적으로 알고리즘을 선택할 수 있는 특징을 가지고 있다.

전략 패턴을 사용하는 주요 목적은 행위의 변화를 클라이언트 코드에 영향을 미치지 않고도 유연하게 변경할 수 있도록 만드는 것이다. 기본 구조는 context, strategy, strategy 구현체로 이뤄진다. 전략 패턴의 예시로 보통 결제에 대해 많이 소개되지만, 이번 글에서는 로그인, 비 로그인 사용자의 최근 본 상품 기능을 예시로 소개한다.

## 로그인 여부에 따른 최근 본 상품 기능 설계하기

서비스에서 비 로그인 사용자에게도 최근 본 상품을 제공하려고 한다. 최근 본 상품 기능에서 로그인 사용자와 비 로그인 사용자의 차이는 회원 식별값이 다르고 테이블의 라이프사이클이 다르다. 따라서 전략 패턴을 사용해서 구분되어야 하는 로직은 사용자 식별값과 접근하는 테이블이다.

![strategy](/assets/img/posts/strategy/strategy-img1.png)
전략 패턴을 적용한 UML 클래스 다이어그램은 위와 같다. Context에서 로그인 여부에 따른 전략을 선택하고 상황에 맞는 로직을 실행하게 된다.

### RecentProductsService

```java
public List<Object> getRecentProducts(HttpServletRequest request, RequestModel requestModel) {
  if (isLogin()) {
    requestModel.setMemberNo(getLoginMbNo());
    recentProductsStrategyContext.setRecentViewedProductsStrategy(loginRecentProductStrategy);
  } else {
    requestModel.setMemberNo(getNonLoginMemberNo(request));
    recentProductsStrategyContext.setRecentViewedProductsStrategy(nonLoginRecentProductStrategy);
  }
  return recentProductsStrategyContext.getRecentProducts();
}
```

Controller에서 호출하는 service단의 코드를 살펴보면, 로그인 여부에 따라 회원 번호를 세팅해준다. 로그인 된 상태라면 세션에서 회원 번호를 가져오고 비 로그인 상태라면 request의 쿠키값에서 비 로그인 고객 식별값을 세팅한다.
로그인 여부에 따라 context에 어떤 구현체를 사용할지 결정하게 된다.

### RecentProductsStrategyContext

```java
@Setter
public class RecentProductsStrategyContext {
  private RecentViewedProductsStrategy recentViewedProductsStrategy;

  public List<Object> getRecentProducts() {
    return recentViewedProductsStrategy.getRecentProducts();
  }
  ...
}
```

```java
public interface RecentViewedProductsStrategy {

    List<Object> getRecentProducts();
}
```

전략 패턴의 context 클래스는 strategy 객체를 참조하고 있다. Context 클래스는 전략의 구현체에 대해서는 고려하지 않는다. Setter를 통해 결정된 strategy의 구현체의 메서드를 호출하기만 하면 된다. Context 클래스는 코드 수정 없이 구현체 기능을 확장할 수 있는 개방-폐쇄 원칙을 따른다.

이렇게 전략 패턴을 사용하면 런타임에서 구현체를 결정하게 되어 유연한 시스템을 구성할 수 있고, 각 전략에 대한 로직만 고려하면 되기 때문에 단일 책임 원칙을 만족한다.

로그인, 비 로그인 상태의 경우에는 전략이 추가될 가능성은 매우 낮지만, 회원 분류에 따른 기능, 결제 등 전략이 추가될 가능성이 있는 서비스에서 높은 유지보수성과 확장성을 가질 수 있다고 생각한다.
