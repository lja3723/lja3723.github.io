---
title: "[Javs] Fluent Interface"
date: 2024-09-04 15:09:00 +0900
categories: [Program-Language, Java]
tags: [Java, Javstudy]
---


## 1. Introduction

요즘 나오는 API들이 즐겨 사용하는 스타일이 있다. 아래 여러 예시를 보면서, 감을 잡아보자.

- Java 8에서 도입된 Stream API

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class StreamExample {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        List<Integer> processedNumbers = numbers.stream()
            .filter(n -> n % 2 == 0)  // 짝수 필터링
            .sorted()                 // 정렬
            .map(n -> n * 2)          // 각 숫자에 2를 곱함
            .collect(Collectors.toList());  // 리스트로 수집

        System.out.println(processedNumbers);  // 출력: [4, 8, 12, 16, 20]
    }
}
```

- StringBuilder

```java
public class StringBuilderExample {
    public static void main(String[] args) {
        String result = new StringBuilder()
            .append("Hello, ")    // 문자열 추가
            .append("world!")     // 또 다른 문자열 추가
            .append(" How are you?")
            .toString();          // 최종 문자열로 변환

        System.out.println(result);  // 출력: "Hello, world! How are you?"
    }
}
```

- java.time.LocalDate

```java
public class LocalDateExample {
    public static void main(String[] args) {
        LocalDate date = LocalDate.now()
            .plusDays(5)    // 현재 날짜에서 5일 추가
            .minusMonths(1) // 1개월 감소
            .withYear(2022); // 연도를 2022로 변경

        System.out.println(date);  // 예: 2022-08-19
    }
}
```

- JPA Criteria API

타입 안전한 방식으로 쿼리를 작성할 수 있게 도와주는 API이다.

```java
CriteriaBuilder cb = entityManager.getCriteriaBuilder();
CriteriaQuery<Employee> cq = cb.createQuery(Employee.class);
Root<Employee> root = cq.from(Employee.class);

cq.select(root)
    .where(cb.equal(root.get("department"), "IT"))
    .orderBy(cb.asc(root.get("salary")));

List<Employee> employees = entityManager.createQuery(cq).getResultList();
```

- MockMvc

웹 앱의 엔드포인트를 테스트할 때 사용하는 Spring의 도구이다.

```java
mockMvc.perform(get("/api/employees/1"))
    .andExpect(status().isOk())
    .andExpect(content().contentType(MediaType.APPLICATION_JSON))
    .andExpect(jsonPath("$.name").value("John Doe"));
```

- BDDMockito

모의 객체 라이브러리로, BDD(Behavior-Driven Development, 행위 주도 개발)이 적용된 Mockito이다.

```java
import static org.mockito.BDDMockito.*;

given(employeeService.getEmployeeById(1L))
    .willReturn(new Employee(1L, "John Doe"));

Employee employee = employeeService.getEmployeeById(1L);

then(employeeService)
    .should()
    .getEmployeeById(1L);
```

- 빌더 패턴 (빌더 클래스 생략)

```java
public class Employee {
    private String name;
    private int age;
    private String department;

    // EmployeeBuilder 클래스 생략

    public static void main(String[] args) {
        Employee employee = new EmployeeBuilder()
            .setName("John Doe")
            .setAge(30)
            .setDepartment("IT")
            .build();
    }
}
```

---

이렇듯 여러 API에서 여러 메서드를 계속 연결하여 호출하는 것을 볼 수 있다. 이러한 코드 형식을 메서드 체이닝이라고 한다.



## 2. 메서드 체이닝(Method Chaining)

### 2.1. 장점

- **가독성 향상**
  - 메서드 체이닝을 사용하면 코드를 마치 자연어처럼 읽을 수 있게 되어 가독성이 크게 향상된다.
- **코드 간결성**
  - 메서드 체이닝은 불필요한 중간 변수 선언을 줄여 코드를 간결하게 만들어 준다.
- **유연한 API 설계**
  - API 설계 시 메서드 체이닝을 활용하면, 사용자가 다양한 방법으로 메서드를 조합할 수 있어 유연한 사용이 가능하다.
- **객체 불변성 지원**
  - 메서드 체이닝을 활용하면 불변 객체를 만들기 쉬워진다.

### 2.2. 단점

- **디버깅 어려움**
  - 메서드 체이닝이 길어질수록, 어느 부분에서 문제가 발생했는지 파악하기 어려진다. 각 메서드가 체인으로 연결되어 있기 때문에, 중간에 발생하는 오류를 추적하는 것이 복잡해질 수 있다.
- **에러 처리 복잡성**
  - 메서드 체이닝은 각 메서드가 연속적으로 호출되기 때문에, 각 메서드의 오류를 개별적으로 처리하기 어렵다.
- **유지보수 어려움**
  - 체이닝이 지나치게 복잡하거나 길어지면, 코드의 유지보수가 어려워질 수 있다. 특히, 체인에 새로운 메서드를 추가하거나 변경해야 할 경우, 모든 호출 경로를 점검해야 할 수 있다.



## 3. Fluent Interface

Fluent Interface는 객체 지향 프로그래밍에서 메서드 체이닝(Method Chaining)을 통해 코드의 가독성을 높이고, 직관적으로 코드를 작성할 수 있도록 설계된 API 디자인 패턴이다. 이 패턴을 사용하면 여러 메서드를 연속해서 호출할 수 있으며, 각 메서드 호출이 객체 자체를 반환함으로써 추가적인 메서드 호출을 이어갈 수 있다.

- 소스코드의 가독성을 높이기 위한 목적으로 사용된다.
- 인터페이스 안에 도메인 특화 언어(DSL)를 이용하여 작성한다.

플루언트 인터페이스는 메서드 체이닝의 장점을 거의 그대로 가져가는 API 설계 방식이며, 요즘 API들이 많이 채택하는 방식이다.

### 3.1. 도메인 특화 언어란?

특정 영역의 문제를 해결하기 위해 맞춤 제작된 컴퓨터 언어로, 일반적인 프로그래밍 언어보다 훨씬 더 '사람에 가까운' 언어를 사용하여 특정 문제를 표현한다. DSL은 주로 사용자 인터페이스, 데이터베이스, 웹사이트 등의 특정 영역에서 사용된다.



## 4. Fluent Interface API 만들기

이 형식의 API를 구현하려면 기본적으로 동일한 클래스에서 객체를 반환하는 메서드를 선언해야 한다. Stream API를 아주 간단하게 구현해 보자.

- 간단히 구현한 SimpleStream의 사용 코드

```java
import java.util.Arrays;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);

        List<Integer> result = SimpleStream.of(numbers)
            .filter(n -> n % 2 == 0) // 짝수만 필터링
            .map(n -> n * 2)          // 각 숫자에 2를 곱함
            .collect();               // 최종 결과를 List로 수집

        System.out.println(result); // 출력: [4, 8, 12]
    }
}
```

- SimpleStream 의 구현

```java
import java.util.ArrayList;
import java.util.List;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.function.Supplier;

public class SimpleStream<T> {
    private final List<T> source; //원본 데이터 저장
    private final List<Operation<T, ?>> operations = new ArrayList<>(); //수행할 연산 저장 (지연 평가 구현)

    private SimpleStream(List<T> source) {
        this.source = source;
    }

    public static <T> SimpleStream<T> of(List<T> source) {
        return new SimpleStream<>(source);
    }

    public <R> SimpleStream<R> map(Function<? super T, ? extends R> mapper) {
        operations.add(new Operation<>(element -> mapper.apply((T) element)));
        return new SimpleStream<>(source);
    }

    public SimpleStream<T> filter(Predicate<? super T> predicate) {
        operations.add(new Operation<>(element -> predicate.test((T) element) ? element : null));
        return new SimpleStream<>(source);
    }

    public List<T> collect() {
        List<T> result = new ArrayList<>(source);
        for (Operation<T, ?> operation : operations) {
            result = (List<T>) applyOperation(result, operation);
        }
        return result;
    }

    private List<T> applyOperation(List<T> elements, Operation<T, ?> operation) {
        List<T> newList = new ArrayList<>();
        for (T element : elements) {
            T transformed = (T) operation.apply(element);
            if (transformed != null) {
                newList.add(transformed);
            }
        }
        return newList;
    }

    private static class Operation<T, R> {
        private final Function<T, R> function;

        Operation(Function<T, R> function) {
            this.function = function;
        }

        R apply(T input) {
            return function.apply(input);
        }
    }
}
```

### 4.1. API 설계 시 주의사항

SOLID 원칙의 I: 인터페이스 분리 원칙을 기억해야 한다. API의 Public 메서드의 수가 너무 많아지면 이것을 위반할 수 있다. 예를 들어 Stream API에는 40개가 넘는 Public 메서드가 있다. 

쉬운 이해를 위해 다음 HTML 문서를 생성하는 API를 봐보자.

```java
public class LargeHtmlDocument {
    private final String content;

    public String html() {
        return format("<html>%s</html>", content);
    }
    public LargeHtmlDocument header(String header) { ... }
    public LargeHtmlDocument headerTwo(String header) { ... }
    public LargeHtmlDocument headerThree(String header) { ... }
    public LargeHtmlDocument headerFour(String header) { ... }
    
    public LargeHtmlDocument unorderedList(String... items) { ... }
    public LargeHtmlDocument orderedList(String... items) { ... }
    
    public LargeHtmlDocument div(Object content) { ... }
    public LargeHtmlDocument span(Object content) { ... }
    public LargeHtmlDocument paragraph(String paragraph) { .. }
    public LargeHtmlDocument horizontalLine() { ...}

    // other methods
}

//사용 예시
class Main {
    public static void main(String[] args) {
        LargeHtmlDocument document = new LargeHtmlDocument("Initial Content");

        String html = document
            .header("Welcome to My Website")           // <h1>Welcome to My Website</h1>
            .paragraph("This is a sample paragraph.")  // <p>This is a sample paragraph.</p>
            .unorderedList("Item 1", "Item 2", "Item 3") // <ul><li>Item 1</li><li>Item 2</li><li>Item 3</li></ul>
            .orderedList("Step 1", "Step 2", "Step 3")  // <ol><li>Step 1</li><li>Step 2</li><li>Step 3</li></ol>
            .div("This is inside a div.")               // <div>This is inside a div.</div>
            .span("This is inside a span.")             // <span>This is inside a span.</span>
            .horizontalLine()                          // <hr/>
            .headerTwo("Subtitle")                      // <h2>Subtitle</h2>
            .paragraph("Another paragraph with more content.") // <p>Another paragraph with more content.</p>
            .html();                                    // <html>...</html>

        System.out.println(html);
    }
}
```

이러한 방식으로 API를 설계할 수도 있지만, 이러면 Public 메서드가 너무 많아진다. 인터페이스 분리 원칙을 위반할 수 있다.
이런 방식 말고, 조금 더 응집력이 있도록 구성할 수 있다. 아래의 예를 봐보자.

```java
String html = new LargeHtmlDocument()
    .head(new HtmlHeader(Type.PRIMARY, "title"))
    .body(new HtmlDiv()
        .append(new HtmlSpan()
            .paragraph("learning OOP from John Doe")
            .append(new HorizontalLine())
            .paragraph("The pillars of OOP:")
      )
      .append(new HtmlList(ORDERED, "Encapsulation", "Inheritance", "Abstraction", "Polymorphism"))
    )
    .footer(new HtmlDiv()
        .paragraph("trademark John Doe")
    )
    .html();
```

API를 이런 식으로 설계하면, LargeHtmlDocument에는 `head()`, `body()`, `footer()`, `html()` 의 4가지의 Public 메서드만 남길 수 있으며, 인터페이스 분리 원칙을 만족할 수 있다.



<br><br>

---

<!-- omit from toc -->
## 참고 문서
{: data-toc-skip=''}

- [[Java/디자인 패턴] 메서드 체이닝(Method Chaining) , 플루언트 인터페이스(Fluent Interface), 빌더 패턴(Builder Pattern)](https://adjh54.tistory.com/440#3.%20Mockito-1)
- [Java에서 Fluent Interface와 Builder Pattern의 차이점](https://www.baeldung.com/java-fluent-interface-vs-builder-pattern)