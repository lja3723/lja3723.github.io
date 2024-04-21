---
title: "[SQL] DML: INSERT, UPDATE, DELETE"
date: 2024-04-21 00:59:00 +0900
categories: [Computer-Science, DB]
tags: ['DB', 'SQL']
---



## 1. INSERT 문

- 테이블에 새 튜플을 삽입하는 명령어

### 1.1. 문법

- `[]`: 생략 가능

```sql
INSERT INTO <table name> [ ( <attr. list> ) ]
    VALUES ( <value list> );
```
{: .nolineno}

### 1.2. 예시

**`질의 3-44`** Book 테이블에 새로운 도서 ‘스포츠 의학’을 삽입하시오. 스포츠 의학은 한솔의학서적에서 출간했으며 가격은 90,000원이다.

<br> **정답**

```sql
INSERT INTO Book(bookid, bookname, publisher, price) 
    VALUES (11, '스포츠 의학', '한솔의학서적', 90000);
```
{: .nolineno}

- 다음과 같이 속성 이름을 생략할 수 있다.
  - 대신 데이터 삽입을 테이블에 정의된 속성 순서와 일치해야 한다.

```sql
INSERT INTO Book
    VALUES (11, '스포츠 의학', '한솔의학서적', 90000);
```
{: .nolineno}

- 다음처럼 속성의 순서를 뒤바꿔서 입력할 수 있다.

```sql
INSERT INTO Book(bookname, bookid, publisher, price) 
    VALUES ('스포츠 의학', 11, '한솔의학서적', 90000);
```
{: .nolineno}

- 다음처럼 여러 값을 동시에 입력할 수 있다.

```sql
INSERT INTO Book(bookid, bookname, publisher, price) 
    VALUES 
        (11, '스포츠 의학', '한솔의학서적', 90000),
        (13, '트렌드 코리아', '한국일보', 25000),
        (14, '올림픽 이야기', '삼성당', 8000),
        (16, 'Zen Golf', 'Pearson', 12000),
           
```
{: .nolineno}

---

<br>

**`질의 3-45`** Book 테이블에 새로운 도서 ‘스포츠 의학’을 삽입하시오. 스포츠 의학은 한솔의학서적에서 출간했으며 가격은 미정이다.

<br> **정답**

```sql
INSERT INTO Book(bookid, bookname, publisher)
    VALUES (14, '스포츠 의학', '한솔의학서적'); -- 이렇게 하면 매칭되지 않은 속성(price)에는 NULL 또는 기본값(설정된 경우)이 들어간다.
```
{: .nolineno}

<br> **질의 결과**

| bookid | bookname          | publisher    | price  |
| ------ | ----------------- | ------------ | ------ |
| 1      | 축구의 역사       | 굿스포츠     | 7000   |
| 2      | 축구 아는 여자    | 나무수       | 13000  |
| ...    | ...               | ...          | ...    |
| 10     | Olympic Champions | Pearson      | 13000  |
| 11     | 스포츠 의학       | 한솔의학서적 | `NULL` |

---

#### 1.2.1. bulk insert

- bulk insert(대량 삽입)이란 한꺼번에 여러 개의 튜플을 삽입하는 방법이다.

<br>

---
**`질의 3-46`** 수입도서 목록(Imported_book)을 Book 테이블에 모두 삽입하시오.(Imported_book 테이블은 스크립트 Book 테이블과 같이 이미 만들어져 있음)

<br> **정답**

```sql
INSERT INTO Book(bookid, bookname, price, publisher)
    SELECT bookid, bookname, price, publisher -- 이렇게 VALUES 대신 SELECT 문의 결과 테이블을 통해 대량으로 삽입할 수 있다.
    FROM Imported_book;
```
{: .nolineno}

<br> **질의 결과**

| bookid | bookname          | publisher      | price  |
| ------ | ----------------- | -------------- | ------ |
| 1      | 축구의 역사       | 굿스포츠       | 7000   |
| 2      | 축구 아는 여자    | 나무수         | 13000  |
| ...    | ...               | ...            | ...    |
| 10     | Olympic Champions | Pearson        | 13000  |
| 11     | 스포츠 의학       | 한솔의학서적   | `NULL` |
| 21     | Zen Golf          | Pearson        | 12000  |
| 22     | Soccer Skills     | Human Kinetics | 15000  |

---



## 2. UPDATE 문

- 특정 속성을 수정하는 명령어

### 2.1. 문법

- `[]`: 생략 가능

```sql
UPDATE  <table name> 
SELECT  <attr. name1> = <value1> [, <attr. name2> = <value2>, ... ]
[ WHERE <search cond.> ];
```
{: .nolineno}

### 2.2. 예시

**`질의 3-47`** Customer 테이블에서 고객번호가 5인 고객의 주소를 ‘대한민국 부산’으로 변경하시오.

<br> **정답**

```sql
-- SET SQL_SAFE_UPATES=0; /* Safe Updates 옵션 미 해제 시 실행 */

UPDATE  Customer
SET     address = '대한민국 부산'
WHERE   custid = 5;
```
{: .nolineno}

<br>

---
**`질의 3-48`** Book 테이블에서 14번 ‘스포츠 의학’의 출판사를 imported_book 테이블의 21번 책의 출판사와 동일하게 변경하시오.

<br> **정답**

```sql
UPDATE  Book
SET     publisher = (
    SELECT  publisher
    FROM    imported_book
    WHERE   bookid = 21)
WHERE   bookid = 14;
```
{: .nolineno}

---

### 2.3. 주의사항

> 다음과 같이 WHERE 절을 생략하면 모든 튜플이 예기치 않게 모두 수정될 수 있다.
{: .prompt-warning}

```sql
UPDATE  Customer
SET     address = '대한민국 서울';
-- 모든 고객의 주소가 '대한민국 서울'로 설정된다.
```
{: .nolineno}



## 3. DELETE 문

- 테이블에 있는 기존 튜플을 삭제하는 명령어

### 3.1. 문법

- `[]`: 생략 가능

```sql
DELETE FROM <table name>
[ WHERE     <search cond.> ];
```
{: .nolineno}

> WHERE 절이 없으면 모든 튜플을 삭제한다.
{: .prompt-warning}

### 3.2. 예시

**`질의 3-49`** Book 테이블에서 도서번호가 11인 도서를 삭제하시오.

<br> **정답**

```sql
DELETE FROM Book
WHERE       bookid = 11;
```
{: .nolineno}

---

<br>

**`질의 3-50`** 모든 고객을 삭제하시오.

<br> **정답**

```sql
DELETE FROM Customer;
```
{: .nolineno}

- 위 SQL은 실행되지 않는데, Orders 테이블에서 Customer.custid 속성을 외래키로 참조하고 있기 때문이다.

---