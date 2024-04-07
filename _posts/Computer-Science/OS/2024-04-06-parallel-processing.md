---
title: "[OS] 컴퓨터구조: 병렬 처리"
date: 2024-04-06 19:10:00 +0900
categories: [Computer-Science, OS]
tags: ['OS', '컴퓨터구조']
image:
  path: /assets/img/posts/OS/2024-04-06-parallel-processing/preview.png
  lqip: data:image/webp;base64,iVBORw0KGgoAAAANSUhEUgAAAAwAAAAGCAMAAADAMI+zAAAAAXNSR0IB2cksfwAAAAlwSFlzAAALEwAACxMBAJqcGAAAAFFQTFRF/////f7+9uPj9/f3/fb28Pb47+Pk4+Hh9vLy3OLm4urt9ufm3t7e9vb23dPT6enp1N7j7e3t3NnZ8/Pz6+Td7fP11+Lm8ebn8e3t4dPT4+PjuTW7gwAAAD1JREFUeJxjYEAGvMyMQCTMwMwvyMPAxc3IxcQqziIhwsPHIMDEyMkkysbCwc0uySAG4rCysQizSwmh6AcARPYB1q+KR64AAAAASUVORK5CYII=
  alt: "Concurrency vs Parallelism"
---
{% assign img_path = "/assets/img/posts/OS/2024-04-06-parallel-processing" %}



## 1. 병렬 처리

병렬 처리란 **동시에 여러 개의 명령을 처리**해 **작업의 능률을 올리는 방식**을 말한다.

### 1.1. Concurrency vs Parallelism

| 동시성 Concurrency                                  | 병렬성 Parallelism                                  |
| --------------------------------------------------- | --------------------------------------------------- |
| 동시에 실행되는 것 같이<br> 보이는 것               | 실제로 동시에 여러 작업이<br> 수행되는 것           |
| **싱글 코어**에서 멀티 쓰레드를<br> 동작시키는 방식 | **멀티 코어**에서 멀티 쓰레드를<br> 동작시키는 방식 |
| 한번에 많은 것을 처리                               | 한번에 많은 일을 처리                               |
| **논리적인 개념**                                   | **물리적인 개념**                                   |

![Concurrency vs Parallelism]({{ img_path }}/1.1. Concurrency vs Parallelism.png){: width='600'}
_Concurrency vs Parallelism_



## 2. 파이프라이닝(Pipelining)

- CPU에서 명령어가 처리되는 과정
  1. **명령어 페치(IF)**: 다음에 실행할 명령어를 명령어 레지스터에 저장
  2. **명령어 해석(ID)**: 명령어 해석
  3. **실행(EX)**: 해석된 결과를 토대로 명령어 실행
  4. **쓰기(WB)**: 실행 결과를 메모리에 저장

| 펄스 1 | 펄스 2 | 펄스 3 | 펄스 4 | 펄스 5 | 펄스 6 | 펄스 7 | 펄스 8 |
| ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| IF     | ID     | EX     | WB     | IF     | ID     | EX     | WB     |

<center>명령어 처리 단계</center>
<br>

**파이프라인이란?**
- 명령어를 겹쳐 실행하는 방법
- 하나의 코어에 여러 스레드를 실행하는 방식
- 명령어를 여러 단계로 분할 후, 각 단계를 동시에 처리하는 하드웨어를 독립적으로 구성하는 방법

![파이프라인 동작과정]({{ img_path }}/2. 파이프라인 동작과정.png){: width='400'}

### 2.1. 병렬 처리 시 고려 사항

- **상호 의존성이 없어야 함**
  - 각 명령이 서로 독립적, 앞 결과가 뒤 명령에 영향이 없어야 함
- **각 단계별 처리 시간이 일정해야 함**
  - 오랜 시간이 걸리는 작업으로 진행이 전반적으로 밀릴 수 있음
  - 병목 현상 발생 → **놀고 있는 Stage 발생 (파이프라인 버블)**

![병렬 처리 시 고려 사항]({{ img_path }}/2.1. 병렬 처리 시 고려 사항.png){: width='480'}

- 전체 작업 시간을 몇 단계로 나눌 지 잘 따져야 함
  - 동시의 처리할 수 있는 작업의 수를 **병렬 처리 깊이 N**으로 정의하자.
  - 이론적으론 **N ↑** 할수록, 동시 작업 개수가 많아져 **성능 ↑**
  - **but** 작업 너무 많이 나누면 각 단계마다 **작업 이동 / 새 작업 불러오는 시간**이 너무 오래 걸려 오히려 **성능 ↓**



## 3. Pipeline hazard

실제로는 처리 시간이 일정하지도 않고, 처리 단계까 균등하지도 않아 각종 문제가 발생하는데 이것을 **Pipeline hazard**라고 한다.

| 해저드(Hazard)    | 발생 이유                                 |
| ----------------- | ----------------------------------------- |
| 구조적 Structural | 자원 충돌로 여러 명령의 동시 수행 시 발생 |
| 제어 Control      | 순차적 명령어가 분기에 의해 버려지는 경우 |
| 데이터 Data       | 미수행된 명령의 결과값 참조 시도 발생     |

### 3.1. Structural hazard(구조 해저드)

- 프로세서 자원 부족할 때, 서로 다른 명령어가 같은 자원에 접근하려 할 때 발생
- **해결 방안**: 추가 자원 확보, 대기 (Stall, Bubble, Nop)

![Structural hazard]({{ img_path }}/3.1. Structural hazard.png){: width='500'}

### 3.2. Control hazard(제어 해저드)

- 프로그램 의존성에 의해 발생
- 분기에 의해 프로그램 제어 순서 바뀌어, 파이프라인 내 명령어들이 무가치해짐
- **해결 방안**: 대기(Stall), 예측(Predict branches), 우선 실행(Fast branches)

![Control hazard]({{ img_path }}/3.2. Control hazard.png){: width='600'}

### 3.3. Data hazard(데이터 해저드)

- 이전 명령어 결과 기반으로 다음 명령 수행 시 발생
  - 예시: `A = C + D;` `E = A * 2;`
- **해결 방안**: 전방전달, 비순차 실행, 코딩 잘하기

![Data hazard]({{ img_path }}/3.3. Data hazard.png){: width='600'}



## 4. 다양한 병렬 처리 기법

### 4.1. Super scalar

- 파이프라인 처리 코어를 여러 개 구성, 복수의 명령어가 동시에 실행되도록 하는 방식
- 대부분 파이프라인 기법과 동일, 하지만 코어를 다중으로 구성해 동시 실행 명령어가 다중이라는 점이 다름

![Super scalar]({{ img_path }}/4.1. Super scalar.png){: width='380'}

### 4.2. Super pipeline

- 파이프라인 각 단계를 세분화해 한 클록 내에 여러 명령어 처리
- 한 클록 내 여러 명령어 실행하면, 다음 명령어가 빠른 시간 안에 처리되므로 병렬 처리 능력 ↑

![Super pipeline]({{ img_path }}/4.2. Super pipeline.png){: width='320'}

### 4.3. Super-pipelined super-scalar

- 슈퍼 파이프라인 기법을 여러 코어에서 동시에 수행하는 방식

![Super-pipelined super-scalar]({{ img_path }}/4.3. Super-pipelined super-scalar.png){: width='320'}

### 4.4. VLIW(Very Long Instruction Word)

- CPU가 병렬 처리 미지원하는 경우 SW적으로 병렬 처리 하는 방법<br>
  → 컴파일 단계에 구성
- 동시 수행 가능 명령어들을 컴파일러가 추출, 하나의 명령어로 압축해 실행

### 4.5. SIMD(Single Instruction Multiple Data)

- 하나의 명령어로 여러 데이터를 한번에 처리하는 기법

![SIMD]({{ img_path }}/4.5. SIMD.png){: width='350'}



## 5. 무어의 법칙, 암달의 법칙

### 5.1. 무어의 법칙(Moore's law)

- CPU 속도가 24개월마다 2배 빨라진다는 내용
- 초기 CPU에만 적용, 현대에선 그렇지 않음

### 5.2. 암달의 법칙(Amdahl's law)

- 컴퓨터 시스템 일부 개선 시 전체 시스템에 미치는 영향과의 관계를 수식으로 나타낸 법칙
- 이 법칙에 따르면, 주변 장치 향상 없이 CPU의 속도를 2GHz → 4GHz로 늘리더라도 컴퓨터 성능이 2배 빨라지지 않음