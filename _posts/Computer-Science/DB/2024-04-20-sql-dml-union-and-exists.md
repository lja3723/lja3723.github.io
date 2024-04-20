---
title: "[SQL] DML: UNION, EXISTS"
date: 2024-04-20 16:34:00 +0900
categories: [Computer-Science, DB]
tags: ['DB', 'SQL']
---



## 1. UNION

테이블은 튜플의 집합이므로 테이블 간 집합 연산(합집합, 차집합, 교집합)을 할 수 있다. 이 중 `UNION`은 합집합 연산이다.

<br>

---
**`질의 3-32`** 대한민국에서 거주하는 고객의 이름과 도서를 주문한 고객의 이름을 보이시오.

<br> **정답**

```sql
SELECT  name
FROM    Customer
WHERE   address LIKE '대한민국%'
UNION
SELECT  name
FROM    Customer
WHERE   custid IN (SELECT custid FROM Orders);
```
{: .nolineno}

<br> **질의 결과**

| name   |
| ------ |
| 김연아 |
| 김연경 |
| 박세리 |
| 박지성 |
| 추신수 |

- `UNION ALL`을 이용하면 중복을 포함한 결과를 얻을 수 있다.

```sql
SELECT  name
FROM    Customer
WHERE   address LIKE '대한민국%'
UNION ALL
SELECT  name
FROM    Customer
WHERE   custid IN (SELECT custid FROM Orders);
```
{: .nolineno}

<br> **질의 결과**

| name   |
| ------ |
| 김연아 |
| 김연경 |
| 박세리 |
| 박지성 |
| 김연아 |
| 김연경 |
| 추신수 |

---

### 1.1. 차집합, 교집합 구하기

- MySQL에서는 **차집합(`MINUS`), 교집합(`INTERSECT`)**을 구하는 연산자가 없다.
- 이런 연산이 필요한 경우 각각 `NOT IN`, `IN` 연산자를 이용하여 구현한다.

#### 1.1.1. MINUS 구현 

**`질의 3-32 변형 1`** 대한민국에서 거주하는 고객의 이름에서 도서를 주문한 고객의 이름을 제외하고 나타내시오.

<br> **정답**

- `NOT IN` 연산자를 이용한다.

```sql
SELECT  name
FROM    Customer
WHERE   address LIKE '대한민국%' AND 
        custid NOT IN ( -- 이처럼 NOT IN을 사용해 MINUS 연산자를 구현한다.
            SELECT  custid
            FROM    Customer
            WHERE   custid IN (SELECT custid FROM Orders));
```
{: .nolineno}

<br> **질의 결과**

| name   |
| ------ |
| 박세리 |

---

#### 1.1.2. INTERSECT 구현

**`질의 3-32 변형 2`** 대한민국에서 거주하는 고객이면서 도서를 주문한 고객을 나타내시오.

<br> **정답**

- `IN` 연산자를 이용한다.

```sql
SELECT  name
FROM    Customer
WHERE   address LIKE '대한민국%' AND 
        custid IN ( -- NOT IN에서 IN으로만 변경하면 된다.
            SELECT  custid
            FROM    Customer
            WHERE   custid IN (SELECT custid FROM Orders));
```
{: .nolineno}

<br> **질의 결과**

| name   |
| ------ |
| 김연아 |
| 김연경 |

---



## 2. EXISTS

- `EXISTS` 연산자 다음에는 부속질의문만 올 수 있으며, 부속질의문이 반환하는 **결과 테이블의 튜플이 존재하는 경우** true를 반환한다. 주로 `WHERE` 절에서 사용한다.
  - 다시 말해 부속질의문의 어떤 행 중 하나라도 조건에 만족하면 true가 된다.
  - 반면 `NOT EXISTS `는 부속질의문의 **모든 행**이 **조건에 만족하지 않아야지만** true가 된다.

- 부속질의문을 실행하던 중 튜플이 하나라도 검색되면 연산을 종료하기 때문에(지연 연산), 일반적으로 **성능이 좋다.**
  - 부속질의문의 `SELECT` 절까지 가지 않고 `WHERE` 에서 조건을 만족하는 경우 바로 결과를 도출하기 때문이다.

- 일반적으로 `EXISTS`에서 사용되는 부속질의문은 상관 부속질의문이다.

<br>

---
**`질의 3-33`** 주문이 있는 고객의 이름과 주소를 나타내시오.

<br> **정답**

```sql
SELECT  name, address
FROM    Customer
WHERE   EXISTS(
    SELECT  *
    FROM    Orders
    WHERE   Orders.custid = Customer.custid);
```
{: .nolineno}

<br> **질의 결과**

| name   | address         |
| ------ | --------------- |
| 박지성 | 영국 맨체스타   |
| 김연아 | 대한민국 서울   |
| 김연경 | 대한민국 경기도 |
| 추신수 | 미국 클리블랜드 |

---

{% assign img_path = "/assets/img/posts/DB/2024-04-20-sql-dml-union-and-exists" %}
![2. EXISTS]({{ img_path }}/2. EXISTS.png){: width='650'}
_EXISTS 연산자의 실행 예시_