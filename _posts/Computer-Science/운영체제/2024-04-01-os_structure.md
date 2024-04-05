---
title: "[운영체제] 운영체제 구조"
date: 2024-04-01 15:37:00 +0900
categories: [Computer-Science, 운영체제]
tags: ['OS']
---
{% assign img_path = "/assets/img/posts/운영체제/2024-04-01-os_structure" %}


## 1. 운영체제의 구조

운영체제는 크게 나누어 **2(+1) 구조**로 볼 수 있다.

![운영체제의 구조]({{ img_path }}/1. 운영체제의 구조.png){: width="300"}

### 1.1. 커널(Kernel)

- **운영체제의 핵심**
- **좁은 의미**의 OS
- OS 핵심 기능(프로세스 / 메모리 / 저장장치 관리 등)의 집합
- 응용 SW는 HW 자원 이용하기 위해 커널에 요청해야 함

### 1.2. 인터페이스(Interface)

- **커널에 명령 전달**, 실행 결과 알려 줌
- API, System Call, Shell, GUI

### 1.3. (+ Device driver)

- 운영체제의 한 부분일 수도, 아닐 수도 있음 (정의마다 다름)
- 커널과 하드웨어를 연결시킴



## 2. 커널과 시스템 콜

> **커널이 하는 일**은 곧 **운영체제가 하는 일**이다.
{: .prompt-danger }

외부로 직접 노출이 되는 것을 방지하기 위해, **User-mode**와 **Kernel-mode**를 분리하였다. 모드가 스위칭될 때 CPU 차원에서 작동모드가 완전히 바뀐다.

![커널과 시스템 콜]({{ img_path }}/2. 커널과 시스템 콜.png){: width='400'}

### 2.1. System Call(syscall)

**시스템 콜(syscall)**이란 커널이 자신의 기능을 함수 형태로 제공해주는 것이다. 커널을 보호하기 위해 아래 작업을 수행한다.

- 입력값의 검증
- 권한의 확인
- 작업 순서 제어

API를 통해 만들어진 프로그램들은 최종적으로 이 **syscall**을 호출하는 형태가 된다.

![시스템 콜]({{ img_path }}/2.1. 시스템 콜.png){: width='500'}

![시스템 콜 2]({{ img_path }}/2.1. 시스템 콜 2.png){: width='500'}



## 3. 눈으로 보는 syscall

```c
#include <stdio.h>

int main() {
    printf("Hello Syscall!!\n");
    return 0;
}
```
{: file="sysc.c"}

위와 같은 `sysc.c`{: .filepath} 파일을 생성하고 리눅스 환경에서 아래 명령어를 실행한다.

```terminal
gcc sysc.c
strace -o sysc.log ./a.out
cat sysc.log
```

출력되는 로그를 살펴보면, 최종 출력(printf)은 syscall인 **write 함수**에 의해 수행됨을 알 수 있다. 파이썬이라 할지라도, 콘솔에 출력하기 위한 최종 방법은 **write 함수**이다. 다만 strace 로그가 훨씬 길다(= 파이썬이 느린 이유)

`strace -T option` 을 이용해 syscall별 소요 시간을 출력할 수 있다.



## 4. 시스템콜의 호출: Interface

- OS가 제공하는 **Library** 또는 **API(Application Programming Interface)**
  - xNIX: POSIX
  - Windows: Win32 API
- 대부분의 xNIC 앱들은 `libc(glibc)` 링킹해서 사용
  - Windows: `kernel32.dll` (redirect to → `ntdll.dll`)
- 확인: `ldd` 명령어
  - `libc.so.6`: 표준 C 라이브러리
  - `a.out`, `echo`, `vim`, 심지어 python까지 모두 `libc` 사용
    - 내부적으론 표준 C 라이브러리 사용한다는 의미이다.

### 4.1. 시스템콜의 Assembly 코드 확인

- `objdump` 명령어를 이용함

```terminal
objdump -M intel --disassemble=__write /lib/x86_64-linux-gnu/libc.so.6
```
- `-M intel`: Intel syntax로 출력
- `--disassemble=__write`: write 함수만 출력

---

> 시스템콜은 일반 함수와는 다르게 언어에서 직접 호출이 불가능하며, 오직 CPU에 의해서만 호출됨
{: .prompt-warning}

- `eax`에 호출하고자 하는 syscall 번호 입력(OS 내부에 indexing 되어있음)
- syscall 명령 실행 → **해당 syscall 호출**함과 동시에 **CPU는 kernel 모드로 전환**된다.

![syscall_assembly]({{ img_path }}/4.1. syscall_assembly.png){: width='500'}

- `linux/arch/x86/entry/syscalls/syscall_64.tbl
`{: .filepath} 를 열어보면 아래 그림과 같이 syscall의 indexing을 확인할 수 있다.

![syscalls]({{ img_path }}/4.1. syscalls.png)

### 4.2. 동적 링킹, 정적 링킹

`printf()`의 내부를 살펴보자.

```c
printf(...) {
    ...
    write(); //이처럼 syscall을 포함하고 있다.
             //그러면 이 함수의 정의는 어디에 있는 것인가?
    ...
}
```

`write()` 함수의 정의는 과연 어디에 있을까? 바로 리눅스에서는 `libc`, 윈도우에서는 `dll` 파일 내에 들어 있다. 왜 프로그램에 없고 이렇게 따로 빼놓았을까?

`write()` 함수를 매번 프로그램에 포함하는 것보다. 미리 만들어(컴파일) 둔 뒤 필요할 때마다 호출하는 것이 공간적으로 더 효율적일 것이다. 이때 호출을 위해서는 프로그램과 `libc(dll)`을 연결해야 하는데 이것을 링킹(linking)이라 한다.

- **dynamic linking**: 소스코드와 dll을 연결하는 것
  - 멀리 갖다 와야되므로 시간적으로 손해
- **static linking**: dll의 내용을 프로그램 내에 완전히 포함하는 것



## 5. 디바이스 드라이버

커널과 하드웨어 그 중간 어딘가에 위치한다.

- **API** → User-Level에 대한 인터페이스
- **디바이스 드라이버** → 하드웨어에 대한 인터페이스
- 하드웨어와 직접 연결되며, 신호에 대한 약속을 정하는 것
  - OS가 그 기준을 정의

![디바이스 드라이버]({{ img_path }}/5. 디바이스 드라이버.png)



## 6. 커널의 형태

- 어떻게 설계하느냐에 따라 모양이 조금씩 달라짐
- 현대에서는 **단일형 커널**과 **계층형 커널**을 하이브리드로 사용하기도 한다.
- 종류: **단일형 커널**, **계층형 커널**, **마이크로 커널**

### 6.1. 단일형 커널

- **초창기 구조**
- 커널의 모든 기능이 한 덩어리로 되어 있음
  - 마치 main 함수에 모든 코드가 들어있는 느낌

![단일형 커널]({{ img_path }}/6.1. 단일형 커널.png){: width='400'}

### 6.2. 계층형 커널

- **대부분의 구조**
- 비슷한 기능을 가진 모듈별로 계층을 만듦 → Kernel stack

![계층형 커널]({{ img_path }}/6.2. 계층형 커널.png){: width='400'}

### 6.3. 마이크로 커널

- 커널은 **아주 기본적 기능**만 제공
- 대부분의 기능은 User-level로 이식, 프로세스간 통신으로 시스템 운영

![마이크로 커널]({{ img_path }}/6.3. 마이크로 커널.png){: width='400'}



## 7. 가상머신

![가상머신]({{ img_path }}/7. 가상머신.png){: width='400'}

OS마다 제공하는 **API/syscall**의 형태가 다르기 때문에, OS가 다르면 프로그램 호환이 안 되는 문제가 발생했다. 이런 문제를 해결하기 위해 등장한 것이 **가상머신**으로, **OS**와 **응용 프로그램** 사이에서 작동한다.

- OS간 이식성 증가 → 하나의 프로그램, 다양한 환경 지원
- 일종의 API 호환 레이어
- 예시
  - Java JVM
  - .NET Framework
  - Python
  - LLVM 등

---

가상머신은 **가상화(Virtualization)**와 다른 듯 하지만, 같은 개념이다.

> 무엇인가를 "에뮬레이션" 하는 것이라면 어떤 SW이든 가상머신이라 할 수 있다.
{: .prompt-info}

- **에뮬레이션**: 대상이 되는 sys를 다른 sys로 구현/재구현한 것
- **시뮬레이션**: 대상이 되는 sys를 추상화한 것(Output만 모방)
- 공통점: 실제 HW를 사용할 필요 없이 작업 처리 환경 제공

Virtualization의 가상머신은 **컴퓨터 HW 시스템을 SW로 구현한 것**이고, 여기서의 가상머신은 **특정 플랫폼의 API 시스템을 SW로 구현한 것**이라는 차이점이 있을 뿐이다.