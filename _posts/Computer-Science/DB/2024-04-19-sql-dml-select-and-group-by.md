---
title: "[SQL] DML: SELECT, GROUP BY"
date: 2024-04-19 14:09:00 +0900
categories: [Computer-Science, DB]
tags: ['DB', 'SQL']
---



## 1. SELECT 문

### 1.1. 구성요소

{% assign img_path = "/assets/img/posts/DB/2024-04-19-sql-dml-select-and-group-by" %}
![1.1. SELECT문 구성요소]({{ img_path }}/1.1. SELECT문 구성요소.png){: width='500'}
_SELECT문의 구성요소_

### 1.2. 기본 문법

- `<>`: SQL 예약어가 아닌 부분, 속성명이나 조건식 등이 올 수 있음
- `[]`: 대괄호 안의 SQL 예약어들은 선택적으로 사용할 수 있음
- `|`: 선택 가능한 문법들 중 한 개를 사용할 수 있음
- attr.: attribute(속성)


```text
SELECT    [ ALL | DISTINCT ] <attr. name(s)>
FROM      <table name(s)>
[ WHERE	  <search condition(s)> ]
[ GROUP BY    <attr. name> ]
[ HAVING      <search condition(s)> ]
[ ORDER BY    <attr. name> [ ASC | DESC ] ]
```
{: file='SELECT문 기본 문법'}



## 2. SELECT, FROM

**`질의 3-1`** 모든 도서의 이름과 가격을 검색하시오.

<br> **정답**

```sql
SELECT  bookname, price
FROM    Book; 
```
{: .nolineno}

<br> **질의 결과**

| bookname          | price |
| ----------------- | ----- |
| 축구의 역사       | 7000  |
| 축구 아는 여자    | 13000 |
| 축구의 이해       | 22000 |
| 골프 바이블       | 35000 |
| 피겨 교본         | 8000  |
| 배구 단계별기술   | 6000  |
| 야구의 추억       | 20000 |
| 야구를 부탁해     | 13000 |
| 올림픽 이야기     | 7500  |
| Olympic Champions | 13000 |

---

<br>

**`질의 3-1 변형`** 모든 도서의 이름과 가격을 검색하시오.

<br> **정답**

```sql
SELECT  price, bookname
FROM    Book; 
```
{: .nolineno}

<br> **질의 결과**

| price | bookname          |
| ----- | ----------------- |
| 7000  | 축구의 역사       |
| 13000 | 축구 아는 여자    |
| 22000 | 축구의 이해       |
| 35000 | 골프 바이블       |
| 8000  | 피겨 교본         |
| 6000  | 배구 단계별기술   |
| 20000 | 야구의 추억       |
| 13000 | 야구를 부탁해     |
| 7500  | 올림픽 이야기     |
| 13000 | Olympic Champions |

---

<br>

**`질의 3-2`** 모든 도서의 도서번호, 도서이름, 출판사, 가격을 검색하시오.

<br> **정답**

```sql
SELECT  bookid, bookname, publisher, price
FROM    Book;
```
{: .nolineno}

또는

```sql
SELECT  *
FROM    Book;
```
{: .nolineno}

---

<br>

**`질의 3-3`** 도서 테이블에 있는 모든 출판사를 검색하시오.

<br> **정답**

```sql
SELECT  publisher
FROM    Book;
```
{: .nolineno}

<br> **질의 결과**

| publisher  |
| ---------- |
| 굿스포츠   |
| 나무수     |
| 대한미디어 |
| 대한미디어 |
| 굿스포츠   |
| 굿스포츠   |
| 이상미디어 |
| 이상미디어 |
| 삼성당     |
| Pearson    |

---

### 2.1. DISTINCT

중복 제거를 위해서는 `DISTINCT` 예약어를 사용한다. **위치에 주의**하자.

<br>

---
**`질의 3-3 변형`** 도서 테이블에 있는 모든 출판사를 검색하시오. 단 중복은 제거한다.

<br> **정답**

```sql
SELECT  DISTINCT publisher
FROM    Book;
```
{: .nolineno}

<br> **질의 결과**

| publisher  |
| ---------- |
| 굿스포츠   |
| 나무수     |
| 대한미디어 |
| 이상미디어 |
| 삼성당     |
| Pearson    |

---



## 3. WHERE

`WHERE` 예약어로 조건을 부여할 수 있다. Filter의 효과가 있다.

| 술어     | 연산자                     | 예                                                |
| -------- | -------------------------- | ------------------------------------------------- |
| 비교     | `=` `<>` `<` `<=` `>` `>=` | price < 20000                                     |
| 범위     | `BETWEEN`                  | price BETWEEN 10000 AND 20000                     |
| 집합     | `IN` `NOT IN`              | price IN (10000, 20000, 30000)                    |
| 패턴     | `LIKE`                     | bookname LIKE '축구의 역사'                       |
| NULL     | `IS NULL` `IS NOT NULL`    | price IS NULL                                     |
| 복합조건 | `AND` `OR` `NOT`           | (price < 20000) AND (bookname LIKE '축구의 역사') |

<br>

---
**`질의 3-4`** 가격이 20,000원 미만인 도서를 검색하시오.

<br> **정답**

```sql
SELECT  *
FROM    Book
WHERE   price < 20000;
```
{: .nolineno}

---

### 3.1. BETWEEN

어떤 속성 값이 연속된 범위 안에 있는 지 확인하고 싶을 때 `BETWEEN` 연산자를 사용할 수 있다. 논리 연산자 `AND`와 함께 사용한다.

- **기본 구문**
  - \<attribute> BETWEEN value1 AND value2

특정 범위에 속하지 않는 조건을 확인할 때, 즉 부정 연산자를 사용하려면 `NOT`을 사용한다.

- **구문**
  - \<attribute> NOT BETWEEN value1 AND value2
  - **또는** NOT \<attribute> BETWEEN value1 AND value2

<br>

---
**`질의 3-5`** 가격이 10,000원 이상 20,000원 이하인 도서를 검색하시오.

<br> **정답**

```sql
SELECT  *
FROM    Book
WHERE   price BETWEEN 10000 AND 20000;
```
{: .nolineno}

또는

```sql
SELECT  *
FROM    Book
WHERE   price >= 10000 AND price <= 20000;
```
{: .nolineno}

<br> **질의 결과**

| bookid | bookname          | publisher  | price |
| ------ | ----------------- | ---------- | ----- |
| 2      | 축구 아는 여자    | 나무수     | 13000 |
| 7      | 야구의 추억       | 이상미디어 | 20000 |
| 8      | 야구를 부탁해     | 이상미디어 | 13000 |
| 10     | Olympic Champions | Pearson    | 13000 |

---

### 3.2. IN, NOT IN

- `IN` 연산자는 집합의 원소인지 판단하는 연산자로, 두 개 이상의 값을 비교할 때 사용할 수 있다. 
- **집합에 속한 원소들 중 하나와 일치**하면 true를 반환한다.
- 반면, `NOT IN` 연산자는 **집합에 속한 모든 원소가 아니어야만** true를 반환한다.

---
<br>

**`질의 3-6`** 출판사가 '굿스포츠' 혹은 '대한미디어'인 도서를 검색하시오.

<br> **정답**

```sql
SELECT  *
FROM    Book
WHERE   publisher IN ('굿스포츠', '대한미디어');
```
{: .nolineno}

<br> **질의 결과**

| bookid | bookname        | publisher  | price |
| ------ | --------------- | ---------- | ----- |
| 1      | 축구의 역사     | 굿스포츠   | 7000  |
| 3      | 축구의 이해     | 대한미디어 | 22000 |
| 4      | 골프 바이블     | 대한미디어 | 35000 |
| 5      | 피겨 교본       | 굿스포츠   | 8000  |
| 6      | 배구 단계별기술 | 굿스포츠   | 6000  |

---

<br>

**`질의 3-6 변형`** 출판사가 '굿스포츠' 혹은 '대한미디어'가 아닌 도서를 검색하시오.

<br> **정답**

```sql
SELECT  *
FROM    Book
WHERE   publisher NOT IN ('굿스포츠', '대한미디어');
```
{: .nolineno}

<br> **질의 결과**

| bookid | bookname          | publisher  | price |
| ------ | ----------------- | ---------- | ----- |
| 2      | 축구 아는 여자    | 나무수     | 13000 |
| 7      | 야구의 추억       | 이상미디어 | 20000 |
| 8      | 야구를 부탁해     | 이상미디어 | 13000 |
| 9      | 올림픽 이야기     | 삼성당     | 7500  |
| 10     | Olympic Champions | Pearson    | 13000 |

---

### 3.3. LIKE

- `LIKE` 연산자를 사용하면 속성의 값이 주어진 정규식(regex)을 만족할 때만 true를 반환한다.
- 찾는 속성이 **텍스트** 또는 **날짜 데이터**를 포함한다면 비교할 값을 **작은 따옴표**`''`로 감싸야 한다.

다음은 문자열 검색시 `LIKE`와 같이 사용할 수 있는 **와일드 문자**이다.

| 와일드 문자 | 의미                          | 사용 예시                                       |
| ----------- | ----------------------------- | ----------------------------------------------- |
| +[^plus]    | 문자열을 연결                 | '골프' + '바이블'                               |
| %           | 0개 이상의 문자열과 일치      | '%축구%': 축구를 포함하는 문자열                |
| []          | 1개의 문자와 일치             | '[0-5]%': 0~5 사이 숫자로 시작하는 문자열       |
| [^]         | 1개의 문자와 불일치           | '[^0-5]%': 0~5 사이 숫자로 시작하지 않는 문자열 |
| _           | 특정 위치의 1개의 문자와 일치 | '_구%': 두 번째 위치에 '구'가 들어가는 문자열   |

[^plus]: 보통 SELECT 구문에서 많이 사용된다.

<br>

---
**`질의 3-7`** '축구의 역사'를 출간한 출판사를 검색하시오.

<br> **정답**

```sql
SELECT  bookname, publisher
FROM    Book
WHERE   bookname LIKE '축구의 역사';
```
{: .nolineno}

<br> **질의 결과**

| bookname    | publisher |
| ----------- | --------- |
| 축구의 역사 | 굿스포츠  |

---

<br>

**`질의 3-8`** 도서이름에 '축구'가 포함된 출판사를 검색하시오.

<br> **정답**

```sql
SELECT  bookname, publisher
FROM    Book
WHERE   bookname LIKE '%축구%';
```
{: .nolineno}

<br> **질의 결과**

| bookname       | publisher  |
| -------------- | ---------- |
| 축구의 역사    | 굿스포츠   |
| 축구 아는 여자 | 나무수     |
| 축구의 이해    | 대한미디어 |

---

<br>

**`질의 3-9`** 도서이름의 왼쪽 두 번째 위치에 '구'라는 문자열을 갖는 도서를 검색하시오.

<br> **정답**

```sql
SELECT  *
FROM    Book
WHERE   bookname LIKE '_구%';
```
{: .nolineno}

<br> **질의 결과**

| bookid | bookname        | publisher  | price |
| ------ | --------------- | ---------- | ----- |
| 1      | 축구의 역사     | 굿스포츠   | 7000  |
| 2      | 축구 아는 여자  | 나무수     | 13000 |
| 3      | 축구의 이해     | 대한미디어 | 22000 |
| 6      | 배구 단계별기술 | 굿스포츠   | 6000  |
| 7      | 야구의 추억     | 이상미디어 | 20000 |
| 8      | 야구를 부탁해   | 이상미디어 | 13000 |

---

### 3.4. 복합조건

`AND`, `OR`, `NOT` 논리연산자를 사용해 복합 조건을 명시할 수 있다.

<br>

---
**`질의 3-10`** 축구에 관한 도서 중 가격이 20,000원 이상인 도서를 검색하시오.

<br> **정답**

```sql
SELECT  *
FROM    Book
WHERE   bookname LIKE '%축구%' AND price >= 20000;
```
{: .nolineno}

<br> **질의 결과**

| bookid | bookname    | publisher  | price |
| ------ | ----------- | ---------- | ----- |
| 3      | 축구의 이해 | 대한미디어 | 22000 |

---

<br>

**`질의 3-11`** 출판사가 '굿스포츠' 혹은 '대한미디어'인 도서를 검색하시오.

<br> **정답**

```sql
SELECT  *
FROM    Book
WHERE   publisher = '굿스포츠' OR publisher = '대한미디어';
```
{: .nolineno}

---



## 4. ORDER BY

- 특정 속성을 기준으로 특정 조건으로 정렬하여 출력하고 싶을 때 `ORDER BY` 절을 사용한다.
- 만약 특정 속성에 대해 정렬한 후 **또 다른 속성으로 다시 정렬**하고자 할 때, 원하는 열 이름을 순서대로 지정하면 된다.

<br>

---
**`질의 3-12`** 도서를 이름순으로 검색하시오.

<br> **정답**

```sql
SELECT    *
FROM      Book
ORDER BY  bookname;
```
{: .nolineno}

<br> **질의 결과**

| bookid | bookname          | publisher  | price |
| ------ | ----------------- | ---------- | ----- |
| 10     | Olympic Champions | Pearson    | 13000 |
| 4      | 골프 바이블       | 대한미디어 | 35000 |
| 6      | 배구 단계별기술   | 굿스포츠   | 6000  |
| 8      | 야구를 부탁해     | 이상미디어 | 13000 |
| 7      | 야구의 추억       | 이상미디어 | 20000 |
| 9      | 올림픽 이야기     | 삼성당     | 7500  |
| 2      | 축구 아는 여자    | 나무수     | 13000 |
| 1      | 축구의 역사       | 굿스포츠   | 7000  |
| 3      | 축구의 이해       | 대한미디어 | 22000 |
| 5      | 피겨 교본         | 굿스포츠   | 8000  |

---

<br>

**`질의 3-13`** 도서를 가격순으로 검색하고, 가격이 같으면 이름순으로 검색하시오.

<br> **정답**

```sql
SELECT    *
FROM      Book
ORDER BY  price, bookname;
```
{: .nolineno}

<br> **질의 결과**

| bookid | bookname          | publisher  | price |
| ------ | ----------------- | ---------- | ----- |
| 6      | 배구 단계별기술   | 굿스포츠   | 6000  |
| 1      | 축구의 역사       | 굿스포츠   | 7000  |
| 9      | 올림픽 이야기     | 삼성당     | 7500  |
| 5      | 피겨 교본         | 굿스포츠   | 8000  |
| 10     | Olympic Champions | Pearson    | 13000 |
| 8      | 야구를 부탁해     | 이상미디어 | 13000 |
| 2      | 축구 아는 여자    | 나무수     | 13000 |
| 7      | 야구의 추억       | 이상미디어 | 20000 |
| 3      | 축구의 이해       | 대한미디어 | 22000 |
| 4      | 골프 바이블       | 대한미디어 | 35000 |

---

### 4.1. ASC, DESC

- 오름차순, 내림차순으로 정렬하고 싶으면 `ASC`, `DESC`를 사용한다.
- 사용 위치를 주의하자.
- 키워드를 생략할 경우 기본값은 `ASC`(오름차순)이다.

<br>

---
**`질의 3-14`** 도서를 가격의 내림차순으로 정렬하고, 가격이 같다면 출판사의 오름차순으로 검색한다.

<br> **정답**


```sql
SELECT    *
FROM      Book
ORDER BY  price DESC, publisher ASC;
```
{: .nolineno}

<br> **질의 결과**

| bookid | bookname          | publisher  | price |
| ------ | ----------------- | ---------- | ----- |
| 4      | 골프 바이블       | 대한미디어 | 35000 |
| 3      | 축구의 이해       | 대한미디어 | 22000 |
| 7      | 야구의 추억       | 이상미디어 | 20000 |
| 10     | Olympic Champions | Pearson    | 13000 |
| 2      | 축구 아는 여자    | 나무수     | 13000 |
| 8      | 야구를 부탁해     | 이상미디어 | 13000 |
| 5      | 피겨 교본         | 굿스포츠   | 8000  |
| 9      | 올림픽 이야기     | 삼성당     | 7500  |
| 1      | 축구의 역사       | 굿스포츠   | 7000  |
| 6      | 배구 단계별기술   | 굿스포츠   | 6000  |

---



## 5. AS: Alias - 별칭 붙이기

SQL에서는 테이블이나 어떤 속성(칼럼)명에 새로운 이름을 붙일 수 있는데, 이를 Alias라고 한다.

### 5.1. 속성명에 별칭 붙이기

속성에 별칭을 붙이는 방법은 다음과 같이 여러 방법을 사용할 수 있다.

```sql
SELECT bookname AS 도서명  -- bookname 대신 '도서명'이라는 별칭을 붙일 수 있다.
                           -- 따옴표를 생략할 수 있다.
SELECT bookname 도서명     -- AS를 이렇게 생략할 수 있다.
SELECT bookname '도서명'   -- 작은따옴표로 별칭을 감쌀 수 있다.
SELECT bookname "도서명"   -- 큰따옴표로도 별칭을 감쌀 수 있다.
SELECT bookname "도서 이름"   -- 별칭에 공백이 포함되면 큰(작은)따옴표는 필수이다.
```
{: .nolineno}

### 5.2. 테이블명에 별칭 붙이기

테이블에 별칭을 붙이는 방법도 속성처럼 여러가지가 있다. 다음은 테이블에 별명을 붙이는 예시이다. (`INNER JOIN` 구문은 추후 설명함)

```sql
SELECT  *
FROM    TB_BOARD AS T1 -- AS 생략 가능
        INNER JOIN TB_BOARD_COMMENT AS T2 -- AS 생략 가능
        ON T1.BBS_SEQ = T2.BBS_SEQ;
```
{: .nolineno}

만약 별명을 사용하지 않으면 아래와 같다.

```sql
SELECT  *
FROM    TB_BOARD
        INNER JOIN TB_BOARD_COMMENT
        ON TB_BOARD.BBS_SEQ = TB_BOARD_COMMENT.BBS_SEQ; -- SQL이 너무 길어진다.
```
{: .nolineno}



## 6. 집계 함수

**집계 함수(aggregate function)**는 테이블의 각 열에 대해 계산하여 하나의 **통계(statistics)**를 출력하는 함수이다.

> **집계(aggregate)**와 **통계(statistics)**는 서로 비슷한 의미로, 데이터베이스에서는 통계보다 집계라는 용어를 사용한다.
{: .prompt-tip}

집계 함수의 종류를 표로 요약하면 아래와 같다.

| 집계 함수 | 문법                                              | 사용 예시              |
| --------- | ------------------------------------------------- | ---------------------- |
| `SUM`     | SUM([ ALL \| DISTINCT ] \<attr. name>)            | SUM(price)             |
| `AVG`     | AVG([ ALL \| DISTINCT ] \<attr. name>)            | AVG(price)             |
| `COUNT`   | COUNT([ [ ALL \| DISTINCT ] \<attr. name> \| * ]) | COUNT(price), COUNT(*) |
| `MAX`     | MAX([ ALL \| DISTINCT ] \<attr. name>)            | MAX(price)             |
| `MIN`     | MIN([ ALL \| DISTINCT ] \<attr. name>)            | MIN(price)             |

- `<>`: SQL 예약어가 아닌 부분, 속성명이나 조건식 등이 올 수 있음
- `[]`: 선택적으로 사용할 수 있음
- `|`: 선택 가능한 것들 중 한 개를 사용할 수 있음
- attr.: attribute(속성)

<br>

---
**`질의 3-15`** 고객이 주문한 도서의 총 판매액을 구하시오.

<br> **정답**

```sql
SELECT  SUM(saleprice)
FROM    Orders;
```
{: .nolineno}

<br> **질의 결과**

| SUM(saleprice) |
| -------------- |
| 118000         |

- SQL 결과는 항상 테이블이므로, 위 질의 결과는 한개의 속성, 한개의 튜플로 구성된 테이블임

---

> 속성명에 아래처럼 별칭을 지어주어 더 의미 있는 결과를 출력할 수 있다.
{: .prompt-info}

```sql
SELECT  SUM(saleprice) AS '총매출' -- AS는 생략 가능 (최신 버전에서 가능)
FROM    Orders;
```
{: .nolineno}

<br> **질의 결과**

| 총매출 |
| ------ |
| 118000 |

---

<br>

**`질의 3-16`** 2번 김연아 고객이 주문한 도서의 총 판매액을 구하시오.

<br> **정답**

```sql
SELECT  SUM(saleprice) AS '총매출'
FROM    Orders
WHERE   custid = 2;
```
{: .nolineno}

<br> **질의 결과**

| 총매출 |
| ------ |
| 15000  |

---

<br>

**`질의 3-17`** 고객이 주문한 도서의 총 판매액, 평균값, 최저가, 최고가를 구하시오.

<br> **정답**

```sql
SELECT  SUM(saleprice) AS Total,
        AVG(saleprice) AS Average,
        MIN(saleprice) AS Minimum,
        MAX(saleprice) AS Maximum
FROM    Orders;
```
{: .nolineno}

<br> **질의 결과**

| Total  | Average | Minimum | Maximum |
| ------ | ------- | ------- | ------- |
| 118000 | 11800   | 6000    | 21000   |

---

### 6.1. COUNT

- 행의 개수를 세는 집계 함수이다.
- 파라미터로 `*` 혹은 특정 속성의 이름이 사용된다.
- `NULL` 값을 제외한 해당 속성의 튜플 개수를 세어준다.
- **예시**
  - `COUNT(*)`: 전체 튜플의 수를 세어준다.
  - `COUNT(publisher)`: 출판사의 수를 세어준다.
  - `COUNT(DISTINCT publisher)`: 중복을 제거한 출판사의 수를 세어준다. (`NULL` 값 제외)

<br>

---
**`질의 3-18`** 마당서점의 도서 판매 건수를 구하시오.

<br> **정답**

```sql
SELECT  COUNT(*)
FROM    Orders;
```
{: .nolineno}

<br> **질의 결과**

| COUNT(*) |
| -------- |
| 10       |

---



## 7. GROUP BY

- `GROUP BY` 절을 사용하면 속성값이 같은 값끼리 그룹을 만들 수 있다.

<br>

---
**`질의 3-19`** 고객별로 주문한 도서의 총 수량과 총 판매액을 구하시오.

<br> **정답**

```sql
SELECT    custid, COUNT(*) AS '도서수량', SUM(saleprice) AS '총액'
FROM      Orders
GROUP BY  custid; -- 묶는 순간 GROUP BY로 묶지 않은 다른 속성명은 소실된다.
```
{: .nolineno}

<br> **질의 결과**

| custid | 도서수량 | 총액  |
| ------ | -------- | ----- |
| 1      | 3        | 39000 |
| 2      | 2        | 15000 |
| 3      | 3        | 31000 |
| 4      | 2        | 33000 |

![7. GROUP BY]({{ img_path }}/7. GROUP BY.png){: width='500'}
_GROUP BY 절의 수행_

---

### 7.1. HAVING

- `HAVING` 절은 `GROUP BY` 절의 결과에 나타나는 그룹을 제한하는 역할을 한다.
  - 다른 표현으로, `GROUP BY` 절로 그룹화된 결과에 조건을 적용하는 데 사용된다.
- 반드시 `GROUP BY` 절과 같이 등장해야 한다.
- 반드시 `WHERE` 절 이후에 등장해야 한다.
- 'HAVING' 절의 조건은 반드시 **집계 함수**로 명시되어야 한다.

`WHERE` 절과의 차이점은 아래와 같다.

- `WHERE`: **개별 행에 대한 조건**을 지정하며, 일반적으로 `SELECT - FROM` 다음에 위치함
- `HAVING`: `GROUP BY`로 그룹화된 결과에 대한 조건을 지정하며, 일반적으로 `GROUP BY` 다음에 위치함

<br>

---
**`질의 3-20`** 가격이 8,000원 이상인 도서를 구매한 고객에 대하여 고객별 주문 도서의 총 수량을 구하시오. 단, 두 권 이상 구매한 고객만 구한다.


<br> **정답**

```sql
SELECT    custid, COUNT(*) AS '도서수량'
FROM      Orders
WHERE     saleprice >= 8000
GROUP BY  custid
HAVING    COUNT(*) >= 2; -- 도서수량 >= 2; 도 가능하다. ('도서수량' >= 2; 는 불가능)
```
{: .nolineno}

<br> **질의 결과**

| custid | 도서수량 |
| ------ | -------- |
| 1      | 2        |
| 3      | 2        |
| 4      | 2        |

---

- HAVING 절이 없는 경우

```sql
SELECT    custid, COUNT(*) AS '도서수량'
FROM      Orders
WHERE     saleprice >= 8000
GROUP BY  custid;
```
{: .nolineno}

<br> **질의 결과**

| custid | 도서수량 |
| ------ | -------- |
| 1      | 2        |
| 2      | 1        |
| 3      | 2        |
| 4      | 2        |

### 7.2. GROUP BY 주의사항

> `GROUP BY`를 사용해 튜플을 그룹으로 묶을 경우, `SELECT` 절에서는 `GROUP BY`에서 **사용한 속성**과 **집계 함수**만 나올 수 있다.
{: .prompt-danger}

- **맞는 예시**

```sql
SELECT    custid, SUM(saleprice)
FROM      Orders
GROUP BY  custid;
```
{: .nolineno}

- **틀린 예시 1**
  - `SELECT` 절에 `GROUP BY`에서 사용되지 않은 속성명은 올 수 없다.

```sql
SELECT    bookid, SUM(saleprice)  -- bookid는 등장 불가능
FROM      Orders
GROUP BY  custid;
```
{: .nolineno}

- **틀린 예시 2**
  - SELECT 문에 **집계 함수**와 **속성명**을 같이 사용한 경우, 반드시 `GROUP BY` 절이 등장해야 한다.

```sql
SELECT    custid, SUM(saleprice)
FROM      Orders; -- GROUP BY가 반드시 등장해야 함
```
{: .nolineno}

### 7.3. HAVING 주의사항

> `WHERE` 절과 `HAVING` 절이 같이 포함된 SQL 문은 검색 조건이 모호해질 수 있다. `HAVING` 절은 **반드시 다음 조건을 준수해야 한다.**
> 1. `GROUP BY` 절과 함께 작성한다.
> 2. `WHERE` 절보다 뒤에 와야 한다.
> 3. 검색 조건에는 `SUM`, `AVG`, `COUNT` 등과 같은 **집계 함수**가 와야 한다.
{: .prompt-danger}

- **맞는 예시**

```sql
SELECT    custid, COUNT(*) AS '도서수량'
FROM      Orders
WHERE     saleprice >= 8000
GROUP BY  custid
HAVING    COUNT(*) >= 2;
```
{: .nolineno}

- **틀린 예시**

```sql
SELECT    custid, COUNT(*) AS '도서수량'
FROM      Orders
HAVING    COUNT(*) >= 2;  -- 순서가 틀렸다.
WHERE     saleprice >= 8000
GROUP BY  custid
```
{: .nolineno}



## 8. SQL 문의 실행 순서

SQL은 순서가 없는 비절차적인 언어지만 내부적으로는 **실행 순서**가 있다. `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`가 포함된 다음 `SELECT` 문의 실행 순서를 알아보자.

```sql
SELECT    custid, COUNT(*) AS '도서수량'  # (5) - 실행 순서
FROM      Orders                          # (1)
WHERE     saleprice >= 8000               # (2)
GROUP BY  custid                          # (3)
HAVING    COUNT(*) > 1                    # (4)
ORDER BY  custid;                         # (6)
```
{: .nolineno}

### 8.1. FROM Orders

| orderid | custid | bookid | saleprice | orderdate  |
| ------- | ------ | ------ | --------- | ---------- |
| 1       | 1      | 1      | 6000      | 2024-07-01 |
| 2       | 1      | 3      | 21000     | 2024-07-03 |
| 3       | 2      | 5      | 8000      | 2024-07-03 |
| 4       | 3      | 6      | 6000      | 2024-07-04 |
| 5       | 4      | 7      | 20000     | 2024-07-05 |
| 6       | 1      | 2      | 12000     | 2024-07-07 |
| 7       | 4      | 8      | 13000     | 2024-07-07 |
| 8       | 3      | 10     | 12000     | 2024-07-08 |
| 9       | 2      | 10     | 7000      | 2024-07-09 |
| 10      | 3      | 8      | 13000     | 2024-07-10 |

### 8.2. WHERE saleprice >= 8000

| orderid | custid | bookid | saleprice | orderdate  |
| ------- | ------ | ------ | --------- | ---------- |
| 2       | 1      | 3      | 21000     | 2024-07-03 |
| 3       | 2      | 5      | 8000      | 2024-07-03 |
| 5       | 4      | 7      | 20000     | 2024-07-05 |
| 6       | 1      | 2      | 12000     | 2024-07-07 |
| 7       | 4      | 8      | 13000     | 2024-07-07 |
| 8       | 3      | 10     | 12000     | 2024-07-08 |
| 10      | 3      | 8      | 13000     | 2024-07-10 |

### 8.3. GROUP BY custid

| custid | COUNT(*) |
| ------ | -------- |
| 1      | 2        |
| 2      | 1        |
| 4      | 2        |
| 3      | 2        |

### 8.4. HAVING COUNT(*) > 1

| custid | COUNT(*) |
| ------ | -------- |
| 1      | 2        |
| 4      | 2        |
| 3      | 2        |

### 8.5. SELECT custid, COUNT(*) AS '도서수량'

| custid | 도서수량 |
| ------ | -------- |
| 1      | 2        |
| 4      | 2        |
| 3      | 2        |

### 8.6. ORDER BY custid

| custid | 도서수량 |
| ------ | -------- |
| 1      | 2        |
| 3      | 2        |
| 4      | 2        |



<br><br><br><br>

---
#### 8.6.1. 각주
{: data-toc-skip=''}