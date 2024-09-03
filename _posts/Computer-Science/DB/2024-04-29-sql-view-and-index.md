---
title: "[SQL 고급] 뷰, 인덱스"
date: 2024-04-21 16:01:00 +0900
categories: [Computer-Science, DB]
tags: ['DB', 'SQL']
---
{% assign img_path = "/assets/img/posts/DB/2024-04-29-sql-view-and-index" %}



## 1. 뷰(view)

- 하나 이상의 테이블을 합하여 만든 가상의 테이블

### 1.1. 장점

- **독립성**: 미리 정의된 뷰를 일반 테이블처럼 사용할 수 있기 때문에 편리하고, **사용자가 필요한 정보만 요구에 맞게 가공**하여 뷰로 만들어 쓸 수 있음
  - 원본 테이블의 구조가 변해도 응용에 영향을 주지 않도록 하는 논리적 독립성 제공

- **편리성 및 재사용성**: 자주 사용되는 복잡한 질의를 뷰로 미리 정의해 놓을 수 있음
  - 복잡한 질의를 간단히 작성

- **보안성**: 사용자별로 필요한 데이터만 선별하여 보여줄 수 있고, 중요한 질의의 경우 질의 내용을 암호화할 수 있음
  - 개인정보(주민번호)나 급여, 건강 같은 민감한 정보를 제외한 테이블을 만들어 사용

### 1.2. 특징

- 원본 데이터 값에 따라 같이 변함
- 독립적인 인덱스 생성이 어려움
- 삽입, 삭제, 갱신 연산에 많은 제약이 따름

![1. 뷰(view)]({{ img_path }}/1. 뷰(view).png){: width='650'}
_뷰의 예시_

### 1.3. 뷰의 생성

#### 1.3.1. 기본 문법

```sql
CREATE VIEW <view_name> [(<col name [, ...n ]>)]
AS SELECT <statement>
```
{: .nolineno}

---

- Book 테이블에서 '축구'라는 문구가 포함된 자료만 보여주는 뷰

```sql
SELECT *
FROM Book
WHERE bookname LIKE '%축구%';
```
{: .nolineno}

- 위 SELECT 문을 이용해 작성한 뷰 정의문

```sql 
CREATE VIEW vw_Book
AS SELECT *
FROM Book
WHERE bookname LIKE '%축구%';
```
{: .nolineno}

#### 1.3.2. 예시

---
**`질의 4-20`** 주소에 '대한민국'을 포함하는 고객들로 구성된 뷰를 만들고 조회하시오. 뷰의 이름은
vw_Customer로 설정하시오.

<br> **정답**

```sql
CREATE VIEW vw_Customer
AS SELECT   *
FROM        Customer
WHERE       address LIKE '%대한민국%';
```
{: .nolineno}

<br> **질의 결과**

- 결과를 확인하기 위해 아래 SQL을 입력한다.

```sql
SELECT  *
FROM    vw_Customer;
```
{: .nolineno}

---

<br>

**`질의 4-21`** Orders 테이블에 고객이름과 도서이름을 바로 확인할 수 있는 뷰를 생성한 후, ‘김연아’ 고객이 구입한 도서의 주문번호, 도서이름, 주문액을 보이시오.

<br> **정답**

```sql
CREATE VIEW vw_Orders (orderid, custid, name, bookid, bookname, saleprice, orderdate)
AS SELECT   od.orderid, od.custid, cs.name, od.bookid, bk.bookname, od.saleprice, od.orderdate
FROM        Orders od, Customer cs, Book bk
WHERE       od.custid = cs.custid AND od.bookid = bk.bookid;
```
{: .nolineno}

<br> **질의 결과**

- 결과를 확인하기 위해 아래 SQL을 입력한다.

```sql
SELECT  orderid, bookname, saleprice
FROM    vw_Orders
WHERE   name = '김연아';
```
{: .nolineno}

---

### 1.4. 뷰의 수정

#### 1.4.1. 기본 문법

```sql
CREATE OR REPLACE VIEW <view_name> [(<col name [, ...n ]>)]
AS SELECT <statement>
```
{: .nolineno}

#### 1.4.2. 예시

<br>

---
**`질의 4-22`** [질의 4-20]에서 생성한 뷰 vw_Customer는 주소가 대한민국인 고객을 보여준다. 이 뷰를 영국을 주소로 가진 고객으로 변경하시오. phone 속성은 필요 없으므로 포함시키지 마시오.

<br> **정답**

```sql
CREATE OR REPLACE VIEW vw_Customer (custid, name, address)
AS SELECT custid, name, address
FROM Customer
WHERE address LIKE '%영국%';
```
{: .nolineno}

<br> **질의 결과**

---

### 1.5. 뷰의 삭제

#### 1.5.1. 기본 문법

```sql
DROP VIEW <view_name> [ , ... n ];
```
{: .nolineno}

#### 1.5.2. 예시

<br>

---
**`질의 4-23`** 앞서 생성한 뷰 vw_Customer를 삭제하시오.

<br> **정답**

```sql
DROP VIEW vw_Customer;
```
{: .nolineno}

<br> **질의 결과**



---



## 2. 데이터베이스의 물리적 저장

![2. 데이터베이스의 물리적 저장]({{ img_path }}/2. 데이터베이스의 물리적 저장.png){: width='650'}
_DBMS와 데이터 파일의 관계_

- 실제 데이터가 저장되는 곳은 보조기억장치임
  - 하드디스크, SSD, USB 메모리 등
- 가장 많이 사용되는 장치는 **하드디스크**이다.

### 2.1. 하드디스크(HDD)의 구조

**시험에 나오지 않음**

- 하드디스크는 원형의 **플레이트(plate)**로 구성되어 있고, 이 플레이트는 논리적으로 트랙으로 나뉘며 트랙은 다시 몇 개의 섹터로 나뉨
- 원형의 플레이트는 초당 빠른 속도로 회전하고, 회전하는 플레이트를 하드디스크의 **액세스 암(arm)**과 **헤더(header)**가 접근하여 원하는 섹터에서 데이터를 가져옴
- 하드디스크에 저장된 데이터를 읽어 오는 데 걸리는 시간은 **모터(motor)**에 의해서 **분당 회전하는 속도(RPM,Revolutions Per Minute)**, 데이터를 읽을 때 **액세스 암이 이동하는 시간(latency time)**, **주기억장치로 읽어오는 시간(transfer time)**에 영향을 받음

![2.1. 하드디스크(HDD)의 구조]({{ img_path }}/2.1. 하드디스크(HDD)의 구조.png){: width='500'}
_하드디스크의 구조_

#### 2.1.1. 엑세스 시간(access time)

**시험에 나오지 않음**

- **액세스 시간** $=$ **탐색시간** $+$ **회전 지연 시간** $+$ **데이터 전송 시간**
  - **탐색시간(seek time)**: 액세스 헤드를 트랙에 이동시키는 시간
  - **회전지연시간(rotational latency time)**: 섹터가 액세스 헤드에 접근하는 시간
  - **데이터 전송시간(data transfer time)**: 데이터를 주기억장치로 읽어오는 시간



### 2.2. MySQL의 DBMS 구조 (시험 X)

![2.2. MySQL의 DBMS 구조]({{ img_path }}/2.2. MySQL의 DBMS 구조.png){: width='500'}
_MySQL의 DBMS 구조_

![2.2. MySQL InnoDB엔진 DB 파일]({{ img_path }}/2.2. MySQL InnoDB엔진 DB 파일.png){: width='500'}
_MySQL InnoDB 엔진 데이터베이스의 파일_



## 3. 인덱스(index)와 B-tree

- 인덱스(index, 색인): 도서의 색인이나 사전과 같이 데이터를 쉽고 빠르게 찾을 수 있도록 만든 데이터 구조

![3. B-tree 구조]({{ img_path }}/3. B-tree 구조.png){: width='650'}
_B-tree 구조_

### 3.1. B-tree에서 검색 예시

![3.1. B-tree에서 검색 예시]({{ img_path }}/3.1. B-tree에서 검색 예시.png){: width='500'}
_B-tree에서 검색 예시_

### 3.2. 인덱스의 특징

- 인덱스는 테이블에서 **한 개 이상의 속성을 이용하여 생성**함
- **빠른 검색**과 함께 **효율적인 레코드 접근**이 가능함
- **순서대로 정렬된 속성**과 **데이터의 위치만 보유**: **테이블보다 작은 공간을 차지**함
- 저장된 값들은 테이블의 부분집합이 됨
- 일반적으로 **B-tree 형태의 구조**를 가짐
- 데이터의 수정, 삭제 등의 변경이 발생하면 인덱스의 재구성이 필요함



## 4. 인덱스와 MySQL

### 4.1. MySQL 인덱스의 종류

- **클러스터 인덱스(Clustered Index)**
  - **기본적인 인덱스**로, 테이블 생성 시 기본키를 지정하면 **기본키에 대하여 클러스터 인덱스를 생성**한다.
  - 기본키를 지정하지 않으면 먼저 나오는 UNIQUE 속성에 대하여 클러스터 인덱스를 생성한다.
  - 기본키나 UNIQUE 속성이 없는 테이블은 MySQL 이 자체 생성한 행번호(Row ID)를 이용하여 클러스터 인덱스를 생성한다.
  - 큰 데이터를 잘게 쪼개서 저장하는 것이 아이디어

- **보조 인덱스(Secondary Index)**
  - **클러스터 인덱스가 아닌 모든 인덱스**는 보조 인덱스
  - 보조 인덱스의 각 레코드는 **보조 인덱스 속성**과 **기본키 속성 값**을 가짐
  - 보조 인덱스를 검색하여 기본키 속성 값을 찾은 다음 클러스터 인덱스로 가서 해당 레코드를 찾는다.

![4.1. 인덱스 혼합 사용 예]({{ img_path }}/4.1. 인덱스 혼합 사용 예.png)
_클러스터 인덱스와 보조 인덱스를 동시에 사용하는 검색_

### 4.2. 클러스터 인덱스

![4.2. 클러스터 인덱스 예]({{ img_path }}/4.2. 클러스터 인덱스 예.png){: width='500'}
_클러스터 인덱스 예_

### 4.3. B-tree 인덱스

![4.3. B-tree 인덱스 예]({{ img_path }}/4.3. B-tree 인덱스 예.png){: width='500'}
_B-tree 인덱스 예_



## 5. 인덱스의 생성

### 5.1. 고려사항

- 인덱스는 **WHERE 절에 자주 사용되는 속성**이어야 함
- 인덱스는 **조인에 자주 사용되는 속성**이어야 함
- 단일 테이블에 **인덱스가 많으면 속도가 느려질 수 있음**(테이블당 4~5개 정도 권장)
- 속성이 가공되는 경우 사용하지 않음
- 속성의 선택도가 낮을 때 유리함(속성의 모든 값이 다른 경우)

### 5.2. 인덱스 생성 문법

```sql
CREATE [UNIQUE] INDEX <index name>
ON <table name> ( <column> [ASC | DESC] [{, <column> [ASC | DESC]}, ...] )[;]
```
{: .nolineno}

### 5.3. 예시

<br>

---
**시험 X!!**

**`질의 4-24`** Book 테이블의 bookname 열을 대상으로 비 클러스터 인덱스 ix_Book을 생성하라.


<br> **정답**

```sql
CREATE INDEX ix_Book ON Book (bookname);
```
{: .nolineno}

<br> **질의 결과**



---

<br>

**시험 X!!**

**`질의 4-25`** Book 테이블의 publisher, price 열을 대상으로 인덱스 ix_Book2를 생성하시오

<br> **정답**

```sql
CREATE INDEX ix_Book2 ON Book(publisher, price);
```
{: .nolineno}

<br> **질의 결과**



---



## 6. 인덱스의 재구성과 삭제

### 6.1. 재구성

- 재구성 문법

```sql
ANALYZE TABLE <table name>;
```
{: .nolineno}

### 6.2. 삭제

- 삭제 문법

```sql
DROP INDEX <table name>;
```
{: .nolineno}