---
title: "[OS] 운영체제 개요"
date: 2024-04-01 12:24:00 +0900
categories: [Computer-Science, OS]
tags: ['OS']
image:
  path: /assets/img/posts/OS/2024-04-01-operating-system-intro/preview.png
  lqip: data:image/webp;base64,iVBORw0KGgoAAAANSUhEUgAAAAoAAAAFCAMAAABLuo1aAAAAAXNSR0IB2cksfwAAAAlwSFlzAAALEwAACxMBAJqcGAAAAJZQTFRF/v7+/P39+Pf17/b59vTr+vPx+vn59/f2+/v7/fT06e706u/15uvy5+zz6O3z5+zy5Orx5Onv5erv5ezz4urz4er03Obx2eLt3ebw2OHqz9nk1d7n3ebv2OLu7PP57fP66fH45u726vH45+715ez05Ovx5Orw6efv6+zt6enr6urr9vb37vHz4uPi4+bf5ujp4+nr9PPyU4KNiAAAABdJREFUeJxjZGCEAy4EUwTBlEMwNeAsAA3PAJdS6Y30AAAAAElFTkSuQmCC
  alt: "운영체제의 기능 요약"
---



## 1. 운영체제란?

**운영체제(OS, Operating System)**란 조직되어 통일된 전체(System)을 관리/운용해 나간다(Operating)는 의미로, 하드웨어(HW, Hardware)를 소프트웨어(SW, Software)로 추상화(Abstraction)[^abstract]한 소프트웨어이다.

### 1.1. OS의 핵심 기능

- 프로세스 관리
- 메모리 관리
- 입출력 및 파일 관리

### 1.2. 관련된 문제

OS는 **여러 종류의 SW를 동시에 작동하게 하는 문제**를 해결하는 과정에서 등장하였으며, OS를 개발하기 위해서는 다음과 같은 문제를 해결해야 했다.

- 누가, 언제, 얼마나 CPU를 사용해야 하는가?
  1. 프로세스 작동 시기는 언제인가?
    - 프로세스는 어떻게 교체하는가?
  2. 프로세스의 메모리 로드 관리는 어떻게 하는가?
  3. CPU와 메모리를 어떻게 연결하는가?
  4. 내려간 프로세스는 어떻게 저장하는가?
   
그리고 아래와 같은 여러 가지 파생 문제가 생겨났다.
- 프로세스 동기화
- 교착상태, 기아상태
- 저장장치 관리
- 분산 시스템, 가상화 시스템

OS의 개념은 HW를 관리하는 소프트웨어에 국한된 것이 아니며, 컴퓨터공학에서 무엇인가를 **관리**하려 할 때 광범위하게 적용될 수 있다. 예를 들어, SW가 무수한 자원을 관리할 수도 있는데, 이 때 OS의 개념이 적용될 수 있다.

[^abstract]: 추상화(abstraction)는 컴퓨터공학의 목표 중 하나이다.



## 2. 운영체제 소개

### 2.1. 운영체제란?

1. **가장 먼저 실행**되는 소프트웨어
1. **컴퓨터 자원을 독점/관리**하는 소프트웨어
1. 저장장치의 **특정 영역에 저장**되어 있음
1. 컴퓨터 자원을 **추상화**하여 **인터페이스를 제공**

### 2.2. 운영체제의 목표

자원관리를 **'잘'** 하기 위한 것으로, 아래의 기능을 갖춰야 한다.
- 효율성(Efficiency)
- 안정성(Robustness, Security)
- 확장 가능성(Scalability, Extensibility, Portability)
- 편리성(Usability, Interactivity)

{% assign img_path = "/assets/img/posts/OS/2024-04-01-operating-system-intro" %}
![운영체제 역할과 목표]({{ img_path }}/2.2. 운영체제 역할과 목표.png){: width='400'}

### 2.3. 운영체제 기능, 하는 일

- **보안** / **네트워크** / **장치** / **프로세스** / **메모리** / **파일시스템** 관리
- 그리고 **기타 관리** (사용자 계정, 통계, 오류 발견/대응 등)

운영체제의 기능과 하는 일은 필요 시 지속적으로 확장되어 왔다.

![운영체제 기능]({{ img_path }}/2.3. 운영체제 기능.png)

- 초기에는 **장치 관리** 기능밖에 존재하지 않았음
- 그러다 여러 프로그램을 동시에 돌려야 되자 **프로세스 관리** 기능이 추가되고,
- 메모리가 한정되었으므로 **메모리 관리 기능**이,
- 작업 보관의 필요성이 생겨 **파일 시스템 관리 기능**이,
- 이 모든 작업이 이상없이 돌아가는지 확인하기 위해 **통계 등 기타 관리 기능**이 생겨남
- 그러다 네트워크가 등장하고, 장치 밖의 자원 관리 위해 **네트워크 관리 기능**이 생기고,
- 외부 사용자 신뢰성 확보 위해 **보안 관리 기능**이,
- 여러 사용자의 관리를 위해 **사용자 계정 관리** 기능 등이 생겨나게 되었음

### 2.4. 운영체제가 없다면?

**작동은 가능**하다. 하지만 **한정된 기능**과 **제어가 쉽지 않다는 것** 뿐이다.
- 추가적인 장치연결 및 SW 설치가 불가능
- 사용 불편, 대응 어려움
  - 공학용 계산기, 펌웨어 등
  - 회로 수준에서 구워져 나온 범위 내에서 작동하는 기기들

> 운영체제가 없다는 것이 **꼭 나쁜 것은 아닐 수도 있다!**
> - **특수 하드웨어를 조작**하는 것에는 적합할 가능성 有
> - 확장성이 결여되었다는 것 → **보안성 증가**했다는 것일 수도...
> - 불편한 인터페이스라는 것 → **비전문가 접근에 대해 보호**받는다는 것
{: .prompt-info}



## 3. 운영체제 역사

### 3.1. 고정 프로그램 방식(Fixed Program Computer)

- 하나의 기계 = **하나의 기능**을 의미
- 기능 변경 = **기계를 새로 만듦**을 의미
- 하드 와이어링 사용
- 회로 수정의 조금의 편의를 제공해주는 수준

### 3.2. 일괄 작업 시스템(Batch-Processing System)

**= 내장 프로그래밍 방식(Stored Program Computer)**

- Reader: 천공 카드 / Printer: 라인 프린터였음
- 전문 연산장치(CPU)와 로직 저장장치(RAM)의 분리
  - 폰 노이만 구조
- 운영체제 기능 일부(**장치 관리**) 제공
- 작업을 **한번에 1개**만 처리 가능
  - 중간에 입력값 제어하거나, 결과 확인 불가능

![일괄 작업 시스템]({{ img_path }}/3.2. 일괄 작업 시스템.png){: width='500'}

### 3.3. 대화형 시스템(Interacive System)

- 모니터/키보드 등장
- 프로그램 실행 도중 사용자 입력 가능
  - 하나의 장치에서 어떻게 여러 개 작업하는 것인가? → 다중 프로그래밍

![대화형 시스템]({{ img_path }}/3.3. 대화형 시스템.png){: width='500'}

### 3.4. 다중 프로그래밍

- 하나의 CPU로 여러 작업 동시 실행
  - CPU사용시간을 각 작업에 분할 할당, 동시에 실행하는 것처럼 보이게 함
- 프로그램 실행 중 I/O 발생 → 다른 프로그램 실행
- 오늘날 대부분의 시스템에서 채택

![다중 프로그래밍]({{ img_path }}/3.4. 다중 프로그래밍.png)

### 3.5. 요약

|구분|시기        |주요 기술            |특징
|----|------------|---------------------|-------------------------------------------
|0기|1940년대     |없음                 |진공관 사용
|1기|1950년대     |카드 리더, 라인프린터|일괄 작업 시스템<br>운영체제의 등장
|2기|1960년대     |키보드, 모니터       |대화형 시스템
|3기|1960년대 초반|C언어                |다중 프로그래밍 기술 개발<br>시분할 시스템
|4기|1970년대 후반|PC                   |개인용 컴퓨터의 등장<br>분산 시스템
|5기|1990년대     |웹                   |클라이언트/서버 시스템
|6기|2000년대     |스마트폰             |P2P 시스템<br>그리드 컴퓨팅<br>클라우드 컴퓨팅<br>사물 인터넷

## 4. 일괄 처리 시스템 VS 다중 프로그래밍

![일괄 처리 시스템 VS 다중 프로그래밍]({{ img_path }}/4. 일괄 처리 시스템 VS 다중 프로그래밍.png)



## 5. 기타 시스템
### 5.1. P2P(Peer-to-Peer) 시스템

- **특정 서버의 과부화**를 해결
  - 단일 장애점 (Single point of failure)
- 서버 고치지 않고, 사용자를 직접 연결함
- **그리드 컴퓨팅**, **클라우드 컴퓨팅**, **사물 인터넷**, 네트워크 OS, 분산 운영체제 동기화(synchronization), 협의(consensus)

### 5.2. 가상화 시스템(Virtualization)

OS는 하드웨어에 대한 추상화: 컴퓨터도 추상화할 수 있다.
- 컴**퓨터 자체를 소프트웨어로 구현한 것**
- 독립공간(sandbox) 지원
- Hypervisor(Virtual Machine Manager), Container(Docker)

### 5.3. 모바일 운영체제

- 모바일 전화기나 스마트폰 등 휴대 장치에서 실행됨
- Network, Power to performance ratio(**전성비**) 추구

### 5.4. 임베디드 운엉체제

- 임베디드 컴퓨터에서 사용됨

### 5.5. 실시간 운영체제

실시간 애플리케이션 또는 태스크를 **정해진 데드라인(deadline) 시간 이내에 처리되도록 보장**하는 것을 목표로 한다.
- 처리시간 예측 가능하고, 보장함 (Timing guarantee)
- pSOS, VxWorks, VRTX, RT-Linux, Lynx



<br><br><br><br>

---
<!-- omit from toc -->
#### 각주
{: data-toc-skip=''}