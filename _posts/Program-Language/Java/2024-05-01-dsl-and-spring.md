---
title: "[Javs] 스프링과 DSL"
date: 2024-05-01 19:09:00 +0900
categories: [Program-Language, Java]
tags: [Java, Javstudy, Spring]
---


## 1. Introduction

[모던 자바 인 액션](https://m.yes24.com/Goods/Detail/77125987) 챕터 10에서 **DSL(Domain Specific Language)**의 개념과 구현법 및 사용법이 소개되었다. 모두 읽고난 뒤 스프링 진영에서 사용하는 DSL은 무엇이 있을까 궁금하여 찾아본 결과 **QueryDSL**라는 라이브러리의 존재를 알게 되었다. 본 글에서는 QueryDSL이 무엇인지, 사용 방법은 무엇인지 아주 간단히 알아본다.

### 1.1. JPA

QueryDSL은 SQL에 관련된 DSL으로, 이를 알아보기전에 우선 JPA를 알아보자. 

아시다시피 JPA는 자바의 ORM 기술을 쉽게 구현하도록 도와주는 API 즉 객체를 DB에 저장 및 관리하기 위한 API로, 스프링에서 SQL을 이용한 CRUD작업을 손쉽게 처리하도록 도와준다.

- 지루하게 반복되는 CRUD 문제를 세련된 방법으로 해결
- 개발자는 인터페이스만 작성
- 스프링 데이터 JPA가 구현 객체를 동적으로 생성해서 주입

예를 들어 기존에는 아래와 같은 클래스로 DB 쿼리를 처리했다고 하자.

```java
public class MemberRepository {
    public void save(Member member) {...}
    public Member findOne(Long id) {...}
    public List<Member> findAll() {...}
    public Member findByUsername(String username) {...}
}

public class ItemRepository {
    public void save(Item item) {...}
    public Item one(Long id) {...}
    public List<Item> findAll() {...}
}
```

JPA를 도입하면 아래처럼 간단해질 수 있다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    Member findByUsername(String username);
}

public interface ItemRepository extends JpaRepository<Item, Long> {
    // 비어있음
}
```

- 스프링 데이터 JPA에서는 JpaRepository 인터페이스를 제공한다.
- 우리가 상상할 수 있는 모든 API를 모두 처리한다.
- 인터페이스가 인터페이스를 상속받을 때는 extends를 사용한다.

JPA를 사용하면 메서드 이름만으로 내부에서 SQL을 자동으로 생성해주는데, 가끔 사용자 정의 SQL을 직접 정의해야 할 상황이 발생한다. `@Query` 어노테이션을 이용해 직접 정의가 가능하다. 어노테이션에 제공하는 문자열을 **JPQL(Java Persistence Query Language)**이라고 한다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Query("select m from Member m where m.username = ?1")
    Member findByUsername(String username, Pageable pageable);
}
```

### 1.2. JPQL의 문제점

- 문자열이다. 타입 체크가 불가능하다.
- 애플리케이션 로딩 시점에 알 수 있다.
- 컴파일 시점에 알 수 있는 방법이 없다.
- 로직 실행 전까지 작동여부 확인이 어렵다.
- 쿼리 실행 시점에 오류를 발견한다.



## 2. QueryDSL

이런 JPQL의 문제점을 해결할 수 있는 것이 QueryDSL이다. 

- **QueryDSL이란?**
  - SQL, JPQL을 코드로 작성할 수 있도록 도와주는 빌더 API
  - 정적 타입을 이용해 SQL과 같은 쿼리를 생성하도록 도와주는 프레임워크

다만 QueryDSL은 오픈소스로, Spring에서 아직 정식 승인을 하지 않아 초기 세팅(gradle 의존성 설정)이 다소 복잡하다는 것이다. 초기 세팅법은 여기서는 생략하며 다른 블로그를 참고하자.

### 2.1. 장점

- 문자가 아닌 코드로 작성할 수 있음
- 컴파일 시점에 문법 오류를 발견할 수 있음
- 코드 자동완성(IDE 도움)단순하고 쉬움
- 코드 모양이 JPQL과 거의 비슷함
- 동적 쿼리임

### 2.2. 예시

- 원래의 JPQL `select m from Member m where m.age > 18`을 QueryDSL로 작성하면 아래 코드와 같다.

```java
JPAFactoryQuery query = new JPAQueryFactory(em);
QMember m = QMember.member;
​List<Member> list = query.selectFrom(m)
                         .where(m.age.gt(18))
                         .orderBy(m.name.desc())
                         .fetch();
```



## 3. QueryDSL 사용 이유

### 3.1. 동적 쿼리 지원

- QueryDSL을 쓰는 진짜 이유는 동적 쿼리 때문이라고 한다.
- JPQL은 정적 쿼리로, 만약 JPQL으로 동적인 쿼리가 필요한 경우 문자열을 더하는 작업을 수행해야 하므로 어렵다.
- 반면 QueryDSL은 코드르 더하는 것이기 때문에 JPQL보다 쉽게 동적 쿼리를 작성할 수 있다는 것이다.

아래 코드에 동적 쿼리의 예가 있다. `BooleanBuilder`에 조건을 넣은 다음 쿼리를 실행하면 된다. 원하는 필드만 뽑아 DTO로 뽑아내는 기능도 QueryDSL이 다 지원한다.

```java
String name = "member";
int age = 9;
​QMember m = QMember.member;
​BooleanBuilder builder = new BooleanBuilder();

if(name != null) {
    builder.and(m.name.contains(name));
}
  
if(age != 0) {
    builder.and(m.age.gt(age));
}

​List<Member> list = query.selectFrom(m)
                         .where(builder)
                         .fetch();
```

### 3.2. 자바 문법으로 작성하는 SQL

- 객체지향 관점에서 중요한 이유이다.
- 코드 자체가 자바이기 때문에 중복코드를 함수로 추출할 수 있다.
- 재사용성과 가독성을 확보한다.
- 리팩토링이 용이하다.

### 3.3. 결론

따라서 스프링에서 만약 JPA를 사용할 때 `@Query`를 사용할 일이 있을 경우 QueryDSL을 사용하는 편이 좋을 것이다.



## 4. DSL 관점에서 살펴보기

DSL 관점에서, QueryDSL은 자바 코드로 작성하기 때문에 **내부적 DSL**이라 볼 수 있다. 반면 JPQL은 문자열으로 별도의 문법으로 작성하므로 **외부적 DSL**이라고 볼 수 있다. 

<br><br>

---

#### **참고자료**
{: data-toc-skip=''}

- [[JPA] Spring Data JPA와 QueryDSL 이해, 실무 경험 공유](https://ict-nroo.tistory.com/117)
- [[Spring JPA] JPA 란?](https://dbjh.tistory.com/77)
- [[Spring] QueryDSL 완벽 이해하기](https://velog.io/@jkijki12/Spring-QueryDSL-%EC%99%84%EB%B2%BD-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)
- [[JPA] JPQL이란? - 객체지향 쿼리 언어 JPQL (1)](https://ittrue.tistory.com/270)