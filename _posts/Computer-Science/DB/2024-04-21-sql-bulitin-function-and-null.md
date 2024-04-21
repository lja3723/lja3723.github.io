---
title: "[SQL 고급] 내장 함수, NULL 처리, 행번호 출력"
date: 2024-04-21 11:20:00 +0900
categories: [Computer-Science, DB]
tags: ['DB', 'SQL']
---



## 1. SQL 함수

SQL 함수란 수학이나 다른 프로그래밍 언어에 존재하는 그것으로, 특정 값이나 열의 값을 입력받은 뒤 결과를 돌려주는 SQL의 기능이다.

- SQL의 함수는 크게 두 가지로 나뉜다.
  - **내장 함수(built-int function)**: DBMS가 기본으로 제공하는 함수
  - **사용자 정의 함수(user-defined function)**: 사용자가 필요에 따라 직접 만드는 함수



## 2. 내장 함수

- 상수나 속성 이름을 입력값으로 받아 단일 값을 결과로 반환한다.
- 각 함수가 기대하는 **입력 형식**이 있다.
  - 예기치 못한 형식의 값이 입력되면 DBMS는 질의문 실행을 중지하고 에러 메시지를 출력한다.
- `SELECT`, `WHERE`, `UPDATE` 등 SQL 문 어디에서나 사용 가능하다.

### 2.1. 종류

<table>
<colgroup>
<col style="text-align:center;">
<col style="text-align:center;">
<col>
</colgroup>
<thead>
  <tr>
    <th colspan="2">구분</th>
    <th>함수</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td rowspan="7">단일행<br>함수</td>
    <td>숫자 함수</td>
    <td>ABS, CEIL, COS, EXP, FLOOR, LN, LOG, MOD, POWER, RAND, <br>ROUND, SIGN, TRUNCATE </td>
  </tr>
  <tr>
    <td>문자 함수<br>(문자 반환)</td>
    <td>CHAR, CONCAT, LEFT, RIGHT, LOWER, UPPER, LPAD, RPAD, <br>LTRIM, RTRIM, REPLACE, REVERSE, RIGHT, SUBSTR, TRIM</td>
  </tr>
  <tr>
    <td>문자 함수<br>(숫자 반환)</td>
    <td>ASCII, INSTR, LENGTH</td>
  </tr>
  <tr>
    <td>날짜/시간 함수</td>
    <td>ADDDATE, CURRENT_DATE, DATE, DATEDIFF, DAYNAME, LAST_DAY, <br>SYSDATE, TIME </td>
  </tr>
  <tr>
    <td>변환 함수</td>
    <td>CAST, CONVERT, DATE_FORMAT, STR_TO_DATE</td>
  </tr>
  <tr>
    <td>정보 함수</td>
    <td>DATABASE, SCHEMA, ROW_COUNR, USER, VERSION</td>
  </tr>
  <tr>
    <td>NULL 관련 함수</td>
    <td>COALESCE, ISNULL, IFNULL, NULLIF</td>
  </tr>
  <tr>
    <td colspan="2">집계 함수</td>
    <td>AVG, COUNT, MAX, MIN, STD, STDDEV, SUM</td>
  </tr>
  <tr>
    <td colspan="2">윈도우 함수<br>(혹은 분석 함수)</td>
    <td>CUME_DIST, DENSE_RANK, FIRST_VALUE, LAST_VALUE, LEAD, <br>NTILE, RANK, ROW_NUMBER</td>
  </tr>
</tbody>
</table>

### 2.2. 숫자 함수

| 함수             | 설명                                  | 예시                  |
| ---------------- | ------------------------------------- | --------------------- |
| `ABS(숫자)`      | 숫자의 절댓값을 계산                  | ABS(-4.5) → 4.5       |
| `CEIL(숫자)`     | 숫자보다 크거나 같은 최소의 정수      | CEIL(4.1) → 5         |
| `FLOOR(숫자)`    | 숫자보다 작거나 같은 최소의 정수      | FLOOR(4.1) → 4        |
| `ROUND(숫자, m)` | 숫자의 반올림, m은 반올림 기준 자릿수 | ROUND(5.36, 1) → 5.40 |
| `LOG(n, 숫자)`   | 숫자의 자연로그 값을 반환             | LOG(10) → 2.30259     |
| `POWER(숫자, n)` | 숫자의 n제곱 값을 계산                | POWER(2, 3) → 8       |
| `SQRT(숫자)`     | 숫자의 제곱근 값을 계산(숫자는 양수)  | SQRT(9.0) → 3.0       |
| `SIGN(숫자)`     | 숫자가 음수면 -1, 0이면 0, 양수면 1   | SIGN(3.45) → 1        |

<br>

---
**`질의 4-1`** $-78$과 $+78$의 절댓값을 구하시오.

<br> **정답**

```sql
SELECT  ABS(-78), ABS(+78)
FROM    DUAL; -- FROM절이 필수인 벤더(오라클)에서 사용하는 가상의 테이블
              -- FROM 절이 필수가 아니면 쓸 필요 없다.
```
{: .nolineno}

<br> **질의 결과**

| ABS(-78) | ABS(+78) |
| -------- | -------- |
| 78       | 78       |

---

<br>

**`질의 4-2`** $4.875$를 소수 첫째 자리까지 반올림한 값을 구하시오.

<br> **정답**

```sql
SELECT ROUND(4.875, 1);
```
{: .nolineno}

<br> **질의 결과**

| ROUND(4.875, 1) |
| --------------- |
| 4.9             |

---

<br>

**`질의 4-3`** 고객별 평균 주문 금액을 백 원 단위로 반올림한 값을 구하시오.

<br> **정답**

```sql
-- saleprice에 NULL이 없으면 
-- AVG(saleprice) 대신 SUM(saleprice)/COUNT(*)를 사용해도 된다.

SELECT  custid '고객번호', ROUND(AVG(saleprice), -2) '평균금액'
FROM    Orders
GROUP BY custid;
```
{: .nolineno}

<br> **질의 결과**

| 고객번호 | 평균금액 |
| -------- | -------- |
| 1        | 13000    |
| 2        | 7500     |
| 3        | 10300    |
| 4        | 16500    |

---

### 2.3. 문자 함수

{% assign img_path = "/assets/img/posts/DB/2024-04-21-sql-bulitin-function-and-null" %}
![2.3. 문자 함수]({{ img_path }}/2.3. 문자 함수.png)
_문자 함수의 종류_

<br>

---
**`질의 4-4`** 도서제목에 야구가 포함된 도서를 농구로 변경한 후 도서 목록을 보이시오.

<br> **정답**

```sql
SELECT  bookid, REPLACE(bookname, '야구', '농구') bookname, publisher, price
FROM    Book;
```
{: .nolineno}

---

<br>

**`질의 4-5`** 굿스포츠에서 출판한 도서의 제목과 제목의 글자 수, 바이트 수를나타내시오.(한글은 2바이트 혹은 UNICODE 경우는 3바이트를 차지함)

<br> **정답**

```sql
SELECT  bookname, CHAR_LENGTH(bookname) '글자수', LENGTH(bookname) '바이트수'
FROM    Book
WHERE   publisher LIKE '굿스포츠';
```
{: .nolineno}

---

<br>

**`질의 4-6`** 마당서점의 고객 중에서 같은 성(姓)을 가진 사람이 몇 명이나 되는지 성별 인원수를 구하시오.

<br> **정답**

```sql
SELECT  SUBSTR(name, 1, 1) '성', COUNT(*) '인원'
FROM    Customer
GROUP BY SUBSTR(name, 1, 1); -- GROUP BY 1; 도 가능(첫 번째 열을 기준으로 그룹화한다는 의미)
```
{: .nolineno}

> SQL에서 첫 문자의 인덱스는 **1부터 시작한다.**
{: .prompt-warning}

<br> **질의 결과**

| 성  | 인원 |
| --- | ---- |
| 박  | 2    |
| 김  | 2    |
| 추  | 1    |

---

### 2.4. 날짜, 시간 함수

날짜, 시간 함수는 날짜 형식(예: `DATE`) 데이터를 가진 열을 대상으로 연산을 수행한다.

![2.4. 날짜, 시간 함수]({{ img_path }}/2.4. 날짜, 시간 함수.png)
_날짜, 시간 함수 종류_

- **interval**: 날짜 간격을 표현하는 파라미터. 문법은 다음과 같다.
  - `INTERVAL <value> [ SECOND | MINUTE | HOUR | DAY | MONTH | YEAR ]`
  - 예시: `INTERVAL 10 DAY`, `INTERVAL -30 HOUR`

- **format**: 날짜와 시간 부분을 나타내는 형식 지정자
  - 종류

![2.4 format 주요 지정자]({{ img_path }}/2.4 format 주요 지정자.png){: width='350'}
_format의 주요 형식 지정자_

<br>

---
**`질의 4-7`** 마당서점은 주문일로부터 10일 후 매출을 확정한다. 각 주문의 확정일자를 구하시오.

<br> **정답**

```sql
SELECT
    orderid '주문번호',
    orderdate '주문일자', 
    ADDDATE(orderdate, INTERVAL 10 DAY) '확정일자'
    -- 만약 INTERVAL -10 DAY 라고 하면 10일 전 날짜가 된다.
FROM Orders;
```
{: .nolineno}

- 다음처럼 작성도 가능하다.

```sql
SELECT
    orderid '주문번호',
    orderdate '주문일자', 
    orderdate + INTERVAL 10 DAY '확정일자'
    -- orderdate - INTERVAL 10 DAY '확정일자' 라고 작성하면 10일 전 날짜가 된다.
FROM Orders;
```
{: .nolineno}

<br> **질의 결과**

| 주문번호 | 주문일자   | 확정일자   |
| -------- | ---------- | ---------- |
| 1        | 2024-07-01 | 2024-07-11 |
| 2        | 2024-07-03 | 2024-07-13 |
| ...      | ...        | ...        |
| 10       | 2024-07-10 | 2024-07-20 |

---

<br>

**`질의 4-8`** 마당서점이 2024년 7월 7일에 주문받은 도서의 주문번호, 주문일, 고객번호, 도서번호를 모두 보이시오.(단, 주문일은 '%Y-%m-%d' 형태로 표시한다)

<br> **정답**

```sql
SELECT  
    orderid '주문번호',
    DATE_FORMAT(orderdate, '%Y-%m-%d') '주문일',
    custid '고객번호',
    bookid '도서번호'
FROM Orders
WHERE DATE_FORMAT(orderdate, '%Y%m%d') = '20240707';
-- 또는 
WHERE orderdate = STR_TO_DATE('20240707', "%Y%m%d");
```
{: .nolineno}

<br> **질의 결과**

| 주문번호 | 주문일     | 고객번호 | 도서번호 |
| -------- | ---------- | -------- | -------- |
| 6        | 2024-07-07 | 1        | 2        |
| 7        | 2024-07-07 | 4        | 8        |

---

<br>

**`질의 4-9`** DBMS 서버에 설정된 현재 날짜와 시간, 요일을 확인하시오.

<br> **정답**

```sql
SELECT  SYSDATE(), DATE_FORMAT(SYSDATE(), "%Y/%m/%d %W %H:%i") 'SYSDATE_FORMAT';
```
{: .nolineno}

---



## 3. NULL 값 처리

`NULL`은 아직 지정되지 않은 값으로 `0`, ` `(공백) 등과 다르며, `=`, `<>`과 같은 비교 연산자로 불가능하다. 그리고 `NULL` 값의 연산을 수행하면 결과 역시 `NULL`으로 반환된다.

### 3.1. 집계 함수 사용 시 NULL 관련 주의사항

- `NULL + 숫자` 연산 결과는 `NULL`이다.
- 집계 함수를 계산할 때 `NULL`이 포함된 행은 집계에서 제외된다.
- 해당되는 행이 하나도 없을 경우
  - `SUM`, `AVG` 결과: `NULL`
  - `COUNT` 결과: `0`

### 3.2. NULL의 처리 방식

- 다음 MyBook 예제 테이블로 `NULL`의 처리 방식을 이해해보자.

| bookid | price  |
| ------ | ------ |
| 1      | 10000  |
| 2      | 20000  |
| 3      | `NULL` |

<br>

```sql
-- MyBook 생성
CREATE TABLE MyBook (bookid INTEGER, price INTEGER);
INSERT INTO MyBook VALUES (1, 10000), (2, 20000), (3, NULL);
```

---

#### 3.2.1. `NULL + 숫자` 연산 결과는 `NULL`

```sql
SELECT  price + 100 
FROM    MyBook 
WHERE   bookid = 3;
```
- **결과**

| price+100 |
| --------- |
| `NULL`    |

---

<br>

####  3.2.2. `NULL`이 포함된 행은 집계에서 제외됨

```sql
SELECT  SUM(price), AVG(price), COUNT(*), COUNT(price)
FROM    MyBook;
```
- **결과** 

| SUM(price) | AVG(price) | COUNT(*) | COUNT(price) |
| ---------- | ---------- | -------- | ------------ |
| 30000      | 15000.0000 | 3        | 2            |

---

<br>

#### 3.2.3. 해당되는 행이 하나도 없을 경우

```sql
SELECT  SUM(price), AVG(price), COUNT(*), COUNT(price)
FROM    MyBook;
WHERE   bookid > 3;
```
- **결과** 

| SUM(price) | AVG(price) | COUNT(*) | COUNT(price) |
| ---------- | ---------- | -------- | ------------ |
| `NULL`     | `NULL`     | 0        | 0            |

---

<br>

#### 3.2.4. NULL 값 확인 방법

- `NULL`을 확인할 때는 `=`, `<>`를 사용하면 안됨
- `IS NULL`, `IS NOT NULL` 연산자를 사용해야 함

### 3.3. IFNULL 함수

- `IFNULL`: NULL 값을 다른 값으로 대치하여 연산하거나 다른 값으로 출력하는 함수
- **문법**
  - `IFNULL(속성, 값)`
  - 속성이 `NULL`이면 `값`으로 대체됨

---
**`질의 4-10`** 이름, 전화번호가 포함된 고객목록을 보이시오. 단, 전화번호가 없는 고객은 '연락처없음'으로 표시한다.

<br> **정답**

```sql
SELECT  name '이름', IFNULL(phone, '연락처없음') '전화번호'
FROM    Customer;
```
{: .nolineno}

<br> **질의 결과**

| 이름   | 전화번호      |
| ------ | ------------- |
| 박지성 | 000-5000-0001 |
| 김연아 | 000-6000-0001 |
| 김연경 | 000-7000-0001 |
| 추신수 | 000-8000-0001 |
| 박세리 | 연락처없음    |

---



## 4. 행번호 출력

(추가 예정)