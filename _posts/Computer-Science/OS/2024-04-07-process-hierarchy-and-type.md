---
title: "[OS] 프로세스의 계층 구조와 종류"
date: 2024-04-07 21:15:00 +0900
categories: [Computer-Science, OS]
tags: ['OS', '프로세스']
---



## 1. init: 모든 프로세스의 조상

유닉스의 모든 프로세스는 **init 프로세스의 자식**이 되어 **트리 구조**를 이룬다.
- Windows는 이렇게까진 아니지만 거의 비슷함

{% assign img_path = "/assets/img/posts/OS/2024-04-07-process-hierarchy-and-type" %}
![init]({{ img_path }}/1. init.png){: width='400'}

![init 2]({{ img_path }}/1. init 2.png){: width='580'}

부모-자식 프로세스 관계를 운영체제별로 확인해보자.

<!-- omit from toc -->
### **예시: 리눅스의 프로세스 목록**
{: data-toc-skip=''}

- `pstree -p` 명령어로 확인 가능

![예시 ]({{ img_path }}/1.1. 예시.png){: width='580'}

![예시 2]({{ img_path }}/1.1. 예시 2.png){: width='580'}
_리눅스에서의 프로세스 목록_

<!-- omit from toc -->
### **예시: 윈도우의 프로세스 목록**
{: data-toc-skip=''}

- `Process Explorer` 프로그램을 통해 확인 가능

![예시 3]({{ img_path }}/1.1. 예시 3.png){: width='580'}
_윈도우에서의 프로세스 목록_



## 2. 계층 구조

- 프로세스는 일반적으로 **부모-자식 관계**이다.
- **#0 프로세스**: 시스템 부팅 시 실행되되는 **최초의 프로세스**, **조상 프로세스**
- 모든 프로세스는 부모 프로세스를 가진다. (#0 프로세스 제외)

### 2.1. 자식 프로세스의 생성

- 모든 프로세스는 프로세스(부모)에 의해 생성된다.
  - 프로세스 생성은 syscall(`fork()`)을 통해서만 가능

### 2.2. 프로세스의 hand-craft 생성

- **PID가 0, 1, 2 등의 몇몇 조상 프로세스**는 syscall로 생성되는 것이 아님
- 부팅 시 OS 차원에서 **수작업(hand-craft)으로 생성**된다.

### 2.3. 왜 계층 구조인가?: fork() - exec()

- **여러 작업을 처리**하기에 용이함
- **프로세스의 재사용**이 용이함
- **관리**가 용이함
  - 프로세스 간 책임 관계 분명해짐 → **자원 회수 등이 쉬워진다.**



## 3. 부모 - 자식 간 실행 관계

- 부모가 자식을 생성한 후 자식의 종료를 기다리는 **정상적인 경우**

![부모 자식 간 실행 관계]({{ img_path }}/3. 부모 자식 간 실행 관계.png){: width='580'}

- 부모가 자식의 종료를 제 때 못 받아주는 경우

> 자식 프로세스는 좀비 프로세스가 된다.
{: .prompt-warning}

![부모자식 간 실행 관계 2]({{ img_path }}/3. 부모 자식 간 실행 관계 2.png){: width='580'}

### 3.1. 좀비 프로세스

**좀비 프로세스**는 비록 메모리를 차지하지는 않지만 **PCB의 낭비가 발생**한다.

- **커널 입장**: PCB 유지하면서 얼마간의 자원을 소모하는 프로세스
  - 커널이 유지할 수 있는 **PCB 테이블 크기에 제한**이 있다.
  - 많은 좀비 프로세스가 발생할 경우, **시스템 성능에 영향**이 있다.
    - ex: 프로세스 스케줄링을 위한 확인
- 시스템 모니터링 시 기분이 나빠지는 심리적인 효과까지(...)

#### 3.1.1. 좀비 프로세스의 제거

1. 부모 프로세스에게 `SIGCHLD` 신호 보내기
  - 부모 프로세스에서 `wait()`을 호출하여 처리한다.
  - 만약 부모에게 `SIGCHLD` 핸들러가 없는 경우 **좀비는 제거되지 못한다.**
2. 부모 프로세스 강제 종료 → 좀비를 **고아(Orphan)화 하기**
  - 좀비는 `init` 프로세스의 자식이 됨
    - `init`이 `wait()` 호출, 좀비 프로세스가 제거된다.

### 3.2. 고아 프로세스(Orphan Process)

- 부모가 먼저 종료한 자식 프로세스

부모 프로세스가 종료될 때 일반적으로 아래의 과정을 거친다.

1. 커널(`exit()` 호출 코드)은 자식 프로세스가 있는지 확인한다.
2. 만약 자식 있으면, **자식 프로세스(고아)를 `init` 프로세스에게 입양**시킨다.
  - **PPID가 1로 변경된다.**
  - (운영체제에 따라, 모든 자식 프로세스를 강제 종료 시키기도 한다.)

![고아 프로세스]({{ img_path }}/3.2. 고아 프로세스.png){: width='580'}

### 3.3. 부모/자식 프로세스 ID의 확인

<!-- omit from toc -->
#### **examples for xNIX OS**
{: data-toc-skip=''}
```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
    pid_t pid, ppid;

    pid = getpid();   // get a PID
    ppid = getppid(); // get a Parent's PID

    printf("Process ID = %d, Parent's ID = %d\n", pid, ppid);
}
```

실행 결과
```terminal
Process ID = 228, Parent's ID = 1
```

---

<!-- omit from toc -->
#### **examples for Windows OS**
{: data-toc-skip=''}
```c
#include <stdio.h>
#include <windows.h>
#include <tlhelp32.h>

int main()
{
    int pid = GetCurrentProcessId();

    HANDLE h = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    PROCESSENTRY32 pe = { 0 };
    pe.dwSize = sizeof(PROCESSENTRY32);

    if (Process32First(h, &pe)) {
        do {
            if (pe.th32ProcessID == pid) {
                printf("Process ID = %d, Parent's ID = %d\n",
                    pid, pe.th32ParentProcessID);
            }
        } while (Process32Next(h, &pe));
    }
    CloseHandle(h);
}
```

실행 결과
```terminal
Process ID = 22960, Parent's ID = 31780
```

---



## 4. 프로세스의 종류

### 4.1. 백그라운드 vs 포그라운드

#### 4.1.1. Background Process

- 터미널에서 실행되었지만, 터미널 사용자와의 **대화가 없는 채 실행**되는 프로세스
- 사용자와 **대화 없이 실행**되는 프로세스
- 사용자 **입력을 필요로 하지 않는** 프로세스
- **idle 상태**로 잠을 자는 프로세스
- **디스크에 swap된 상태**의 프로세스

#### 4.1.2. Foreground Process

- 실행되는 동안 **터미널 사용자의 입력을 독점**하는 프로세스

<br>

### 4.2. CPU 집중 vs I/O 집중

#### 4.2.1. CPU Intensive Process

- 대부분의 시간을 **계산 중심의 작업(CPU 작업)**을 하느라 보내는 프로세스
  - `예시`
    - 배열 곱
    - 인공지능 연산
    - 이미지 처리 등
- **CPU 속도**가 성능 좌우 (**CPU bound**)

#### 4.2.2. I/O Intensive Process

- 대부분의 시간을 **입출력 작업(I/O 작업)**을 하느라 보내는 프로세스
  - `예시` 
    - 네트워크 전송
    - 파일 입출력에 집중된 프로세스
    - 파일 서버
    - 웹 서버 등
- **입출력 장치/시스템의 속도**가 성능 좌우 (**I/O bound**)