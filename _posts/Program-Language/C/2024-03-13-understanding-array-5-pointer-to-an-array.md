---
title: "[배열의 이해] 5. 배열 형식 포인터(pointer to an array)"
date: 2024-03-13 16:35:00 +0900
categories: [Program-Language, C]
tags: ["배열의 이해"]
---


이 포스팅은 이전 네이버 블로그의 [해당 게시물](https://blog.naver.com/lja3333/222267225300){: target='_blank'}에서 마이그레이션되었다.

## 1. 배열 형식 포인터(pointer to an array)

`int`, `double`, `unsigned long`, `char` 와 같은 C/C++에서 기본적으로 제공하는 **기본 형식(fundamental types)**에 대한 포인터 변수를 선언하고 사용하는 방법은 간단하다.

```c
int num1 = 35;
double kg = 67.1;
char ch = 't';

int * pint = &num1;   //int형 변수를 가리키는 포인터 int *
double * pdouble = &kg;   //double형 변수를 가리키는 포인터 double *
char * pchar = &ch;   //char형 변수를 가리키는 포인터 char *
```
{: .nolineno }

이처럼 타입 `T`를 가리키는 포인터, 즉 `T`형 포인터의 형식은 `T *` 이다. 그러면 [배열 형식(array type)(이전 글 참고)]({% link _posts/Program-Language/C/2024-03-12-understanding-array-1-array-type.md %})을 가리키는 포인터도 가능할까? 물론이다. 배열 형식도 하나의 형식이므로, 배열 형식을 가리키는 포인터, **즉 배열 형식 포인터(pointer to an array)**또는 **배열 포인터(array pointer)**를 선언하고 사용할 수 있다. 우선 배열의 원소가 int이고 배열의 길이가 6인 1차원 배열 형식 `int [6]`의 포인터를 살펴보자.

```c
int arr[6];
int (*parr)[6] = &arr;  //int [6]형 변수 arr를 가리키는 포인터 int (*)[6];
//int *parr[6] == &arr;    //잘못된 선언(배열의 원소가 int *이고 배열의 길이가 6인 배열)
```
{: .nolineno }

`int [6]` 형 포인터 `parr`의 선언은 `int (*parr)[6];` 과 같다. 여기서 괄호는 꼭 필요한데, 왜냐하면 배열 첨자 연산자(`[ ]`)의 우선순위가 간접 참조 연산자(`*`)보다 더 높기 때문이다. 따라서 `int *parr[6]` 와 같이 괄호를 생략해 버리면, 컴파일러는 이를 포인터 선언이 아닌 배열의 선언으로 해석하게 된다. 즉 `int *parr[6]` 는 `int [6]`형 변수를 가리키는 포인터가 아니라 배열의 원소가 `int *`이고 배열의 길이가 6인 배열, 즉 포인터의 배열이 되는 것이다.

한편 변수의 선언문에서 변수의 형식을 알기 위해서는 변수의 이름을 생략하면 되므로 `int (*parr)[6];`으로 선언된 포인터 변수 `parr`의 형식은 `int (*)[6]`이다. 이것을 일반화하여, 배열의 원소가 `T`형이고 배열의 길이가 `N`인 배열 형식 `T [N]`를 가리키는 포인터, 다시 말해` T [N]` 형식의 포인터 선언은 다음과 같고, 형식은` T (*)[N]` 이다.

```c
T arr[N];
T (*ptr)[N] = &arr;   //배열 형식 포인터; ptr의 형식은 T (*)[N]
```
{: .nolineno }

---

### 1.1. 배열 형식 포인터의 간접 참조

이제 배열 형식 포인터에 간접 참조 연산자(indirection operator, `*`)를 적용하면 어떻게 되는지 살펴보자.

```c
int arr[6] = {1, 2, 3, 4, 5, 6};
int (*ptr)[6] = &arr;
*ptr;    //간접 참조 연산자 적용
```
{: .nolineno }

포인터 변수 `ptr`은 `int [6]`형 변수 `arr`의 시작 메모리를 가리키는 포인터이다. 여기에 간접 참조 연산자를 적용한 `*ptr`은 간접 참조 연산자의 성질에 의해, 포인터가 가리키는 위치(즉 배열 `arr`)를 지정하는 형식이 `int [6]` (`ptr`이 가리키는 변수의 형식이 `int [6]`이므로) 인 `lvalue`이다. 그런데 배열 형식은 포인터로 붕괴하므로, `*ptr`은 간접 참조 연산이 수행되는 즉시(immediately) 배열의 첫번째 원소를 가리키는 포인터로 붕괴되어 형식이 `int *`인 포인터가 된다.[^1-1] 이제 이러한 배열 형식 포인터의 산술 연산을 살펴보자.

[^1-1]: 여기서 간접 참조 연산자는 배열 붕괴의 예외사항이 아니므로 배열이 포인터로 붕괴하였다.

---

## 2. 배열 형식 포인터의 산술(Arithmetic) 연산

### 2.1. 기본 형식 포인터의 산술연산

우선 기본 형식에 대한 포인터의 산술 연산을 짚고 넘어가겠다. 다음 코드를 보자.

```c
char* ptr1 = (char*)0x100;   //임의의 값 저장(실무에서는 비권장)
int* ptr2 = (int*)0x200;     //임의의 값 저장(실무에서는 비권장)
printf("%#x %#x\n", ptr1, ptr1 + 2); //0x100 0x102 출력
printf("%#x %#x\n", ptr2, ptr2 + 2); //0x100 0x108 출력
```
{: .nolineno }

포인터가 아닌 형식의 산술연산은 수학적 산술 연산과 동일한 반면, 기본 형식 포인터의 산술 연산은 조금 다르다. 형식 `T`에 대한 포인터 `ptr` 의 값을 `n` 만큼 증가시키면 `ptr`의 값은 `n` 만큼 증가하는 것이 아니라 `sizeof(T) * n` 만큼 증가하고, 마찬가지로 `n` 만큼 감소시키면 `sizeof(T) * n` 만큼 감소한다.

### 2.2. 배열 형식 포인터의 산술연산

```c
short (*ptr)[7] = (short (*)[7])250;    //임의의 값 저장(실무에서는 비권장)
printf("%d %d %d\n", ptr, ptr + 3, ptr - 2);   //250 292 222 출력
```
{: .nolineno }

기본 형식 포인터의 산술연산의 원리는 배열 형식 포인터의 산술연산에도 동일하게 적용된다.

`short (*ptr)[7] = (short (*)[7])250` :  `int`형 상수 250을 포인터 변수에 그대로 할당하지 않고, 명시적 형변환을 통해 `short [7]` 형을 가리키는 포인터 형식 `short (*)[7]` 으로 변환하여 할당하는 것으로, 이로써 `ptr`은 주소가 250인 메모리 공간을 가리키게 된다.

`ptr`이 가리키는 형식이 `short [7]`이므로 `ptr + 3`의 값은 `250 + sizeof(short [7]) * 3 == 250 + 14*3 == 292`이고, `ptr - 2` 의 값은 `250 - 14*(2) == 222`임을 쉽게 알 수 있다.

이처럼 포인터가 가리키는 형식이 기본 형식이든지 배열 형식이든지 간에, 산술연산의 원리는 동일하다는 것을 알았다. 다음은 포인터에 대한 산술연산의 성질을 정리한 것이다.

- 형식 `T`에 대한 포인터 `ptr`(즉 `T *` 타입)이 선언되어 있고 여기에 적절한 주소값 `k`가 할당되어 있을 때, 산술연산 `ptr ± n`(`n`은 정수 계열 형식) 의 결과는 형식이 `T *`이고 값이 `k ± sizeof(T) * n` 인 rvalue이다.

---

지금까지 살펴본 배열 형식 포인터에 대한 내용을 정리해 보자.

<!-- omit from toc -->
### examples
{: data-toc-skip=''}

```c
#include <stdio.h>

int main(void)
{
    int arr[6] = { 1, 2, 3, 4, 5, 6 };
    int(*ptr)[6] = &arr;

    printf("%lu\n", sizeof(*ptr));     //24 출력
    printf("%lu\n", sizeof(*ptr + 2));  //8(32비트 시스템에서는 4) 출력
    printf("%d\n", (*ptr)[2]);  //3 출력
    printf("%d\n", ptr[0][4]);  //5 출력

    int diff1 = (int)(ptr + 2) - (int)ptr;
    int diff2 = (int)(*ptr + 2) - (int)*ptr;
    printf("%d %d\n", diff1, diff2); // 48 8 출력

    return 0;
}
```

`sizeof(*ptr)`의 값이 24인 것을 통해 우리는 `*ptr`의 형식이 `int [6]`임을 알 수 있다.[^decay-exception] 반면 `sizeof(*ptr + 2)`의 값은 8[^32bit]인데, 왜냐하면 `*ptr + 2`에서 가장 먼저 `*ptr`이 포인터로 붕괴한 후 포인터 덧셈이 수행되고, 그 결과는 포인터이기 때문이다.[^result-pointer]

표현식 `(*ptr)[2]`를 보자. 괄호 연산이 먼저 수행되므로 `*ptr`은 배열의 첫 원소를 가리키는 포인터이고, 여기에 첨자 연산자 `[2]`를 적용하면 배열 `arr`의 세 번째 원소인 `arr[2]`를 의미하는 lvalue이므로 값은 3이다.[^operator-order]

이제 표현식 `ptr[0][4]`을 보자. 첨자 연산자는 왼쪽에서 오른쪽으로 평가되므로 `ptr[0]`이 먼저 해석된다. `ptr[0]`은 첨자 연산자의 정의에 의해 `*((ptr) + (0)) == *ptr` 이므로 `ptr[0][4]`은 `(*ptr)[4]`이다. 이것은 `arr[4]`를 의미하는 lvalue이므로 값은 5이다.

마지막으로 `diff1`과 `diff2`의 값을 살펴보자. `diff1`은 값 `ptr + 2`와 `ptr`의 차이를 저장한다. `ptr`은 배열 `arr`의 시작 주소값(이 값을 `k`라 하자)이 저장된, `int [6]`을 가리키는 포인터이므로 `ptr + 2`의 값은 `k + sizeof(int [6]) * 2` 이다. 그러므로 `diff1`의 값은 `sizeof(int [6]) * 2 == 48`이다. 반면 `diff2`는 값` *ptr + 2`와 `*ptr`의 차이를 저장한다. `*ptr`은 `k`가 저장된 `int *`형 포인터이므로 `*ptr + 2`의 값은 `k + sizeof(int) * 2` 이며, 따라서 `​diff2`의 값은 `sizeof(int) * 2 == 8`이다.

[^decay-exception]: 배열 붕괴의 예외로 인하여 포인터로 붕괴하지 않았기 때문이다.
[^32bit]: 32비트 시스템에서는 4
[^result-pointer]: 덧셈의 피연산자중 한 피연산자가 포인터 형식이면 덧셈 연산 결과의 형식도 포인터 형식이다.
[^operator-order]: 표현식 `*ptr`를 괄호로 감싸는 이유는 앞서 말했듯 배열 첨자 연산자(`[ ]`)의 우선순위가 간접 참조 연산자(`*`)보다 더 높기 때문이다.

---

### 2.3. 배열의 붕괴의 3가지 예외 - 단항 주소 연산자 & 

이제 배열 형식 포인터를 배웠으므로, [저번 글(4. 배열의 포인터로의 붕괴)]({% link _posts/Program-Language/C/2024-03-13-understanding-array-4-array-to-pointer-decay.md %})에서 설명을 미루었던 배열의 붕괴 예외 2번, **배열 형식이 단항 주소 연산자 `&` 의 피연산자로 쓰이는 경우**를 설명하겠다. 위 코드를 다시 보자.

```c
int arr[6];
int (*parr)[6] = &arr;  //int [6]형 변수를 가리키는 포인터 int (*)[6];
```
{: .nolineno }

배열 형식이 단항 주소 연산자 `&`의 피연산자로 쓰이는 경우에는 예외에 의해 배열의 붕괴 현상이 발생하지 않는다. 그 이유는 배열 형식에 대해 `&` 연산자를 적용할 때 배열 형식에 대한 포인터 형식이 되어야 하기 때문이다. 만약 이런 예외가 없다면 어떻게 될까?

배열 형식이 `&` 연산자의 피연산자에 쓰이는 경우에도 배열의 붕괴 현상이 일어난다고 가정해 보자. 그러면 위 코드에서 `arr`은 배열의 원소 `int`를 가리키는 포인터 `int *`가 되고, `&` 연산자가 적용이 된` &arr`은 `int *`에 대한 포인터 형식, 즉 이중 포인터 `int **`이 될 것이다. 따라서 이렇게 잘못된 포인터 형식이 도출되는 것을 방지하기 위해, 배열 형식이 `&`의 피연산자로 쓰이는 경우를 배열의 붕괴의 예외로 명시한 것이다.

---

## 3. 다차원 배열 형식 포인터(pointer to a multidimensional array)

지금까지 1차원인 배열 형식을 가리키는 포인터에 대해 알아보았다. 이번에는 다차원인 배열 형식을 가리키는 포인터, **다차원 배열 형식 포인터(pointer to a multidimensional array)** 또는 **다차원 배열 포인터**에 대해 살펴볼 것이다.

```c
int arr[2][3];
int (*ptr)[2][3] = &arr;  //int [2][3]형 변수 arr를 가리키는 포인터 int (*)[2][3];
```
{: .nolineno }

다차원 배열 형식 포인터도 1차원 배열 형식 포인터와 크게 다르지 않다. 2차원 배열 `int [2][3]` 을 가리키는 포인터 `ptr`의 형식은 식별자를 제거한 `int (*)[2][3]`이다. 

`T`는 배열이 아닌 형식이며 `arr` 이 `N`차원 배열일 때($N \geq 1$ 인 자연수), `arr`을 가리키는 포인터의 선언을  일반화하면 다음과 같다. (단 $s1, s2, ..., sN$은 자연수)

```c
T arr[s1][s2]...[sN];           //arr의 형식은 T [s1][s2]...[sN]
T (*ptr)[s1][s2]...[sN] = &arr; 
//배열 형식 arr를 가리키는 포인터; ptr의 형식은 T (*)[s1][s2]...[sN]
```
{: .nolineno }

### 3.1. 다차원 배열의 포인터로의 붕괴(multidimensional array decay)

다차원 배열 형식도 배열 형식이므로 배열의 붕괴가 일어난다. 즉 다차원 배열 형식이 표현식에 나타날 때, 1차원 배열 형식과 마찬가지로, 형식이 배열의 원소 형식에 대한 포인터이고 배열의 첫 번째 원소의 주소값을 값으로 가지는 rvalue로 변환된다.

<!-- omit from toc -->
### examples
{: data-toc-skip=''}

```c
unsigned int arr[2][5] = {
    {1, 2, 3, 4, 5},
    {6, 7, 8, 9, 10}
};
unsigned int (*ptr)[5] = arr;
```
{: .nolineno}

`arr`의 형식은 `unsigned int [2][5]` 이고 이것은 원소의 개수가 2이고 원소의 형식이 `unsigned int [5]` 인 2차원 배열이다. 표현식 `unsigned int (*ptr)[5] = arr;` 에서 `arr` 은 배열의 포인터로의 붕괴에 의해 형식이 배열의 원소를 가리키는 포인터인 `unsigned int (*)[5]` 가 된다. 그러므로 이것을 같은 형식을 가진 포인터 변수` ptr`에 대입했다.

### 3.2. 다차원 배열 형식 포인터의 간접 참조

지금까지 다룬 배열에 대한 내용을 이해했다면 다차원 배열 형식 포인터의 간접 참조는 그렇게 어려운 개념이 아니다.

<!-- omit from toc -->
### examples
{: data-toc-skip=''}

```c
int arr[2][3][6] = {
    {
        {1, 2, 3, 4, 5, 6},
        {7, 8, 9, 10, 11, 12},
        {13, 14, 15, 16, 17, 18}
    },
    {
        {100, 200, 300, 400, 500, 600},
        {700, 800, 900, 1000, 1100, 1200},
        {1300, 1400, 1500, 1600, 1700, 1800}
    }
};
int (*ptr)[2][3][6] = &arr;
*ptr;       //형식: int [2][3][6] -> int (*)[3][6]
**ptr;      //형식: int [3][6] -> int (*)[6]
***ptr;     //형식: int [6] -> int *
****ptr;    //형식: int
```

포인터 변수` ptr`은 `int [2][3][6]`형 변수 `arr`의 시작 메모리 주소를 저장한다. 

표현식 `*ptr`은 간접 참조 연산자의 성질에 의해 형식이 `int [2][3][6]` 인 lvalue이며, 배열의 붕괴에 의해 즉시 `int (*)[3][6]` 형 포인터(rvalue)로 변환된다. 

표현식 `**ptr`은 간접 참조 연산자의 성질에 의해 형식이 `int [3][6]` 인 lvalue이며, 배열의 붕괴에 의해 즉시 `int (*)[6]` 형 포인터(rvalue)로 변환된다. 

표현식 `***ptr`은 간접 참조 연산자의 성질에 의해 형식이 `int [6]` 인 lvalue이며, 배열의 붕괴에 의해 즉시 `int *` 형 포인터(rvalue)로 변환된다. 

마지막으로 표현식 `****ptr`은 간접 참조 연산자의 성질에 의해, 마침내 `int`형 값 하나를 저장할 수 있는 lvalue가 된다.

한편 `ptr`, `*ptr`, `**ptr`, `***ptr`(표현식 `ptr`, `ptr[0]`, `ptr[0][0]`, `ptr[0][0][0]`과 동치)은 배열 `arr`의 첫 번째 원소의 주소값을 표현하며(이들은 rvalue이므로 별도의 메모리 공간이 마련되지 않는다), `****ptr` 은 값이 1인 lvalue로 관찰 가능하고 수정 가능하다.

---

## 4. 배열 형식 포인터의 성질

위 예시를 통해, 배열 형식 포인터에는 다음과 같은 성질이 있음을 알 수 있다.

- `T`는 배열이 아닌 형식, `N`은 2 이상의 자연수, $s1, s2, ..., sN$은 자연수이고 **배열의 붕괴의 예외가 적용되지 않을 때**, 형식이 `T (*)[s1][s2]...[sN]`인 포인터 `ptr`에 대해 `*ptr`은 형식이` T (*)[s2][s3]...[sN]` 인 rvalue이다.

<!-- omit from toc -->
### examples
{: data-toc-skip=''}

```c
unsigned mean[3][5][2][8];
unsigned(*p)[5][2][8] = mean;
p[0];
p[0][0];
p[0][0][0];
p[0][0][0][0];
```

`p[0]`, `p[0][0]`, `p[0][0][0]`, `p[0][0][0][0]` 은 `*p`, `**p`, `***p`, `****p`와 동치이며, 그 형식은 각각 `unsigned int (*)[2][8]`, `unsigned int (*)[8]`, `unsigned int *`, `unsigned int` 다.

여기서 주의해야 할 점이 있는데, 배열 형식 포인터의 성질은 **"배열의 붕괴의 예외가 적용되지 않을 때 성립한다."** `p[0]`, `p[0][0]`, `p[0][0][0]`은 간접 참조 연산자의 성질에 의해 본질적으로 배열 형식이므로 배열의 붕괴의 예외가 적용되면 이 표현식들은 포인터가 아니라 배열 형식으로 간주된다. 예컨대 `sizeof(p[0][0])` 의 값은 64비트 시스템에서 8이 아니라 64이다.

<!-- omit from toc -->
## 참고 문서
{: data-toc-skip=''}

- [[#] 기본 제공 형식(fundamental types)](https://docs.microsoft.com/ko-kr/cpp/cpp/fundamental-types-cpp?view=msvc-160){: target='_blank'}
- [[#] 배열 형식 포인터](https://www.geeksforgeeks.org/pointer-array-array-pointer/){: target='_blank'}
- [[#] 포인터 산술 연산](https://docs.microsoft.com/ko-kr/cpp/cpp/additive-operators-plus-and?view=msvc-160#pointer-addition){: target='_blank'}
- [[#] 다차원 배열 포인터의 연산](https://www.geeksforgeeks.org/multidimensional-pointer-arithmetic-in-cc/){: target='_blank'}
- [[#] [배열의 이해] 1. 배열 형식(array type)]({% link _posts/Program-Language/C/2024-03-12-understanding-array-1-array-type.md %}){: target='_blank'}
- [[#] [배열의 이해] 3. 배열 및 포인터와 관련된 여러 연산자의 성질]({% link _posts/Program-Language/C/2024-03-13-understanding-array-3-operator-properties-of-array-and-pointer.md %}){: target='_blank'}
- [[#] [배열의 이해] 4. 배열의 포인터로의 붕괴]({% link _posts/Program-Language/C/2024-03-13-understanding-array-4-array-to-pointer-decay.md %}){: target='_blank'}



<br><br><br><br>

---
<!-- omit from toc -->
#### 각주
{: data-toc-skip=''}