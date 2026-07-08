# 🚀 [TIL] Java Stream `.toString()` 문제 해결 & Enum 내부 동작 원리

<br>

![Java](https://img.shields.io/badge/Java-17-orange?style=for-the-badge&logo=openjdk)
![TIL](https://img.shields.io/badge/TIL-Learning-blue?style=for-the-badge)
![GitHub](https://img.shields.io/badge/GitHub-Markdown-success?style=for-the-badge&logo=github)

</div>

---

# 📚 목차

- [1. 학습 배경](#-1-학습-배경)
- [2. Stream `.toString()` 문제](#-2-stream-tostring-문제)
- [3. 해결 방법](#-3-해결-방법)
- [4. 더 좋은 방법 : replace()](#-4-더-좋은-방법--replace)
- [5. Enum 구조 분석](#-5-enum-구조-분석)
- [6. Enum을 사용하는 이유](#-6-enum을-사용하는-이유)
- [7. Enum 내부 동작 원리](#-7-enum-내부-동작-원리)
- [8. 핵심 정리](#-8-핵심-정리)
- [9. 느낀 점](#-9-느낀-점)

---

# 📖 1. 학습 배경

코딩 테스트 문제를 풀면서 두 가지 궁금증이 생겼다.

### ❓첫 번째

왜 Stream 뒤에 `.toString()`을 붙이면 문자열이 만들어지지 않을까?

### ❓두 번째

Enum은

```java
LAMB("양꼬치", 12000)
```

처럼 선언하는데,

- 생성자는 언제 호출되는가?
- 객체는 어디에 저장되는가?
- `getPrice()`는 어떻게 값을 가져오는가?

이번 글에서는 단순히 사용법이 아니라 **동작 원리까지** 이해하는 것을 목표로 정리하였다.

---

# 💥 2. Stream `.toString()` 문제

## 문제 코드

```java
import java.util.stream.Stream;

class Solution {

    public String solution(String str, String letter) {

        return Stream.of(str.split(""))
                .filter(e -> !e.equals(letter))
                .toString();
    }

}
```

실행 결과는 예상과 달랐다.

```text
java.util.stream.ReferencePipeline$2@6d06d69c
```

문자열이 아니라 Stream 객체의 정보가 출력된다.

---

## 🤔 왜 이런 결과가 나올까?

많은 사람들이

```java
.toString()
```

을 사용하면

> "안에 있는 데이터를 문자열로 만들어주겠지."

라고 생각하기 쉽다.

하지만 **Stream은 데이터를 저장하는 컬렉션이 아니라 데이터를 흘려보내는 파이프라인(Pipeline)** 이다.


```java
stream.toString()
```

Stream 내부 요소를 출력하는 것이 아니라

**Object의 `toString()`**

을 그대로 호출하는 것이다.

따라서 객체 정보만 반환된다.

---

# ✅ 3. 해결 방법

문자열을 하나로 합치려면

**최종 연산(Terminal Operation)** 이 필요하다.

대표적인 방법이

```java
Collectors.joining()
```

이다.

```java
import java.util.stream.Collectors;
import java.util.stream.Stream;

class Solution {

    public String solution(String str, String letter) {

        return Stream.of(str.split(""))
                .filter(e -> !e.equals(letter))
                .collect(Collectors.joining());

    }

}
```

---

## 실행 흐름

```text
문자열
      │
      ▼
 split("")
      │
      ▼
 Stream<String>
      │
      ▼
 filter()
      │
      ▼
 Stream<String>
      │
      ▼
 Collectors.joining()
      │
      ▼
 String 생성
```

---

# ⚡ 4. 더 좋은 방법 : `replace()`

사실 이번 문제는 Stream을 사용할 필요도 없다.

문자 하나를 제거하는 목적이라면

```java
class Solution {

    public String solution(String str, String letter) {

        return str.replace(letter, "");

    }

}
```

이 방법이 가장 좋다.

## 이유

✅ 코드가 짧다.

✅ 가독성이 좋다.

✅ Stream 생성 비용이 없다.

✅ 성능도 더 좋다.

---

# 🏷️ 5. Enum 구조 분석

다음은 양꼬치 가격 계산 문제이다.

```java
class Solution {

    public int solution(int n, int k) {

        int lambTotalPrice = totalPrice(Menu.LAMB, n);
        int drinkTotalPrice = totalPrice(Menu.DRINK, k);
        int discountPrice = discount(Menu.DRINK, n);

        return lambTotalPrice + drinkTotalPrice - discountPrice;
    }

    private int totalPrice(Menu menu, int quantity) {
        return menu.getPrice() * quantity;
    }

    private int discount(Menu menu, int lambQuantity) {
        return menu.getPrice() * (lambQuantity / 10);
    }

}

enum Menu {

    LAMB("양꼬치", 12000),
    DRINK("음료수", 2000);

    private final String name;
    private final int price;

    Menu(String name, int price) {
        this.name = name;
        this.price = price;
    }

    public int getPrice() {
        return price;
    }

}
```

---

# 🎯 6. Enum을 사용하는 이유

## 일반 상수의 문제점

```java
public static final int LAMB_PRICE = 12000;
public static final int DRINK_PRICE = 2000;
```

이 방법은 간단하지만 문제가 있다.

### ❌ 타입 안전성이 없다.

```java
calculate(99999);
```

잘못된 숫자가 들어와도 컴파일러는 알 수 없다.

---

### ❌ 데이터가 흩어진다.

```
양꼬치
12000

음료수
2000
```

관련 있는 데이터가 서로 따로 관리된다.

---

## Enum은 객체(Object)이다.

```java
LAMB("양꼬치", 12000)
```

이 한 줄은

- 객체 생성
- 생성자 호출
- 데이터 저장

을 동시에 수행하는 Java의 축약 문법이다.

즉,

Java Enum은 단순한 상수가 아니라

**클래스(Class)** 이다.

---

# 🔍 7. Enum 내부 동작 원리

컴파일 이후에는 아래와 비슷한 코드가 만들어진다.

```java
public final class Menu extends Enum<Menu> {

    public static final Menu LAMB =
            new Menu("LAMB", 0, "양꼬치", 12000);

    public static final Menu DRINK =
            new Menu("DRINK", 1, "음료수", 2000);

    private final String name;
    private final int price;

    private Menu(String enumName,
                 int ordinal,
                 String name,
                 int price) {

        super(enumName, ordinal);

        this.name = name;
        this.price = price;
    }

    public int getPrice() {
        return price;
    }

}
```

---

## 객체 생성 과정

```text
프로그램 시작
        │
        ▼
Menu 클래스 로딩
        │
        ▼
static 객체 생성
        │
        ▼
Menu.LAMB 생성
        │
        ▼
생성자 호출
        │
        ▼
name = "양꼬치"

price = 12000
```

---

## getPrice() 호출 과정

```java
Menu.LAMB.getPrice();
```

실제로는

```text
Menu
   │
   ▼
static 객체 LAMB
   │
   ▼
getPrice()
   │
   ▼
price 반환
   │
   ▼
12000
```

객체가 생성될 때 저장된 값을 그대로 반환하는 것이다.

---

# 💡 핵심 포인트

## Stream

❌

```java
stream.toString();
```

→ 객체 정보 출력

✅

```java
stream.collect(Collectors.joining());
```

→ 문자열 생성

---

## String

단순 문자열 치환은

```java
replace()
```

가 가장 효율적이다.

---

## Enum

```java
LAMB("양꼬치",12000)
```

↓

컴파일 후

```java
public static final Menu LAMB =
        new Menu(...);
```

으로 변환된다.

즉,

Enum은

> **"미리 생성된 단 하나의 객체(Singleton)"**

를 사용하는 구조이다.

---

# 📝 9. 느낀 점

이번 학습에서는 단순히 문법을 외우는 것이 아니라 **"왜 이렇게 동작하는가?"**에 집중했다.

특히 다음 내용을 이해한 것이 가장 큰 수확이었다.

- Stream은 데이터를 저장하는 컬렉션이 아니라 **파이프라인**이라는 점
- `.toString()`이 문자열을 합쳐주는 메서드가 아니라는 점
- `Collectors.joining()`이 필요한 이유
- Enum은 단순한 상수가 아니라 **객체**라는 점
- `LAMB("양꼬치",12000)`는 객체 생성과 생성자 호출을 동시에 수행하는 축약 문법이라는 점
- 컴파일 이후에는 `public static final Menu LAMB = new Menu(...)` 형태로 변환되어 관리된다는 점

---

</div>
