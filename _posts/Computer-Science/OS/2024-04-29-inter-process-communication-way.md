---
title: "[OS] IPC: 프로세스 통신 방법"
date: 2024-04-30 15:35:00 +0900
categories: [Computer-Science, OS]
tags: ['OS', '동기화', 'IPC']
---



## 1. 파일을 이용한 통신

- 데이터를 쓰고 읽는 함수를 통해 이뤄진다.
  - 예시: `open`, `write`, `read`

```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>

int main()
{
    int fd;
    char buf[124]={0,};
    fd = open("temp.txt", O_RDWR);
    write(fd, "Test", 5);
    lseek(fd,0,SEEK_SET);
    read(fd, buf, 5);
    printf(buf);
    close(fd);
    return 0;
}
```

{% assign img_path = "/assets/img/posts/OS/2024-04-29-inter-process-communication-way" %}
![1. 파일을 이용한 통신]({{ img_path }}/1. 파일을 이용한 통신.png){: width='350'}
_파일 입출력 연산_



## 2. 파일 디스크립터를 이용한 통신

- 프로세스끼리 파일 디스크립터를 공유해서 사용한다.
  - `fork` 시스템콜을 이용한다.

```c
#include <fcntl.h>
#include <stdio.h>
#include <unistd.h>

int global = 100;  // 부모-자식 간 공유되지 않는다.

int main() {
    char str[128] = {
        0,
    };
    int fd = open("temp.txt", O_RDWR);  // 부모 자식이 동시에 파일을 공유한다!
    pid_t pid;
    pid = fork();

    if (pid > 0) {  // parent
        write(fd, "Can you hear me?", 17);
        sleep(2);
        lseek(fd, 17, SEEK_SET);
        read(fd, str, 12);
        printf("parent(%X): %s \n", getpid(), str);
        close(fd);
    } else if (pid == 0) {  // child
        sleep(1);           // 타이밍을 맞추기 위한(동기화를 위한) 방법
        lseek(fd, 0, SEEK_SET);
        read(fd, str, 17);
        printf("child(%X): %s \n", getpid(), str);
        write(fd, "Yes, I can!", 12);
    }

    return 0;
}
```

> 간혹 전역변수를 통해 부모-자식 간 통신을 구현하려고 할 수 있는데, 프로세스간에는 메모리를 공유하지 않으므로 불가능하다!
{: .prompt-warning}

### 2.1. 파일 디스크립터

프로세스가 파일을 다룰 때 사용하는 개념으로 **0 이상의 정수값**으로 표현된다. 운영체제가 **특정 파일에 할당**해주는 값으로, 시스템 함수 `open()`, `read()`, `write()` 등에서 작업을 처리하기 위한 값이다.

파일 디스크립터 값은 프로세스마다 별개이다. 프로세스는 자기만의 파일 디스크립터 테이블을 하나씩 자기고 있기 때문이다. 이 테이블을 **File Descriptor Table**라 하며, 이 배열의 **index가 바로 파일 디스크립터 값**이다. 

- 예시: `A.txt`, `B.txt`, **프로세스 C**, **프로세스 D**가 있을 때
  1. `A.txt`를 가리키는 디스크립터 값이 **프로세스 C**와 **D**에서 서로 다를 수 있다.
  2.  **프로세스 C**에서 `A.txt`를 가리키는 디스크립터 값, **프로세스 D**에서` B.txt`를 가리키는 디스크립터 값이 서로 같을 수 있다. 

`open()` 이외에 파일 디스크립터를 생성하는 함수가 더 있는데 아래와 같은 함수들이 파일 디스크립터를 새로 생성한다.

- `open()`, `creat()`(오타가 아니라 [실제 함수 이름이다!](https://pubs.opengroup.org/onlinepubs/9699919799/functions/creat.html)), `socket()`, `accept()`, `pipe()` 등

![2. 파일 디스크립터]({{ img_path }}/2. 파일 디스크립터.png){: width='650'}
_File descriptor 예_

#### 2.1.1. Open file table

- 프로세스에 의해 open된 파일의 입출력 동작을 지원하기 위한 테이블
- 테이블 자체는 커널이 단 하나를 생성하며, 모든 프로세스가 접근할 수 있게 공유되어 있다.
- 파일이 열릴 때마다 **entry(테이블의 행)**가 하나씩 할당된다.
- 각 엔트리에는 아래와 같은 내용을 담고 있다.
  - **파일 위치 지시자(File Offset)**: 입출력 작업이 진행되는 위치
  - **파일 상태 플래그(File Status Flags)**
  - **참조 카운터(Reference Count)**: 해당 파일을 참조하는 파일 디스크립터의 개수
  - **파일의 inode 정보**: 파일 시스템 내 파일을 식별하는 inode에 대한 포인터

- **같은 프로세스**에서 동일 파일을 3번 `open`하면 새 엔트리가 3개가 생성된다.
- **다른 프로세스**에서 동일 파일을 `open`해도 새 엔트리가 생성된다.
  - 위 두 가지 경우 모두 엔트리의 ref count는 1이다.

```c
//fd1, fd2, fd3은 모두 다른 값

//엔트리 생성, ref count: 1
int fd1 = open("fileA.txt", O_RDONLY); 
//별도의 엔트리 또 생성 (2개째), ref count: 1
int fd2 = open("fileA.txt", O_RDONLY); 
//또 엔트리 생성 (3개째), ref count: 1
int fd3 = open("fileA.txt", O_RDONLY); 
```
{: .nolineno}

- `fork`나 `dup` 함수를 사용하는 경우 **새 엔트리가 생성되지 않으며**, 새로 만들어진 파일 디스크립터는 **기존의 엔트리를 참조**하게 된다. 그리고 기존 엔트리의 **Ref Count가 1이 증가**한다.

```c
//fd1과 fd2 값은 같음

// open file table 엔트리 생성, ref count = 1
int fd1 = open("fileA.txt", O_RDONLY); 
// 동일 open file table 엔트리, ref count = 2
int fd2 = dup(fd1);                    
```
{: .nolineno}


```c
// open file table 엔트리 생성, ref count = 1
int fd1 = open("fileA.txt", O_RDONLY); 
if (fork() == 0) {
    // 자식 프로세스
    // fd1은 부모와 동일한 open file table 엔트리를 가리킴, ref count = 2
}
```
{: .nolineno}

#### 2.1.2. 예약된 파일 디스크립터

- 파일 디스크립터 값 중 **0, 1, 2**는 미리 예약되어 있으며, 각각 다음의 의미를 가진다.
  - **0:** `stdin`
    - standard input(표준 입력)
    - 프로그램에 대한 입력을 나타내는 stream
    - 일반적으로 **키보드**에 해당됨
  - **1:** `stdout`
    - standard output(표준 출력)
    - 출력을 위한 stream
    - 현재 shell을 실행한 **console이나 terminal**에 해당됨
  - **2:** `stderr`
    - standard error(표준 오류)
    - 디버깅 정보, 에러를 출력하는 데 사용됨

- **예시**

```c
#include <fcntl.h>
#include <stdio.h>
#include <unistd.h>

int main() {
    char buf[124] = {};
    int fd = open("temp.txt", O_RDWR);

    printf("fd: %d \n", fd); // 출력값은? 3

    write(fd, "Test", 5);
    lseek(fd, 0, SEEK_SET);
}
```
{: file='std_fd_ex.c'}

#### 2.1.3. Redirect 연산

- 리눅스의 **redirect** 연산에서 파일 디스크립터를 사용한다.

- `echo "hello" > temp.txt`
  - echo의 출력(0, stdin)의 방향을 파일로 바꾸어 주라는 뜻이다.
  - 리다이렉트를 쓰지 않으면 `echo "hello" > 0`과 동일한 의미이다.

위의 예시 `std_fd_ex.c`{: .filepath}을 컴파일한 프로그램 이름이 `program`일 때, `./program 3> temp.txt` 명령으로 실행하면 fd 값이 4가 된다.
  - `3> temp.txt` : 3번 파일 디스크립터를 temp.txt에 할당하라는 뜻이다.
  - 3번 파일 디스크립터를 redirect로 연 후 시작하므로, fd의 값은 4가 나오게 된다.



## 3. 파이프를 이용한 통신

**파이프(pipe)**란 데이터 스트림을 한 프로그램에서 다른 프로그램으로 전달하는 메커니즘으로, 유닉스 계열 OS에서 제공하는 동기화 통신 방식이며 `|` 기호로 흔히 표현된다.

파이프에 쓰기 연산을 하면 데이터가 전송되고, 읽기 연산을 하면 데이터를 받을 수 있어 **단방향 통신**이라고 할 수 있다.

- 0번: 읽기, 1번: 쓰기
- 예: `ls | grep passed` - ls의 출력 결과를 grep 명령어로 전달
- 파일 입출력과 같이 `open()` 함수로 Pipe descriptor를 열고 작업을 한 후 `close()` 함수로 마무리한다.

![3. 파이프를 이용한 통신]({{ img_path }}/3. 파이프를 이용한 통신.png){: width='400'}
_파이프를 이용한 통신_

<!-- omit from toc -->
### 예시
{: data-toc-skip=''}

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h> //pipe 사용 위함
#include <pthread.h>

int main(){
    int pipefd[2]; // [0] for read, [1] for write
    pid_t pid;
    char buf[128];
    
    // function signature: int pipe(int pipefd[2]);
    // 주의! pipe()를 이용한 통신은 부모-자식 간 프로세스 관계에서만 가능함
    if(pipe(pipefd) < 0){ 
        printf("pipe error\n");
        return 1;
    }

    pid = fork();
    if(pid > 0){ //parent process
        close(pipefd[0]);
        strcpy(buf, "Hello?\n");
        //부모에서 쓰기용 [1] 파이프에 데이터 기록
        write(pipefd[1], buf, strlen(buf)); 
    }
    else if(pid == 0) { //child process
        close(pipefd[1]);
        //자식에서 읽기용 [0] 파이프에서 데이터 읽음
        read(pipefd[0], buf, 128);
        printf(">> I am child! I can hear the parent!!: %s\n",buf);
    }
    else {
        printf("fork error\n");
    }

    return 0;
}
```
- 자식이 부모에게 역으로 데이터를 전달하려면 어떻게 해야 할까?
  - 파이프를 하나 더 만들면 된다. `pipeA[0]`, `pipeB[1]`를 각각 부모의 읽기/쓰기 파이프로 사용하고 `pipeA[1]`, `pipeB[0]`를 각각 자식의 쓰기/읽기 파이프로 사용하자.

> `pipe()`를 이용한 통신은 부모-자식 간 프로세스에서만 가능하다! 아무 관련 없는 프로세스끼리 파이프 통신을 하기 위해선 아래에서 설명할 **FIFO**를 사용해야 한다.
{: .prompt-warning}

### 3.1. 파이프의 종류

- **이름 없는 파이프**
  - 일반적으로 파이프라고 하면 이름 없는 파이프를 가리킨다.

- **이름 있는 파이프**(명명 파이프, Named Pipe, FIFO)
  - FIFO라 불리는 특수 파일을 이용한다.
  - **서로 관련 없는 프로세스 간 통신**에 사용한다.

- **차이점**
  - pipe는 열려져 있는 pipe에 대해서는 open할 수 없지만, FIFO는 가능하다.
    - 즉, FIFO는 **서로 다른 (부모자식이 아닌) process**간에 data를 주고받을 수 있다(`pipe()`는 주고받지 못한다).
  - pipe는 파일시스템에 이미지를 생성하지 않는 반면에 FIFO는 **파일 시스템상에 이미지를 가진다.**

<!-- omit from toc -->
### FIFO 예시: Receive
{: data-toc-skip=''}

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>

int main() {
    int counter = 0;
    int fd;
    char buff[128];

    if ( mkfifo("temp.txt", 0666) < 0){
        perror( "mkfifo() error");
        return 1;
    }
    fd = open("temp.txt", O_RDWR);
    // <- why RW? (NOT O_RDONLY?) It's for both read and write access.
    // RDONLY(읽기 권한)만 주면 쓰레기 값을 계속 읽는다.
    // 쓰기 권한을 같이 줌으로써 reference count를 1을 부여해 read()에서 blocking의 효과를 부여한다.

    while(1){
        memset( buff, 0, 128);
        read(fd, buff, 128); //읽기권한만 주면 파이프에 아무 것도 없으면 스킵되므로, 무한반복이 됨
        printf( "%d: %s\n", counter++, buff);
    }
    close(fd);
}
```

<!-- omit from toc -->
### FIFO 예시: Sender
{: data-toc-skip=''}

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <unistd.h>

int main( void)
{
    int fd;
    char *str = "I am the sender!";
    fd = open("temp.txt", O_WRONLY);
    write(fd, str, strlen(str));
    close(fd);
} 
```

위 두 프로그램 실행 시 `temp.txt`{: .filepath}가 만들어지는데, `ls -al`으로 확인하면 파일 타입이 `p`인 것을 확인할 수 있다. 이 상태에서 `echo "hello" > temp.txt` 를 실행하면 파이프로 데이터를 밀어넣는 것이 된다.

![3. 파이프 파일 예시]({{ img_path }}/3. 파이프 파일 예시.png){: width='450'}
_FIFO 예시_



## 4. 소켓을 이용한 통신

소켓 프로그래밍은 일종의 네트워크 프로그래밍(Server/Client)으로, 서로 다른 컴퓨터가 각각 서버-클라이언트가 되어 각 컴퓨터의 프로세스끼리 통신하는 방법이다. 

소켓 통신을 하고자 하는 프로세스는 자신의 소켓과 상대방의 소켓을 연결해야 한다. 운영체제 함수를 이용하여 시스템에 있는 프로세스가 소켓을 바인딩하게 한 후 소켓에 쓰기 연산을 하면 데이터가 전송되고, 읽기 연산을 하면 데이터를 받게 된다. 소켓 통신에 필요한 함수 호출 흐름을 아주 간략히 하면 아래와 같다.

1. `socket()`: 소켓 생성
2. `bind()`: 주소 할당
3. `listen()`: 통신 요청을 기다림
4. `connect()`: 통신 요청
5. `accept()`: 통신 수락
6. `read()`/`write()`: 통신
7. `close()`: 소켓 종료

![4. 소켓을 이용한 통신 방법]({{ img_path }}/4. 소켓을 이용한 통신 방법.png){: width='400'}
_소켓을 이용한 통신: 도식_

![4. 소켓을 이용한 통신 방법 2]({{ img_path }}/4. 소켓을 이용한 통신 방법 2.png){: width='350'}
_소켓을 이용한 통신: API 호출 흐름도_

<!-- omit from toc -->
### 예시: Server
{: data-toc-skip=''}

```c
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/tcp.h>
#include <arpa/inet.h>
//#include "sock.h"

int main() {
    char buffer[256];
    char buffer2[256];
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    if (fd < 0) return -1;

    struct sockaddr_in saddr;
    memset(&saddr, 0, sizeof(saddr));
    saddr.sin_family = AF_INET;
    saddr.sin_addr.s_addr = htonl(INADDR_ANY);
    saddr.sin_port = htons(7777); // port number

    // open a server
    if (bind(fd, (struct sockaddr *) &saddr, sizeof(saddr)) < 0)
        return -1;

    if (listen(fd, 16) < 0)
        return -1;

    while (1) {
        struct sockaddr_in caddr;
        int len = sizeof(caddr);
        int client_fd = accept(fd, (struct sockaddr*) &caddr, &len);
        if (client_fd < 0) continue;

        // read from client
        memset(buffer, 0, sizeof(buffer));
        read(client_fd, buffer, sizeof(buffer));
        printf(buffer);
        sprintf(buffer2, "I got you!: %s", buffer);
        write(client_fd, buffer2, strlen(buffer2));
        close(client_fd);
    }

    return 0;
}

```

<!-- omit from toc -->
### 예시: Client
{: data-toc-skip=''}

```c
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <netinet/tcp.h>
#include <netdb.h>

int main() {
    char buffer[256];
    int sockfd = socket(AF_INET, SOCK_STREAM, 0);

    if (sockfd < 0) return -1;

    struct hostent* hptr = gethostbyname("127.0.0.1");
    if (hptr->h_addrtype != AF_INET) return -1;

    struct sockaddr_in saddr;
    memset(&saddr, 0, sizeof(saddr));
    saddr.sin_family = AF_INET;
    saddr.sin_addr.s_addr =
    ((struct in_addr*) hptr->h_addr_list[0])->s_addr;
    saddr.sin_port = htons(7777); // port number

    // connect to the server
    if (connect(sockfd, (struct sockaddr*) &saddr, sizeof(saddr)) < 0)
        return -1;

    char *msg = "This is a socket communication!!";
    write(sockfd, msg, strlen(msg));
    memset(buffer, 0, sizeof(buffer));
    read(sockfd, buffer, sizeof(buffer));
    printf(buffer);
    close(sockfd);

    return 0;
}
```



## 5. Shared memory를 이용한 통신

커널이 관리하는 일정한 크기의 **공유 메모리 공간(Shared memory)**을 통해 통신하는 방법으로, Shared memory는 마치 **힙 메모리를 읽듯이** 접근 및 사용할 수 있다. 이 방법은 아래와 같은 특징이 있다.

- IPC 방법 중 가장 속도가 빠름
- 사용하려는 프로세스간 할당한 크기가 동일해야 사용 가능
- 동기화가 중요한 포인트로, 신경써야 할 부분

![5. Shared memory]({{ img_path }}/5. Shared memory.png){: width='500'}
_Shared memory를 이용한 통신_

### 5.1. 사용되는 함수

- `int shmget(key, size, flag)`: shared memory get
  - 공유 메모리를 생성하거나 접근하기 위한 함수
  - **key**: 공유 메모리 할당 및 접근하기 위한 고유한 키 값, 커널에서 관리됨
    - key가 지시하는 메모리 공간에 이미 공유 메모리 세그먼트가 생성되어 있으면 그 공간의 id를 반환하고, 없으면 새 공간 생성하여 id를 반환한다.
  - **size**: 새로 할당할 공유 메모리의 최소 크기
    - 이미 할당된 메모리에 접근할 땐 0으로 입력받는다.
  - **flag**: 함수 동작 관련 플래그, `IPC_CREAT`, `IPC_EXCL을` 사용할 수 있음
  - **return**: key에 해당하는 공유 메모리의 id

- `void* shmat(shmid, shmaddr, flag)`: shared memory attach
  - 공유 메모리를 프로세스에서 사용할 수 있게 하는 함수
  - **shmid**: `shmget()`으로 얻은 id
  - **shmaddr**: 공유 메모리가 붙을 주소
    - `NULL`로 지정시 시스템은 적절한 주소에 주소를 붙인다.
  - **return**: 붙은 공유 메모리의 주소로 힙 메모리(`malloc()` 반환값)처럼 사용하면 됨

- `void* shmctl(shmid, cmd, buf)`: shared memory control
  - 공유 메모리를 제어하기 위한 함수
  - **shmid**: `shmget()`으로 얻은 id
  - **cmd**: 공유 메모리를 제어하기 위한 command
    - **IPC_RMID**: 공유 메모리 공간을 삭제했다고 표시하는 명령어. 그러나 바로 삭제되지는 않고, 공유 메모리에 attach했던 모든 프로세스가 detach하는 순간 삭제됨
  - **return**: 작업 성공 시 0을 반환

<!-- omit from toc -->
### 예시: Write
{: data-toc-skip=''}

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <unistd.h>

int main() {
    int shmid = 0;
    char *shared_memory = 0;

    // get shared memory
    shmid = shmget((key_t)7777, 128, 0666 | IPC_CREAT);
    if (shmid == -1) return -1;

    // attach to the shared memory
    shared_memory = (char *)shmat(shmid, (void *)0, 0);
    if (shared_memory == -1) return -1;
    while (1) {
        strcpy(shared_memory, "Hello?\n");
        sleep(1);
        strcpy(shared_memory, "Can you hear me?\n");
        sleep(1);
    }
}
```

<!-- omit from toc -->
### 예시: Read
{: data-toc-skip=''}

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <unistd.h>

int main() {
    int shmid;
    char *shared_memory = 0;

    // get shared memory
    shmid = shmget((key_t)7777, 128, 0666 | IPC_CREAT);
    if (shmid == -1) return -1;

    // attach to the shared memory
    shared_memory = (char *)shmat(shmid, (void *)0, 0666 | IPC_CREAT);
    if (shared_memory == -1) return -1;

    while (1) {
        printf(shared_memory);
        sleep(1);
    }
}
```

<!-- omit from toc -->
### 예시: Clear
{: data-toc-skip=''}

> Shared memory는 프로세스와 관계없이 커널에서 만들어지기 때문에 프로그램을 닫아도 유지된다. 즉 **Memory Leak**이 발생할 수 있으므로 따로 Clear를 위한 코드를 실행해야 한다.
{: .prompt-warning}

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <unistd.h>

int main() {
    int shmid;
    char *buff = NULL;
    void *shared_memory = (void *)0;

    // get shared memory
    shmid = shmget((key_t)7777, 128, 0666 | IPC_CREAT);
    if (shmid == -1) return -1;


    printf("%X \n", shmctl(shmid, IPC_RMID, 0));
}
```

### 5.2. 단점

동기화가 되어 있긴 하지만 다른 프로세스는 메모리에 쓰인 값이 언제 쓰인 것인지 알 수 없다. 즉 쓰인 값이 기존 값인지 새 값인지 알 수 없다.



## 6. Message queue를 이용한 통신

- 일종의 파이프로, 커널 메모리 공간을 활용한다. 
- 구조체를 기반으로 작동하므로 메세지 경계가 명확하다. 
  - block 단위임(파이프는 stream으로 경계가 명확하지 않음)
- 동기화를 고려할 필요가 없다.
- 데이터가 기존 데이터인지 새 데이터인지 식별이 가능하다.

![6. Message queue]({{ img_path }}/6. Message queue.png){: width='500'}
_Message queue를 이용한 통신_

<!-- omit from toc -->
### 예시: Sender
{: data-toc-skip=''}

```c
#include <stdio.h>
#include <string.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>

struct msgbuf {
    long mtype;      /* message type, must be > 0 */
    char mtext[128]; /* message data */
};

int main() {
    key_t key_id;
    struct msgbuf msg = { 5, { 0, } };  // type, text
    // Message Queue
    key_id = msgget((key_t)7777, IPC_CREAT | 0666);
    if (key_id == -1) return -1;
    while (1) {
        strcpy(msg.mtext, "This is MQ!\n");
        msgsnd(key_id, &msg, sizeof(struct msgbuf), IPC_NOWAIT);
        sleep(1);
        strcpy(msg.mtext, "Wow~~!!\n");
        msgsnd(key_id, &msg, sizeof(struct msgbuf), IPC_NOWAIT);
        sleep(1);
    }
    return 0;
}
```

<!-- omit from toc -->
### 예시: Receiver
{: data-toc-skip=''}

```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <sys/stat.h>
#include <sys/types.h>

struct msgbuf {
    long mtype;      /* message type, must be > 0 */
    char mtext[128]; /* message data */
};

int main() {
    key_t key_id;
    struct msgbuf msg;
    key_id = msgget(7777, IPC_CREAT | 0666);
    if (key_id < 0) return -1;
    while (1) {
        msgrcv(key_id, &msg, sizeof(struct msgbuf), 5, 0);
        printf("%s", msg.mtext);
    }
    return 0;
}
```



## 7. Memory map을 이용한 통신

- 파일을 프로세스의 메모리의 특정 부분에 **일정 부분 맵핑**시켜 사용한다.
  - Shared Memory와 마찬가지로 메모리를 공유한다는 측면에 있어서는 서로 비슷하나, Memory Map의 경우 **열린 파일을 메모리에 맵핑시켜서 공유**한다는 점이 차이가 있다.
- 파일로 대용량 데이터를 공유 할 때 유리하며, 메모리에 맵핑된 파일을 읽거나 쓰면 `read()` 나 `write()` 시스템 콜을 사용할 때 발생하는 **불필요한 복사**를 방지한다.
  - **추가적인 복사**는 사용자 영역의 버퍼로 데이터를 읽고 써야 하기 때문에 발생함
- mmap 함수의 종류는 많다.

![7. Memory map]({{ img_path }}/7. Memory map.png){: width='600'}
_Memory map을 이용한 통신_

<!-- omit from toc -->
### 예시: Sender
{: data-toc-skip=''}

```c
#include <errno.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
    int fd;
    char *file = NULL;
    struct stat sb;
    int flag = PROT_WRITE | PROT_READ;

    fd = open("temp.txt", O_RDWR | O_CREAT);
    if (fstat(fd, &sb) < 0) return -1;

    file = (char *)mmap(0, 40, PROT_WRITE | PROT_READ, MAP_SHARED, fd, 0);

    while (1) {
        strcpy(file, "This is the memory map!\n");
        sleep(1);
        strcpy(file, "wow!!\n");
        sleep(1);
    };

    munmap(file, 40);
    close(fd);

    return 0;
}
```

<!-- omit from toc -->
### 예시: Receiver
{: data-toc-skip=''}

```c
#include <errno.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
    int fd;
    char *file = NULL;
    struct stat sb;
    int flag = PROT_WRITE | PROT_READ;

    fd = open("temp.txt", O_RDWR | O_CREAT);
    if (fstat(fd, &sb) < 0) return -1;

    file = (char *)mmap(0, 40, PROT_WRITE | PROT_READ, MAP_SHARED, fd, 0);
    while (1) {
        printf("%s", file);
        sleep(1);
    }

    munmap(file, 40);
    close(fd);
    return 0;
}
```



## 8. IPC 방법 정리

|     IPC 종류      | 사용 시기                      | 공유 매개체 | 통신 단위      | 통신 방향 | 통신 가능 범위     |
| :---------------: | ------------------------------ | ----------- | -------------- | --------- | ------------------ |
|     **PIPE**      | 부모 자식 간 단방향 통신 시    | 파일        | Stream         | 단방향    | 동일 시스템        |
|  **Named PIPE**   | 다른 프로세스와 단방향 통신 시 | 파일        | Stream         | 단방향    | 동일 시스템        |
| **Mesage Queue**  | 다른 프로세스와 단방향 통신 시 | 메모리      | 구조체         | 단방향    | 동일 시스템        |
| **Shared Memory** | 다른 프로세스와 양방향 통신 시 | 메모리      | 구조체         | 양방향    | 동일 시스템        |
|  **Memory Map**   | 다른 프로세스와 양방향 통신 시 | 파일+메모리 | 페이지         | 양방향    | 동일 시스템        |
|    **Socket**     | 다른 시스템간 양방향 통신 시   | 소켓        | Stream(Packet) | 양방향    | 동일 + 외부 시스템 |