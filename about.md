---
layout: lecture
title: "왜 우리가 이 수업을 가르칠까요?"
---

전통적인 컴퓨터 과학(CS) 교육을 듣는 동안 운영 체제, 프로그래밍 언어, 기계학습에 이르기까지 CS 내의 고급 주제들을 가르치는 많은 수업을 듣게 될 것입니다.
그러나 많은 기관에서는 거의 다루지 않고 학생들이 스스로 학습할 수 있도록 남겨진 필수적인 한 가지 주제가 있는데 바로 컴퓨터 생태계 활용 능력입니다.

지난 수 년간, 우리는 MIT에서 몇 가지 수업을 하는 것을 도왔고, 계속해서 많은 학생들이 사용하는 툴에 대한 제한된 지식을 가지고 있다는 것을 알았습니다.
컴퓨터는 수동 작업을 자동화하기 위해 만들어졌지만 학생들은 종종 손으로 반복 작업을 수행하거나 version control과 text editor와 같은 강력한 도구를 충분히 활용하지 않았습니다.
최상의 경우, 이는 비효율성과 시간 낭비를 초래하고, 최악의 경우 데이터 손실이나 특정 작업을 완료할 수 없는 등의 문제를 초래합니다.

이러한 주제들은 대학 커리큘럼의 일부로 가르치지 않습니다. 학생들은 이러한 도구들을 어떻게 사용하는지를 결코 알 수 없거나, 적어도 어떻게 효율적으로 사용하는지를 알지 못하며,
따라서 _간단해야 할_ 과제에 시간과 노력을 낭비합니다. 표준 CS 커리큘럼은 학생들의 삶을 훨씬 더 쉽게 만들 수 있는 컴퓨팅 생태계에 대한 중요한 주제들이 빠져 있습니다.

# 여러분의 CS 교육에서 누락된 학기

이 문제를 해결하기 위해 우리는 효과적인 컴퓨터 과학자와 프로그래머가 되기 위해 중요하다고 생각하는 모든 주제를 다루는 수업을 운영하고 있습니다.
이 수업은 실제적이고 실용적이며, 여러분이 접하게 될 다양한 상황에서 즉시 적용할 수 있는 툴과 기술에 대한 실습 소개를 제공합니다.
이 수업은 2020년 1월 MIT의 "독립 활동 기간" 동안 운영되고 있는데, 이는 단축 수업을 위한 한 달짜리 학기입니다.
수업 자체는 MIT 학생들만 수강할 있지만 녹화된 수업 영상과 함께 모든 수업 자료를 대중에게 제공할 예정입니다.

만약 이 내용이 여러분에게 해당하는 것처럼 들린다면, 이 수업이 무엇을 가르칠 것인지에 대한 구체적인 예들이 아래에 나와 있습니다:

## Command shell

별칭(aliases), 스크립트 및 빌드 시스템을 사용하여 일반 작업과 반복 작업을 자동화하는 방법에 대해 알아봅니다.
일반 문서에서 복사-붙여넣기 명령어를 더 이상 사용하지 않을 겁니다.
"이 15개의 명령을 차례로 실행"을 더 이상 하지 않을 겁니다.
"이것을 실행하는 방법을 잊었다"거나 "이 argument의 사용법을 잊어버렸다"는 말은 더 이상 하지 않을 겁니다.

예를 들어, 여러분의 shell history를 빠르게 검색하는 것은 엄청난 시간 절약이 될 수 있습니다.
아래와 같이 명령 `변환`을 위한 shell history 탐색과 관련된 몇 가지 트릭을 보여 줍니다.

<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="/static/media/demos/history.mp4" type="video/mp4">
</video>

## Version control

Version control을 _올바르게_ 사용하는 방법과 이를 활용하여 난관에서 여러분을 구하고, 다른 사람들과 협력하며, 문제가 있는 변경사항을 신속하게 찾아 격리하는 방법에 대해 알아봅니다.
`rm -rf; git clone`을 수행하는 일은 더 이상 없을 겁니다.
병합 충돌은 더 이상 없을 겁니다(글쎄요, 최소한 더 적은 수라고 해야겠군요).
주석이 들어간 큰 코드 블록은 더 이상 없을 겁니다.
어떤 요소가 여러분의 코드를 깨트리는지 더 이상 애태우지 마세요.
"안돼, 설마 작업 중인 코드를 삭제했나?!" 라고 더 이상 외칠 일이 없습니다.
Pull requests로 다른 사람의 프로젝트에 기여하는 방법까지 알려드리겠습니다!

아래와 같이 `git bisect`를 사용하여 어떤 커밋이 unit test를 깨트렸는지 찾은 후에 `git revert`로 수정하는 것입니다.

<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="/static/media/demos/git.mp4" type="video/mp4">
</video>

## Text editing

로컬 및 원격으로 command-line에서 파일을 효율적으로 편집하고 고급 편집기 기능을 활용하는 방법에 대해 알아봅니다.
파일을 앞뒤로 복사하는 일은 더 이상 하지 않게 될 겁니다.
반복적인 파일 편집이 더 이상 필요 없게 됩니다.

Vim 매크로는 가장 좋은 기능 중 하나인데, 아래와 같이 html 테이블을 중첩된 vim 매크로를 사용하여 csv 형식으로 빠르게 변환시킬 수 있습니다.

<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="/static/media/demos/vim.mp4" type="video/mp4">
</video>

## Remote machines

SSH 키 및 터미널 멀티플렉싱을 사용하여 원격 시스템과 작업할 때 정상적으로 유지하는 방법에 대해 알아봅니다.
두 개의 명령을 한 번에 실행하기 위해 많은 터미널을 열어둘 필요가 더 이상 없을 겁니다.
연결할 때마다 더 이상 암호를 입력하지 않아도 됩니다.
인터넷 연결이 끊겼거나 노트북을 재부팅해야 한다고 해서 모든 것을 잃을 일은 더 이상 없을 겁니다.

아래와 같이 세션을 원격 서버에서 활성 상태로 유지하기 위해 `tmux`를 사용하고 네트워크 로밍과 연결을 지원하기 위해 `mosh`를 사용합니다.

<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="/static/media/demos/ssh.mp4" type="video/mp4">
</video>

## Finding files

신속하게 원하는 파일을 찾는 방법에 대해 알아 봅니다.
원하는 코드가 있는 파일을 찾을 때까지 프로젝트의 파일을 클릭하는 일은 더 이상 하지 않아도 됩니다.

아래와 같이 `fd`가 있는 파일과 `rg`가 있는 코드 스니펫을 빠르게 찾아볼 수 있습니다.
또한 `fasd`를 사용하여 최근/자주 사용하는 파일/폴더도 `cd`와 `vim`을 실행할 수 있습니다.

<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="/static/media/demos/find.mp4" type="video/mp4">
</video>

## Data wrangling

Command-line에서 직접 데이터 및 파일을 빠르고 쉽게 수정, 보기, 구문 분석, 그래프 그리기 및 계산하는 방법에 대해 알아봅니다.
로그 파일에서 복사, 붙여넣기를 더 이상 하지 않아도 됩니다.
데이터에 대한 통계를 더 이상 수동으로 계산하지 않아도 됩니다.
이제는 스프레드시트에 그래프를 그리지 않아도 됩니다.

## Virtual machines

가상 머신을 사용하여 새로운 운영 체제를 시도하고, 관련 없는 프로젝트를 분리하며, 주 머신을 깨끗하고 깔끔하게 유지하는 방법에 대해 알아봅니다.
보안 랩을 수행하는 동안 실수로 컴퓨터를 더 이상 손상시키지 않아도 됩니다.
버전이 다른 수백만 개의 무작위로 설치된 패키지는 더 이상 필요 없습니다.

## Security

여러분의 모든 비밀을 세상에 즉시 드러내지 않고 인터넷에 접속하는 방법에 대해 알아봅니다.
너무 취약한 자신의 정보과 일치하는 비밀번호를 더 이상 사용하지 않을 겁니다.
보안되지 않은 개방형 WiFi 네트워크는 더 이상 없을 겁니다.
암호화되지 않은 메시지는 더 이상 없을 겁니다.

# Conclusion

위의 내용과 더불어 더 많은 내용이 12개의 수업에서 수업 내용과 함께 연습 문제를 포함되어 있습니다.
1월을 기다릴 수 없다면 지난해 IAP 때 운영했던 [Hacker Tools](https://hacker-tools.github.io/lectures/) 강의를 살펴봐도 좋습니다.
Hacker Tools는 이 수업의 선구자격로서, 같은 주제를 많이 다루고 있습니다.

우리는 가상으로든 직접으로든 1월에 당신을 만나기를 희망합니다!

Happy hacking,<br>
Anish, Jose, 그리고 Jon
