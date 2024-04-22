---
title: "[SQL] DDL: CREATE, ALTER, DROP"
date: 2024-04-20 23:13:00 +0900
categories: [Computer-Science, DB]
tags: ['DB', 'SQL']
---



## 1. 데이터 정의어(DDL)

데이터를 저장하기 위해 우선 데이터를 저장할 테이블의 구조를 만들어야 하는데, **데이터 정의어**(**DDL**, Data Definition Language)로 이 작업을 수행할 수 있다. 대표적으로 `CREATE`, `ALTER`, `DROP`이 있다.



## 2. CREATE 문

`CREATE` 문은 다음과 같은 사항을 정의하는 명령어이다.
- 테이블의 구성
- 속성·속성에 관한 제약 조건 명시
- 기본키·외래키 지정
  - `PRIMARY KEY`: 기본키 지정 시 사용
  - `FOREIGN KEY`: 외래키 지정 시 사용

다음 옵션으로 외래키 속성의 수정·튜플의 삭제 시의 동작을 지정할 수 있다.
- `ON UPDATE`
- `ON DELETE`

### 2.1. 기본 문법

- `<>`: SQL 예약어가 아닌 부분, 속성명이나 조건식 등이 올 수 있음
- `[]`: 선택적으로 사용 가능
- `|`: 1개를 선택함 
- `{}`: 반복 가능
- `attr.`: attribute(속성)


```sql
CREATE TABLE <table name> (
    {
        <attr. name> DATATYPE
            [ NOT NULL | UNIQUE | DEFAULT <default value> | CHECK( <check condition> ) ]  
    }
    [ PRIMARY KEY ( <attr. name(s)> ) ]
    {
        [ FOREIGN KEY ( <attr. name> ) REFERENCES <table name(attr. name)> ( <ref. attr. name> ) ]
            [ ON DELETE [ CASCADE | SET NULL ] ]
    }
);
```

#### 2.1.1. 문법 설명

  - `NOT NULL`: NULL 값을 허용하지 않는 제약
  - `UNIQUE`: 유일한 값이어야 하는 제약
  - `DEFAULT`: 기본값 설정
  - `CHECK`: 속성 값에 대한 제약 조건 부여
    - 검사 조건을 **반드시 괄호로 감싸야 함**

- 키의 유형 지정
  - `PRIMARY KEY`: 해당 속성이 기본키임을 지정함
    - 속성명을 **반드시 괄호로 감싸야 함**
    - 여러 속성을 기본키로 지정할 경우 `(att1, att2)` 처럼 작성하면 됨
  - `FOREIGN KEY`: 해당 속성이 왜래키임을 지정함. 여러 번 등장 가능
    - 속성명을 **반드시 괄호로 감싸야 함**
    - `ON DELETE`: 참조되던 튜플이 삭제될 시 외래 키 속성에 대한 동작을 지정
      - `CASCADE`: 자신도 연쇄적으로 삭제됨
      - `SET NULL`: 외래키 값을 `NULL`로 지정
      - 명시하지 않을 경우 기본 값은 `RESTRICT`(`NO ACTION`)이다.

> ON UPDATE 에 대한 설명은 나중에 찾아볼 예정이다.
{: .prompt-warning}

### 2.2. 예시

<br>

**`질의 3-34`** 다음과 같은 속성을 가진 NewBook 테이블을 생성하시오. 정수형은 INTEGER를 사용하며 문자형은 가변형 문자타입인 VARCHAR을 사용한다.

- bookid(도서번호) - INTEGER
- bookname(도서이름) - VARCHAR(20)
- publisher(출판사) - VARCHAR(20)
- price(가격) - INTEGER

<br> **정답**

```sql
CREATE TABLE NewBook (
    bookid    INTEGER,
    bookname  VARCHAR(20),
    publisher VARCHAR(20),
    price     INTEGER
);
```
{: .nolineno}

### 2.3. 기본키 지정 예시

기본키를 지정하려면 아래 방법 중 하나를 사용하자.

```sql
-- 방법 1
CREATE TABLE NewBook (
    bookid    INTEGER PRIMARY KEY,
    bookname  VARCHAR(20),
    publisher VARCHAR(20),
    price     INTEGER
);
```
{: .nolineno}

```sql
-- 방법 2
CREATE TABLE NewBook (
    bookid    INTEGER,
    bookname  VARCHAR(20),
    publisher VARCHAR(20),
    price     INTEGER,
    PRIMARY KEY (bookid) -- 괄호가 필수다!!!
);
```
{: .nolineno}

- 복합키를 기본키로 지정하고 싶다면 다음과 같이 할 수 있다.

```sql
CREATE TABLE NewBook (
    bookname  VARCHAR(20),
    publisher VARCHAR(20),
    price     INTEGER,
    PRIMARY KEY (bookname, publisher) -- 복합키의 기본키 지정 방법
);
```
{: .nolineno}

다음은 **`질의 3-34`**에 조금 더 복잡한 제약사항을 추가한 예시이다.

---

<br>

**`질의 3-34 변형`** bookname은 NULL 을 가질 수 없고, publisher에는 같은 값이 있으면 안되며, price에 값이 입력되지 않은 경우 10,000을 기본값으로 저장한다. 또 가격은 최소 1,000원 이상으로 한다.

<br> **정답**

```sql
CREATE TABLE NewBook (
    bookname  VARCHAR(20) NOT NULL,
    publisher VARCHAR(20) UNIQUE,
    price     INTEGER     DEFAULT 10000 CHECK(price >= 1000)/* 괄호가 필수다! */,
    PRIMARY KEY (bookname, publisher)
);
```
{: .nolineno}

---

<br>

**`질의 3-35`** 다음과 같은 속성을 가진 NewCustomer 테이블을 생성하시오.

- custid(고객번호) - INTEGER, 기본키
- name(이름) – VARCHAR(40)
- address(주소) – VARCHAR(40)
- phone(전화번호) – VARCHAR(30)

<br> **정답**

```sql
CREATE TABLE NewCustomer (
    custid  INTEGER PRIMARY KEY,
    name    VARCHAR(40),
    address VARCHAR(40),
    phone   VARCHAR(30)
);
```
{: .nolineno}

---

### 2.4. 외래키 지정 예시

**`질의 3-36`** 다음과 같은 속성을 가진 NewOrders 테이블을 생성하시오.

- orderid(주문번호) - INTEGER, 기본키
- custid(고객번호) - INTEGER, NOT NULL 제약조건, 외래키(NewCustomer.custid, 연쇄삭제)
- bookid(도서번호) - INTEGER, NOT NULL 제약조건
- saleprice(판매가격) - INTEGER
- orderdate(판매일자) – DATE

<br> **정답**

```sql
CREATE TABLE NewOrders (
    orderid   INTEGER,
    custid    INTEGER   NOT NULL,
    bookid    INTEGER   NOT NULL,
    saleprice INTEGER,
    orderdate DATE,

    -- 괄호 사용 여부에 주의하자!!
    PRIMARY KEY (orderid),
    FOREIGN KEY (custid) REFERENCES NewCustomer (custid) ON DELETE CASCADE
);
```
{: .nolineno}

---

### 2.5. 외래키 지정시 주의사항

- 외래키 제약 조건 명시할 때 반드시 **참조되는 테이블(부모 릴레이션)**이 존재해야 한다.
- 또한 참조되는 테이블의 **기본키**여야 한다.
- 외래키 지정 시 `ON DELETE` 또는 `ON UPDATE` 옵션은 참조되는 테이블의 튜플이 **삭제·수정**될 때 취할 수 있는 동작을 지정한다.
- `NO ACTION`은 어떤 동작도 하지 않으며, 기본값이다.

### 2.6. 사용가능한 데이터 타입 종류

| 데이터 타입                      | 설명                                                          | ANSI SQL 표준 타입                          |
| -------------------------------- | ------------------------------------------------------------- | ------------------------------------------- |
| `INTEGER`<br>`INT`               | 4바이트 정수형                                                | `INTEGER`, `INT`<br>`SMALLINT`              |
| `NUMERIC(m,d)`<br>`DECIMAL(m,d)` | 전체자리수 m, 소수점이하 자리수 d를 가진 숫자형               | `DECIMAL(p, s)`<br>`NUMERIC[(p,s)]`         |
| `CHAR(n)`                        | 문자형 고정길이, 문자를 저장하고 남은 공간은 공백으로 채운다. | `CHARACTER(n)`<br>`CHAR(n)`                 |
| `VARCHAR(n)`                     | 문자형 가변길이                                               | `CHARACTER VARYING(n)`<br>`CHAR VARYING(n)` |
| `DATE`                           | 날짜형, 연도, 월, 날, 시간을 저장한다.                        |                                             |



## 3. ALTER 문

- 생성된 테이블의 속성·속성에 관한 제약 사항을 변경한다.
- 기본키, 외래키를 변경할 수 있음

### 3.1. 기본 문법

```sql
ALTER TABLE <table name> 
    [ADD <attr. name> DATATYPE]
    [DROP COLUMN <attr. name>]
    [MODIFY <attr. name> DATATYPE]
    [MODIFY <attr. name> DATATYPE [ NULL | NOT NULL ]]
    [ADD PRIMARY KEY (<attr. name>) ]
    [ [ ADD | DROP] CONSTRAINT <constraint name> ]
```

- `ADD`, `DROP COLUMN`: 속성을 추가·제거 시 사용
- `MODIFY`: 속성 변경 시 사용
- `ADD CONSTRAINT <제약이름>`, `DROP CONSTRAINT <제약이름>`: 제약사항 추가·삭제 시 사용

### 3.2. 예시

**`질의 3-37`** NewBook 테이블에 VARCHAR(13)의 자료형을 가진 isbn 속성을 추가하시오.

<br> **정답**

```sql
ALTER TABLE NewBook ADD isbn VARCHAR(13);
```
{: .nolineno}

<br>

---
**`질의 3-38`** NewBook 테이블의 isbn 속성의 데이터 타입을 INTEGER형으로 변경하시오.

<br> **정답**

```sql
ALTER TABLE NewBook MODIFY isbn INTEGER;
```
{: .nolineno}

<br>

---
**`질의 3-39`** NewBook 테이블의 isbn 속성을 삭제하시오.

<br> **정답**

```sql
ALTER TABLE NewBook DROP COLUMN isbn; -- COLUMN을 적어야 한다.
```
{: .nolineno}

<br>

---
**`질의 3-40`** NewBook 테이블의 bookid 속성에 NOT NULL 제약조건을 적용하시오.

<br> **정답**

```sql
ALTER TABLE NewBook MODIFY bookid INTEGER NOT NULL; -- 연산 대상 속성의 데이터 타입을 적어야 한다.
```
{: .nolineno}

<br>

---
**`질의 3-41`** NewBook 테이블의 bookid 속성을 기본키로 변경하시오.

<br> **정답**

```sql
ALTER TABLE NewBook ADD PRIMARY KEY (bookid); -- 괄호를 잊지 말자!
```
{: .nolineno}

<br>

---



## 4. DROP 문

- 테이블을 삭제하는 명령어
- 테이블의 구조와 데이터를 모두 삭제하므로 사용에 주의해야 한다.

> 이 작업은 복구할 수 없다!
{: .prompt-danger}

- 문법
  - `DROP TABLE <table name>`

### 4.1. 예시

**`질의 3-42`** NewBook 테이블을 삭제하시오.

<br> **정답**

```sql
DROP TABLE NewBook;
```
{: .nolineno}

---

<br>

**`질의 3-43`** NusCustomer 테이블을 삭제하시오. 만약 삭제가 거절되면 원인을 파악하고 돤련된 테이블을 같이 삭제하시오((NewOrders 테이블이 NewCustomer를 참조하고 있음).

<br> **정답**

```sql
DROP TABLE NewOrders; -- 이 테이블을 먼저 삭제해야 함
DROP TABLE NewCustomer;
```
{: .nolineno}

---