---
title: "[SQL] DML: JOIN (feat. INNER, OUTER)"
date: 2024-04-19 23:30:00 +0900
categories: [Computer-Science, DB]
tags: ['DB', 'SQL']
---



## 1. 조인

**두 개 이상의 테이블**에 **동시**에 질의를 할 수 있다. 이것은 **조인(Join) 연산**을 이용해 할 수 있는데, 이것은 한 테이블의 행에 다른 테이블의 행을 연결해 결합하는 연산이다.

가장 기본적으로 조인을 아래처럼 할 수 있다.

```sql
-- 카테시안 곱과 동일하다.
SELECT  *
FROM    Customer, Orders;
```
{: .nolineno}

- **질의 결과**

{% assign img_path = "/assets/img/posts/DB/2024-04-19-sql-dml-join" %}
![1. 조인]({{ img_path }}/1. 조인.png){: width='500'}
_Customer와 Orders 테이블의 합체_

- `Customer`와 Orders `테이블의` 튜플 개수는 각각 5, 10이고 이들을 합체한 결과 테이블의 튜플 개수는 $5 \times 10 = 50$이다.
- 두 테이블을 이처럼 아무 조건 없이 `SELECT` 시키면 관계대수의 **카티전 프로덕트 연산**이 된다.
- 이렇게 얻은 결과는 **논리에 맞지 않으므로**, 적절한 조건을 걸어주어야 의미가 있다.

### 1.1. 조인 연산의 예시

**`질의 3-21`** 고객과 고객의 주문에 관한 데이터를 모두 보이시오.

<br> **정답**

```sql
SELECT  *
FROM    Customer, Orders
WHERE   Customer.custid = Orders.custid; -- 사실 요즘 DBMS는 조건에 사용된 속성을 기준으로 결과가 자동으로 정렬이 된다.
```
{: .nolineno}

<br> **질의 결과**

- `orderid` 오름차순으로 되어있음을 확인할 수 있다.

| custid | name   | address         | phone         | orderid | custid | bookid | saleprice | orderdate  |
| ------ | ------ | --------------- | ------------- | ------- | ------ | ------ | --------- | ---------- |
| 1      | 박지성 | 영국 맨체스타   | 000-5000-0001 | 1       | 1      | 1      | 6000      | 2024-07-01 |
| 1      | 박지성 | 영국 맨체스타   | 000-5000-0001 | 2       | 1      | 3      | 21000     | 2024-07-03 |
| 2      | 김연아 | 대한민국 서울   | 000-6000-0001 | 3       | 2      | 5      | 8000      | 2024-07-03 |
| 3      | 김연경 | 대한민국 경기도 | 000-7000-0001 | 4       | 3      | 6      | 6000      | 2024-07-04 |
| 4      | 추신수 | 미국 클리블랜드 | 000-8000-0001 | 5       | 4      | 7      | 20000     | 2024-07-05 |
| 1      | 박지성 | 영국 맨체스타   | 000-5000-0001 | 6       | 1      | 2      | 12000     | 2024-07-07 |
| 4      | 추신수 | 미국 클리블랜드 | 000-8000-0001 | 7       | 4      | 8      | 13000     | 2024-07-07 |
| 3      | 김연경 | 대한민국 경기도 | 000-7000-0001 | 8       | 3      | 10     | 12000     | 2024-07-08 |
| 2      | 김연아 | 대한민국 서울   | 000-6000-0001 | 9       | 2      | 10     | 7000      | 2024-07-09 |
| 3      | 김연경 | 대한민국 경기도 | 000-7000-0001 | 10      | 3      | 8      | 13000     | 2024-07-10 |

---

<br>

**`질의 3-22`** 고객과 고객의 주문에 관한 데이터를 고객번호 순으로 정렬하여 보이시오.

<br> **정답**

```sql
SELECT    *
FROM      Customer, Orders
WHERE     Customer.custid = Orders.custid;
ORDER BY  Customer.custid;
```
{: .nolineno}

<br> **질의 결과**

| custid | name   | address         | phone         | orderid | custid | bookid | saleprice | orderdate  |
| ------ | ------ | --------------- | ------------- | ------- | ------ | ------ | --------- | ---------- |
| 1      | 박지성 | 영국 맨체스타   | 000-5000-0001 | 1       | 1      | 1      | 6000      | 2024-07-01 |
| 1      | 박지성 | 영국 맨체스타   | 000-5000-0001 | 2       | 1      | 3      | 21000     | 2024-07-03 |
| 1      | 박지성 | 영국 맨체스타   | 000-5000-0001 | 6       | 1      | 2      | 12000     | 2024-07-07 |
| 2      | 김연아 | 대한민국 서울   | 000-6000-0001 | 3       | 2      | 5      | 8000      | 2024-07-03 |
| 2      | 김연아 | 대한민국 서울   | 000-6000-0001 | 9       | 2      | 10     | 7000      | 2024-07-09 |
| 3      | 김연경 | 대한민국 경기도 | 000-7000-0001 | 4       | 3      | 6      | 6000      | 2024-07-04 |
| 3      | 김연경 | 대한민국 경기도 | 000-7000-0001 | 8       | 3      | 10     | 12000     | 2024-07-08 |
| 3      | 김연경 | 대한민국 경기도 | 000-7000-0001 | 10      | 3      | 8      | 13000     | 2024-07-10 |
| 4      | 추신수 | 미국 클리블랜드 | 000-8000-0001 | 5       | 4      | 7      | 20000     | 2024-07-05 |
| 4      | 추신수 | 미국 클리블랜드 | 000-8000-0001 | 7       | 4      | 8      | 13000     | 2024-07-07 |

> 열 이름을 표기하는 방법
> 
> `Customer.custid`, `Orders.custid`와 같이 `테이블이름.열이름` 형식으로 표현함으로써 해당 속성명이 어느 테이블의 속성인지 명시할 수 있다.
{: .prompt-info}

---

<br>

**`질의 3-23`** 고객의 이름과 고객이 주문한 도서의 판매가격을 검색하시오.

<br> **정답**

```sql
SELECT  name, saleprice
FROM    Customer, Orders
WHERE   Customer.custid = Orders.custid;
```
{: .nolineno}

<br> **질의 결과**

| name   | saleprice |
| ------ | --------- |
| 박지성 | 6000      |
| 박지성 | 21000     |
| 박지성 | 12000     |
| 김연아 | 8000      |
| 김연아 | 7000      |
| 김연경 | 6000      |
| 김연경 | 12000     |
| 김연경 | 13000     |
| 추신수 | 20000     |
| 추신수 | 13000     |

---

> 모든 SQL 질의의 결과는 단일 테이블이므로, **`질의 3-23`** 결과 테이블에 SQL 문법을 적용할 수 있다.
{: .prompt-tip}

<br>

**`질의 3-24`** 고객별로 주문한 모든 도서의 총 판매액을 구하고, 고객별로 정렬하시오.

<br> **정답**

```sql
SELECT    name, SUM(saleprice)
FROM      Customer, Orders
WHERE     Customer.custid = Orders.custid
GROUP BY  name  -- Customer.name 으로 명시적으로 표현 가능하다.
ORDER BY  name;
```
{: .nolineno}

<br> **질의 결과**

| name   | SUM(saleprice) |
| ------ | -------------- |
| 김연경 | 31000          |
| 김연아 | 15000          |
| 박지성 | 39000          |
| 추신수 | 33000          |

---



## 2. 3개 이상의 테이블에 대한 조인

3개 이상의 테이블을 조인할 수 있다.

<br>

---
**`질의 3-25`** 고객의 이름과 고객이 주문한 도서의 이름을 구하시오.

<br> **정답**

```sql
SELECT  Customer.name, Book.bookname
FROM    Customer, Orders, Book
WHERE   Orders.custid = Customer.custid AND Orders.bookid = Book.bookid;
```
{: .nolineno}

<br> **질의 결과**

| name   | bookname          |
| ------ | ----------------- |
| 박지성 | 축구의 역사       |
| 박지성 | 축구의 이해       |
| 김연아 | 피겨 교본         |
| 김연경 | 배구 단계별기술   |
| 추신수 | 야구의 추억       |
| 박지성 | 축구 아는 여자    |
| 추신수 | 야구를 부탁해     |
| 김연경 | Olympic Champions |
| 김연아 | Olympic Champions |
| 김연경 | 야구를 부탁해     |

다음은 조인 과정 중 데이터들이 연결되는 모습을 나타낸 것이다.

![2. 3개 이상의 테이블에 대한 조인]({{ img_path }}/2. 3개 이상의 테이블에 대한 조인.png){: width='650'}
_테이블 데이터들이 연결되는 모습_

---

<br>

**`질의 3-26`** 가격이 20,000원인 도서를 주문한 고객의 이름과 도서의 이름을 구하시오.

<br> **정답**

```sql
SELECT  Customer.name, Book.bookname
FROM    Customer, Orders, Book
WHERE   Orders.custid = Customer.custid AND Orders.bookid = Book.bookid 
        AND Book.price = 20000;
```
{: .nolineno}

<br> **질의 결과**

| name   | bookname    |
| ------ | ----------- |
| 추신수 | 야구의 추억 |

---


## 3. INNER JOIN

일반적으로 조인이라고 하면 이 **INNER JOIN**을 말하는데, 지금까지는 조인을 하기 위해 `FROM` 절에서 쉼표로 테이블을 열거하였다.

이 방법 대신 `INNER JOIN` 문을 사용할 수 있다. 쉼표로 열거하는 기존의 방법도 INNER JOIN이라고 할 수 있지만, `WHERE` 조건을 생략하면 카티전 곱이 되기 쉽다는 단점이 있다.

> **참고** 
> 
> SQL에서는 주로 **동등 조인**을 사용한다.
{: .prompt-info}

### 3.1. 방법 1

- `조인조건`이 `WHERE` 절으로 들어가는 방법이다.
- 지금까지 해왔던 조인 방식이다.

```sql
SELECT  '속성1', '속성2', etc
FROM    TABLE1, TABLE2
WHERE   '조인조건' AND '검색조건'
```
{: .nolineno}

### 3.2. 방법 2

- `INNER JOIN` 문을 사용하면 `ON` 문이 뒤따라와야 하며, 여기에는 `조인조건`을 명시한다.
- `조인조건`을 `ON` 구문을 이용해 따로 명시해야 하기 때문에, **가독성이 조금 더 좋아진다.**
- `[]`: 생략 가능하다. 즉 `INNER` 키워드는 생략 가능하며, `JOIN`문은 기본적으로 **INNER JOIN**을 수행한다.

```sql
SELECT  '속성1', '속성2', etc
FROM    TABLE1 [INNER] JOIN TABLE2 
            ON '조인조건'
WHERE   '검색조건'
```
{: .nolineno}

<br>

---
**`질의 3-26 (feat. INNER JOIN)`** 가격이 20,000원인 도서를 주문한 고객의 이름과 도서의 이름을 구하시오.

- **기존 답**

```sql
SELECT  Customer.name, Book.bookname
FROM    Customer, Orders, Book
WHERE   Orders.custid = Customer.custid AND Orders.bookid = Book.bookid 
        AND Book.price = 20000;
```
{: .nolineno}

<br> **정답**

```sql
SELECT  Customer.name, Book.bookname
FROM    Customer
    INNER JOIN  Orders
        ON Orders.custid = Customer.custid
    INNER JOIN  Book
        ON Orders.bookid = Book.bookid 
WHERE   Book.price = 20000;
```
{: .nolineno}

- `INNER`를 생략해 더 간단히 표기가 가능하다.

```sql
SELECT  Customer.name, Book.bookname
FROM    Customer 
          JOIN Orders ON Orders.custid = Customer.custid
          JOIN Book   ON Orders.bookid = Book.bookid 
WHERE   Book.price = 20000;
```
{: .nolineno}

- `JOIN`과 `ON`을 묶어서 더욱 간단히 표기할 수 있다.

```sql
SELECT  Customer.name, Book.bookname
FROM    Customer JOIN Orders JOIN Book
            ON Orders.custid = Customer.custid AND Orders.bookid = Book.bookid 
WHERE   Book.price = 20000;
```
{: .nolineno}

---

### 3.3. 방법 1과 방법 2 비교

- 두 방법 모두 사용할 수 있지만, **방법 2**를 사용하는 것이 가독성에서 조금 더 뛰어다고 할 수 있다.
- 현대 DBMS에서 INNER JOIN에 한해서는 두 방법은 **성능 상에 차이가 거의 없다**고 알려져 있다.




## 4. OUTER JOIN(외부조인)

### 4.1. 문법

외부 조인은 다음 문법으로 가능하다.

- `[]`: 생략 가능
  - 따라서 `OUTER`를 생략할 수 있다.
- `{}`: `|`으로 구분된 조건들 중 하나를 선택해야 함
  - 이것을 아예 생략하면 INNER JOIN이 된다.

```sql
SELECT  '속성1', '속성2', etc
FROM    TABLE1 { LEFT | RIGHT | FULL } [OUTER] JOIN TABLE2 
            ON '조인조건'
WHERE   '검색조건'
```
{: .nolineno}

- `LEFT` `RIGHT` `FULL`은 관계대수에서 배웠던 그것으로, **LEFT 외부조인**을 수행하면 왼쪽 테이블의 튜플은 값으로 가득 채워지지만 오른쪽 테이블에는 `NULL` 값이 포함될 수 있다.

### 4.2. 예시

**`질의 3-23`**를 보면 도서를 주문하지 않은 고객 '박세리'는 결과에 포함되지 않는다. 이 고객까지 결과에 포함하려면 어떻게 해야 할까? 바로 **외부 조인**을 사용하면 된다.

<br>

---
**`질의 3-27 (3-23 변형)`** 도서를 구매하지 않은 고객을 포함하여, 고객의 이름과 고객이 주문한 도서의 판매 가격을 구하시오.

<br> **틀린 답(3-23의 정답)**

```sql
SELECT  Customer.name, Orders.saleprice
FROM    Customer, Orders
WHERE   Customer.custid = Orders.custid;
-- Orders.custid에 주문하지 않은 고객의 custid가 없으므로,
-- 따라서 주문하지 않은 고객은 결과에 포함되지 않는다.
```
{: .nolineno}

<br> **정답**

```sql
SELECT  Customer.name, Orders.saleprice
FROM    Customer 
            LEFT OUTER JOIN 
        Orders ON Customer.custid = Orders.custid; -- WHERE과 같은 역할을 수행한다. JOIN문 사용 시 이처럼 ON을 사용한다.
```
{: .nolineno}

<br> **질의 결과**

| name   | saleprice |
| ------ | --------- |
| 박지성 | 6000      |
| 박지성 | 21000     |
| 박지성 | 12000     |
| 김연아 | 8000      |
| 김연아 | 7000      |
| 김연경 | 6000      |
| 김연경 | 12000     |
| 김연경 | 13000     |
| 추신수 | 20000     |
| 추신수 | 13000     |
| 박세리 | `NULL`    |

---

- `ON`은 `WHERE` 절과 같은 역할을 수행하고 있다. 
- 이처럼 **`JOIN` 절과 함께라면 `WHERE` 대신 `ON`을 사용해야 한다.**