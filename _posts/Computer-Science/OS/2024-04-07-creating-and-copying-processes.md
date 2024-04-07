---
title: "[OS] 프로세스의 생성과 복사"
date: 2024-04-07 17:35:00 +0900
categories: [Computer-Science, OS]
tags: ['OS', '프로세스']
---
{% assign img_path = "/assets/img/posts/OS/2024-04-07-creating-and-copying-processes" %}



## 1. 프로세스의 생성

### 1.1. 프로세스가 생성되는 경우

1. **시스템 부팅** 과정에서 필요한 프로세스 생성
2. 사용자 로그인 후 **사용자 제어(대화)** 위한 프로세스 생성
  - bash
  - explorer.exe
  - finder.app 등
3. 새 프로세스 생성하는 **사용자 명령** (ex: `vi hello.c`)
4. **배치 작업** 실행 시(`at`, `batch` 명령)
5. 사용자 응용 프로그램이 **syscall**로 새 프로세스 생성

> 단순히 메모리에 올라갔다고 프로세스가 되는 것이 아니다.
> - 그냥 메모리에 올라가져**'만'** 있으면 아무 의미가 없다.
> - **PCB**가 존재해 **OS가 제어 가능한 형태**가 되어야 함 → CPU 할당 받아 실행이 가능해진다. 
{: .prompt-danger}

### 1.2. 프로세스 생성 과정

1. 생성할 **실행파일 경로**를 OS에 전달
2. OS는 **메모리에 프로그램 적재**
  - **Code 영역**에 프로그램 코드 적재
  - **Data 영역**에 전역/정적 변수 할당
  - **스택/힙**은 아직 아무것도 없으므로 초기화만 시킴
3. **PCB** 공간 할당 받고(malloc), 필요 정보 채움
  - 프로세스 식별자 결정 → **새 PID 번호 할당**
  - 프로세스 정보 기록
  - 프로세스 테이블에서 새 항목 할당
  - 새로 할당된 프로세스 테이블에 PCB 연결
4. PCB에 프로세스 **상태를 ready로 표시**하고, 준비 큐에 장착

### 1.3. 복사에 의한 프로세스 생성

프로세스 생성 과정은 이처럼 할 일이 많으므로, 다른 프로세스를 생성하기 위해 **기존의 프로세스를 복사**하여 생성하는 것이 상대적으로 간편하다. 현대 운영체제는 상당수가 이를 따르고 있다.

- **Linux**: `fork()` 시스템 콜
- **Windows** `CreateProcess()` 등 시스템 콜

> 엄밀히 말하면 윈도우에는 `fork()`는 없다. `CreateProcess()`는 `fork()` + `exec()`에 가깝다.
{: .prompt-info}

UNIX 계열 OS는 시스템 부팅 시 **0번 프로세스만 자체적으로 생성**한다. **나머지 프로세스는 복제를 통해 생성**한다. 이렇게 함으로써 얻는 이점은 아래와 같다.

- 자주 사용되는 프로세스에 대해 매번 반복할 필요 X (예: `bash shell`)
- 관리상 편리해짐 (**프로세스 계층 구조**)
- Process 간 통신


## 2. fork() 시스템 콜

- **`fork()`**: 새로운 프로세스를 실행 중 프로세스로부터 복사하는 함수
- 실행 중인 프로세스와 똑같은 프로세스가 하나 더 만들어진다.
  - `int pid = fork();`
- **부모 프로세스**: fork를 호출한 프로세스
- **자식 프로세스**: fork된 프로세스
- 부모 프로세스의 **모든 환경, 메모리, PCB** 등을 복사
- 부모와 동일한 모양이지만 **독립된 주소 공간**에 위치함

> 단, PCB에서 아래 내용은 달라진다.
{: .prompt-warning}

- **PID**
- **PPID(Parent PID)**: 부모의 PID로 초기화
- **CPID(Chlid PID)**: 자식이 없을 경우 -1
- **메모리 관련 정보**: 독립된 주소 공간을 소유하기 때문

![fork() 시스템 콜]({{ img_path }}/2. fork() 시스템 콜.png){: width='550'}

### 2.1. 실행 과정

```c
pid_t pid; //pid 변수 선언

pid = fork(); //자식 프로세스 생성
              //자식 프로세스는 이 라인 밑에서부터 실행됨

if (pid > 0) {
    /* 부모 프로세스가 실행할 코드 */
}
else if (pid == 0) {
    /* 자식 프로세스가 실행할 코드 */
}
else {
    /* fork() 오류 처리하는 코드 작성 */
}
```

- 자식은 부모의 PC(Program Counter)도 복제, `pid = fork();` 그 다음 줄부터 실행된다.
  - 즉 `pid = fork();` 이전 라인은 실행이 불가능
- **`fork()` 함수의 리턴값**
  - **부모 프로세스**: 자식 프로세스의 PID (> 0)
  - **자식 프로세스**: 0
  - **Error**: 음수

![fork() 실행 과정]({{ img_path }}/2.1. fork() 실행 과정.png){: width='650'}

#### **code example for xNIX**
{: data-toc-skip=''}

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

int main() {
    pid_t pid;
    int i, sum = 0;

    pid = fork();  // fork! -> create a child process
    if (pid > 0) { // Run by the parent
        printf("Parent: fork()'s return == Child pid = %d\n", pid);
        printf("Parent: pid = %d\n", getpid());

        wait(NULL); // Wait for the child
        printf("Parent has been finished\n");
        return 0;
    } else if (pid == 0) { // Run by the child
        printf("\t-Child: fork()'s return pid = %d\n", pid);
        printf("\t-Child: pid = %d, parent's pid = %d\n", getpid(), getppid());

        for (i = 1; i <= 100; i++)
            sum += i;

        printf("\t-Child: sum = %d\n", sum);
        return 0;
    } else { // Error
        printf("fork error");
        return 0;
    }
}
```

#### **실행 결과**
{: data-toc-skip=''}

```terminal
Parent: fork()'s return == Child pid = 482
Parent: pid = 481
    -Child: fork()'s return pid = 0
    -Child: pid = 482, parent's pid = 481
    -Child: sum = 5050
```

---

### 2.2. 장점

- 프로세스 생성 **속도 빠름**
- 추가 작업 없이 **자원 상속 가능**
- 시스템 관리 효율적으로 가능 (**프로세스 계층 구조**)

### 2.3. 단점

- 매번 모든 Context의 복사본 만드는 것은 **매우 비효율적**
- 맨 처음 만든 프로그램 프로세스(0번 프로세스) 이외에는 **다른 프로그램 동작 불가**
  - **해결책**: exec() 시스템 콜 - fork() 후 실행



## 3. exec() 시스템 콜: Process Overlay

- 기존의 프로세스를 **새 프로세스로 전환(재사용)**하는 함수
- 현재 실행 중인 프로세스 주소 공간에 새로운 응용프로그램 적재 후 실행
  - `fork()`: 새 프로세스를 **복사하는 syscall**
  - `exec()`: 프로세스 그대로 둔 채 **내용만 바꾸는 syscall**
    - `execlp()`, `execv()`, `execvp()`, ...
- 실행 파일을 로딩 후, 현재 프로세스 이미지 위에 단순이 덮어쓰는 것

> 프로세스를 **새로 생성하는 것이 아니다.**
> - 프로세스 PID가 **변경되지 않는다.**
> - 프로세스 메모리 공간(코드, 데이터, 힙, 스택)에 **새 프로그램이 적재**되는 것이다.
> - **보통 `fork()` 통해 생성된 자식 프로세스가 `exec()`를 실행한다.**
> - loader가 exec를 통해 호출된다.
{: .prompt-danger}

![exec() 시스템 콜]({{ img_path }}/3. exec() 시스템 콜.png){: width='550'}

### 3.1. 실행 과정

#### 메모리
{: data-toc-skip=''}

- Code 영역에 있는 기존의 내용을 지우고, 새로운 코드를 덮어씌움
- Data 영역이 새 변수로 채워지고, 힙/스택 영역이 리셋됨

#### PCB
{: data-toc-skip=''}

- PID, PPID, CPID, 메모리 관련 정보는 유지
  - 새 프로세스가 전환 되더라도, 종료 후 부모 프로세스로 돌아올 수 있음
- **Program counter**를 비롯한 **register**, **파일 정보 등이 모두 리셋**됨
  - 오버레이된 프로그램의 맨 처음부터 실행하게 된다.

![실행 과정]({{ img_path }}/3.1. 실행 과정.png){: width='600'}

![실행 과정 2]({{ img_path }}/3.1. 실행 과정 2.png){: width='650'}



## 4. wait() 시스템 콜

- 자식 프로세스가 끝날 때까지 무한히 기다리다가, 자식 프로세스가 종료되면 그 다음 코드부터 이어서 실행을 계속할 수 있게 하는 시스템 콜
- Linux shell에서의 예:
  - foreground process 있는 경우: shell은 wait 상태
  - 프로세스 실행 중 상태에서 `Ctrl + Z` → 프로세스 일시중단 → 쉘이 wait에서 빠져나온다.

#### **code example for xNIX**
{: data-toc-skip=''}

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

int main() {
    pid_t pid;
    int status;

    pid = fork(); //새 프로세스 분기

    //자식 프로세스는 여기서부터 실행된다.
    if (pid > 0) {
        printf("Parent: Wating for the child\n");
        wait(&status); // Wait.
        printf("Parent: child's exit code=%d\n", WEXITSTATUS(status));
        return 0;
    } else if (pid == 0) {
        execlp("/bin/ls", "ls", NULL); // run ls on the child
                                       // 이게 실행되면 이 밑의 코드는 싹 무시된다!
                                       // (자식은 절대 그 존재를 모른다)
    } else {
        printf("fork error!");
        return 0;
    }
}
```

#### **실행 결과**
{: data-toc-skip=''}

```terminal
Parent: Wating for the child
forkEx.c  Makefile    threadEx2.c  waitEx.c  # 자식 프로세스 /bin/ls 실행 결과
main      replit.nix  threadEx.c             # 자식 프로세스 /bin/ls 실행 결과
Parent: child's exit code=0
```

---

## 5. exit() 시스템 콜

- 작업의 종료를 알리는 시스템 콜
- 종료를 명시적으로 알림으로서 부모는 자식이 사용하던 자원을 빨리 회수한다.

### 5.1. 종료 코드

- 부모 프로세스에게 상태나 종료의 이유를 전달하는 값 (ex: `exit(1)`)
- 통상 `0`을 정상 종료로, 나머지 `1 ~ 255` 범위를 그 외 종료로 사용한다.
- `main()` 함수의 리턴 값이 곧 `exit()` 함수의 종료 코드 값이다.
  - `return 4; == exit(4);`
  - 내부적으로 `main()`의 `return`문은 `exit()` syscall이 실행되도록 컴파일된다.
  - OS가 자동적으로 `exit()` 호출되도록 해 프로그램을 종료시키도록 하게 한다.

> **종료 코드**를 **부모가 확인**해야 **최종적으로 자식 프로세스가 종료**된다.
{: .prompt-warning}

### 5.2. exit() 통한 프로세스 종료 과정

1. 프로세스의 모든 자원 반환
  - 코드, 데이터, 스택, 힙 등 모든 메모리 자원 반환
  - 열어 놓은 파일/소켓 등을 닫는다.
2. PCB에 프로세스 상태를 `Terminated`로 변경, PCB에 종료 코드 저장
  - 아직 PCB가 프로세스 테이블에서 **제거된 것이 아님**
3. 자식 프로세스들을 `init` 프로세스에게 입양
4. 부모 프로세스에게 `SIGCHLD`(일종의 종료 알림 신호) 전송
  - **부모의 의무**: `SIGCHLD` 신호 수신하고, `wait() `시스템 호출로 자식의 종료 코드 읽어주기
    - **죽은 자식이 남긴 정보를 확인**해야 자식 프로세스의 **PCB가 완전히 제거**된다.

> 부모가 자식의 종료신호를 **제 때 확인하지 못할 경우, 자식은 좀비 프로세스가 된다.**
> - PCB가 남아있으므로, `ps` 명령어 등으로 존재를 확인할 수 있다.
{: .prompt-danger}

#### **code example for xNIX**
{: data-toc-skip=''}

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

int main() {
    pid_t pid, zompid;
    int status;

    pid = fork();
    if (pid > 0) {            // parent's code
        sleep(10);              // sleep for 10 sec
        zompid = wait(&status); // wait for the child
        printf("Parent: child %d has been finished with %d\n", zompid,
                     WEXITSTATUS(status));
        return 0;
    } else if (pid == 0) { // child's code
        printf("Child: I am done %d \n", getpid());
        exit(100); // exit code 100
    } else {
        printf("fork error\n");
        return 0;
    }
}
```

#### **실행 결과**
{: data-toc-skip=''}

```terminal
Child: I am done 592 # 문장 출력 후 10초 뒤에 아래 문장이 출력된다.
Parent: child 592 has been finished with 100  
```

![exit()]({{ img_path }}/5.2. exit().png){: width='650'}
_자식 프로세스(PID 592)가 좀비(Status=Z)가 된 모습. 자식까지 모두 종료된 뒤에는 사라진다._

- **메모리 정리**는 이미 모두 되었으므로, **점유하는 메모리 size = 0**이다.

> 좀비 프로세스란, 단순히 PCB 정리가 안된 것 뿐이다.
{: .prompt-info}

---



## 6. 정리 in xNIX OS

![정리]({{ img_path }}/6. 정리 (in xNIX OS).png){: width='650'}

- **`fork()` → `exec()`**의 구조
  - `fork()` 통해 프로세스 생성
  - `exec()` 통해 필요한 프로세스 실행
  - **부모 프로세스**: 생성을 한 프로세스 → `wait()`으로 대기
  - **자식 프로세스**: 생성된 프로세스 → `exit()`으로 종료 알림
- 모든 프로세스의 조상: init(1번 프로세스)

### 6.1. fork() → exec() 거치는 이유

- 프로세스 **생성 과정 간소화**
- 프로세스 관리 쉬워짐: **프로세스 계층 구조**
  - 예시: `abort()`(부모가 신호 보내서 자식 죽이는 syscall)
- **프로세스 간 통신(IPC)**
  - 프로세스는 독자적인 메모리 공간 가진 것처럼 운영
  - 서로 간섭/통신 불가, 통신이 필요하면 `SIGCHLD` 같은 신호 보내기
  - 또는 **파일을 이용해 서로 의사소통**
    - 여기서 `fork()`가 사용됨
      - 같은 파일 핸들러를 두 프로세스가 공유

```c
int fd; // 파일 시스템 변수

foo() {
    fd = open("pile"); // 파일을 연다.
    if(fork() == 0) { // 자식일 경우
        read(fd,...); // 하나의 파일로 서로 의사소통함
    }
    else { // 부모일 경우
        write(fd,...); // 하나의 파일로 서로 의사소통함
    }
}
```



### (사족) 윈도우의 프로세스 생성 과정

- `CreateProcess()` 함수 $ \approx $ `fork()` + `exec()`
- **`fork()`와 차이점**
  - `fork()`: 자식 프로세스가 부모 프로세스의 주소공간 상속밭음
  - `CreateProcess()`: 자식 프로세스에게 구체적으로 어떤 프로그램을 실행할 것인지 요구(주소값이 명확해야 한다.)

#### **code example for Windows**
{: data-toc-skip=''}

```c
#include <stdio.h>
#include <windows.h>

int main() {
    STARTUPINFO si = { 0 };
    PROCESS_INFORMATION pi = { 0 };

    si.cb = sizeof(si);

    if (!CreateProcessA(NULL,
        "C:\\windows\\system32\\notepad.exe",
        NULL, NULL, FALSE, 0, NULL, NULL, &si, &pi)) {
        printf("Error!\n");
        return -1;
    }

    WaitForSingleObject(pi.hProcess, INFINITE);
    printf("Child has been finished\n");

    CloseHandle(pi.hProcess);
    CloseHandle(pi.hThread);

    return 0;
}
```

#### **실행 결과**
{: data-toc-skip=''}

![윈도우의 프로세스 생성 과정]({{ img_path }}/7. 윈도우의 프로세스 생성 과정.png){: width='650'}
_(1) 실행 시 메모장이 자동으로 실행된다. 종료하기 전까지는 콘솔에 아무 것도 표시되지 않는다._

![윈도우의 프로세스 생성 과정 2]({{ img_path }}/7. 윈도우의 프로세스 생성 과정 2.png){: width='650'}
_(2) 메모장을 닫으면 콘솔에 `"Child has been finished\n"`가 출력된다._