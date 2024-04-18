---
title: "[DB] 데이터베이스 시스템 구성"
date: 2024-04-17 21:03:00 +0900
categories: [Computer-Science, DB]
tags: ['DB']
---
{% assign img_path = "/assets/img/posts/DB/2024-04-17-relational-data-model" %}



## 1. 관계 데이터 모델 개념

### 1.1. 릴레이션(relation)

- **relation**: 행과 열로 구성된 테이블

| 용어                  | 한글 용어         | 비고                 |
| --------------------- | ----------------- | -------------------- |
| relation              | 릴레이션, 테이블  | "관계"라고 하지 않음 |
| relational data model | 관계 데이터 모델  |                      |
| relational database   | 관계 데이터베이스 |                      |
| relational algebra    | 관계대수          |                      |
| relationship          | 관계              |                      |

![1.1. 릴레이션]({{ img_path }}/1.1. 릴레이션.png){: width='650'}

- 도서번호 	= {1, 2, 3, 4, 5}
- 도서이름 	= {축구의 역사, 축구아는 여자, 축구의 이해, 골프 바이블, 피겨 교본}
- 출판사 	= {굿스포츠, 나무수, 대한미디어}
  - 중복이 없음
- 가격 		= {7000, 13000, 22000, 35000, 8000}

**첫 번째 행**(1, 축구의 역사, 굿스포츠, 7000)의 경우 네 개의 집합에서 각각 원소 한 개씩 선택하여 만들어진 것으로, 이 원소들이 **관계(relationship)**를 맺고 있다.

### 1.2. 관계(relationship)

1. 릴레이션 내에서 생성되는 관계
  - 릴레이션 내 데이터들의 관계
2. 릴레이션 간 생성되는 관계
  - 릴레이션 간 관계(참조로 관계 생성)

![1.2. 관계]({{ img_path }}/1.2. 관계.png){: width='340'}
_릴레이션 간의 관계_



## 2. 릴레이션 스키마와 인스턴스

![2. 릴레이션 스키마와 인스턴스]({{ img_path }}/2. 릴레이션 스키마와 인스턴스.png){: width='650'}
_도서 릴레이션_

### 2.1. 릴레이션 스키마

- 스키마의 요소
  - **속성(attribute)**: 릴레이션 스키마의 열
  - **도메인(domain)**: 속성이 가질 수 있는 값의 집합
  - **차수(degree)**: 속성의 개수
- 스키마의 표현
  - `릴레이션 이름(속성1: 도메인1, 속성2: 도메인2, 속성3: 도메인3, ...)`
  - 예) 도서(도서번호, 도서이름, 출판사, 가격)

### 2.2. 릴레이션 인스턴스

- 인스턴스 요소
  - **튜플(tuple)**: 릴레이션의 행
  - **카디날리티(cardinality)**: 튜플의 수

- 릴레이션 구조와 관련된 용어

| 릴레이션 용어      | 같은 의미로 통용되는 용어 | 파일 시스템 용어 |
| ------------------ | ------------------------- | ---------------- |
| 릴레이션(relation) | 테이블(table)             | 파일(file)       |
| 스키마(schema)     | 내포(intension)           | 헤더(header)     |
| 인스턴스(instance) | 외연(extension)           | 데이터(data)     |
| 튜플(tuple)        | 행(row)                   | 레코드(recored)  |
| 속성(attribute)    | 열(column)                | 필드(field)      |



## 3. 릴레이션 특징

1. 속성의 값은 단일 값을 가짐
2. **속성은 서로 다른 이름 가짐**
3. 한 속석 값은 모두 같은 도메인 값을 가짐
4. 속성의 순서는 상관 X
5. **릴레이션 내 중복 튜플 허용 X**
6. 튜플 순서는 상관 X
  - 관계 데이터 모델은 데이터를 릴레이션으로 표현함
  - 릴레이션의 **제약조건**과 관계 연산 위한 **관계대수**를 정의

![3. 릴레이션 특징이 위배된 경우]({{ img_path }}/3. 릴레이션 특징이 위배된 경우.png){: width='550'}
_릴레이션 특징이 위배된 경우_



## 4. 관계 데이터 모델

- 관계 데이터 모델은 데이터를 2차원 테이블 형태인 릴레이션으로 표현함
- 릴레이션에 대한 제약조건과 관계 연산을 위한 관계대수를 정의함

![4. 관계 데이터 모델]({{ img_path }}/4. 관계 데이터 모델.png){: width='500'}
_관계 데이터베이스 시스템_