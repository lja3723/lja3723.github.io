---
title: "[SQL 고급] 부속질의"
date: 2024-04-21 15:29:00 +0900
categories: [Computer-Science, DB]
tags: ['DB', 'SQL']
---



## 1. 부속질의란?

- 하나의 SQL 문 안에 **다른 SQL 문이 중첩**된(nested) 질의
- 다른 테이블에서 가져온 데이터로 현재 테이블에 있는 정보를 찾거나 가공할 때 사용
- 보통 데이터가 대량일 때 데이터를 모두 합쳐서 연산하는 조인보다 필요한 데이터만 찾아서 공급해주는 부속질의가 성능이 더 좋음
- **주질의(main query, 외부질의)**와 **부속질의(sub query, 내부질의)**로 구성됨

{% assign img_path = "/assets/img/posts/DB/2024-04-29-sql-subquery" %}
![1. 부속질의란]({{ img_path }}/1. 부속질의란.png){: width='500'}

### 1.1. 부속질의의 종류 

| 명칭            | 위치      | 영문 및 동의어                      | 설명                                                                                  |
| --------------- | --------- | ----------------------------------- | ------------------------------------------------------------------------------------- |
| 스칼라 부속질의 | SELECT 절 | scalar subquery                     | SELECT 절에서 사용되며 단일 값을 반환하기 때문에 스칼라 부속질의라고 함.              |
| 인라인 뷰       | FROM 절   | inline view, table subquery         | FROM 절에서 결과를 뷰(view) 형태로 반환하기 때문에 인라인 뷰라고 함.                  |
| 중첩질의        | WHERE 절  | nested subquery, predicate subquery | WHERE 절에 술어와 같이 사용되며 결과를 한정시키기 위해 사용됨. 상관 혹은 비상관 형태. |



## 2. 스칼라 부속질의(scala subquery)

- SELECT 절에서 사용되는 부속질의로, 부속질의의 결과 값을 단일 행, 단일 열의 스칼라 값으로 반환함
- 원칙적으로 스칼라 값이 들어갈 수 있는 모든 곳에 사용 가능하며, 일반적으로 SELECT 문과 UPDATE SET 절에 사용됨
- 주질의와 부속질의와의 관계는 상관/비상관 모두 가능함

![2. 스칼라 부속질의]({{ img_path }}/2. 스칼라 부속질의.png){: width='500'}
_스칼라 부속질의_

---
**`질의 4-12`** 마당서점의 고객별 판매액을 보이시오(고객이름과 고객별 판매액을 출력)

<br> **정답**

```sql
SELECT    (SELECT name
          FROM    Customer cs
          WHERE   cs.custid = od.custid) 'name', SUM(saleprice) 'total'
FROM      Orders od
GROUP BY  od.custid;
```
{: .nolineno}

<br> **질의 결과**

| name   | total |
| ------ | ----- |
| 박지성 | 39000 |
| 김연아 | 15000 |
| 장미란 | 31000 |
| 추신수 | 33000 |

---

![2. 스칼라 부속질의 - 질의 4-12]({{ img_path }}/2. 스칼라 부속질의 - 질의 4-12.png){: width='500'}
_작동 방식_

<br>

**`질의 4-13`** Orders 테이블에 각 주문에 맞는 도서이름을 입력하시오.


<br> **정답**

```sql
UPDATE  Orders
SET     bookname = ( SELECT bookname
                     FROM   Book
                     WHERE  Book.bookid=Orders.bookid );
```
{: .nolineno}

<br> **질의 결과**

(삽입예정)

---



## 3. 인라인 뷰(inline view)

- FROM 절에서 사용되는 부속질의
- 테이블 이름 대신 인라인 뷰 부속질의를 사용하면 보통의 테이블과 같은 형태로 사용할 수 있음
- 부속질의 결과 반환되는 데이터는 다중 행, 다중 열이어도 상관없음
- 다만 가상의 테이블인 뷰 형태로 제공되어 상관 부속질의로 사용될 수는 없음

---
**`질의 4-14`** 고객번호가 2 이하인 고객의 판매액을 보이시오(고객이름과 고객별 판매액 출력).

<br> **정답**

```sql
SELECT    cs.name, SUM(od.saleprice) 'total'
FROM      /* 인라인 뷰 */(SELECT custid, name
          FROM Customer -- 이것을 가지고 상관질의를 사용하면 안 됨
          WHERE custid <= 2) cs,
          Orders od
WHERE     cs.custid = od.custid
GROUP BY  cs.name;
```
{: .nolineno}

<br> **질의 결과**

(삽입예정)

---

![3. 인라인 뷰]({{ img_path }}/3. 인라인 뷰.png){: width='500'}
_인라인 뷰_



## 4. 중첩질의

- 중첩질의(nested subquery) : WHERE 절에서 사용되는 부속질의
- WHERE 절은 보통 데이터를 선택하는 조건 혹은 술어(predicate)와 같이 사용됨 → 중첩질의를 술어 부속질의(predicate subquery)라고도 함

| 술어             | 연산자              | 반환 행 | 반환 열 | 상관 |
| ---------------- | ------------------- | ------- | ------- | ---- |
| 비교             | =, >, <, >=, <=, <> | 단일    | 단일    | 가능 |
| 집합             | IN, NOT IN          | 다중    | 단일    | 가능 |
| 한정(quantified) | ALL, SOME(ANY)      | 다중    | 단일    | 가능 |
| 존재             | EXISTS, NOT EXISTS  | 다중    | 다중    | 필수 |

한정자 -> 시험을 내고 싶었던 내용
- 한정자는 실무에서 정말 많이 쓰이는 내용임

### 4.1. 비교 연산자

- 부속질의가 반드시 단일 행, 단일 열을 반환해야 하며, 아닐 경우 질의를 처리할 수 없음

---
**`질의 4-15`** 평균 주문금액 이하의 주문에 대해서 주문번호와 금액을 보이시오.

<br> **정답**

```sql
SELECT  orderid, saleprice
FROM    Orders
WHERE   saleprice <= (SELECT AVG(saleprice) FROM Orders);
```
{: .nolineno}

<br> **질의 결과**



---

<br>

**`질의 4-16`** 각 고객의 평균 주문금액보다 큰 금액의 주문 내역에 대해서 주문번호, 고객번호, 금액을 보이시오.

<br> **정답**

```sql
SELECT  orderid, custid, saleprice
FROM    Orders od
WHERE   saleprice > (SELECT AVG(saleprice)
                    FROM    Orders so
                    WHERE   od.custid=so.custid);
```
{: .nolineno}

<br> **질의 결과**



---

### 4.2. IN, NOT IN

- IN 연산자는 주질의 속성 값이 부속질의에서 제공한 결과 집합에 있는지 확인하는 역할을 함
- IN 연산자는 부속질의의 결과 다중 행을 가질 수 있음
- 주질의는 WHERE 절에 사용되는 속성 값을 부속질의의 결과 집합과 비교해 하나라도 있으면 참이 됨
- NOT IN은 이와 반대로 값이 존재하지 않으면 참이 됨

---
**`질의4-17`** 대한민국에 거주하는 고객에게 판매한 도서의 총판매액을 구하시오.

<br> **정답**

```sql
SELECT  SUM(saleprice) 'total'
FROM    Orders
WHERE   custid IN (SELECT custid
                  FROM    Customer
                  WHERE   address LIKE '%대한민국%');
```
{: .nolineno}

<br> **질의 결과**
| total |
| ----- |
| 46000 |

---

### 4.3. ALL, SOME(ANY)

- ALL은 모두, SOME(ANY)은 어떠한(최소한 하나라도)이라는 의미
- 구문 구조

```sql
scalar_expression 
  { cmp_op( =, <>, !-, >, !>, <, <=, !< ) } 
  { ALL | SOME | ANY } (subquery)
```
{: .nolineno}

> **주의사항**
> 
> ANY 연산자는 일부 오래된 데이터베이스 시스템에서 지원하지 않을 수 있다.
{: .prompt-warning}

### 4.4. EXISTS, NOT EXISTS

- 데이터의 존재 유무를 확인하는 연산자
- 주질의에서 부속질의로 제공된 속성의 값을 가지고 부속질의에 조건을 만족하여 값이 존재하면 참이 되고, 주질의는 해당 행의 데이터를 출력함
- NOT EXIST의 경우 이와 반대로 동작함
- 구문 구조

```sql
WHERE [NOT] EXISTS (subquery)
```
{: .nolineno}