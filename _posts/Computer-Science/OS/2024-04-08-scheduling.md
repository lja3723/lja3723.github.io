---
title: "[OS] 스케줄링"
date: 2024-04-08 09:18:00 +0900
categories: [Computer-Science, OS]
tags: ['OS', '스케줄링']
---



## 1. 스케줄링 개요

**자원(Resource)**이 부족한 곳에서는 항상 스케줄링이 존재한다. 이것은 일상생활에서도 마찬가지이다. **컴퓨터 시스템** 안에서는 다음과 같은 스케줄링들이 있다.

- 작업(job) 스케줄링
  - 대기 중 배치 작업 중 메모리에 적재할 작업
- **CPU 스케줄링**
  - 프로세스/스레드 중 하나 선택해 CPU에 할당
  - 오늘날의 CPU 스케줄링은 **스레드 스케줄링**
- 디스크 스케줄링
  - 디스크 장치 내에서, 입출력 요청 처리 순서
- 프린터 스케줄링
- 네트워크 스케줄링
- 서버에서의 요청 처리(예: 티켓팅 등)

### 1.1. 다중 프로그래밍과 스케줄링

- 다중 프로그래밍의 도입 목적
  - 여러 프로세스가 하나의 CPU를 돌아가며 사용해야 함
    - 어떤 순서로 작업시킬지, 스케줄링 개념 필요
  - 스케줄링이 잘 되면 **CPU idle 시간↓** → **CPU 활용률↑**

- 그래서 다중 프로그래밍에선 두 가지 스케줄링 개념 필요
  1. **Job Scheduling**
    - 디스크 장치로부터 메모리에 올릴 작업 선택하기
  2. **CPU Scheduling**
    - 메모리에 적재된 작업 중 CPU에 실행시킬 프로세스 선택

{% assign img_path = "/assets/img/posts/OS/2024-04-08-scheduling" %}
![다중 프로그래밍과 스케줄링]({{ img_path }}/1.1. 다중 프로그래밍과 스케줄링.png){: width='650'}
_다중 프로그래밍과 스케줄링_




## 2. 스케줄링 단계에 따른 구분

- 발생 빈도 및 할당 자원에 따른 구분

### 2.1. 고수준 스케줄링

- **Long-term scheduling, Job scheduling**
- 시스템 내 전체 작업 수 조절하는 것
- 시스탬 내 동시 실행 가능한 프로세스 개수 결정
  - 다중 프로그래밍의 정도(Degree) 결정

> 장기 스케줄링, 승인 스키줄링이라고도 한다.
{: .prompt-info }

### 2.2. 중간 수준 스케줄링

- **Mid-term scheduling, Swapping**
- 중지/활성화로 전체 시스템의 활성화된 프로세스 수 조절 → 과부하 방지
- 일부 프로세스를 정지 상태로 옮김 → 프로세스가 원만히 작동하도록 지원
  - 메모리 할당 관련

### 2.3. **저수준 스케줄링**

- Short-term scheduling, Processor/CPU scheduling
- 어떤 프로세스에게 CPU 할당할지, 어떤 프로세스를 대기 상태로 보낼지 등 결정

> 아주 짧은 시간 일어나므로 **단기 스케줄링**이라고도 한다.
{: .prompt-info }

<!-- omit from toc -->
#### **참고**
{: data-toc-skip=''}

![alt text]({{ img_path }}/2. 스케줄링 단계.png){: width='470'}
_프로세스 라이프사이클과 스케줄링 단계 (1)_

![alt text]({{ img_path }}/2. 스케줄링 단계 2.png){: width='580'}
_프로세스 라이프사이클과 스케줄링 단계 (2)_



## 3. 스케줄링 결정 기준

프로세스는 **CPU 연산 작업**과 **I/O 작업의 연속**이다. 따라서 아래 두 가지로 구분할 수 있다.
- CPU burst 프로세스
- I/O burst 프로세스

|                   | CPU burst 프로세스 | I/O burst 프로세스 |
| ----------------- | ------------------ | ------------------ |
| 주요 연산         | CPU 연산           | 입출력 작업        |
| 필요 CPU 할당시간 | 많음               | 적음               |

- **프로세스 수행 시간** $=$ **CPU 사용시간** $+$ **I/O 대기 시간**

![스케줄링 결정 기준]({{ img_path }}/3. 스케줄링 결정 기준.png){: width='530'}

![스케줄링 결정 기준 2]({{ img_path }}/3. 스케줄링 결정 기준 2.png){: width='530'}

### 3.1. CPU 스케줄링

- 실행 준비 상태의 스레드 중 하나를 선택하는 과정
- CPU 사용 순서를 경정
- **기본 목표: CPU를 놀게 하지 않는 것(idle의 지양)**
  - CPU 활용률 극대화 → 시스템 처리율 향상
  - 달성 방법: **타임 슬라이싱(Time Slicing)**

> 대부분의 OS에서 하나의 스레드가 너무 오래 CPU를 사용하도록 허용하지 않는다.
{: .prompt-warning}

### 3.2. 타임 슬라이스(Time slice)

- 스케줄된 스레드에게 한 번 할당하는 CPU 시간
- 커널이 스케줄을 단행하는 주기 시간
  - 타이머 인터럽트 도움 받아 타임 슬라이스 단위로 CPU 스케줄링
  - 현재 실행 중 스레드 강제 중단(preemption), 준비 리스트에 삽입

> 타임 퀀텀(time quantum), 타임 슬롯(time slot)이라고도 함
{: .prompt-info}

#### 3.2.1. 시간 할당 기준(Criteria)들

시간을 그러면 얼마나 할당할 것인가? **아래 기준들**을 고려함

|                     기준                     | 설명                                                           | 입장            |
| :------------------------------------------: | -------------------------------------------------------------- | --------------- |
| CPU 활용률 / Efficiency<br>(CPU utilization) | 전체 시간 중 CPU 사용시간 비율                                 | OS 입장         |
|                    처리율                    | 단위시간당 처리하는 프로세스 수                                | OS 입장         |
|         응답시간<br>(response time)          | 대화식 사용자의 경우, 명령에 <br>응답하는데 걸리는 시간        | 사용자 입장     |
|          대기시간<br>(waiting time)          | 스레드가 준비 큐에서 머무르는 시간                             | OS, 사용자 입장 |
|  소요시간[^batch-sys]<br>(turnaround time)   | 프로세스(스레드)가 시스템에 도착<br> 후 완료될때까지 걸린 시간 | 사용자 입장     |

- **공평성(fairness) / Load balancing**
  - CPU를 스레드들에게 공평하게 배분
  - 무한정 대기하는 기아 스레드(starving thread) 생기지 않도록 스케줄
  - **사용자 입장**

- **시스템 정책(policy enforcement) 우선**
  - 컴퓨터 시스템의 특별한 목적 달성하기 위한 스케줄링
  - **OS 입장**
 - **예)** 실시간 시스템: 스레드가 완료시간(deadline) 내 이뤄지도록 하는 정책
 - **예)** 급여 시스템: 안전 관리 스레드 우선 정책 등

- 그 외 기준들
  - 자원 활용률(resource efficiency)
  - 우선순위(Priority)...

[^batch-sys]: 배치 처리 시스템에서의 주된 스케줄링 기준이다.

![대기시간, 실행시간, 응답시간, 반환시간]({{ img_path }}/3.2.1. 대기시간, 실행시간, 응답시간, 반환시간.png){: width='480'}
_대기시간, 실행시간, 응답시간, 반환시간_



## 4. CPU 스케줄링이 실행되는 4가지 상황

### 4.1. 스레드가 시스템 호출 끝에 I/O를 요청하여 블록될 때

- 스레드를 블록 상태로 만들고 스케줄링
- **CPU의 활용률 향상 목적**

### 4.2. 스레드가 자발적으로 CPU를 반환할 때

- yield() syscall 통해 스레드가 자발적으로 CPU 반환
- 커널은 현제 스레드를 준비 리스트에 넣고, 새로운 스레드 선택
- **CPU의 자발적 양보**

### 4.3. 스레드의 타임 슬라이드가 소진되어 타이머 인터럽트 발생

- **균등한 CPU 분배 목적**

### 4.4. 더 높은 순위의 스레드가 요청한 입출력 작업 완료, 인터럽트 발생

- 현재 스레드를 강제 중단시켜 준비 리스트에 넣고
- 높은 순위의 스레드 깨워 스케줄링
- **우선순위를 지키기 위한 목적**



## 5. Scheduler와 Dispatcher

- 스케줄링 관련 코드는 **커널 내 코드 형태로 있다. (중요하다는 의미)**
  - 별도로 실행되는 커널 프로세스 or 스레드 형태가 X
  - syscall이나 ISR(Interrupt Service Routine)이 끝나는 마지막 단계에서 실행
- **스케줄러 & 디스페처 모두 실행시간이 짧도록 작성된다.**

### 5.1. 스케줄러

- 스케줄러 타이머가 주기적으로 인터럽트 발생시킴
- 인터럽트 코드는 현재 실행 중 프로세스의 실행시간을 업데이트 시킴
- 스케줄링해야 하는 경우, 스케줄링 플래그 비트가 스케줄링 타이머 인터럽트에 설정됨
- 인터럽트 종료(return)
- 커널은 인터럽트의 리턴에 스케줄링 플래그 비트가 설정되었는지 여부를 판별, 되어있으면 디스패처 작동

### 5.2. 디스패처

- **컨텍스트 스위칭 실행하는 커널 코드**
- 스케줄러에 의해 선택된 스레드를 CPU가 실행하도록 하는 작업
- 커널 모드 → 사용자 모드 전환
- 새로 선택된 스레드가 이전에 중단된 곳에스 실행하도록 점프

### 5.3. Scheduling process

![Scheduling process]({{ img_path }}/5.3. Scheduling process.png){: width='650'}
_스케줄링 프로세스_



## 6. 선점과 비선점

- 실행 중인 스레드의 강제 중단 여부에 따라 CPU 스케줄링 타입을 구분할 수 있다.
- **오늘날의 범용 OS**: 선점 스케줄링 타입을 사용한다.

<!-- omit from toc -->
#### **비교**
{: data-toc-skip=''}

|   구분    | 선점형                                                  | 비선점형                                           |
| :-------: | ------------------------------------------------------- | -------------------------------------------------- |
| 작업 방식 | 실행중 작업 중단시키고<br> 새 작업 실행 가능            | 실행중작업 완료될 때까지<br> 다른 작업 불가능      |
|   장점    | 프로세스가 CPU 독점 불가<br>대화형/시분할 시스템에 적합 | CPU스케줄러 작업량↓<br>Context Switching 오버헤드↓ |
|   단점    | Context Switching 오버헤드↑                             | 기다리는 프로세스 많아 처리율↓                     |
|   사용    | 시분할 방식 스케줄러                                    | 일괄 작업방식 스케줄러                             |
|  중요도   | 높음                                                    | 낮음                                               |

### 6.1. 비선점 스케줄링(non-preemptive scheduling)

- 현재 실행중인 스레드를 강제로 중단시키지 않음
- 일단 스레드가 CPU를 할당 받아 실행을 시작하면, 작업 완료되거나 CPU를 더이상 사용할 수 없는 상황이 될 때까지 **스레드를 강제 중단시키지 않고** 스케줄링도 하지 않는 방식

#### 6.1.1. 스케줄링 시점

1. CPU를 더 이상 사용할 수 없게 된 경우
  - I/O로 인한 블록 상태
  - sleep 등
2. 자발적으로 CPU 양보할 때
3. 실행 중 종료할 때

![비선점 스케줄링]({{ img_path }}/6.1. 비선점 스케줄링.png){: width='620'}
_비선점 스케줄링 예시_

### 6.2. 선점 스케줄링(preemptive scheduling)

- 현재 실행 중인 스레드 강제 중단시키고 다른 스레드 선택, CPU 할당하는 방식
- **오늘날 대부분의 범용 OS가 사용**하는 방식

### 6.2.2. 스케줄링 시점

1. 타임 슬라이스가 소진되어 타이머 인터럽트가 발생할 때
2. 인터럽트나 시스템 호출 종료 시점에서, 더 높은 순위의 스레드가 준비 상태일 때

![선점 스케줄링]({{ img_path }}/6.2. 선점 스케줄링.png){: width='620'}
_선점 스케줄링 예시_



## 7. 우선순위

- **우선순위 높은 프로세스**가 CPU를 먼저, 더 오래 차지한다.
- 시스템에 따라 높은 우선순위를 의미하는 숫자는 높은 숫자일 수도 있고, 낮은 숫자일 수도 있음[^priority-linux]

[^priority-linux]: 리눅스는 숫자가 작을수록 우선순위가 높다. (가장 높은 우선순위는 -20, `nice` 명령어로 설정 가능 (단 루트권한 필요))

### 7.1. 우선순위의 일반적인 결정 기준

|    우선순위 높음    |        |    우선순위 낮음    |
| :-----------------: | :----: | :-----------------: |
|   Kernel 프로세스   | **vs** | User-level 프로세스 |
| I/O burst 프로세스  | **vs** | CPU burst 프로세스  |
| Foreground 프로세스 | **vs** | Background 프로세스 |

![우선순위]({{ img_path }}/7. 우선순위.png){: width='620'}
![스케줄링 시 고려 사항]({{ img_path }}/7. 스케줄링 시 고려 사항.png){: width='380'}

### 7.1. 프로세스의 우선순위를 배정하는 방식

#### 7.1.1. 정적 스케줄링(Static Scheduling)

- OS가 프로세스에 우선순위 한 번 부여하면 끝날때까지 바뀌지 않는 방식
- 구현이 쉬움
- 시스템 변화 대응 어려우므로 **작업 효율↓**

#### 7.1.2. 동적 스케줄링(Dynamic Scheduling)

- 프로세스 생성 시 부여받은 **우선순위가 작업 중간에 변하는** 방식
- 구현 어려움
- **시스템 효율성↑**



## 8. 기아 현상과 에이징

### 8.1. 기아(starvation)

- 스레드가 스케줄링에서 선택되지 못한 채 오랜 동안 준비 리스트에 있는 상황
- 사례
  - **우선순위 기반 시스템**: 높은 순위 스레드가 계속 시스템에 들어오는 경우
  - **짧은 스레드 우선실행 시스템**: 자신보다 짧은 스레드가 계속 도착하는 경우
- 스케줄링 알고리즘 설계 시 **기아 발생을 면밀히 평가**한다.
  - 기아가 발생하지 않도록 설계하는 것이 바람직함

### 8.2. 에이징(aging)

- **기아의 해결책**
- 스레드가 준비 리스트에 **머무르는 시간에 비례**하여 **스케줄링 순위를 높이는 기법**
- 비록 오래 기다릴 수도 있지만, 언젠가는 가장 높은 순위에 도달하는 것을 보장



## 9. 스케줄링: 큐(Queue)에 관한 이야기

**준비 상태로 변한 프로세스**는 **자신의 우선순위에 해당하는 큐의 마지막에 삽입**된다.
- CPU 스케줄러는 **우선순위가 가장 높은 큐**의 맨 앞에 있는 프로세스에 CPU 할당함
- 대기상태에서는 시스템의 효율을 높이기 위해 같은 입출력끼리 모아둠

### 9.1. 준비 상태의 다중 큐

![준비 상태의 다중 큐]({{ img_path }}/9.1. 준비 상태의 다중 큐.png){: width='570'}

### 9.2. 대기 상태의 다중 큐

![대기 상태의 다중 큐]({{ img_path }}/9.2. 대기 상태의 다중 큐.png){: width='570'}



<br><br><br><br>

---
<!-- omit from toc -->
#### 각주
{: data-toc-skip=''}