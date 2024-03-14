---
title: "비트레이트 계산기(Bitrate Calculator) Ver.1.1.1"
date: 2024-03-14 10:11:00 +0900
categories: [Project, 토이프로젝트]
tags: [my_tag]
image:
  path: /assets/img/posts/project/toy-project/prev-img-bitrate-calc.png
  lqip: data:image/webp;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAJBAMAAADJBLEBAAAAAXNSR0IB2cksfwAAAAlwSFlzAAAWJQAAFiUBSVIk8AAAAC1QTFRF////+vr68vLy7u7u9/j49fX1/Pz8XmWK6+vs6OnrZGuN5ubm29vbVl6D4eHhNo4UmgAAAFRJREFUeJxjWLW2HASqGDomCYopKRuLMQi3hAhE71BMYBAycWAIChFjYAhuCWAQsQhkYEh0TVBKUQYyXAXYjKM7HBgYOozcGICgj0FYVTGtU+jMbgBcehOpv59uNwAAAABJRU5ErkJggg==
  alt: "비트레이트 계산기(Bitrate Calculator) V.1.1.1"
---


이 포스팅은 이전 네이버 블로그의 [해당 게시물](https://blog.naver.com/lja3333/221328077325)에서 마이그레이션되었다.

이번에 제가 직접 개발한 프로그램인 **비트레이트 계산기(Bitrate Calculator)**를 소개해볼까 합니다. 비트레이트가 뭘까요? 비트레이트는 단위 시간동안 처리되는 비트(Bit)를 말합니다.

비트레이트를 계산해서 뭐하냐구요? 사실 이 프로그램은 더 정확하게 말하면 '**영상 비트레이트**'를 계산하는 프로그램입니다. 영상 비트레이트는 동영상의 화질을 결정하는 중요한 요소 중 하나인데요, 영상 비트레이트가 높으면 높을수록 동일한 해상도와 동일한 프레임 수(FPS) 대비 선명하고 깨끗한 품질의 영상이 됩니다. 더 자세히 알고 싶으면 아래의 링크를 참고해 주시기 바랍니다.

<https://support.wecandeo.com/docs/videopack-guide-quality-determinant>
{: style="text-align: center"}

그런데 사실 동일한 해상도와 동일한 프레임 수에서, 영상 비트레이트가 계속 높아져도 어느 순간부터는 화질에 변화가 없게 됩니다. 즉 비트레이트가 계속 높아져 영상의 용량(Capacity)은 의미 없이 계속 커지지만 품질은 그대로라는 겁니다. 이러한 사실로 아래와 같은 결론을 얻을 수 있습니다.

<br>

"*영상의 용량이 가장 낮으면서 품질이 가장 높아지는 영상 비트레이트가 존재한다.*"
{: style="font-size: 120%; text-align: center"}

<br>

이 프로그램의 주요한 기능 중 하나가 바로 이러한 영상 비트레이트, 즉 **최대 영상 비트레이트**를 계산하는 것입니다. 영상의 **해상도**와 **프레임 수**를 입력하면 최대 영상 비트레이트를 바로 계산합니다.

더불어 영상의 **런타임 시간(재생 시간)**과 영상의 **오디오 비트레이트(Audio Bitrate)**를 입력하고 영상을 재 인코딩할 때 사용할 **코덱(Codec)**을 선택하면, 최대 영상 비트레이트로 영상을 재 인코딩했을 때 예상되는 영상의 용량 또한 계산해 줍니다.

제가 개발한 비트레이트 계산기(Bitrate Calculator)는 최대 영상 비트레이트를 계산할 뿐만 아니라, **출력 영상 크기 기준 영상 비트레이트**를 계산하는 기능도 존재합니다.

출력 영상 크기 기준 영상 비트레이트란 영상을 인코딩한 결과물인 **출력 영상의 용량**을 지정하면, 실제로 영상을 인코딩 했을 때 지정한 용량이 되도록 하는 영상 비트레이트를 의미합니다.

예를 들어 런타임 시간이 15분 30초, 해상도가 1920 x 1080, 초당 프레임이 30Fps, 오디오 비트레이트가 192Kbps인 영상을 H.264라는 코덱을 사용해 재 인코딩했을 때 나온 출력 영상의 용량이 800MB가 되게 하고 싶을 때, 이 영상의 최대 영상 비트레이트는 10,328Kbps이지만 출력 영상 크기 기준 영상 비트레이트는 6,855Kbps가 됩니다. 즉 해당 영상을 6,855Kbps의 영상 비트레이트로 인코딩하면 출력 영상의 용량이 800MB가 되는 것입니다.

마지막으로 이 프로그램의 또 다른 기능은 **해상도 변환**입니다. 1920 x 1080 해상도의 영상을 **비율을 유지**하면서 가로길이를 1280으로 줄이거나 또는 늘이고 싶을 때, 변환 예상 해상도와 그 때의 최대 영상 비트레이트를 바로 계산해서 보여줍니다. 

이외에 여러가지 편리한 부기능이 프로그램 속에 숨어있으니 잘 찾아보시고, 유용하게 사용하시길 바랍니다. 초창기 버전(콘솔)은 <https://cafe.naver.com/cafec/362331> , 이전 버전은 <https://cafe.naver.com/cafec/376158> 에서 확인해보실 수 있겠습니다.

**비트레이트 계산기(Bitrate Calculator) Ver.1.1.1 다운로드**는 아래에서 가능합니다. 감사합니다.

<br>

[프로그램 다운로드](/assets/file/posts/Project/toy-project/Bitrate%20Calculator%20v.1.1.1.exe)
{: style="font-size: 125%; font-weight: bold; text-align: center;"}