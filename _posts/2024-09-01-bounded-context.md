---
title: "[DDD] 도메인 모델과 바운디드 컨텍스트"
description: "도메인 모델과 바운디드 컨텍스트에 대해 이해한다. MSA환경에서의 바운디드 컨텍스트에 대해서도 다룬다."
date: 2024-09-01 15:50 +0900
lastmod: 2024-09-01 15:50 +0900
categories: architecture
tags:
  - msa
  - ddd
  - bounded context
---

### 도메인 모델과 경계

Bounded context는 의미적으로 구분되는 경계를 말한다. 하나의 상품에서 카테고리 컨텍스트와 재고 컨텍스트는 서로 의미가 다르다. 이렇게 하나의 모델을 구성하고 있어도 하위 모델들은 명시적으로 구분되는 경계를 가져야 한다.

### Bounded Context

모델의 경계를 결정하고, 한 개의 바운디드 컨텍스트는 논리적으로 한 개의 모델을 갖는다. 바운디드 컨텍스트는 실제로 사용자에게 기능을 제공하는 물리적 시스템으로 도메인 모델은 바운디드 컨텍스트 안에서 도메인을 구현하게 된다.

### Bounded Context의 구현

바운디드 컨텍스트는 도메인 기능을 사용자에게 제공하는 데 필요한 표현 영역, 응용 서비스, 인프라스트럭처 영역을 포함한다. 도메인 모델의 구조와 동일한 데이터베이스 테이블도 바운디드 컨텍스트에 포함된다. 도메인 기능을 위해 필요한 것들이 모두 하나의 바운디드 컨텍스트에 속하는 것이다.

![image](/assets/img/posts/bounded_context/img1.png)

하나의 바운디드 컨텍스트에 두 가지 구현 방법이 사용될 수도 있다. 한 바운디드 컨텍스트에 CQRS 패턴을 적용하게 되면 위 구조와 같은 방식으로 구현할 수 있다.
한 바운디드 컨텍스트에 대해 도메인의 상태 변경 부분은 도메인 기능으로 구현하고 조회 부분은 단순한 리포지토리 구조로 구현할 수 있다.

### 바운디드 컨텍스트 간 통합

```java
@Override
public List<Product> getRecommendationOf(ProductId id) {
  List<RecommendationItem> items = getRecItems(id.getValue());
  return toProducts(items);
}

private List<Product> toProducts(List<RecommendationItem> items) {
  return items.stream()
      .map(item -> toProductId(item.getItemId()))
      .map(productRepository::findById)
      .collect(Collectors.toList());
}

private ProductId toProductId(String itemId) {
  return ProductId.getValue(itemId);
}

private List<RecommendationItem> getRecItems(String itemId) {
  return externalRecClient.getRecs(itemId);
}
```

상품 정보를 노출할 때 추천 상품 목록이 함께 노출되어야 한다고 가정해보자. 추천 서비스 모듈에서 한 상품에 대한 추천 상품을 조회하기 위해 외부 모듈을 호출한다. 추천 서비스 모듈로부터 추천 상품 리스트를 얻으면 각 상품의 정보를 데이터베이스로부터 얻는다.

```java
@RequiredArgsConstructor
public class RecSystemClient implements ProductRecommendationService {
...
  private final TranslateProducts translateProducts;

  @Override
  public List<Product> getRecommendationOf(ProductId id) {
    List<RecommendationItem> items = getRecItems(id.getValue());
    List<ProductCouponItem> couponItems = getProductCoupons(id.getValue());

    return translateProducts.toProducts(items, couponItems);
  }
  ...
}
```

만약 상품을 노출하기 위해 필요한 정보가 많아지거나 상품 정보를 조합에 복잡하다면 외부 모듈에서 얻어온 정보들을 가지고 재조합을 위한 클래스를 별도로 두어 처리하는 것도 고려해 볼 수 있다.

### 바운디드 컨텍스트 간 관계

서비스에는 여러 바운디드 컨텍스트가 존재하고 두 개 이상의 바운디드 컨텍스트 간의 결합이 필요한 경우가 있다. 상품을 노출할 때 리뷰, 할인, 추천 상품 등의 정보가 포함 되어야 한다.
![image](/assets/img/posts/bounded_context/img2.png)
카탈로그와 추천 바운디드 컨텍스트 관계를 나타낸 그림이다. `고객`인 카탈로그 컨텍스트는 추천 컨텍스트가 제공하는 데이터와 기능에 의존한다. 상류 컨텍스트는 공급자 역할을 하고 하류 컨텍스트는 소비하는 고객 역할을 한다. 공급자가 제공하는 데이터를 소비하기 위한 방법으로는 Rest API 호출을 하거나 데이터 파이프라인으로 사용할 수 있다.

### Open Host Service(OHS)

공개 오픈 서비스는 공급자 서비스를 말한다. 공급자는 고객 서비스가 사용할 API를 정의하고 공개하여 서비스의 일관성을 유지한다. 하나의 공급자에는 여러 고객이 소비하는 관계로 구성될 수 있다.
![image](/assets/img/posts/bounded_context/img3.png)

예를 들어, 회원 정보를 여러 모듈에서 사용한다고 생각해보자. 회원은 로그인을 하게 되면 회원의 정보를 redis session에 저장한다. 다른 모듈은 access token을 사용해 redis session에서 회원 정보를 얻는다. 회원 정보를 얻는 api를 구성하는 대신 redis를 사용해서 간접적으로 공급자 역할을 할 수 있다.

### Anticorruption Layer(ACL)

![image](/assets/img/posts/bounded_context/img4.png)
안티 코럽션 계층은 외부 시스템의 도메인에 의해 본래의 도메인이 깨지는 것을 방지해 준다. RecSystemClient는 외부 시스템과 연동을 처리하는 부분이다. 추천과 쿠폰 바운디드 컨텍스트에서 얻은 데이터를 조합하는 부분으로 다른 컨텍스트에 영향을 받지 않고 본래의 컨텍스트를 유지할 수 있다.

### Shared Kernel

공유 커널은 두 바운디드 컨텍스트가 동일한 모델을 공유하는 방법이다. 주문 정보는 고객에 의해 발생한 주문 정보를 가지고 셀러, 관리자 모듈에서 동일한 모델을 사용할 수 있다.

- 고객은 주문한 내역을 확인한다
- 셀러는 주문을 확인하고 물건을 준비한다
- 관리자는 쇼핑몰에서 발생한 주문 통계를 확인한다

간단하게 생각하면 세 가지의 모듈에서 하나의 주문을 표현하는 모델을 가지고 공유할 수 있다. 이렇게 여러 모듈이 공유하는 모델을 공유 커널이라고 한다.

공유 커널은 여러 모듈을 담당하는 팀들의 논의에 의해 모델을 정의하고 정책이 변경된다면 모델을 소비하는 서비스들은 변경이 필요할 수 있다.
MSA 아키텍처에서는 주문 도메인을 공유 커널로 사용하는 것은 제약이 많을 수 있다. 각 모듈에서 필요한 제약 조건이 다를 수 있고, 모델의 변경이 필요한 경우 이를 논의하기 위해 개발이 지연될 수 있다.

### Context Map

![image](/assets/img/posts/bounded_context/img5.png)
컨텍스트 맵은 시스템의 전체적인 구조를 확인할 수 있다. 바운디드 컨텍스트 간의 관계를 나타내고 고객/공급자 관계를 명시하면서 어떤 관계를 맺고 있는지 명확하게 알 수 있다. 대규모 시스템의 구조를 이해하기 쉬운 지도 역할을 해준다.

### 정리

MSA 아키텍처에서는 하나의 micro service가 바운디드 컨텍스트로 볼 수 있다. 회원, 주문, 상품, 리뷰 등 각각의 모듈은 모델이 갖는 의미와 경계를 명확하게 이해할 수 있다.
바운디드 컨텍스트 간의 관계를 생각하며 설계 한다면, 각 컨텍스트의 역할이 명확해 질 수 있을 것 같다. 컨텍스트 맵을 그려보며 고객/공급자 관계를 명시하고 공유 커널을 사용하거나, 각 모듈이 공급자에 의존한 형태로 데이터를 처리하도록 설계할 수 있을 것이라고 생각한다.
