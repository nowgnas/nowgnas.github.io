---
title: "[Effective Java] 싱글턴과 빌더 패턴"
date: 2023-08-05 15:48:52
tags: [builder, lotteon, singleton, 롯데e커머스, 비트컴퓨터]
categories: api
description: Effective Java를 읽으며 정리한 내용입니다.
---

> Effective Java를 읽으며 정리한 내용입니다.

## Builder 패턴

Builder는 spring boot에서 `@Builder` 어노테이션으로 많이 사용했었다. 이번 글에서는 lombok에서 제공되는 어노테이션이 아닌 직접 빌더 패턴을 만들어 본다.

> 생성자에 매개변수가 많다면 빌더를 고려하라 - item2

### 점층적 생성자 패턴 - telescoping constructor pattern

생성자에 필요한 매개변수가 많을 경우 매개변수 개수에 따라 생성자를 다르게 사용할 수 있다. 1개, 2개, 3개 등 개수에 따라 생성자에 사용되는 변수가 다르게 들어가게 되는데 이것을 점층적 생성자 패턴이라고 한다.

```java
public class NutritionFacts {
    private final int servingSize;  // (mL, 1회 제공량)     필수
    private final int servings;     // (회, 총 n회 제공량)  필수
    private final int calories;     // (1회 제공량당)       선택
    private final int fat;          // (g/1회 제공량)       선택
    private final int sodium;       // (mg/1회 제공량)      선택
    private final int carbohydrate; // (g/1회 제공량)       선택

    public NutritionFacts(int servingSize, int servings) {...}

    public NutritionFacts(int servingSize, int servings, int calories) {...}

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {...}

    public static void main(String[] args) {
        NutritionFacts cocaCola =
                new NutritionFacts(240, 8, 100, 0);
    }
}
```

위 예시와 같이 생성자를 만들 때 매개변수를 순서대로 넣게 된다. 사용자가 설정하고 싶지 않은 값(ex. 0)까지 넣어 줘야 하는 불편함이 있고, 매개변수 개수가 많을 경우 들어가는 값의 의미를 알기 힘들다.

### 자바 빈즈 패턴

선택 매개변수가 많을 때 사용할 수 있는 자바 빈즈 패턴이다. 특징은 매개변수가 없는 생성자로 객체를 만든 후 setter메서드들을 호출해 원하는 매개변수의 값을 설정하는 방법이다.

```java
public class NutritionFacts {
    // 매개변수들은 (기본값이 있다면) 기본값으로 초기화된다.
    private int servingSize  = -1; // 필수; 기본값 없음
    private int servings     = -1; // 필수; 기본값 없음
    private int calories     = 0;
    private int fat          = 0;
    private int sodium       = 0;
    private int carbohydrate = 0;

    public NutritionFacts() { }
    // Setters
    public void setServingSize(int val)  { servingSize = val; }
    public void setServings(int val)     { servings = val; }
    public void setCalories(int val)     { calories = val; }
    public void setFat(int val)          { fat = val; }
    public void setSodium(int val)       { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts();
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
        cocaCola.setCalories(100);
        cocaCola.setSodium(35);
        cocaCola.setCarbohydrate(27);
    }
}
```

생성자를 만들고 setter로 값을 할당하는 방식으로 점층적 생성자 패턴보다 직관적인 코드가 되었다. 자바 빈즈 패턴은 객체를 생성하기 위해 여러 메서드를 호출해야 하는 불편함이 있다. 또한 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태이다. 자바 빈즈 패턴은 클래스를 불변으로 만들 수 없고 \*스레드 안정성을 얻으려면 개발자가 추가로 작업을 해야 한다.

### 빌더 패턴

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 - 기본값으로 초기화한다.
        private int calories      = 0;
        private int fat           = 0;
        private int sodium        = 0;
        private int carbohydrate  = 0;

				public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
		public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                .calories(100).sodium(35).carbohydrate(27).build();
    }
}
```

점층적 생성자 패턴의 안정성과 자바 빈즈 패턴의 가독성을 갖춘 빌더 패턴이다. 클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻는다. 그 후 빌더 객체가 제공하는 setter 메서드로 원하는 매개변수를 설정할 수 있다. `build()` 메서드를 호출하여 원하는 객체를 얻는다.

Spring boot에서는 Builder 어노테이션으로 간편하게 사용할 수 있지만, 빌더 패턴의 구체적 동작을 알 수 있는 내용이었다.

---

\*스레드 안정성: 멀티 스레드 프로그래밍에서 일반적으로 함수나 변수, 혹은 객체가 여러 스레드로부터 동시에 접근이 이뤄져도 프로그램 실행에 문제가 없음을 말한다. 하나의 함수가 한 스레드로부터 호출되어 실행 중일 때, 다른 스레드가 그 함수를 호출하여 동시에 실행해도 각 스레드에서 함수 결과가 올바른 것.
