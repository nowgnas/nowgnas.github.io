---
title: "[DDD] 애그리거트"
date: 2024-07-21 00:03 +0900
lastmod: 2024-07-21 00:03 +0900
tags:
  - msa
  - ddd
  - aggregate
categories: architecture
---

[이전 글](https://nowgnas.github.io/posts/domain/)에서 도메인 모델과 도메인 주도 개발을 위한 아키텍처를 알아봤다. 이번 글에서는 도메인 모델을 잘 이해할 수 있도록 만들어주는 aggregate(애그리거트)에 대해 알아본다.

## Aggregate

애그리거트는 관련된 객체를 하나로 묶는 역할을 한다. 애그리거트로 묶어서 보면 모델간의 관계를 모델 수준과 상위 수준에서 이해하기 용이하다. 애그리거트 단위로 모델의 일관성을 관리하기 때문에 복잡한 도메인을 단순한 구조로 만들고, 복잡도가 낮아져 기능 확장이나 변경에 비용이 줄어든다.

도메인 모델을 사용하지 않는 경우 세부적인 모델만 이해한 상황에서 로직을 구현하게 되면, 당장 동작하는 코드를 작성할 수 있지만, 유지보수가 어려운 코드를 만들게 된다.
애그리거트는 관련된 모델들을 하나로 묶어 관리하며, 속한 객체들은 동일하거나 유사한 라이프 사이클을 갖게 된다. 이런 특징을 이용해 모델의 일관성을 유지할 수 있다.

## Aggregate root와 데이터 일관성

애그리거트 루트는 애그리거트에 속한 객체들의 일관성 유지를 책임진다. 애그리거트는 도메인 규칙을 구현한 기능을 제공하여 데이터 일관성이 깨지지 않도록 한다.

애그리거트 루트에 속한 객체들을 변경하거나 유효성 검사를 하는 등의 동작은 모두 애그리거트 내에서 이뤄진다.

```java
if (state != OrderState.PAYMENT_WAITING && state != OrderState.PREPARING) {
  throw new IllegalStateException();
}
```

배송 정보를 변경할 경우 현재 주문 상태가 정보 변경 가능한 상태인지 확인하는 로직이다. 배송 주소를 변경하거나 배송 메세지를 변경할 때 사용할 수 있다. 이런 검사 로직을 응용 영역에서 사용한다면 매번 서비스 로직에 검사 로직을 추가해야 한다. 만약 정책의 변경으로 검사 로직에 변경이 발생하면 코드 중복으로 인해 사용한 부분을 모두 찾아 변경해야 한다.

```java
public class Order {
	...
	public void changeShippingAddress(Address address) {
	  verifyNotYetShipped();
	  setAddress(address);
	}

	public void changeReceiver(Receiver receiver) {
	  verifyNotYetShipped();
	  setReceiver(receiver);
	}

	private void verifyNotYetShipped() {
	  if (state != OrderState.PAYMENT_WAITING && state != OrderState.PREPARING) {
	    throw new IllegalStateException();
	  }
	}
	private void setReceiver(Receiver receiver) {...}
	private void setAddress(Address address) {...}

  ...
}
```

위 코드는 주문 상태 변경이 가능한지와 주문의 상태를 변경하기 위한 set 메서드를 사용하여 개선한 부분이다. 애그리거트를 통해서만 도메인 로직을 구현하고, set 메서드를 private으로 구성하여 외부에서 내부 상태를 변경하지 못하도록 하여 애그리거트의 일관성을 깨지지 않게 한다.

## 레포지토리와 애그리거트

애그리거트는 개념상 완전한 한 개의 도메인 모델을 표현하기 때문에 객체의 영속성을 처리하는 레포지토리는 애그리거트 단위로 존재한다.
애그리거트를 저장할 때 애기리거트에 속한 모든 구성 요소가 저장되어야 한다. 애그리거트를 조회하게 되면 애그리거트를 구성하는 하위 객체 정보가 함께 조회되어야 한다. 예를 들어 Order 애그리거트를 조회하게 되면 Orderer, Address 등 하위 객체도 함께 조회되어 정보를 제공해야 한다. 그렇지 않으면 도메인 기능을 실행 중 값이 없는 경우 NPE가 발생할 수 있다.

한 객체에서 다른 애그리거트 루트를 참조하는 것은 구현이 쉬워지지만, 도메인 규칙을 따르지 못할 수 있고, 의존 결합도가 높아져 결과적으로는 애그리거트 변경을 어렵게 만든다.
MSA 환경에서는 도메인 별로 서비스가 분산되어 있고, 데이터베이스도 분리되어 있다. 이런 경우 주문과 회원 도메인이 분리되어 있다면 Order 내의 객체가 Member를 참조한다면 조회하지 못하는 경우가 발생한다.

Id를 이용한 애그리거트 참조 방식을 사용하면 모든 객체가 참조로 연결되지 않고, 한 애그리거트에 속한 객체들만 참조로 연결된다. Id 참조를 통해 구현 복잡도를 낮추고, 응용 서비스에서 id를 이용해 애그리거트를 조회하는 방식으로 구현할 수 있다.

### 애그리거트를 팩토리로 사용하기

```java
public ProductId registerNewProduct(NewProductRequest req) {
    Store store = storeRepository.findById(req.getStoreId())
            .orElseThrow(NullPointerException::new);
    ProductId id = productRepository.nextId();
    Product product = store.createProduct(id, req.getProductInfo());
    productRepository.save(product);
    return id;
}
```

```java
public class Store {

    private Status status;


    public Product createProduct(ProductId productId, ProductInfo productInfo) {
        if (isBlocked()) throw new StoreBlockedException();
        return ProductFactory.create(productId, productInfo);
    }

    private boolean isBlocked() {
        return this.status.isValue();
    }
}
```

Store 애그리거트를 Product를 생성하는 팩토리 역할을 하도록 구성하면 도메인 로직을 따르게 할 수 있고 응용 서비스에서 store의 상태를 확인하지 않아도 된다. Store 애그리거트의 목적은 Store의 차단 여부이므로 차단 여부를 확인하고 Product의 생성은 ProductFactory라는 다른 팩토리에 위임하는 방식으로 구현하여 책임을 분리할 수 있다.
