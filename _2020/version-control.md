---
layout: lecture
title: "Version Control (Git)"
date: 2019-01-22
ready: true
video:
  aspect: 56.25
  id: 2sjqTHE0zok
---

버전관리 시스템(Version control systems, VCSs)은 소스코드(또는 다른 파일들 혹은 폴더들)의 변화를 추적하기 위해 사용된다. VCSs는 이름에서 알 수 있듯이 변경된 사항들의 관리를 도와주며, 더 나아가 협력을 용이하게 한다. 버전관리 시스템은 폴더와 폴더 내용물의 변화를 일련의 스냅샷으로 추적하는데, 각각의 스냅샷은 최상위 디렉토리 내 파일과 폴더의 전반적인 상태를 `요약된 형태로 보관(encapsulates)`한다. 또한 버전관리 시스템은 각 스냅샷의 생성자와 메시지 등 메타데이터를 관리한다.

버전관리는 어째서 유용할까? 혼자서 작업할 땐 프로젝트의 예전 스냅샷을 보거나, 어떤 변경점이 왜 생성됐는지 로그를 유지할 수 있게 해주고, 동시에 여러 개발 브랜치에서 작업할 수 있게 해주는 등 많은 이유가 있다. 또한 다른 사람들과 일할 땐 다른 사람들이 무엇을 변경했는지 보거나, 함께 개발하며 생기는 충돌을 해결하기 위한 매우 귀중한 도구이다.

현대의 버전관리 시스템은 다음과 같은 질문들을 쉽게 해결해준다.

- 누가 이 모듈을 작성했는가?
- 특정 파일의 특정 라인은 언제 수정됐는가? 누구에 의해서? 왜 수정되었는가?
- 지난 1000개의 수정들 중 특정 유닛 테스트가 고장난 건 언제, 누구에 의한 건가?

여러 버전관리 시스템이 존재하지만, 사실상 표준은 **Git**이다.
이 만화는[XKCD comic](https://xkcd.com/1597/) Git의 평판을 잘 보여준다.

![xkcd 1597](https://imgs.xkcd.com/comics/git.png)

Git의 인터페이스는 leaky abstraction 이기 때문에, top-down 방식으로 배우면 (인터페이스 / 커맨드-라인 인터페이스로 시작하는 것) 크게 혼란스러울 수 있다. 몇 가지 유용한 명령어를 기억하여 이를 마법 주문이라 생각할 수도 있고, 뭔가가 잘못될 시 언제든지 위 만화에서 얘기하는 것처럼 프로젝트를 지우고 새로운 복사본을 받을 수 있다.
(역자 주 - leaky abstraction란 추상화는 사용자의 입장에서 사용하기는 쉽지만, 그 기저의 원리를 알지 못하기에 어떤 문제가 발생하면 이를 이해하거나 해결할 수 없음을 의미합니다.)

Git은 명백히 못난 인터페이스를 지니고 있지만, 그 아래 깔려있는 디자인과 아이디어는 아름답다. 못난 인터페이스는 _기억_ 해야 하지만, 아름다운 디자인은 _이해_ 할 수 있다. 이런 이유로, 우리는 bottom-up 방식으로, Git의 데이터 모델부터 시작하여 나중에 커맨드-라인 인터페이스를 배우자. 데이터 모델을 이해하고 나면, 명령어가 실제로 어떻게 데이터모델을 조작하는지 더 쉽게 이해할 수 있다.

# Git의 데이터 

버전관리에 대해서는 여러 접근법이 있다. Git은 잘 고안된 데이터 모델을 가지고 있으며, 이는 버전관리의 멋진 기능들인 히스토리를 관리, 브랜치 기능의 지원 그리고 협업을 가능케 한다.

## 스냅샷

Git은 어떤 최상위 디렉토리 내에서 파일과 폴더들의 집합의 히스토리를 스냅샷의 연속이라는 형태로 구축한다. Git에서 하나의 파일은 "blob"으로 불리며, 하나의 바이트 묶음(a bunch of bytes)이다. 디렉토리는 "tree"라고 불리고, 다른 blob이나 tree에 이름이 매핑된다(디렉토리가 다른 디렉토리를 포함할 수 있도록). 스냅샷은 최상위 tree이며 지속적으로 추적된다. 예를 들어 다음과 같은 트리가 있다고 하자.

A directory is called a "tree", and it maps names to blobs or trees (so directories can contain other directories).

```
<root> (tree)
|
+- foo (tree)
|  |
|  + bar.txt (blob, contents = "hello world")
|
+- baz.txt (blob, contents = "git is wonderful")
```

최상위 트리는 두 구성 요소로 이루어져 있다. foo라는 tree와 baz.txt라는 blob이다. (foo는 bar.txt라는 blob 요소를 포함한다)

## 모델링 히스토리: 스냅샷 연결 Modeling history: relating snapshots

버전관리 시스템은 어떻게 스냅샷을 연결해야 할까? 한 가지 간단한 모델은 직선 형태의 히스토리를 갖는 것이다. 시간순의 스냅샷 리스트가 히스토리가 되는 것이다. 많은 이유로, Git은 이런 간단한 모델을 사용하지 않는다.

Git에서, 히스토리란 스냅샷들의 유향 비순환 그래프(directed acyclic graph: DAG)이다. 어쩌면 이게 수준 높은 수학 용어로 들릴 수 있는데, 겁먹을 필요 없다. 이게 뜻하는 바는, 각각의 스냅샷은 시간상 이전에 발생한 "부모(parents)" 스냅샷의 집합을 바라본다는 것이다. 예를 들어, 두 개의 개발 브랜치를 합치려면(merging) 하나의 스냅샷이 여러 부모로부터 만들어질 수 있어야 하기 때문에, 하나의 부모(위에서 언급한 직선의 히스토리에서 말하는)가 아닌 부모의 집합이 되는 것이다.

Git에서 이런 스냅샷들을 "커밋(commit)"이라고 부른다. 커밋 히스토리를 시각화하면 다음처럼 보일 수 있다.

```
o <-- o <-- o <-- o
            ^  
             \
              --- o <-- o
```

위의 아스키 그림에서, 'o'는 각각의 커밋(스냅샷)이다. 화살표는 각 커밋의 부모를 향한다(화살표가 향하는 방향이 이후가 아닌 이전을 뜻한다). 세 번째 커밋 이후에, 히스토리는 두 개의 분리된 브랜치가 된다. 이는 두개의 분리된 기능이 동시에 그리고 독립적으로 개발되고 있음을 뜻한다. 이후에 이 브랜치들이 병합(merged)되고 하나의 새로운 스냅샷이 되어, 두 기능을 통합할 수 있다. 새로운 히스토리가 만들어지는 건 다음에 나올 그림과 같으며, 새로 만들어진 병합 커밋은 굵은 'o'로 확인할 수 있다.

<pre>
o <-- o <-- o <-- o <---- <strong>o</strong>
            ^            /
             \          v
              --- o <-- o
</pre>

Git에서 커밋은 불변이다(immutable). 실수가 고쳐질 수 없다는 뜻은 아니지만, 커밋 히스토리를 수정한다는 건 실제로는 완전히 새로운 커밋들을 만드는 것이고, 새로운 커밋들로 참조가 수정된다는 것이다.

## 의사코드(pseudocode)로 보는 데이터 모델

의사코드로 작성된 Git의 데이터 모델을 보면 도움이 될 것이다. 

```
// 파일은 바이트 묶음이다
type blob = array<byte>

// 디렉토리는 파일과 디렉토리를 포함한다
type tree = map<string, tree | blob>

// 커밋은 부모, 메타데이터, 최상위 트리로 이루어진다
type commit = struct {
    parent: array<commit>
    author: string
    message: string
    snapshot: tree
}
```

이는 깔끔하고 단순한 히스토리 모델이다.

## 객체(objects)와 주소 지정

하나의 "객체(object)"는 blob이나 tree 혹은 commit이다:

```
type object = blob | tree | commit
```

Git의 데이터 스토어에서, 모든 객체는 그 객체의 [SHA-1 hash](https://en.wikipedia.org/wiki/SHA-1)를 가지고 주소가 지정된다.

```
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

blob과 tree 그리고 커밋은 모두 오브젝트라는 공통점을 가지고 있다. 이 세 오브젝트가 다른 오브젝트를 참조할 때, 실제로 디스크상의 그 데이터를 _포함_하는 게 아니라, 해쉬값을 참조한다.

[위에](#snapshots)(visualized using `git cat-file -p 698281bc680d1995c5f4caaf3359721a5a58d48d`) 있는 directory 구조의 tree를 예로 들면 다음과 같다.

```
100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo
```

tree는 `baz.txt` (a blobk) 과 'foo' (a tree)의 포인터를 갖는다. 만약 다음 명령어 `git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85`를 사용하여 baz.txt의 내용을 살펴보면 다음과 같다.

```
git is wonderful
```

## 참조

이제, 모든 스냅샷은 그 스냅샷의 SHA-1 hash로 식별될 수 있다. 이는 불편하다. 사람은 16진수 문자열 40자를 외우는 데 능하지 않기 때문이다.

이 문제에 대한 Git의 해결법은, SHA-1 hashes에 사람이 읽을 수 있는 이름을 짓고 "참조"라고 부르는 것이다. 참조는 커밋을 가리키는 포인터이다. 불변인 오브젝트와 달리 참조는 변할 수 있다 (새로운 커밋을 가리키도록 수정될 수 있다). 예를 들어 `master`라는 참조는 보통 메인 개발 브랜치의 가장 최신 커밋을 가리킨다.

```
references = map<string, string>

def update_reference(name, id):
    references[name] = id

def read_reference(name):
    return references[name]

def load_reference(name_or_id):
    if name_or_id in references:
        return load(references[name_or_id])
    else:
        return load(name_or_id)
```

이를 사용하여 Git은 히스토리 상의 특정 스냅샷을 가리키기 위해 하나의 긴 16진수 문자열이 아닌 사람이 읽을 수 있는 이름("master"와 같은)을 사용할 수 있다.


One detail is that we often want a notion of "where we currently are" in the
history, so that when we take a new snapshot, we know what it is relative to
(how we set the `parents` field of the commit). In Git, that "where we
currently are" is a special reference called "HEAD".

## Repositories (저장소)

마침내, 우리는 (대충) Git _리포지토리_가 무엇인지 정의할 수 있다. 'objects'와 'references'인 데이터다.

디스크 상에서, Git이 저장하는 건 모두 objects나 references이고, 이게 Git의 데이터 모델에 대한 전부이다. 모든 `git` 명령어는 objects를 추가하거나 references를 추가/수정하여 커밋 DAG를 조작하는 것이다.

어떤 명령어든 실행할 때마다, 해당 명령어가 그래프 데이터 구조 상에서는 어떤 조작을 할지 생각해 봐라. 반대로, 네가 커밋 DAG에 특정한 조작을 가하려고 한다면(예를 들어, "커밋되지 않은 변경 사항을 무시하고, 'master' ref가 커밋 `5d83f9e`를 가리키게 해라"), 분명 이를 위한 명령어가 있을 것이다. (이 경우에는 `git checkout master; git reset --hard 5d83f9e`이다).

# Staging area

스테이징 영역은 데이터 모델과는 다른 컨셉이지만, 커밋을 생성하는 인터페이스의 일부이다

위에 기술한대로 스냅샷을 구현한다고 가정하자. 한 가지 방법은, working directory의 _현재 상태_에 기반하여 새로운 스냅샷을 생성하는 "create snapshot" 커맨드를 만드는 것이다.  어떤 버전관리 도구는 그렇게 작동하지만, Git은 아니다. 우리는 깨끗한 스냅샷을 원하고, 현재의 상태를 가지고 스냅샷을 만드는 건 이상적이지 않다. 예를 들어서, 네가 두 가지 분리된 기능을 개발했다고 하자. 그러면 각각의 기능을 분리하여 두 개의 분리된 커밋을 만들고 싶을 것이다. 아니면 또 다른 예로, 디버그를 위한 로그가 코듭 전반에 더해졌고, 버그를 수정한 부분도 함께 있다고 하자. 그럼 버그를 수정한 부분만 커밋하고, 디버깅용 로그들은 다 없애고 싶을 것이다.

Git은 "스테이징 영역"이라고 불리는 메카니즘을 통해 다음 스냅샷에 어떤 변경사항 포함되어야 할지 정할 수 있게 하여 이러한 시나리오가 가능케 한다.

# Git command-line interface

중복되는 정보를 피하기 위해서, 아래의 명령어를 상세하게 설명하지 않겠다. 더 이상의 정보를 위해서는 [Pro Git](https://git-scm.com/book/en/v2)를 매우 추천하며 혹은 강의 동영상을 봐라.

## Basics

{% comment %}

`git init` 명령어는 새로운 깃 저장소를 초기화하고, `.git` 디렉토리 안에 저장소 메타데이터를 저장한다:

```console
$ mkdir myproject
$ cd myproject
$ git init
Initialized empty Git repository in /home/missing-semester/myproject/.git/
$ git status
On branch master

No commits yet

nothing to commit (create/copy files and use "git add" to track)
```

위 출력을 어떻게 해석할까? "No commits yet(커밋이 없다)"는 기본적으로 우리 버전 히스토리가 비어있다는 뜻이다. 이를 고쳐보자.

```console
$ echo "hello, git" > hello.txt
$ git add hello.txt
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

        new file:   hello.txt

$ git commit -m 'Initial commit'
[master (root-commit) 4515d17] Initial commit
 1 file changed, 1 insertion(+)
 create mode 100644 hello.txt
```

With this, we've `git add`ed a file to the staging area, and then `git
commit`ed that change, adding a simple commit message "Initial commit". If we
didn't specify a `-m` option, Git would open our text editor to allow us type a
commit message.

Now that we have a non-empty version history, we can visualize the history.
Visualizing the history as a DAG can be especially helpful in understanding the
current status of the repo and connecting it with your understanding of the Git
data model.

The `git log` command visualizes history. By default, it shows a flattened
version, which hides the graph structure. If you use a command like `git log
--all --graph --decorate`, it will show you the full version history of the
repository, visualized in graph form.

```console
$ git log --all --graph --decorate
* commit 4515d17a167bdef0a91ee7d50d75b12c9c2652aa (HEAD -> master)
  Author: Missing Semester <missing-semester@mit.edu>
  Date:   Tue Jan 21 22:18:36 2020 -0500

      Initial commit
```

This doesn't look all that graph-like, because it only contains a single node.
Let's make some more changes, author a new commit, and visualize the history
once more.

```console
$ echo "another line" >> hello.txt
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   hello.txt

no changes added to commit (use "git add" and/or "git commit -a")
$ git add hello.txt
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   hello.txt

$ git commit -m 'Add a line'
[master 35f60a8] Add a line
 1 file changed, 1 insertion(+)
```

Now, if we visualize the history again, we'll see some of the graph structure:

```
* commit 35f60a825be0106036dd2fbc7657598eb7b04c67 (HEAD -> master)
| Author: Missing Semester <missing-semester@mit.edu>
| Date:   Tue Jan 21 22:26:20 2020 -0500
|
|     Add a line
|
* commit 4515d17a167bdef0a91ee7d50d75b12c9c2652aa
  Author: Anish Athalye <me@anishathalye.com>
  Date:   Tue Jan 21 22:18:36 2020 -0500

      Initial commit
```

Also, note that it shows the current HEAD, along with the current branch
(master).

We can look at old versions using the `git checkout` command.

```console
$ git checkout 4515d17  # previous commit hash; yours will be different
Note: checking out '4515d17'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at 4515d17 Initial commit
$ cat hello.txt
hello, git
$ git checkout master
Previous HEAD position was 4515d17 Initial commit
Switched to branch 'master'
$ cat hello.txt
hello, git
another line
```

Git can show you how files have evolved (differences, or diffs) using the `git
diff` command:

```console
$ git diff 4515d17 hello.txt
diff --git c/hello.txt w/hello.txt
index 94bab17..f0013b2 100644
--- c/hello.txt
+++ w/hello.txt
@@ -1 +1,2 @@
 hello, git
 +another line
```

{% endcomment %}

- `git help <command>`: get help for a git command
- `git init`: creates a new git repo, with data stored in the `.git` directory
- `git status`: tells you what's going on
- `git add <filename>`: adds files to staging area
- `git commit`: creates a new commit
    - Write [good commit messages](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)!
    - Even more reasons to write [good commit messages](https://chris.beams.io/posts/git-commit/)!
- `git log`: shows a flattened log of history
- `git log --all --graph --decorate`: visualizes history as a DAG
- `git diff <filename>`: show differences since the last commit
- `git diff <revision> <filename>`: shows differences in a file between snapshots
- `git checkout <revision>`: updates HEAD and current branch

## Branching and merging

{% comment %}

Branching allows you to "fork" version history. It can be helpful for working
on independent features or bug fixes in parallel. The `git branch` command can
be used to create new branches; `git checkout -b <branch name>` creates and
branch and checks it out.

Merging is the opposite of branching: it allows you to combine forked version
histories, e.g. merging a feature branch back into master. The `git merge`
command is used for merging.

{% endcomment %}

- `git branch`: shows branches
- `git branch <name>`: creates a branch
- `git checkout -b <name>`: creates a branch and switches to it
    - same as `git branch <name>; git checkout <name>`
- `git merge <revision>`: merges into current branch
- `git mergetool`: use a fancy tool to help resolve merge conflicts
- `git rebase`: rebase set of patches onto a new base

## Remotes

- `git remote`: list remotes
- `git remote add <name> <url>`: add a remote
- `git push <remote> <local branch>:<remote branch>`: send objects to remote, and update remote reference
- `git branch --set-upstream-to=<remote>/<remote branch>`: set up correspondence between local and remote branch
- `git fetch`: retrieve objects/references from a remote
- `git pull`: same as `git fetch; git merge`
- `git clone`: download repository from remote

## Undo

- `git commit --amend`: edit a commit's contents/message
- `git reset HEAD <file>`: unstage a file
- `git checkout -- <file>`: discard changes

# Advanced Git

- `git config`: Git is [highly customizable](https://git-scm.com/docs/git-config)
- `git clone --depth=1`: shallow clone, without entire version history
- `git add -p`: interactive staging
- `git rebase -i`: interactive rebasing
- `git blame`: show who last edited which line
- `git stash`: temporarily remove modifications to working directory
- `git bisect`: binary search history (e.g. for regressions)
- `.gitignore`: [specify](https://git-scm.com/docs/gitignore) intentionally untracked files to ignore

# Miscellaneous

- **GUIs**: there are many [GUI clients](https://git-scm.com/downloads/guis)
out there for Git. We personally don't use them and use the command-line
interface instead.
- **Shell integration**: it's super handy to have a Git status as part of your
shell prompt ([zsh](https://github.com/olivierverdier/zsh-git-prompt),
[bash](https://github.com/magicmonty/bash-git-prompt)). Often included in
frameworks like [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh).
- **Editor integration**: similarly to the above, handy integrations with many
features. [fugitive.vim](https://github.com/tpope/vim-fugitive) is the standard
one for Vim.
- **Workflows**: we taught you the data model, plus some basic commands; we
didn't tell you what practices to follow when working on big projects (and
there are [many](https://nvie.com/posts/a-successful-git-branching-model/)
[different](https://www.endoflineblog.com/gitflow-considered-harmful)
[approaches](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)).
- **GitHub**: Git is not GitHub. GitHub has a specific way of contributing code
to other projects, called [pull
requests](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests).
- **Other Git providers**: GitHub is not special: there are many Git repository
hosts, like [GitLab](https://about.gitlab.com/) and
[BitBucket](https://bitbucket.org/).

# Resources

- [Pro Git](https://git-scm.com/book/en/v2) is **highly recommended reading**.
Going through Chapters 1--5 should teach you most of what you need to use Git
proficiently, now that you understand the data model. The later chapters have
some interesting, advanced material.
- [Oh Shit, Git!?!](https://ohshitgit.com/) is a short guide on how to recover
from some common Git mistakes.
- [Git for Computer
Scientists](https://eagain.net/articles/git-for-computer-scientists/) is a
short explanation of Git's data model, with less pseudocode and more fancy
diagrams than these lecture notes.
- [Git from the Bottom Up](https://jwiegley.github.io/git-from-the-bottom-up/)
is a detailed explanation of Git's implementation details beyond just the data
model, for the curious.
- [How to explain git in simple
words](https://smusamashah.github.io/blog/2017/10/14/explain-git-in-simple-words)
- [Learn Git Branching](https://learngitbranching.js.org/) is a browser-based
game that teaches you Git.

# Exercises

1. If you don't have any past experience with Git, either try reading the first
   couple chapters of [Pro Git](https://git-scm.com/book/en/v2) or go through a
   tutorial like [Learn Git Branching](https://learngitbranching.js.org/). As
   you're working through it, relate Git commands to the data model.
1. Clone the [repository for the
class website](https://github.com/missing-semester/missing-semester).
    1. Explore the version history by visualizing it as a graph.
    1. Who was the last person to modify `README.md`? (Hint: use `git log` with
       an argument)
    1. What was the commit message associated with the last modification to the
       `collections:` line of `_config.yml`? (Hint: use `git blame` and `git
       show`)
1. One common mistake when learning Git is to commit large files that should
   not be managed by Git or adding sensitive information. Try adding a file to
   a repository, making some commits and then deleting that file from history
   (you may want to look at
   [this](https://help.github.com/articles/removing-sensitive-data-from-a-repository/)).
1. Clone some repository from GitHub, and modify one of its existing files.
   What happens when you do `git stash`? What do you see when running `git log
   --all --oneline`? Run `git stash pop` to undo what you did with `git stash`.
   In what scenario might this be useful?
1. Like many command line tools, Git provides a configuration file (or dotfile)
   called `~/.gitconfig`. Create an alias in `~/.gitconfig` so that when you
   run `git graph`, you get the output of `git log --all --graph --decorate
   --oneline`.
1. You can define global ignore patterns in `~/.gitignore_global` after running
   `git config --global core.excludesfile ~/.gitignore_global`. Do this, and
   set up your global gitignore file to ignore OS-specific or editor-specific
   temporary files, like `.DS_Store`.
1. Clone the [repository for the class
   website](https://github.com/missing-semester/missing-semester), find a typo
   or some other improvement you can make, and submit a pull request on GitHub.
