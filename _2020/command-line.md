---
layout: lecture
title: "Command-line Environment"
date: 2019-01-21
ready: true
video:
  aspect: 56.25
  id: e8BO_dYxk5c
---

이 강의에서는 shell을 사용할 때 워크플로우를 개선할 수 있는 몇 가지 방법을 살펴볼 것입니다. 
그동안 우리는 shell을 사용해왔지만, 주로 다른 명령을 실행했습니다. 
이제 여러 프로세스를 추적하면서 동시에 실행하는 방법, 특정 프로세스를 중지하거나 일시 중지하는 방법, 프로세스가 백그라운드에서 실행되도록 하는 방법 등을 살펴보겠습니다.

우리는 또한 별칭을 정의하고 dotfile을 사용하여 설정함으로써 여러분의 shell과 다른 도구들을 향상시키는 다른 방법에 대해서도 배울 것입니다. 
이 두 가지 모두 긴 명령을 입력할 필요 없이 모든 시스템에서 동일한 구성을 사용하여 시간을 절약할 수 있습니다. 
우리는 SSH를 이용하여 어떻게 원격 서버에서 작업을 하는지에 대해 알아볼 것입니다.


# 작업 제어

예를 들어 명령이 완료되는 데 시간이 너무 오래 걸리는 경우(예컨데 `find`를 사용하여 매우 큰 디렉토리 구조를 검색)와 
같이 작업이 실행 중일 때 중단해야 하는 경우가 있습니다.
대부분 경우 `Ctrl-C`를 하게 되면 명령이 중단됩니다.
하지만 이것이 실제로 어떻게 작동하고, 왜 때때로 프로세스를 멈추는데 실패할까요?

## 프로세스 제거하기

당신의 shell은 _신호_ 라고 불리는 UNIX 통신 메커니즘을 사용하여 정보를 프로세스에 전달하고 있습니다. 
프로세스가 실행을 중지하는 신호를 수신하면 해당 신호를 처리하고, 신호가 전달한 정보를 기반으로 실행 흐름을 잠재적으로 변화시킵니다. 
따라서 신호는 _소프트웨어 중단_ 입니다.

`Ctrl-C`를 입력하면 shell이 `SIGINT` 신호를 프로세스에 전달합니다.

다음은 `SIGINT`를 캡처하여 무시하고 더 이상 멈추지 않는 파이썬 프로그램의 작은 예제 입니다. 
이 프로그램을 제거하려면 `Ctrl-\`를 입력하여 `SIGQ` 신호를 보냅니다.

```python
#!/usr/bin/env python
import signal, time

def handler(signum, time):
    print("\nI got a SIGINT, but I am not stopping")

signal.signal(signal.SIGINT, handler)
i = 0
while True:
    time.sleep(.1)
    print("\r{}".format(i), end="")
    i += 1
```

이 프로그램에 두 번의 `SIGINT`를 보내고, 그 다음에 `SIGQUIT`를 보내면 다음과 같은 일이 일어납니다. 
`^`는 termial에 입력할 때 `Ctrl`이 표시되는 방식입니다.

```bash
$ python sigint.py
24^C
I got a SIGINT, but I am not stopping
26^C
I got a SIGINT, but I am not stopping
30^\[1]    39913 quit       python sigint.py
```

`SIGINT`와 `SIGQ`는 보통 터미널 관련 요청과 관련되어 있는 반면, 보다 멋있게 종료하는 과정을 요청하는 일반적인 신호는 `SIGTERM` 신호입니다.
이 신호를 보내기 위해 [`kill`](https://www.man7.org/linux/man-pages/man1/kill.1.html) 명령을 `kill -TERM <PID>`라는 구문과 함께 사용할 수 있습니다.

## 프로세스의 일시 중지와 백그라운드

신호는 프로세스를 제거하는 것 이상의 다른 일들을 할 수 있습니다. 
예를 들어 `SIGSTOP`은 프로세스를 일시 중지시킵니다. 
터미널에서 `Ctrl-Z`를 입력하면 shell이 Terminal Stop(터미널 버전 `SIGSTOP`)을 줄인 `SIGTSTP` 신호를 보냅니다.

그 다음 각각 [`fg`](https://www.man7.org/linux/man-pages/man1/fg.1p.html) 또는 [`bg`](http://man7.org/linux/man-pages/man1/bg.1p.html),)를 사용하여 포어그라운드 또는 백그라운드에서 일시 중지된 작업을 계속할 수 있습니다.

[`jobs`](https://www.man7.org/linux/man-pages/man1/jobs.1p.html) 명령어에는 현재 터미널 세션과 관련된 미완료 작업 목록을 보여줍니다. 
목록의 pid([`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) 사용)를 이용하여 해당 작업을 참조할 수 있습니다.
보다 직관적으로 작업 번호(`jobs`로 보여지는) 다음에 해당하는 % 기호를 사용하는 프로세스도 참조할 수 있습니다. 
마지막 백그라운드 작업을 참조하려면 특수 매개 변수 `$!`를 사용하면 됩니다.

한 가지 더 알아야 할 것은 명령어의 `&` 접미사가 명령을 백그라운드에서 실행해 귀찮을 수 있는 쉘의 `STDOUT`를 여전히 사용하지만(이 경우 쉘 재조정을 사용) 프롬프트를 다시 제공한다는 점입니다.

이미 실행 중인 프로그램을 백그라운드 실행으로 변경하려면 `Ctrl-Z`에 이어 `bg`를 실행하면 됩니다.
백그라운드 프로세스는 여전히 터미널의 하위 프로세스로서 터미널을 닫으면 죽는다는 점에 유의하세요 (또 다른 신호인 `SIGHUP`를 전송). 
그런 일이 일어나는 것을 방지하기 위해 [`nohup`](https://www.man7.org/linux/man-pages/man1/nohup.1.html) (`SIGHUP`를 무시하는 wrapper) 
으로 프로그램을 실행하거나 프로세스가 이미 시작되었다면 `disown`을 사용합니다.
대안으로 다음 섹션에서 볼 수 있듯이 터미널 멀티플렉서를 사용할 수 있습니다.

아래는 이러한 개념들 중 몇 가지를 소개하기 위한 예제입니다.

```bash
$ sleep 1000
^Z
[1]  + 18653 suspended  sleep 1000

$ nohup sleep 2000 &
[2] 18745
appending output to nohup.out

$ jobs
[1]  + suspended  sleep 1000
[2]  - running    nohup sleep 2000

$ bg %1
[1]  - 18653 continued  sleep 1000

$ jobs
[1]  - running    sleep 1000
[2]  + running    nohup sleep 2000

$ kill -STOP %1
[1]  + 18653 suspended (signal)  sleep 1000

$ jobs
[1]  + suspended (signal)  sleep 1000
[2]  - running    nohup sleep 2000

$ kill -SIGHUP %1
[1]  + 18653 hangup     sleep 1000

$ jobs
[2]  + running    nohup sleep 2000

$ kill -SIGHUP %2

$ jobs
[2]  + running    nohup sleep 2000

$ kill %2
[2]  + 18745 terminated  nohup sleep 2000

$ jobs

```

특수신호는 프로세스를 캡처할 수 없고 항상 즉시 종료되기 때문에 `SIGKILL`입니다. 
하지만 고아가 된 하위 프로세스를 방치하는 등의 부작용을 낳을 수 있습니다.

이것과 다른 신호에 대해 자세히 알아보려면 [`여기`](https://en.wikipedia.org/wiki/Signal_(IPC)) 또는 [`man signal`](https://www.man7.org/linux/man-pages/man7/signal.7.html)을 입력 또는 `kill -t`를 입력하세요.


# 터미널 멀티플렉서

커맨드라인 인터페이스를 사용할 때 한 번에 두 개 이상의 작업을 실행하는 경우가 많을 겁니다.
예를 들어, 에디터와 프로그램을 나란히 실행하길 원한다고 합시다.
새로운 터미널 창을 열면 가능하겠지만 터미널 멀티플렉서를 사용하는 것이 더 다재다능한 해결방법 입니다.

[`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html)와 같은 터미널 멀티플렉서는 창과 탭을 사용하여 
터미널 창을 동시에 실행가능하게 분할할 수 있어 여러 shell 세션과 상호 작용이 가능합니다.
또한 터미널 멀티플렉서는 현재의 터미널 세션을 빠져나왔다가 나중에 다시 연결할 수 있습니다.
이는 `nohup`과 유사한 트릭을 사용할 필요가 없기 때문에 원격 컴퓨터로 작업할 때 워크플로우를 훨씬 더 좋게 만들 수 있습니다.

요즘 가장 인기 있는 터미널 멀티플렉서는 [`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html) 입니다. 
`tmux`는 환경 관리 설정이 뛰어나며 관련된 키 바인딩을 사용하여 여러 탭과 창을 만들고 빠르게 탐색할 수 있습니다.

`tmux`는 키 바인딩을 알고 싶어하는데 (1) `Ctrl+b` 키보드를 누르고, (2) `Ctrl+b` 키보드를 놓은 후, 
(3) `x`를 누르는 `<C-b> x` 형식을 갖고 있습니다. 
`tmux`는 다음과 같은 개체 계층을 가지고 있습니다:
- **Sessions** - 세션은 하나 이상의 윈도우가 있는 독립된 작업 공간
    + `tmux` 새로운 세션 시작
    + `tmux new -s NAME` 세션 이름과 함께 새로운 세션 시작
    + `tmux ls` 현재 세션 목록 나열
    + `tmux` 실행 중 `<C-b> d`를 입력  현재 세션에서 빠져나오기
    + `tmux a` 마지막 세션으로 들어가기. `-t` 옵션을 써서 특정 세션으로 접속 가능

- **Windows** - 에디터 또는 브라우저의 탭과 동일하게 동일한 세션에서 시각적으로 분리된 부분
    + `<C-b> c` 새로운 윈도우를 생성. `<C-d>`를 입력하면 윈도우 닫기
    + `<C-b> N` _N_ 번째 윈도우로 이동. 윈도우마다 번호가 있음을 유의
    + `<C-b> p` 이전 윈도우로 이동
    + `<C-b> n` 다음 윈도우로 이동
    + `<C-b> ,` 현재 윈도우 이름 바꾸기
    + `<C-b> w` 현재 윈도우 목록 나열

- **Panes** - vim 분할처럼 창은 동일한 시각 디스플레이에 여러 개의 shell을 가질 수 있음
    + `<C-b> "` 현재 창을 가로로 나누기
    + `<C-b> %` 현재 창을 세로로 나누기
    + `<C-b> <direction>` _direction_ 방향의 창으로 이동. 여기서 Direction은 화살표 키를 의미
    + `<C-b> z` 현재 창의 확대/축소 전환
    + `<C-b> [` scrollback 시작. `<space>`를 누르면 선택을 시작하고 `<enter>`를 누르면 선택 내용이 복사
    + `<C-b> <space>` 창 배열을 순환

추가적인 자료는 [`여기`](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)에 `tmux`에 대한 간단한 튜토리얼이 있고, 
[여기](http://linuxcommand.org/lc3_adv_termmux.php)에는 원래의 `screen` 명령을 다루는 보다 자세한 설명이 있습니다. 
대부분의 UNIX 시스템에 설치되므로 [`screen`](https://www.man7.org/linux/man-pages/man1/screen.1.html)을 숙지하는 것이 좋을 것입니다.

# 별칭

많고 장황한 옵션을 포함하는 긴 명령을 입력하는 것은 지루한 일입니다.
그런 이유로 대부분의 shell은 _별칭 지정_ 을 지원합니다.
별칭은 shell이 자동으로 대체할 수 있는 다른 명령을 위한 짧은 형식입니다.
예를 들어 bash의 별칭은 다음과 같은 구조를 가지고 있습니다.

```bash
alias alias_name="command_to_alias arg1 arg2"
```

[`alias`](https://www.man7.org/linux/man-pages/man1/alias.1p.html)는 하나의 argument를 취하는 shell 명령이기 때문에 등호 `=` 주위에 빈 공간이 없다는 점을 유의하세요.

별칭은 많은 편리한 기능을 가지고 있습니다:

```bash
# 일반적인 플래그의 짧은 명령어 만들기
alias ll="ls -lh"

# 평소에 자주 사용하는 명령어 저장하기
alias gs="git status"
alias gc="git commit"
alias v="vim"

# 오타 방지
alias sl=ls

# 더 좋은 기본값을 위해 기존 명령어 덮어쓰기
alias mv="mv -i"           # -i 지정 위치에 동일 파일이 있을 경우 덮어 쓸때 물어봄
alias mkdir="mkdir -p"     # -p 상위 디렉토리가 필요하다면 만들기
alias df="df -h"           # -h 사람이 읽을 수 있는 형식으로 출력

# 별칭을 시용하여 별칭을 구성
alias la="ls -A"
alias lla="la -l"

# 앞에 \가 붙은 상태로 별칭 실행하면 빌칭 실행 무시
\ls
# 또는 unalieas를 사용하여 별칭 사용불가
unalias la

# 별칭 정의를 얻기으려면 별칭을 호출
alias ll
# ll='ls -lh' 이 출력됨
```

별칭은 기본적으로 shell 세션을 유지하지 않는다는 점을 알아두세요.
별칭을 영구적으로 만들려면 다음 절에서 소개할 `.bashrc`나 `.zshrc`와 같은 shell 시작 파일에 포함시켜야 합니다.


# Dotfiles

Many programs are configured using plain-text files known as _dotfiles_
(because the file names begin with a `.`, e.g. `~/.vimrc`, so that they are
hidden in the directory listing `ls` by default).

Shells are one example of programs configured with such files. On startup, your shell will read many files to load its configuration.
Depending on the shell, whether you are starting a login and/or interactive the entire process can be quite complex.
[Here](https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html) is an excellent resource on the topic.

For `bash`, editing your `.bashrc` or `.bash_profile` will work in most systems.
Here you can include commands that you want to run on startup, like the alias we just described or modifications to your `PATH` environment variable.
In fact, many programs will ask you to include a line like `export PATH="$PATH:/path/to/program/bin"` in your shell configuration file so their binaries can be found.

Some other examples of tools that can be configured through dotfiles are:

- `bash` - `~/.bashrc`, `~/.bash_profile`
- `git` - `~/.gitconfig`
- `vim` - `~/.vimrc` and the `~/.vim` folder
- `ssh` - `~/.ssh/config`
- `tmux` - `~/.tmux.conf`

How should you organize your dotfiles? They should be in their own folder,
under version control, and **symlinked** into place using a script. This has
the benefits of:

- **Easy installation**: if you log in to a new machine, applying your
customizations will only take a minute.
- **Portability**: your tools will work the same way everywhere.
- **Synchronization**: you can update your dotfiles anywhere and keep them all
in sync.
- **Change tracking**: you're probably going to be maintaining your dotfiles
for your entire programming career, and version history is nice to have for
long-lived projects.

What should you put in your dotfiles?
You can learn about your tool's settings by reading online documentation or
[man pages](https://en.wikipedia.org/wiki/Man_page). Another great way is to
search the internet for blog posts about specific programs, where authors will
tell you about their preferred customizations. Yet another way to learn about
customizations is to look through other people's dotfiles: you can find tons of
[dotfiles
repositories](https://github.com/search?o=desc&q=dotfiles&s=stars&type=Repositories)
on Github --- see the most popular one
[here](https://github.com/mathiasbynens/dotfiles) (we advise you not to blindly
copy configurations though).
[Here](https://dotfiles.github.io/) is another good resource on the topic.

All of the class instructors have their dotfiles publicly accessible on GitHub: [Anish](https://github.com/anishathalye/dotfiles),
[Jon](https://github.com/jonhoo/configs),
[Jose](https://github.com/jjgo/dotfiles).


## Portability

A common pain with dotfiles is that the configurations might not work when working with several machines, e.g. if they have different operating systems or shells. Sometimes you also want some configuration to be applied only in a given machine.

There are some tricks for making this easier.
If the configuration file supports it, use the equivalent of if-statements to
apply machine specific customizations. For example, your shell could have something
like:

```bash
if [[ "$(uname)" == "Linux" ]]; then {do_something}; fi

# Check before using shell-specific features
if [[ "$SHELL" == "zsh" ]]; then {do_something}; fi

# You can also make it machine-specific
if [[ "$(hostname)" == "myServer" ]]; then {do_something}; fi
```

If the configuration file supports it, make use of includes. For example,
a `~/.gitconfig` can have a setting:

```
[include]
    path = ~/.gitconfig_local
```

And then on each machine, `~/.gitconfig_local` can contain machine-specific
settings. You could even track these in a separate repository for
machine-specific settings.

This idea is also useful if you want different programs to share some configurations. For instance, if you want both `bash` and `zsh` to share the same set of aliases you can write them under `.aliases` and have the following block in both:

```bash
# Test if ~/.aliases exists and source it
if [ -f ~/.aliases ]; then
    source ~/.aliases
fi
```

# Remote Machines

It has become more and more common for programmers to use remote servers in their everyday work. If you need to use remote servers in order to deploy backend software or you need a server with higher computational capabilities, you will end up using a Secure Shell (SSH). As with most tools covered, SSH is highly configurable so it is worth learning about it.

To `ssh` into a server you execute a command as follows

```bash
ssh foo@bar.mit.edu
```

Here we are trying to ssh as user `foo` in server `bar.mit.edu`.
The server can be specified with a URL (like `bar.mit.edu`) or an IP (something like `foobar@192.168.1.42`). Later we will see that if we modify ssh config file you can access just using something like `ssh bar`.

## Executing commands

An often overlooked feature of `ssh` is the ability to run commands directly.
`ssh foobar@server ls` will execute `ls` in the home folder of foobar.
It works with pipes, so `ssh foobar@server ls | grep PATTERN` will grep locally the remote output of `ls` and `ls | ssh foobar@server grep PATTERN` will grep remotely the local output of `ls`.


## SSH Keys

Key-based authentication exploits public-key cryptography to prove to the server that the client owns the secret private key without revealing the key. This way you do not need to reenter your password every time. Nevertheless, the private key (often `~/.ssh/id_rsa` and more recently `~/.ssh/id_ed25519`) is effectively your password, so treat it like so.

### Key generation

To generate a pair you can run [`ssh-keygen`](https://www.man7.org/linux/man-pages/man1/ssh-keygen.1.html).
```bash
ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/id_ed25519
```
You should choose a passphrase, to avoid someone who gets hold of your private key to access authorized servers. Use [`ssh-agent`](https://www.man7.org/linux/man-pages/man1/ssh-agent.1.html) or [`gpg-agent`](https://linux.die.net/man/1/gpg-agent) so you do not have to type your passphrase every time.

If you have ever configured pushing to GitHub using SSH keys, then you have probably done the steps outlined [here](https://help.github.com/articles/connecting-to-github-with-ssh/) and have a valid key pair already. To check if you have a passphrase and validate it you can run `ssh-keygen -y -f /path/to/key`.

### Key based authentication

`ssh` will look into `.ssh/authorized_keys` to determine which clients it should let in. To copy a public key over you can use:

```bash
cat .ssh/id_ed25519.pub | ssh foobar@remote 'cat >> ~/.ssh/authorized_keys'
```

A simpler solution can be achieved with `ssh-copy-id` where available:

```bash
ssh-copy-id -i .ssh/id_ed25519.pub foobar@remote
```

## Copying files over SSH

There are many ways to copy files over ssh:

- `ssh+tee`, the simplest is to use `ssh` command execution and STDIN input by doing `cat localfile | ssh remote_server tee serverfile`. Recall that [`tee`](https://www.man7.org/linux/man-pages/man1/tee.1.html) writes the output from STDIN into a file.
- [`scp`](https://www.man7.org/linux/man-pages/man1/scp.1.html) when copying large amounts of files/directories, the secure copy `scp` command is more convenient since it can easily recurse over paths. The syntax is `scp path/to/local_file remote_host:path/to/remote_file`
- [`rsync`](https://www.man7.org/linux/man-pages/man1/rsync.1.html) improves upon `scp` by detecting identical files in local and remote, and preventing copying them again. It also provides more fine grained control over symlinks, permissions and has extra features like the `--partial` flag that can resume from a previously interrupted copy. `rsync` has a similar syntax to `scp`.

## Port Forwarding

In many scenarios you will run into software that listens to specific ports in the machine. When this happens in your local machine you can type `localhost:PORT` or `127.0.0.1:PORT`, but what do you do with a remote server that does not have its ports directly available through the network/internet?.

This is called _port forwarding_ and it
comes in two flavors: Local Port Forwarding and Remote Port Forwarding (see the pictures for more details, credit of the pictures from [this StackOverflow post](https://unix.stackexchange.com/questions/115897/whats-ssh-port-forwarding-and-whats-the-difference-between-ssh-local-and-remot)).

**Local Port Forwarding**
![Local Port Forwarding](https://i.stack.imgur.com/a28N8.png  "Local Port Forwarding")

**Remote Port Forwarding**
![Remote Port Forwarding](https://i.stack.imgur.com/4iK3b.png  "Remote Port Forwarding")

The most common scenario is local port forwarding, where a service in the remote machine listens in a port and you want to link a port in your local machine to forward to the remote port. For example, if we execute  `jupyter notebook` in the remote server that listens to the port `8888`. Thus, to forward that to the local port `9999`, we would do `ssh -L 9999:localhost:8888 foobar@remote_server` and then navigate to `locahost:9999` in our local machine.


## SSH Configuration

We have covered many many arguments that we can pass. A tempting alternative is to create shell aliases that look like
```bash
alias my_server="ssh -i ~/.id_ed25519 --port 2222 -L 9999:localhost:8888 foobar@remote_server
```

However, there is a better alternative using `~/.ssh/config`.

```bash
Host vm
    User foobar
    HostName 172.16.174.141
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    LocalForward 9999 localhost:8888

# Configs can also take wildcards
Host *.mit.edu
    User foobaz
```

An additional advantage of using the `~/.ssh/config` file over aliases  is that other programs like `scp`, `rsync`, `mosh`, &c are able to read it as well and convert the settings into the corresponding flags.


Note that the `~/.ssh/config` file can be considered a dotfile, and in general it is fine for it to be included with the rest of your dotfiles. However, if you make it public, think about the information that you are potentially providing strangers on the internet: addresses of your servers, users, open ports, &c. This may facilitate some types of attacks so be thoughtful about sharing your SSH configuration.

Server side configuration is usually specified in `/etc/ssh/sshd_config`. Here you can make changes like disabling password authentication, changing ssh ports, enabling X11 forwarding, &c. You can specify config settings on a per user basis.

## Miscellaneous

A common pain when connecting to a remote server are disconnections due to shutting down/sleeping your computer or changing a network. Moreover if one has a connection with significant lag using ssh can become quite frustrating. [Mosh](https://mosh.org/), the mobile shell, improves upon ssh, allowing roaming connections, intermittent connectivity and providing intelligent local echo.

Sometimes it is convenient to mount a remote folder. [sshfs](https://github.com/libfuse/sshfs) can mount a folder on a remote server
locally, and then you can use a local editor.


# Shells & Frameworks

During shell tool and scripting we covered the `bash` shell because it is by far the most ubiquitous shell and most systems have it as the default option. Nevertheless, it is not the only option.

For example, the `zsh` shell is a superset of `bash` and provides many convenient features out of the box such as:

- Smarter globbing, `**`
- Inline globbing/wildcard expansion
- Spelling correction
- Better tab completion/selection
- Path expansion (`cd /u/lo/b` will expand as `/usr/local/bin`)

**Frameworks** can improve your shell as well. Some popular general frameworks are [prezto](https://github.com/sorin-ionescu/prezto) or [oh-my-zsh](https://ohmyz.sh/), and smaller ones that focus on specific features such as [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) or [zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search). Shells like [fish](https://fishshell.com/) include many of these user-friendly features by default. Some of these features include:

- Right prompt
- Command syntax highlighting
- History substring search
- manpage based flag completions
- Smarter autocompletion
- Prompt themes

One thing to note when using these frameworks is that they may slow down your shell, especially if the code they run is not properly optimized or it is too much code. You can always profile it and disable the features that you do not use often or value over speed.

# Terminal Emulators

Along with customizing your shell, it is worth spending some time figuring out your choice of **terminal emulator** and its settings. There are many many terminal emulators out there (here is a [comparison](https://anarc.at/blog/2018-04-12-terminal-emulators-1/)).

Since you might be spending hundreds to thousands of hours in your terminal it pays off to look into its settings. Some of the aspects that you may want to modify in your terminal include:

- Font choice
- Color Scheme
- Keyboard shortcuts
- Tab/Pane support
- Scrollback configuration
- Performance (some newer terminals like [Alacritty](https://github.com/jwilm/alacritty) or [kitty](https://sw.kovidgoyal.net/kitty/) offer GPU acceleration).

# Exercises

## Job control

1. From what we have seen, we can use some `ps aux | grep` commands to get our jobs' pids and then kill them, but there are better ways to do it. Start a `sleep 10000` job in a terminal, background it with `Ctrl-Z` and continue its execution with `bg`. Now use [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) to find its pid and [`pkill`](http://man7.org/linux/man-pages/man1/pgrep.1.html) to kill it without ever typing the pid itself. (Hint: use the `-af` flags).

1. Say you don't want to start a process until another completes, how you would go about it? In this exercise our limiting process will always be `sleep 60 &`.
One way to achieve this is to use the [`wait`](https://www.man7.org/linux/man-pages/man1/wait.1p.html) command. Try launching the sleep command and having an `ls` wait until the background process finishes.

    However, this strategy will fail if we start in a different bash session, since `wait` only works for child processes. One feature we did not discuss in the notes is that the `kill` command's exit status will be zero on success and nonzero otherwise. `kill -0` does not send a signal but will give a nonzero exit status if the process does not exist.
    Write a bash function called `pidwait` that takes a pid and waits until the given process completes. You should use `sleep` to avoid wasting CPU unnecessarily.

## Terminal multiplexer

1. Follow this `tmux` [tutorial](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/) and then learn how to do some basic customizations following [these steps](https://www.hamvocke.com/blog/a-guide-to-customizing-your-tmux-conf/).

## Aliases

1. Create an alias `dc` that resolves to `cd` for when you type it wrongly.

1.  Run `history | awk '{$1="";print substr($0,2)}' | sort | uniq -c | sort -n | tail -n 10`  to get your top 10 most used commands and consider writing shorter aliases for them. Note: this works for Bash; if you're using ZSH, use `history 1` instead of just `history`.


## Dotfiles

Let's get you up to speed with dotfiles.
1. Create a folder for your dotfiles and set up version
   control.
1. Add a configuration for at least one program, e.g. your shell, with some
   customization (to start off, it can be something as simple as customizing your shell prompt by setting `$PS1`).
1. Set up a method to install your dotfiles quickly (and without manual effort) on a new machine. This can be as simple as a shell script that calls `ln -s` for each file, or you could use a [specialized
   utility](https://dotfiles.github.io/utilities/).
1. Test your installation script on a fresh virtual machine.
1. Migrate all of your current tool configurations to your dotfiles repository.
1. Publish your dotfiles on GitHub.

## Remote Machines

Install a Linux virtual machine (or use an already existing one) for this exercise. If you are not familiar with virtual machines check out [this](https://hibbard.eu/install-ubuntu-virtual-box/) tutorial for installing one.

1. Go to `~/.ssh/` and check if you have a pair of SSH keys there. If not, generate them with `ssh-keygen -o -a 100 -t ed25519`. It is recommended that you use a password and use `ssh-agent` , more info [here](https://www.ssh.com/ssh/agent).
1. Edit `.ssh/config` to have an entry as follows

```bash
Host vm
    User username_goes_here
    HostName ip_goes_here
    IdentityFile ~/.ssh/id_ed25519
    LocalForward 9999 localhost:8888
```
1. Use `ssh-copy-id vm` to copy your ssh key to the server.
1. Start a webserver in your VM by executing `python -m http.server 8888`. Access the VM webserver by navigating to `http://localhost:9999` in your machine.
1. Edit your SSH server config by doing  `sudo vim /etc/ssh/sshd_config` and disable password authentication by editing the value of `PasswordAuthentication`. Disable root login by editing the value of `PermitRootLogin`. Restart the `ssh` service with `sudo service sshd restart`. Try sshing in again.
1. (Challenge) Install [`mosh`](https://mosh.org/) in the VM and establish a connection. Then disconnect the network adapter of the server/VM. Can mosh properly recover from it?
1. (Challenge) Look into what the `-N` and `-f` flags do in `ssh` and figure out what a command to achieve background port forwarding.
