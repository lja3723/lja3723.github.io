---
title: "[배열의 이해] 1. 배열 형식(array type)"
date: 2024-03-12 22:25:00 +0900
categories: [Program-Language, C]
tags: ["배열의 이해"]
---


이 포스팅은 이전 네이버 블로그의 [해당 게시물](https://blog.naver.com/lja3333/222144304364){: target='_blank'}에서 마이그레이션되었다.

## 1. 배열 형식(array type)

- **배열은 \<배열\> 이다**

```c
long double k;
double *st;
int arr[6];
```
{: .nolineno }

위 코드는 여러 변수의 선언문이다. 어떤 변수의 선언문을 보고 그 변수의 타입을 알기 위해서는 선언문에서 변수의 이름을 제외하면 된다. 그러므로 `k` 의 타입, `st` 의 타입은 각각 `long double`, `double *`이다. 그러면 `arr` 의 타입은 무엇일까? 보통 배열의 이름은 포인터라고 알려져 있기에 `int *` 라고 생각할 수도 있겠다. 그러나 틀렸다. 예상했을 수도 있겠지만, `arr` 의 타입은  `int [6]` 이다. 이것은 '원소의 형식은 `int` 이고, 원소의 개수(a.k.a. 배열의 길이)가 `6`인 배열'이라는 의미이다. 이렇게 첨자([], subscript)와 형식(type)으로 표현된 새로운 형식은 배열에 대한 형식, 배열을 표현하는 형식이라는 의미에서 **배열 형식(array type)**이라고 한다. 다른 표현으로는 **배열의 형식**, **배열 자료형**, **배열형** 등이 있다. 그러므로 배열은 포인터 같은 것이 아니라 말 그대로, '배열'이다.

### examples
{: data-toc-skip=''}

```c
float y[3]; // y의 타입은 float [3]
const char* str[8]; 
//str의 타입은 const char* [8] - 원소의 형식은 const char*, 원소의 개수가 8개인 배열이라는 의미
```
{: .nolineno }

---

## 2. 다차원(multidimensional) 배열의 본질

- **배열의 배열(arrays of arrays)**

2차원 배열은, 사실 정확하게 말하면 **배열의 배열**이다. 다시 말해 배열의 원소가 **배열**인 배열이라는 것이다.

```c
char board[7][15];  //board의 타입은 char [7][15]
```
{: .nolineno }

board의 타입은 `char [7][15]` 이다. 이것의 의미는 '원소의 형식은 `char [15]`이고 원소의 개수가 7인 배열'과 같다. 여기서 **원소의 형식이 `char [15]`인 배열 형식**이라는 것이 핵심이다. `char [15]`의 의미를 풀어서 `board` 의 타입을 다시 설명하면 다음과 같다.

> 원소의 형식은 '원소의 형식은 char 이고 원소의 개수가 15인 배열'이고, 원소의 개수가 7인 배열

<br>
```c
double coord[5][2][6]; //coord의 타입은 double [5][2][6]
```
{: .nolineno }

3차원 배열인 `coord` 의 타입은 `double [5][2][6]`다. 이는 배열의 원소가 `double [2][6]` 이고 원소의 개수가 5인 배열이라는 의미이다. '배열의 배열'의 배열인 셈이다.

이처럼 배열 형식을 표현하는 문장에서 배열 원소의 개수는 가장 왼쪽의 첨자에 적힌 수와 같고, 배열 원소의 형식은 가장 왼쪽의 첨자를 제외한 나머지 첨자와 배열 이름 왼쪽의 형식으로 이루어진다. 그런데 간혹 배열 원소의 개수를 가장 오른쪽에 적힌 수와 같다고 착각하여 배열 형식에 대해 잘못된 해석을 할 수 있다. 주의하자.

### 2.1. 〈다차원〉이라는 명칭이 붙은 이유

메모리의 구조는 선형적이므로, 2차원 배열의 원소도 메모리에 선형적으로 배치된다. 또한 2차원 배열은 본질적으로 배열의 배열임을 알았다. 그런데 왜 보통 우리는 배열의 배열이라 하지 않고 2차원 배열이라고 부르는 것일까? 그것은 배열의 배열의 원소가 2차원의 형태로 배열되어 있다고 가정해도 전혀 어색하지 않고, 사용하기 더 편리하기 때문이다. 그래서 그냥 '배열'을 1차원 배열, '배열의 배열'을 2차원 배열, '배열의 배열의 배열'을 3차원 배열, '배열의 배열의 ... 배열의 배열'을 다차원 배열이라고 명칭하게 된 것이다.

이처럼 다차원 배열의 본질은 '배열의 배열의 ... 배열의 배열'이지만, 필자는 앞으로 언급의 편의성을 위해 '배열의 배열의 ... 배열의 배열'이라는 표현 대신 다차원 배열이라는 표현을 사용하겠다.

---

## 3. 배열 형식의 크기

**sizeof 연산자의 적용**

```c
sizeof( typename or variable or lvalue or rvalue )   //sizeof 연산자의 사용법
```
{: .nolineno }

`sizeof` 연산자는 임의의 변수가 메모리를 차지하는 크기를 바이트 단위로 알려준다. 사용법은 위와 같다. 괄호 안의 `typename or variable or lvalue` 에는 크기를 알고 싶은 타입의 이름이나 타입형 변수, 또는 `lvalue` 나 `rvalue` 를 적어준다. (`lvalue` 와 `rvalue` 에 대해서는 추후에 언급한다.)

### examples
{: data-toc-skip=''}

```c
sizeof(int);   //int형은 4바이트이므로 결과는 4
sizeof(double);  //8
sizeof(short);  //2
long double k;
sizeof(k);   //k는 long double형 변수이므로 결과는 16
```
{: .nolineno }

---
<br>배열형에 대하여 `sizeof` 연산자를 적용할 경우, 그 결과는 `sizeof( 배열의 원소의 형 ) * (배열의 원소 개수)` 이다.

### examples
{: data-toc-skip=''}

```c
short toon[12];
sizeof(toon);  
//toon 배열은 원소가 short형이고 원소의 개수가 12이므로 결과는 sizeof(short) * 12 = 24
sizeof(unsigned char [7]);  
//배열의 원소가 unsigned char이고 원소의 개수가 7인 배열형이므로 결과는 sizeof(unsigned char) * 7 = 7
```
{: .nolineno }

---

### 3.1. 다차원 배열형에 대한 sizeof 연산자의 적용

다차원 배열형에 대해  `sizeof` 연산자를 적용할 때에도 1차원 배열에 대해  `sizeof` 연산자를 적용한 것과 동일한 규칙(`sizeof( 배열의 원소의 형 ) * (배열의 원소 개수)`)이 적용된다.

### examples
{: data-toc-skip=''}

```c
unsigned short gt[3][8][4];
sizeof(gt);
```
{: .nolineno }

1. `gt` 의 형식은 `unsigned short [3][8][4]`이므로 배열 원소의 타입은 `unsigned short [8][4]`, 원소의 개수는 3이다. 따라서 `sizeof(gt)`는 `sizeof(unsigned short[8][4]) * 3`
2. `unsigned short [8][4]`는 배열의 원소 타입이 `unsigned short [4]`이고 원소의 개수는 8인 배열형이므로 `sizeof(unsigned short[8][4])`는 `sizeof(unsigned short [4]) * 8`
3. `unsigned short [4]`는 배열의 원소의 타입이 `unsigned short` 이고 원소의 개수는 4인 배열형이므로 `sizeof(unsigned short [4])`는 `sizeof(unsigned short) * 4​`
4. 따라서 아래가 성립한다.
 - `sizeof(gt)` 
 - $\equiv$ `sizeof(unsigned short [3][8][4])` 
 - $\equiv$ `sizeof(unsigned short[8][4]) * 3` 
 - $\equiv$ `sizeof(unsigned short [4]) * 8 * 3` 
 - $\equiv$ `sizeof(unsigned short) * 4 * 8 * 3` 
 - $\equiv$ `2 * 4 * 8 * 3` 
 - $\equiv$ `192`

위의 과정 1. ~ 4. 의 연쇄적인 연산을 간략화 및 일반화하면 다음과 같다.

| `arr` 이 아래와 같은 $N$차원 배열$(N \geq 1)$이라고 하자. (단 $T$는 배열형이 아닌 형식이고 $s1, s2, ... , sN$은 자연수) <br>그러면 `sizeof(arr) == sizeof(T) * s1 * s2 * ... * sN` 의 값은 항상 참이다. |

```c
T arr[s1][s2]...[sN];
sizeof(arr) == sizeof(T) * s1 * s2 * ... * sN  // 값은 항상 참이다
```
{: .nolineno }

## 참고 문서
{: data-toc-skip=''}

- [[#] 배열형이란?](https://www.geeksforgeeks.org/arrays-in-c-cpp/?ref=lbp){: target='_blank'}
- [[#] 배열의 선언](https://docs.microsoft.com/ko-kr/cpp/c-language/array-declarations?view=msvc-160){: target='_blank'}
- [[#] 다차원 배열](https://modoocode.com/20){: target='_blank'}
- [[#] 배열의 배열과 배열의 메모리 구조](http://www.tcpschool.com/c/c_array_oneDimensional){: target='_blank'}