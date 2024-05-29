---
title: "[배열의 이해] 4. 배열의 포인터로의 붕괴(array to pointer decay)"
date: 2024-03-13 15:58:00 +0900
categories: [Program-Language, C]
tags: ["배열의 이해"]
---


이 포스팅은 이전 네이버 블로그의 [해당 게시물](https://blog.naver.com/lja3333/222145547884){: target='_blank'}에서 마이그레이션되었다.

[앞선 글(1. 배열 형식(array type))]({% link _posts/Program-Language/C/2024-03-12-understanding-array-1-array-type.md %})에서 배열은 포인터가 아닌 말 그대로 배열이라고 하였다. 그런데도 왜 보통 배열의 이름은 포인터라고 하는 것일까? 그것은 배열의 포인터로의 붕괴가 일어나면 결과적으로 배열의 이름은 포인터과 같아지기 때문(equivalent)이다. 그러나 이것은 배열의 이름과 포인터가 서로 동등하다(identical)는 것은 아니다.

## 1. 배열의 포인터로의 붕괴(array to pointer decay)

**배열의 포인터로의 붕괴(array to pointer decay)**, 또는 **배열의 붕괴(array decay)**는 배열 형식(array type)이 표현식에 나타날 때 **배열의 첫 번째 원소를 가리키는 포인터로 변환되는 현상**을 말한다. 이때 **포인터의 형식은 배열의 원소의 형식**이며, 변환의 결과는 rvalue이다. 다시 말하면 배열 형식이 표현식에 나타날 때, 그 즉시 값이 배열의 첫 번째 원소의 주소값이고 형식이 배열의 원소 형식에 대한 포인터인 rvalue로 변환된다. 이러한 변환은 C/C++의 암시적 형변환(implicit conversion) 중 하나이다. 또한 변환의 결과가 rvalue이므로, **변환 결과로 생긴 포인터를 위한 별도의 메모리 공간이 따로 할당되는 것은 아니라는 것**을 알 수 있다!

<!-- omit from toc -->
### examples
{: data-toc-skip=''}

```c
char str[] = {'d', 'e', 'c', 'a', 'y'};
char* ptr = str;
printf("%c", str[2]);
```
{: .nolineno }

`str`의 형식은 `char [5]`으로, 배열 원소의 형식은 `char`이고 배열의 길이는 5이다. `char* ptr = str;` 에서 배열 형식 `str`이 표현식에 나타났다. 따라서 `str`은 배열의 포인터로의 붕괴(array to pointer decay) 현상에 의해 형식이 `char *` 이고 값이 배열의 첫 번째 원소의 주소값인 rvalue로 붕괴가 일어난다. 이 값은 곧 대입 연산자에 의해 `char*` 형 변수 `ptr`에 할당된다.

`printf("%c", str[2]);` 표현식에서 `str[2]`가 해석되는 과정을 살펴보자. 

1. 배열 첨자 연산자(subscript operator)가 연산자 정의에 의해 역참조 연산자 표현식 `*((str) + (2))` 으로 대체된다. 
2. 배열 형식 `str`이 표현식에 나타났으므로 `str`은 배열의 붕괴에 의해 값이 배열의 첫 번째 원소의 주소값이고 형식이 `char *` 인 rvalue로 변환된다.
3. `str + 2` 표현식에 의해 포인터 덧셈 연산이 수행되어 값이 배열의 세 번째 원소의 주소값이고 형식이 `char *` 인 rvalue가 된다. 
4. 이 값에 간접 참조 연산자가 적용되어 형식이 `char`이고 배열의 세 번째 원소를 의미하는 lvalue가 된다. 다섯째, lvalue는 lvalue conversion에 의해 값 `'c'`로 대체된다. 
 
따라서 `printf("%c", str[2]);`는 이러한 연쇄적인 연산에 의해 `printf("%c", 99);`가 되어 콘솔창에 `c`가 출력된다.

---

### 1.1. 배열의 붕괴의 3가지 예외

배열의 붕괴는 항상 일어나는 것은 아니다. 3가지 예외가 있는데 그것은 아래와 같다.

1. 배열 형식이 sizeof 연산자의 피연산자로 쓰일 경우
2. 배열 형식이 단항 주소 연산자 & 의 피연산자로 쓰일 경우
3. 문자 배열 형식을 문자 리터럴(string literal)으로 초기화(initialize)할 때 

<!-- omit from toc -->
### examples
{: data-toc-skip=''}

```c
int arr[5];
printf("%lu\n", sizeof(arr));
printf("%lu\n", sizeof(arr + 2));
```
{: .nolineno }

두 표현식 `printf("%lu\n", sizeof(arr));` 과 `printf("%lu\n", sizeof(arr + 2));` 를 보자. 이들은 64-bit 운영체제에서 각각 20과 8을 출력한다. 첫 표현식의 `sizeof(arr)` 에서 `arr`은 포인터로 붕괴하지 않고 배열 형식을 유지한다(예외 1번), 반면 두 번째 표현식의 `sizeof(arr + 2)`에서 `arr`은 이미 `arr + 2` 표현식에서 포인터로 붕괴하였고, 덧셈 연산자의 결과도 역시 포인터 형식이다.  따라서 `sizeof`의 결과로 포인터 형식의 크기인 8이 출력된다. (예외 2번에 대한 예시는 추후에 다룰 것이다.)

---

## 2. 배열의 붕괴로 인한 배열의 길이 정보의 손실

배열 형식은 3가지 예외를 제외하고 형식이 배열의 원소에 대한 포인터이고 값이 배열의 원소의 첫 번째의 주소값인 rvalue로 붕괴한다고 하였다. 이런 변환의 결과를 자세히 살펴보면 배열의 원소 형식에 대한 정보는 남아있지만, 배열의 원소의 개수, 즉 **배열의 길이에 대한 정보가 손실되었다**는 것을 알 수 있다. 이러한 이유로 배열이 포인터로 변환하는 현상을 배열의 붕괴(decay)라고 명명한 것이다.

<!-- omit from toc -->
### examples
{: data-toc-skip=''}

```c
#include <stdio.h>

void func(unsigned short h[]) {
    printf("%lu", sizeof(h));
}

int main(void) {
    unsigned short k[7];
    func(k);
    return 0;
}
```

위 프로그램을 컴파일하고 실행시켜 보면 8을 출력한다. `unsigned short [7]` 형식의 `k`가 함수 `func`의 argument로 전달되는데, 이 과정에서 배열의 붕괴가 발생하여 `k`는 형식이 `unsigned short*` 이고 값이 배열의 첫 번째 원소의 주소값인 rvalue가 된다. 따라서 `func` 내부에서 `sizeof(h)`를 하게 되면 포인터의 크기인 8이 출력된다. 배열의 길이 정보가 손실된 것이다.

### 2.1. 배열을 함수의 매개변수로 전달할 때 길이 정보를 전달하는 이유

이처럼 배열의 이름은 배열의 붕괴로 인해 배열의 길이 정보가 손실되므로, 배열을 함수의 매개변수로 전달할 때 배열의 이름과 함께 배열의 길이도 전달해 주는 것이다. 배열 원소 형식이 `T`이고 배열의 원소가 `N`개인 배열 형식 `k`가 있을때, 다음 식은 항상 참이다.

```c
T k[N];
sizeof(k) / sizeof(*k) == N; // 값은 항상 참
```
{: .nolineno}

`sizeof(k)`는 배열 형식에 대한 `sizeof` 연산에 의해 `sizeof(T) * N` 과 같다. 한편 표현식 `*k` 에서 `k`는 배열의 붕괴에 의해 배열의 첫 번째 원소를 가리키는 형식이 `T *`인 rvalue로 붕괴되고, 여기에 간접 참조 연산자 `*`를 적용하므로 `*k`는 형식이 `T`인 lvalue이다. 따라서` sizeof(*k)`는 `sizeof(T)`와 같다. 즉 `sizeof(k) / sizeof(*k)`는 `(sizeof(T) * N) / sizeof(T)`이며, 배열 원소의 크기 `sizeof(T)`가 상쇄되므로 배열의 길이 `N` 만 남는다. 그래서 `sizeof(k) / sizeof(*k) == N` 은 항상 참이다.

<!-- omit from toc -->
### examples
{: data-toc-skip=''}

```c
#include <stdio.h>

void func(unsigned short h[], int size) {
    for (int i = 0; i < size; i++)
        printf("%d ", h[i]);
}

int main(void) {
    unsigned short k[7] = {1, 2, 3, 4, 5, 6, 7};
    func(k, sizeof(k) / sizeof(k[0]));
    return 0;
}
```

함수 `func`의 프로토타입(prototype)을 보면 배열 원소에 대한 포인터 매개변수와 배열의 크기 정보를 받기 위한 `int`형 매개변수가 선언되었다는 것을 알 수 있다. 따라서 함수를 호출할 때 배열의 이름과 배열의 길이 정보를 넘겨준다. `sizeof(k) / sizeof(k[0])` 는  `sizeof(k) / sizeof(*k)`와 같고 이는 k의 배열의 길이인 7과 같다. 따라서 함수 `func`의 안의 반복문은 7번 반복하며 배열의 모든 원소를 순서대로 출력한다.

---

<!-- omit from toc -->
## 참고 문서
{: data-toc-skip=''}

- [[#] 배열의 포인터로의 붕괴(1)](https://cinsk.github.io//cfaqs/html/node8.html#6.3){: target='_blank'}
- [[#] 배열의 포인터로의 붕괴(2)](https://en.cppreference.com/w/cpp/language/array#Array-to-pointer_decay){: target='_blank'}
- [[#] 배열의 포인터로의 붕괴의 예외](https://stackoverflow.com/questions/17752978/exceptions-to-array-decaying-into-a-pointer){: target='_blank'}
- [[#] 배열의 포인터로의 붕괴로 인한 길이 정보의 손실](https://int-i.github.io/cpp/2020-04-12/array-deacy/){: target='_blank'}
- [[#] 배열의 대한 오해와 답변](https://yeogue.tistory.com/24){: target='_blank'}