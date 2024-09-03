---
title: "[DB] 데이터베이스 프로그래밍"
date: 2024-05-08 15:09:00 +0900
categories: [Computer-Science, DB]
tags: ['DB']
---



## 1. 데이터베이스 프로그래밍

- 프로그래밍이란?
  - 프로그램을 설계하고 소스코드를 작성하여 디버깅하는 과정
- 데이터베이스 프로그래밍이란?
  - DBMS에 데이터를 정의하고 저장된 데이터를 읽어와 데이터를 변경하는 프로그램을 작성하는 과정
  - 일반 프로그래밍과의 차이점: 프로그래밍 도중 SQL을 포함하고 있음

{% assign img_path = "/assets/img/posts/DB/2024-05-08-db-programming" %}
![1. 데이터베이스 프로그래밍]({{ img_path }}/1. 데이터베이스 프로그래밍.png){: width='500'}

### 1.1. 데이터베이스 프로그래밍 방법

#### 1.1.1. SQL 전용 언어를 사용하는 방법

- SQL 자체의 기능을 확장하여 변수, 제어, 입출력 등 기능을 추가한 새로운 언어를 사용하는 방법
- 오라클: 저장 프로그램 언어를 사용함
- SQL Server: T-SQL이라는 언어를 사용함

#### 1.1.2. 일반 프로그래밍 언어에 SQL을 삽입하여 사용하는 방법

- 일반 프로그래밍 언어(GPL(General Programming Language): 자바, C, C++ 등)에 SQL을 삽입하여 사용하는 방법
- GPL로 작성된 응용 프로그램에서 DB에 저장된 데이터를 관리, 검색함
- 삽입된 SQL문은 DBMS의 컴파일러가 처리함

#### 1.1.3. 웹 프로그래밍 언어에 SQL을 삽입하여 사용하는 방법

- 호스트 언어가 JSP, ASP, PHP 등 웹 스크립트 언어인 경우임

#### 1.1.4. 4GL(4th Generation Language)

- DB 관리 기능과 비주얼 프로그래밍 기능을 갖춘 ‘GUI 기반 SW 개발 도구’를 사용하여 프로그래밍하는 방법
- Delphi, Power Builder, Visual Basic 등이 존재

### 1.2. DBMS 플랫폼 및 DB 프로그래밍의 유형

![1.2. DBMS 플랫폼 및 DB 프로그래밍의 유형]({{ img_path }}/1.2. DBMS 플랫폼 및 DB 프로그래밍의 유형.png)
_DBMS 플랫폼 및 DB 프로그래밍의 유형_

### 1.3. DBMS의 종류와 특징

| 특징          | Access           | SQL Server              | Oracle               | MySQL              | DB2                   | SQLite              |
| ------------- | ---------------- | ----------------------- | -------------------- | ------------------ | --------------------- | ------------------- |
| 제조사        | 마이크로소프트사 | 마이크로소프트사        | 오라클사             | 오라클사           | IBM사                 | 리처드 힙(오픈소스) |
| 운영체제 기반 | 윈도우           | 윈도우,리눅스           | 윈도우,유닉스,리눅스 | 유닉스             | 모바일 OS             |
| 용도          | 개인용 DBMS      | 윈도우 기반 기업용 DBMS | 대용량 DB위한 응용   | 대용량 DB위한 응용 | 대용량 DB를 위한 응용 | 모바일 전용 DB      |



## 2. 저장 프로그램

- DB 응용 프로그램을 작성하는 데 사용하는 MySQL의 SQL 전용 언어
- SQL 문에 변수, 제어, 입출력 등의 프로그래밍 기능을 추가하여 SQL 만으로 처리하기 어려운 문제를 해결함
- 저장 프로그램은 Workbench에서 바로 작성하고 컴파일한 후 결과를 실행함

### 2.1. 프로시저

- 정의를 위해 CREATE PROCEDURE 문을 사용함
- 정의 방법
  - 프로시저는 선언부와 실행부(BEGIN-END)로 구성됨
  - 선언부에서는 변수와 매개변수를 선언하고 실행부에서는 프로그램 로직을 구현함
  - 매개변수(parameter)는 저장 프로시저가 호출될 때 그 프로시저에 전달되는 값
  - 변수(variable)는 저장 프로시저나 트리거 내에서 사용되는 값
  - 소스코드에 대한 설명문은 /*와 */ 사이에 기술
  - 설명문이 한 줄이면 이중 대시(--) 기호 다음에 기술해도 됨

### 2.2. 삽입 작업을 하는 프로시저

```sql
use madang;
delimiter //  /* delimiter를 //로 변경 */
CREATE PROCEDURE InsertBook(
    IN myBookID INTEGER,
    IN myBookName VARCHAR(40),
    IN myPublisher VARCHAR(40),
    IN myPrice INTEGER)
BEGIN
    INSERT INTO Book(bookid, bookname, publisher, price)
    VALUES(myBookID, myBookName, myPublisher, myPrice);
END;
// /* delimiter의 사용 */
delimiter ; /* delimiter를 ;로 복원 */
```
{: .filename='InsertBook.sql'}

```sql
/* 프로시저 InsertBook을 테스트하는 부분 */
CALL InsertBook(13, '스포츠과학', '마당과학서적', 25000);
SELECT * FROM Book;
```

- delimiter란?
  - 코드 블록을 하나의 덩어리라고 알려주는 구문

### 2.3. 제어문을 사용하는 프로시저

- 어떤 조건에서 어떤 코드가 실행되어야 하는 지를 제어하기 위한 문법
- 절차적 언어의 구성요소를 포함함

//표 추가 바람
//예제 5-2 추가 바람

### 2.4. 결과를 반환하는 프로시저

//예제 5-3 추가바람

### 2.5. 커서를 사용하는 프로시저

- 커서(cursor)는 실행 결과 테이블을 한 번에 한 행씩 처리하기 위하여 테이블의 행을 순서대로 가리키는 데 사용함
- 사용시 DB가 느려지므로 일반적으로 잘 사용되지는 X

| 키워드                             | 역할        |
| ---------------------------------- | ----------- |
| CUROR <cursor 이름> IS <커서 정의> | 커서를 생성 |
| OPEN <cursor 이름> | 커서의 사용을 시작|
| FETCH <cursor 이름> INTO <변수> | 행 데이터를 가져옴 |
| CLOSE <cursor 이름> | 커서의 사용을 끝냄 |

//예제 5-4 추가예정



## 3. 트리거

- 데이터의 변경(INSERT, DELETE, UPDATE)문이 실행될 때 자동으로 따라서 실행되는 프로시저

![3. 트리거]({{ img_path }}/3. 트리거.png){: width='500'}
_데이터 변경과 트리거의 수행_

### 트리거의 주요 목적

- 목적: 제약조건과 무결성을 지키기 위해 사용
- 목적 2: 로그를 기록하기 위해 사용

//예제 5-5 추가예정



## 사용자 정의 함수

- 수학의 함수, 프로그래밍의 함수와 마찬가지로 입력된 값 가공하여 결과 값을 되돌려 줌
- 문법적 차이가 조금 있음

```sql
delimiter //
CREATE FUNCTION fnc_Interest(
Price INTEGER) RETURNS INT
BEGIN
    DECLARE myInterest INTEGER;
    -- 가격이 30,000원 이상이면 10%, 30,000원 미만이면 5%
    IF Price >= 30000 THEN SET myInterest = Price * 0.1;
        ELSE SET myInterest := Price * 0.05;
    END IF;
    RETURN myInterest;
END; //
delimiter ;
```

```sql
/* Orders 테이블에서 각 주문에 대한 이익을 출력 */
SELECT custid, orderid, saleprice, fnc_Interest(saleprice) interest
FROM Orders;
```
//표 5-4, 5-5 추가바람