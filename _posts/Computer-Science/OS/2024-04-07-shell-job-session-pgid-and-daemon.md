---
title: "[OS] 셸, 작업/세션 관리와 PGID, Daemon"
date: 2024-04-07 21:15:00 +0900
categories: [Computer-Science, OS]
tags: ['OS', '프로세스']
---
{% assign img_path = "/assets/img/posts/OS/2024-04-07-shell-job-session-pgid-and-daemon" %}



## 1. 셸, 작업/세션 관리

### 1.1. 셸(Shell)이란?

- 사용자와 운영체제(kernel) 사이의 **인터페이스 역할**을 하는 프로그램/환경>

![셸(Shell)이란]({{ img_path }}/1.1. 셸(Shell)이란.png)
_셸의 다양한 예시들_

### 1.2. 작업(Job)이란?

- 셸이 **Background로 프로세스를 제어**하는 동작 구조

![작업(Job)이란]({{ img_path }}/1.2. 작업(Job)이란.png){: width='500'}
_job의 예시_

### 1.3. 세션(Session)이란?

- 사용자가 Terminal(ex: ssh)을 사용해 시스템에 로그인 했을 때, **활동하는 상태 등을 관리하는 단위**
- 각 세션에는 해당 세션을 제어하는 Terminal 존재 → **Session ID 통해 관리**

![세션(Session)이란]({{ img_path }}/1.3. 세션(Session)이란.png){: width='500'}
_Session의 예시_

#### 1.3.1. 세션의 확인

- `ps ajx` 명령어 사용

![세션의 확인]({{ img_path }}/1.3.1. 세션의 확인.png){: width='550'}

![세션의 확인 2]({{ img_path }}/1.3.1. 세션의 확인 2.png){: width='550'}
_`ps ajx` 실행 결과_

---

- 터미널이 닫히면 세션이 닫히고, 실행중이던 **모든 프로세스가 종료**[^closing-terminal]된다.
  - 따라서 세션이 안 끊기도록 유지를 잘 할 필요가 있음

만약 긴 작업을 돌려야 한다면 어떻게 해야 될까? 터미널이 닫혀도 계속 실행되게 할 수 없을까?

> **작업을 계속 켜 둘 방법이 없을까?**

[^closing-terminal]: 단말이 끊길 경우, 세션에 `SIGHUP` 시그널이 전달되어 종료되는 원리이다.

### 1.4. 작업을 오래 돌리는 방법

1. ssh의 경우, **`timeout` 시간을 증가**시키거나, **`heartbeat`를 주기적으로 보낸다.**
2. **nohup 명령어를 사용**한다.
  - `SIGHUP`을 무시하도록 설정하고 프로세스를 가동하는 것이다.
  - 만약 세션이 종료되어도, 프로세스를 종료하지 않도록 한다.

#### **example: nohup이 없을 때**
{: data-toc-skip=''}

![nohup이 없을 때]({{ img_path }}/1.4. nohup이 없을 때.png){: width='600'}

#### **example: nohup이 있을 때**
{: data-toc-skip=''}

![nohup이 있을 때]({{ img_path }}/1.4. nohup이 있을 때.png){: width='600'}



## 2. 프로세스 그룹(PGID)

- 여러 프로세스를 **하나로 묶어서 관리하는 단위** (PGID)

### 2.1. Foreground Process Group

- 현재 사용자가 **직접 사용하고 있는 프로세스**(터미널에서 접근 가능)
  - **세션당 하나만 존재**
- 만약 여러 프로세스가 **pipe**로 연결된다면 **하나의 프로세스 그룹**임

### 2.2. Background Process Group

- 사용자와 **직접적인 상호작용 없이 실행되고 있는 프로세스**
- 터미널 세션과 관련 없이 실행 → 사용자 입력을 기다리지 않음

![프로세스 그룹(PGID)]({{ img_path }}/2. 프로세스 그룹(PGID).png){: width='550'}


#### **예시**
{: data-toc-skip=''}

- `ps ajx | less` 명령어

![프로세스 그룹 예시]({{ img_path }}/2. 프로세스 그룹 예시.png){: width='650'}

- `ps ajx | cat | cat | cat | cat` 명령어

![프로세스 그룹 예시 2]({{ img_path }}/2. 프로세스 그룹 예시 2.png){: width='650'}



## 3. 데몬(Daemon)[^daemon]

**상주하는 프로세스**로, **백그라운드로 실행**된다.

- 터미널 입출력 필요 X → **단말 할당이 없음**
- 로그인 세션 종료해도 영향 받지 않도록 **독자적인 세션** 가짐
- 데몬 생성한 프로세스가 데몬 종료 여부를 신경쓰지 않도록, **`init`가 부모**가 됨

> 위의 조건을 전부 만족하지 않더라도, 그냥 상주하기만 하면 편의상 **Deamon** 혹은 **Service**라고 부른다.
{: .prompt-info}

[^daemon]: 윈도우에서는 Service라고 불리는 개념이다.



<br><br><br><br>

---
#### 각주
{: data-toc-skip=''}