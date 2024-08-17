---
title: "[DDD] 도메인 서비스"
description: 도메인 서비스를 이용한 여러 애그리거트 연산하기
date: 2024-08-17 14:38 +0900
lastmod: 2024-08-17 14:38 +0900
categories: architecture
tags:
  - msa
  - ddd
  - domain service
---

MSA 아키텍처에서는 하나의 루트 애그리거트로 도메인 로직을 구현하지 못할 가능성이 높다. 상품을 등록하거나 구매하는 경우 각 동작에 필요한 여러 유효성 검사가 필요하다.
상품을 등록하는 부분에서는 상품을 등록할 수 있는 사용자인지, 상품 등록이 가능한 셀러인지 등의 로직이 필요하고, 상품 구매 시에는 상품에 대한 할인 정책, 배송비, 총 지불 금액 등의 로직이 필요하다.

이런 경우 어느 도메인 영역에서 검사 및 연산이 필요한지 모호할 수 있다. 최대한 애그리거트 루트가 책임 범위를 넘어서는 기능을 구현하지 않도록 설계해야 한다. 애그리거트 루트의 범위를 벗어나는 로직은 다른 도메인 로직이 애그리거트에 포함되어 도메인 개념 관리가 어렵게 된다. 이는 도메인 기능을 별도의 서비스로 구현하는 방법으로 해결 가능하다.

## 도메인 서비스

도메인 서비스는 도메인 영역에 위치한 도메인 로직을 표현할 때 사용한다. 여러 애그리트가 필요한 연산이나, 구현을 위해 다른 모듈의 시스템을 사용해야 하는 경우에 도메인 서비스를 사용할 수 있다.

### 도메인 서비스의 연산

쇼핑몰에서 주문을 한다고 가정해보자. 주문할 때 할인 정책에 따라 할인 받을 수 있다. 쿠폰을 사용하거나 회원 등급에 따라 할인 받을 수 있는데 주문 애그리거트에서 주문에 대한 할인 금액을 연산하기에는 적절하지 않을 수 있다.  
별도의 서비스에서 주문에 대한 할인 금액을 연산할 수 있다.

```java
@Service
@RequiredArgsConstructor
public class OrderService {
  private final DiscountCalculationService discountCalculationService;
  private final OrderRepository orderRepository;

...

  private Order createOrder(OrderNo orderNo, OrderRequest orderRequest) {
    Member member = findMember(orderRequest.getOrdererId());
    Order order = Order.createOrder(orderNo, orderRequest);
    order.calculateAmounts(this.discountCalculationService, member.getMemberGrade());
    return order;
  }
}
```

응용 서비스 영역에서 주문은 생성하는 부분이다. 클라이언트로부터 `OrderRequest`를 받아 주문자의 정보를 얻은 후 주문을 생성한다. 주문에 대한 할인 금액 연산은 `calculateAmounts`에서 이뤄지는데, `discountCalculationService`와 회원의 등급을 파라미터로 넘겨 연산한다.

```java
public class Order {

...

  public void calculateAmounts(DiscountCalculationService discountService, MemberGrade grade) {
    Money calculatedTotalAmounts = getTotalAmounts();
    Money discountAmounts =
        discountService.calculateDiscountAmount(this.orderLines, this.coupons, grade);

    this.paymentAmounts = calculatedTotalAmounts.minus(discountAmounts);
  }
}
```

```java
@Component
public class DiscountCalculationService {
  public Money calculateDiscountAmount(
      List<OrderLine> orderLines, List<Coupon> coupons, MemberGrade grade) {
    Money couponDiscount =
        coupons.stream().map(this::calculateDiscount).reduce(new Money(0), Money::add);
    Money memberShipDiscount = calculateDiscount(grade);
    return couponDiscount.add(memberShipDiscount);
  }
}
```

Order 도메인 영역에서 `DiscountCalculationService`를 사용해 주문에 대한 할인 금액을 연산할 수 있다. 쿠폰, 회원의 등급에 따른 할인 금액을 서비스에서 연산하고 Order 애그리거트에서 할인된 금액을 얻는다.

```java
public class Order {
	@AutoWired
	private DiscountCalculationService discountCalculationService;

	...
}
```

Order 도메인 영역에서 `DiscountCalculationService`를 주입하지 않고 파라미터로 받은 이유는 Order 애그리거트는 Order 도메인에 대한 정보만 제공해야 하기 때문이다. 또한, `DiscountCalculationService`는 Order를 구성하는 필드가 아니기 때문이다.

위 예시 처럼 특정 도메인 영역이 로직을 가지기 어려운 경우 도메인 서비스를 이용해 해결할 수 있다. 위 예시에서는 모놀리식 구조인 경우 가능하다. 만약 MSA 환경에서 모듈이 분리되어 있다면, 회원 정보는 redis session에서 얻는 방법, 쿠폰 정보는 다른 모듈에 HTTP 호출을 통해 데이터를 얻어 도메인 서비스를 완성할 수 있다.
