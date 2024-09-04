---
title: "[OS] 동기화 기법"
date: 2024-05-13 09:24:00 +0900
categories: [Computer-Science, OS]
tags: ['OS', '동기화']
---



## 1. 동기화 종류

- 크게 3가지 종류가 있음
  - 뮤텍스(mutex)
  - 스핀락(spinlock)
  - 세마포어(semaphore): 자원이 여러개인 경우

### 1.1. locks 방식

- 뮤텍스(mutex), 스핀락(spinlock)
- 상호배제가 되도록 만들어진 락(lock) 활용
- Mutex는 동기화 대상이 only 1개일 때 사용
- 락을 소유한 스레드만이 임계구역 진입
- 락을 소유하지 않은 스레드는 락이 풀릴 때까지 대기
- 둘의 차이는 busy-wait, sleep-wait (큐 존재)

{% assign img_path = "/assets/img/posts/OS/2024-05-13-synchronization-way" %}
(< 사진삽입필요 >)

### 1.2. wait-signal 방식

- 세마포어(semaphore)
- n개 자원을 사용하려는 m개 멀티스레드의 원활한 관리
- 카운터를 활용!
- 자원을 소유하지 못한 스레드는 대기(wait)
- 자원을 다 사용한 스레드는 알림(signal)
- (사실 얘도 Busy-wait, sleep-wait로 구분됨)

(< 사진삽입필요 >)



## 2. Mutex

- **MUT**ual **EX**clusion의 줄임말
- 잠김/열림 중 한 상태를 가지는 락 변수 이용
  - 한 스레드만 임계구역에 진입시킴, 다른 스레드는 큐에 대기
  - sleep-waiting lock 기법

![2. Mutex]({{ img_path }}/2. Mutex.png){: width='500'}

### 2.1. 구성 요소

1. 락 변수
   - true/false 중 한 값
   - true : 락을 잠근다. 락을 소유한다.
   - false : 락을 연다. 락을 해제한다.

2. 대기 큐
   - 락이 열리기를 기다리는 스레드 큐

3. 연산
   - lock 연산(임계구역의 entry 코드)
     - 락이 잠김 상태(lock = true)이면, 현재 스레드를 Block 상태로 만들고 대기 큐에 삽입
     - 락이 열린 상태이면, 락을 잠그고 임계구역 진입
   - unlock 연산(임계구역의 exit 코드)
     - lock = false, 락을 열린 상태로 변경
     - 대기 큐에서 기다리는 스레드 하나 깨움

### 2.2. 그림

(< 사진삽입 필요 >)

### 2.3. 예시

```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

const int count = 200000;
int sum = 0; // global variable, shared data
pthread_mutex_t lock;

void* myThread1(void *p) {
    for(int i = 0; i < count; i++) {
        pthread_mutex_lock(&lock); // entry
        sum += 1;
        pthread_mutex_unlock(&lock); // exit
    }
    return 0;
}

void* myThread2(void *p) {
    for(int i = 0; i < count; i++) {
        pthread_mutex_lock(&lock); // entry
        sum -= 1;
        pthread_mutex_unlock(&lock); // exit
    }
    return 0;
}

int main() {
    pthread_t tid1, tid2; // thread id
    int count = 200000;
    int *ret1, *ret2;

    printf("Start!\n");
    pthread_mutex_init(&lock, NULL); // init mutex
    pthread_create(&tid1, NULL, myThread1, NULL);
    pthread_create(&tid2, NULL, myThread2, NULL);
    pthread_join(tid1, (void**)&ret1); // waiting for 'tid1'
    pthread_join(tid2, (void**)&ret2); // waiting for 'tid2'
    printf("sum = %d\n", sum);
    pthread_mutex_destroy(&lock); // destroy mutex
    return 0;
} 
```


## 3. Spinlock

- 이전 포스팅의 while(lock)으로 대기하던 것과 같음
  - 바쁜 대기(busy wait) 기법

- 뮤텍스의 non-blocking 모델 (Busy-waiting)
  - 락이 잠겨 있을 때 블록되지 않고 락이 풀릴 때까지 검사 코드 실행
    - 큐가 아닌 While loop로 대기!

- 단일 CPU(단일 코어)를 가진 운영체제에서 비효율적, **멀티 코어에 적합**
  - 단일 코어 CPU에서 의미 없는 CPU 시간 낭비
  - 스핀락을 검사하는 스레드의 타임 슬라이스가 끝날 때까지 다른 스레드 실행 안 됨, 다른 스레드의 실행 기회 뺏음
  - 여러 스레드/프로세스가 동시 접근을 시도하는 경우 **기아 상태에 빠질 수 있음**

- 락을 소유한 다른 스레드가 실행되어야 락이 풀림.
- 임계구역의 실행 시간이 짧은 경우 효과적

### 3.1. 그림

(< 사진삽입필요 >)

### 3.2. 예시

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <pthread.h>

const int count = 200000;
int sum = 0; // global variable, shared data
pthread_spinlock_t lock; //스핀락 변수 생성

void* myThread1(void *p) {
    for(int i = 0; i < count; i++) {
        pthread_spin_lock(&lock); // 임계구역 entry 코드, 스핀락 잠그기
        sum += 1;
        pthread_spin_unlock(&lock); // 임계구역 exit 코드, 스핀락 열기
    }
    return 0;
}

void* myThread2(void *p) {
    for(int i = 0; i < count; i++) {
        pthread_spin_lock(&lock); // entry
        sum -= 1;
        pthread_spin_unlock(&lock); // exit
    }
    return 0;
}

int main() {
    pthread_t tid1, tid2; // thread id
    int count = 200000;
    int *ret1, *ret2;
    
    printf("Start!\n");
    pthread_spin_init(&lock, NULL); // init spin(스핀락 변수 초기화)
    pthread_create(&tid1, NULL, myThread1, NULL);
    pthread_create(&tid2, NULL, myThread2, NULL);
    pthread_join(tid1, (void**)&ret1); // waiting for 'tid1'
    pthread_join(tid2, (void**)&ret2); // waiting for 'tid2'
    printf("sum = %d\n", sum);
    pthread_spin_destroy(&lock); // destroy spin
    return 0;
} 
```



## 4. Mutex vs Spinlock

### 4.1. 락이 잠기는 시간이 긴 경우 : 뮤텍스

- 락을 얻지 못했을 때, CPU를 다른 스레드에게 양보하는 것이 효율적
- 락이 잠기는 시간이 짧은 경우 : 스핀락이 효율적

### 4.2. 단일 CPU를 가진 시스템 : 뮤텍스

- 단일 CPU에서 스핀락은 크게 의미 없음

### 4.3. 멀티 코어(멀티 CPU)를 가진 시스템 : 스핀락

- 잠자고 깨는 컨텍스트 스위칭 없이 바로 자원 사용
- 임계구역은 가능한 짧게 작성하므로

### 4.4. 사용자 응용프로그램 : 뮤텍스, 커널 코드 : 스핀락

- 커널 코드나 인터럽트 서비스 루틴을 빨리 실행되어야 하고, 인터럽트 서비스 루틴 내에서 잠잘 수 없기 때문

### 4.5. 스핀락을 사용하면 기아 발생 가능

- 스핀락은 무한 경쟁 방식이어서 기아가 발생 가능
- 락을 소유한 스레드가 락을 풀지 않고 계속 실행하거나 종료해버린 경우, 코딩이 잘못된 경우 시



## 5. 세마포어(semaphore)

- semaphore 뜻: 수기(手旗) 신호, 수기 신호로 알리다
- n개의 **공유 자원을 다수 스레드가 공유하여 사용**하도록 돕는 자원 관리 기법
  - 보다 다수의 자원을 다수의 프로세스/쓰레드를 대상으로...
    - e.g., n개의 프린터가 있는 경우, 프린터를 사용하고자 하는 다수 스레드의 프린터 관리
- 세마포어는 동시에 여러 개의 프로세스/스레드가 임계 구역에 접근할 수 있도록 카운터를 가지고 있는 입구! 
  - → 일정 수가 초과하면 못들어갑니다!

![5. 세마포어]({{ img_path }}/5. 세마포어.png){: width='600'}
_세마포어의 개념_

### 5.1. 구성 요소 (counter semaphore)

1. 자원 : n 개
2. 대기 큐 : 자원을 할당받지 못한 스레드들이 대기하는 큐
3. counter 변수
  - 사용 가능한 자원의 개수를 나타내는 정수형 전역 변수
  - n으로 초기화(counter = n)
4. P/V 연산
  - P 연산 (wait 연산): 자원 요청 시 실행하는 연산 (자원 사용 허가를 얻는 과정)
  - V 연산 (signal 연산): 자원 반환 시 실행하는 연산 (자원 사용이 끝났음을 알리는 과정)

### 5.2. P 연산, V 연산

- P/V를 wait/signal로 표기하기도 함
  - P 연산 : 자원 사용을 허가하는 과정, 사용가능 자원 수 1 감소(counter--)
  - V 연산 : 사용 사용을 마치는 과정, 사용가능 자원 수 1 증가(counter++)
- 세마포 종류 2가지 - 자원을 할당받지 못한 경우의 행동에 따라 구분

```c
// busy-wait 세마포어
P(S) {
    while S <=0; // 아무것도 하지 않음 (반복문)
    S--;
}

V(S) {
    S++;
}
```

```c
// sleep-wait 세마포어
P(S) {
    S--;
    if S < 0
    // 이 프로세스를 재움 큐에 추가 (잠 듦)
}

V(S) {
    S++;
    if S <= 0
    // 재움 큐로부터 프로세스를 제거 (깨어남)
}
```

### 5.3. 세마포어 통한 자원 관리 예제

- 4개의 인스턴스를 가진 자원에 대해, 4개의 스레드(T1~T4)가 할당 받아 사용, 2개의 스레드 T5와 T6는 자원을 기다리고 있는 상태
- counter 변수는 사용 가능한 자원의 개수를 나타냄
  - 음수이면 대기 중인 스레드의 수를 나타냄

![5.3. 세마포어 통한 자원 관리 예제]({{ img_path }}/5.3. 세마포어 통한 자원 관리 예제.png){: width='550'}
_세마포어 통한 자원 관리 예_

### 5.4. 이진 세마포어: 특수한 경우

- 이진 세마포(binary semaphore)
- 자원이 1개있는 경우 멀티스레드 사이의 자원 관리
- 1개의 자원에 대해 1개의 스레드만 액세스할 수 있도록 보호
- 뮤텍스와 매우 유사

#### 5.4.1. 구성 요소

1. 세마포 변수 S
  - 0 과 1 중 하나를 가지는 전역 변수, S는 1로 초기화
2. 대기 큐
  - 사용 가능한 자원이 생길 때까지 스레드들이 대기하는 큐
  - 스레드 스케줄링 알고리즘 필요
3. 2개의 원자 연산
  - wait 연산(P 연산) – 자원 사용 허가를 얻는 과정
    - S가 1 감소 시키고, 0보다 작으면 대기 큐에서 잠듦, 0보다 크거나 같으면, 자원 사용하는 코드 실행
  - signal 연산(V 연산) – 자원 사용이 끝났음을 알리는 과정
    - S를 1증가시키고, 0보다 크면 그냥 리턴, 0보다 작거나 같으면

### 5.5. 예시

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <pthread.h>
#include <semaphore.h> //세마포어 사용
#include <unistd.h>

sem_t sem; //세마포어 구조체 생성

void* myThread1(void *p) {
    int cnt = -1;

    sem_wait(&sem); // P 연산, 자원 사용 요청
    sem_getvalue(&sem, &cnt); // get the semaphore cnt
    printf("%s uses the resource. Remaining %d \n", (char*)p, cnt);

    sleep(1);

    sem_post(&sem); // V 연산, 자원 사용 끝
    sem_getvalue(&sem, &cnt); // get the semaphore cnt
    printf("%s finished using the resource. Remaining %d \n", (char*)p, cnt);
}

int main() {
    int cnt = -1;
    pthread_t tid[5];
    char *name[] = {"t1", "t2", "t3", "t4", "t5" };

    printf("Start!\n");
    sem_init(&sem, 0, 3); // set the semaphore count to 3
    sem_getvalue(&sem, &cnt);

    printf("1. Semaphore: %d \n", cnt);
    for(int i = 0; i < 5 ; i++)
        pthread_create(&tid[i], NULL, myThread1, (void*)name[i]);

    for(int i = 0; i < 5 ; i++)
        pthread_join(tid[i], NULL);
    sem_getvalue(&sem, &cnt);

    printf("2. Semaphore: %d \n", cnt);
    sem_destroy(&sem);
    return 0;
}
```

### 5.6. 세마포어의 오사용

- 프로세스가 세마포어를 사용하지 않고 바로 임계구역에 들어간 경우로 임계구역을 보호할 수 없음
- P를 두 번 사용하여 wake_up 신호가 발생하지 않은 경우로 프로세스 간의 동기화가 이루어지지 않아 세마포어 큐에서 대기하고 있는 프로세스들이 무한 대기에 빠짐
- P와 V를 반대로 사용하여 상호 배제가 보장되지 않은 경우로 임계구역을 보호할 수 없음
- (C/C++에서의 포인터 문제와 유사!)

### 5.7. 세마포어 vs 뮤텍스

- 가장 큰 차이점: **동기화 대상의 갯수**
  - Mutex는 동기화 대상이 only 1개일 때 사용
  - Semaphore는 동기화 대상이 1개 이상일 때 사용

- 세마포어는 뮤텍스가 될 수 있지만, **뮤텍스는 세마포어가 될 수 없음**
  - Mutex는 0, 1로 이루어진 이진 상태를 가지므로 Binary Semaphore!

- Mutex는 자원 소유 가능 + 책임을 가지는 반면, **Semaphore는 자원 소유 불가**
  - 뮤텍스는 상태 0, 1 뿐이므로 Lock 가질 수 있음
  - Semaphore는 책임이 없음

- Mutex는 소유하고 있는 스레드만이 현재 Mutex를 해제할 수 있음
  - 반면, Semaphore는 Semaphore를 소유하지 않는 스레드가 Semaphore를 해제할 수 있음

## 6. Monitor(모니터)

- 공유 자원을 내부적으로 숨기고 공유 자원에 접근하기 위한 **인터페이스만 제공**함으로써 자원을 보호하고 프로세스 간에 동기화를 시킴 (→ 추상화)

- **작동원리**
  - 임계구역으로 지정된 자원에 접근하고자 하는 프로세스는 직접 P나 V를 사용하지 않고 모니터에 작업 요청
  - 모니터는 요청받은 작업을 모니터 큐에 저장한 후 순서대로 처리하고 그 결과만 해당 프로세스 에 알려줌
- 일종의 클래스 개념
  - c.f., 스마트 포인터

![6. 모니터의 작동 원리]({{ img_path }}/6. 모니터의 작동 원리.png){: width='400'}

- 자바로 작성한 모니터 내부 코드

```java
monitor shared_balance {
    private int balance = 10;
    private boolean busy = false;
    private condition mon;

    public increase(int amount) {
        if (busy == true) mon.wait();
        busy = true;
        balance += amount;
        mon.signal();
    }
}
```