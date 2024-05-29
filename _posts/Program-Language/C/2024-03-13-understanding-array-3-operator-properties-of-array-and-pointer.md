---
title: "[배열의 이해] 3. 배열 및 포인터와 관련된 여러 연산자의 성질"
date: 2024-03-13 14:09:00 +0900
categories: [Program-Language, C]
tags: ["배열의 이해"]
---


이 포스팅은 이전 네이버 블로그의 [해당 게시물](https://blog.naver.com/lja3333/222145277147){: target='_blank'}에서 마이그레이션되었다.

이 글에서 다룰 연산자는 크게 3가지로, 배열 첨자 연산자(`[ ]`), 단항 간접 참조 연산자(`*`), 단항 주소 연산자(`&`)이다. 

## 1. 첨자 연산자(subscript operator): 배열 원소의 접근​

배열 선언 후 배열의 원소에 접근하려면 첨자 연산자(subscript operator) `[ ]` 를 사용한다.

<!-- omit from toc -->
### examples
{: data-toc-skip=''}

```c
char str[] = "multidimentional array";
char ch = str[7]; //ch의 값은 'm'
```
{: .nolineno }

---

### 1.1. 배열 첨자 연산자와 간접 참조 연산자(indirection operator) 간의 자유로운 변환

첨자 연산자의 정의에 따르면, 표현식 `a[b]`는 표현식 `*((a) + (b))`와 동일한 표현이다. 따라서 `str[7]`은 `*(str + 7)`으로, 또 `*(str + 7)`은 `str[7]`으로 자유롭게 변환이 가능하다.

<!-- omit from toc -->
### a[b] == b[a]​
{: data-toc-skip=''}

`a + b == b + a` 와 같은 덧셈의 교환 법칙에 의해, 표현식 `a[b]`, 즉 `*((a) + (b))`는 `*((b) + (a))`로 쓸 수 있으며 이는 표현식 `b[a]`와 같다. 따라서 아래와 같은 상당히 이상해 보이는 표현도 오류 없이 컴파일이 가능하고, 실제로 잘 동작한다!

<!-- omit from toc -->
### examples
{: data-toc-skip=''}

```c
char ch = 7[str]; //str[7]과 동일함, ch의 값은 'm'
```
{: .nolineno }

---

## 2. 간접 참조 연산자(역참조 연산자, indirection operator)의 성질

단항 연산자인 간접 참조 연산자(`*`)는 포인터가 가리키는 변수의 값을 읽어오는데 사용하는 연산자이다. 또한 역참조 연산자(dereference operator)라고도 한다. 간접 참조 연산자의 성질은 다음과 같다.

1. 간접 참조 연산자의 피연산자의 형식은 반드시 포인터 형식이이어야 한다. 이때 피연산자는 lvalue, rvalue 모두 가능하다.
2. 간접 참조 연산자의 연산 결과는 포인터가 가리키는 위치를 지정하는 lvalue이고, 그 형식은 피연산자가 '`T`를 가리키는 포인터'일 경우 `T`이다.

<!-- omit from toc -->
### examples
{: data-toc-skip=''}

```c
double num = 2.728182; 
double* p = &num; 
double n2 = *p + 5; //역참조 연산자의 사용(*p)
```
{: .nolineno }

위 코드에서 `p`는 포인터 형식이므로 간접 참조 연산자 `*`의 피연산자가 될 수 있다. 그리고 `*p`의 연산 결과는 포인터가 가리키는 위치를 지정하는 lvalue이므로 `*p`는 num을 의미하며 그 형식은 `p`가 '`double`을 가리키는 포인터'이므로 `double *`이다.

---

## 3. 주소 연산자(address operator)의 성질

단항 연산자인 주소 연산자(`&`)는 피연산자의 주소를 반환한다. 성질은 다음과 같다.

1. 주소 연산자의 피연산자는 lvalue이어야 한다.
2. 주소 연산자의 연산 결과는 rvalue이며, 형식은 피연산자의 형식에서 파생된 포인터 형식이다.

<!-- omit from toc -->
### examples
{: data-toc-skip=''}

```c
double num = 2.728182; 
double* p = &num; //주소 연산자의 사용(&num)
```
{: .nolineno }

`num`은 lvalue이므로 `&`의 피연산자가 될 수 있다. `&num`의 연산 결과는 rvalue이며, 피연산자의 형식이 `double`이므로 `&num`의 형식은 `double`에서 파생된 포인터 형식인 `double *` 이다.

---

<!-- omit from toc -->
## 참고 문서
{: data-toc-skip=''}

- [[#] 첨자 연산자의 성질](https://www.ibm.com/support/knowledgecenter/en/ssw_ibm_i_71/rzarg/arsubex.htm){: target='_blank'}
- [[#] 주소 연산자의 성질](https://docs.microsoft.com/ko-kr/cpp/cpp/address-of-operator-amp?view=msvc-160){: target='_blank'}
- [[#] 간접 참조 연산자의 성질](https://docs.microsoft.com/ko-kr/cpp/c-language/indirection-and-address-of-operators?view=msvc-160){: target='_blank'}