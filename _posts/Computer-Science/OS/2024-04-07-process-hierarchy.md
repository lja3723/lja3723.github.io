---
title: "[OS] 프로세스의 계층 구조"
date: 2024-04-07 21:15:00 +0900
categories: [Computer-Science, OS]
tags: ['OS', '프로세스']
---
{% assign img_path = "/assets/img/posts/OS/2024-04-07-process-hierarchy" %}



## 1. init: 모든 프로세스의 조상

- 유닉스의 모든 프로세스는 init 프로세스의 자식이 되어 트리 구조를 이룬다
  - Windows도 거의 비슷함

![init]({{ img_path }}/1. init.png){: width='400'}

![init 2]({{ img_path }}/1. init 2.png){: width='580'}

### 1.1. 예시: 리눅스의 프로세스 목록

- `pstree -p` 명령어 (부모-자식 프로세스 확인)

![예시]({{ img_path }}/1.1. 예시.png){: width='580'}
_리눅스에서의 프로세스 목록_

![예시 2]({{ img_path }}/1.1. 예시 2.png){: width='580'}
_윈도우에서의 프로세스 목록 (using ProcessExplorer.exe)_