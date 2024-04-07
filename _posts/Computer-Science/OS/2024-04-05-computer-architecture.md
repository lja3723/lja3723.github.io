---
title: "[OS] 컴퓨터구조: 하드웨어"
date: 2024-04-05 19:05:00 +0900
categories: [Computer-Science, OS]
tags: ['OS', '컴퓨터구조']
image:
  path: /assets/img/posts/OS/2024-04-05-computer-architecture/preview.png
  lqip: data:image/webp;base64,iVBORw0KGgoAAAANSUhEUgAAAAwAAAAGCAMAAADAMI+zAAAAAXNSR0IB2cksfwAAAAlwSFlzAAALEwAACxMBAJqcGAAAAG9QTFRF/////v7+9Pb48vT36evu5uns/f3+2OPwzdztydfoxtXm0t/u+/z9+fn57u7v5eTl7e/w7vDy4OLl3+Hk6uzv4+nt4ODi8vLz9fX19/f35ubm5uXm6+rq3dzd29vc8vLy6+7v4ODh/f39/Pz8+/v72F0M2QAAADNJREFUeJxjZEACjEAEAv8YmH+BOWwgDjPjFwZGXkYkIAnEv0FyjG85GZUYQTr/g/T/BwD4GQhYBOU8kgAAAABJRU5ErkJggg==
  alt: "컴퓨터 계층 구조"
---
{% assign img_path = "/assets/img/posts/OS/2024-04-05-computer-architecture" %}



운영체제를 본격적으로 공부하기 위해서는 컴퓨터 아키텍쳐(컴퓨터 구조)를 잘 알아야 하며, 이 게시글에서는 컴퓨터의 하드웨어적 내용을 요약한다.

## 1. 컴퓨터의 계층 구조

- 응용 프로그램 ↔ 운영체제 ↔ 하드웨어
  - 사용자는 응용프로그램 통해 컴퓨터 활용
  - **응용프로그램은 하드웨어 직접 접근 X**
    - 반드시 OS를 통해서만 가능하다.
  - 하드웨어는 OS의 배타적 독점적 지배를 받는다.

이런 설계의 배경에는 **추상화**가 있다.

![컴퓨터의 계층 구조]({{ img_path }}/1. 컴퓨터의 계층 구조.png){: width='600'}



## 2. 컴퓨터 하드웨어 구성

- 필수 장치: CPU, 메모리
- **CPU**: 명령어를 해석/실행하는 장치(Patch - Decode - Execute)
- **메모리**: 명령어 및 필요한 데이터를 저장하는 곳
- 나머지 주변 장치: **입출력장치**, **저장장치**

컴퓨터의 모든 장치는 최종적으로 **메인보드**와 연결되며, 메인보드 자체에 각종 장치/기능들(LAN card, Sound card)이 포함되기도 한다.

### 2.1. 버스(BUS)

메인보드에 연결된 각 장치들은 메인보드 내 **버스(BUS)**를 통해 통신한다.

- **CPU 내부 버스 (Back-Side Bus)**
- **시스템 버스 (Front-Side Bus)**
- **I/O 버스**
- 데이터 버스
- 주소 버스
- 제어 버스

![버스]({{ img_path }}/2.1. 버스.png){: width='600'}

### 2.2. 폰 노이만 구조

프로그램은 보통 저장장치에 **파일 형식으로 저장**되어 있고, CPU가 이것을 실행하려면 **메인 메모리에 적재**되어 있어야 한다.

- **메인 메모리**가 유일한 작업 공간
  -  메모리 관리는 매우 중요한 이슈



## 3. 클럭, 버스의 동작방식

- 컴퓨터 장치는 Clock 단위로 작업함
- **Clock**: 장치들의 작동을 동기화해주는 주기적인 타이밍 신호
  - Clock Generator가 생성해 줌

### 3.1. CPU가 메모리에 데이터를 입출력하는 과정

- **제어 버스**로 Read/Write 모드 결정
- **주소 버스**에 입출력하고자 하는 메모리 주소 적어 둠
- 다음 클럭 사이클(Next clock cycle)에 **데이터 버스**로 해당 데이터가 입출력 됨

![데이터 입출력 과정 1]({{ img_path }}/3.1. 데이터 입출력 과정 1.png){: width='550'}

![데이터 입출력 과정 2]({{ img_path }}/3.1. 데이터 입출력 과정 2.png){: width='550'}



## 4. 병목현상

- 읽을 데이터 크기 > 버스의 크기[^bus_size] 일 때 발생
  - 데이터를 **여러 클럭**에 나눠 읽을 수밖에 없음
- 메모리에 갑자기 **많은 입출력**이 동시에 발생하면?
  - 늦게 요청된 작업의 입출력은 **앞 처리를 기다려야만 함**

메모리가 CPU보다 느리므로, 메모리는 항상 병목에 시달릴 수 밖에 없다. 따라서 **병목현상을 극복하기 위해서는 특별한 설계**가 필요한데, 크게 **물리적 설계**, **구조적 설계**가 있다.

[^bus_size]: CPU가 한 번에 처리할 수 있는 데이터의 크기로, Word라고 한다.

### 4.1. 병목 극복법: 물리적 설계

- 메모리 클럭 ↑, 클럭 낭비 ↓
  - **DDR**: Double Data Rate
    - 클럭 웨이브에서 **상승 엣지**와 **하강 엣지**를 모두 데이터 처리에 할당한다.
- **듀얼 채널** 구성
  - 메모리 꽂는 슬롯에 칸 별로 색이 칠해진 이유
  - **병렬화**를 통해 한번에 더 많은 데이터에 접근한다.
- **CPU와 가깝게 위치**시켜 이동시간을 아낌

### 4.2. 병목 극복법: 구조적 설계

- **버퍼(Buffer)**[^buffer]를 도입하기
  - 속도 차이가 있는 두 장치 사이에서 차이를 완하하는 역할을 한다.
  - 일정량의 데이터를 모아 옮김으로 차이를 완화
- **스풀(Spool)**: 버퍼의 SW적 구현 형태
  - `ex` 프린터

[^buffer]: 데이터 전달 간 임시 저장하는 장치/장소는 모두 버퍼라고 부른다.

![구조적 설계]({{ img_path }}/4.2. 구조적 설계.png){: width='500'}

![캐시 구조]({{ img_path }}/4.2. 캐시 구조.png){: width='500'}



## 5. 메모리
### 5.1. 메모리 계층 구조

- **캐시(Cache)**: 메모리 ↔ CPU 간 속도 차이 완화를 위해서 메모리에서 **앞으로 사용이 예상[^predict]되는 데이터를 미리 가져 와서** 임시 저장해두는 장소
  - CPU에 대한 버퍼
  - 메모리의 메모리
- CPU는 메모리 접근이 필요할 때, 캐시를 우선 방문하여 데이터에 접근한다.

[^predict]: 예상하기 위해서는 지역성(locality)을 이용하는데, 크게 시간 지역성(ex: for loop)와 공간 지역성(ex: array)가 있다.

### 5.2. 메모리의 종류

- **휘발성** 메모리, **비휘발성** 메모리
- Random Access Memory(**RAM**), Read Only Memory(**ROM**)
- Sequential access memory (ex: 자기테이프)

![메모리의 종류]({{ img_path }}/5.2. 메모리의 종류.png){: width='350'}



<br><br><br><br>

---
#### 각주
{: data-toc-skip=''}