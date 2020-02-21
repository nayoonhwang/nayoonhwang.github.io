---
layout: post
title:  "Git revert 실습해보기"
date:   2020-02-21 16:00:22 +0900
categories: git
---

충분한 테스트 없이 커밋한 코드를 원격저장소에 푸쉬했을때, 해당 커밋에 문제가 발생하는 경우 긴급하게 이전 상태로 되돌려야하는 경우가 발생할 수 있습니다. 로컬에서 작업한 커밋의 경우에는 git reset 을 통해 쉽게 되돌릴수 있는데요. 예를 들어 최근 두개의 커밋전 상태로 돌리려면&nbsp; git reset --hard HEAD~2 이런식으로 돌려버리면됩니다.

원격저장소에 반영된 코드의 경우에는 혼자 사용하는 레포지토리가 아니고, 동료들이 이미 해당 커밋을 pull하였을 수도 있기 때문에&nbsp;git revert를 사용하여야합니다.

git revert를 실습을 통해 알아보겠습니다!

* * *

1. 연습용 레포지토리를 하나 만들고 3개의 커밋을 push했습니다.

이중 첫번째 커밋빼고 두,세번째 커밋은 revert해보겠습니다.

2. git log를 출력해봅니다.

```
commit 6fa203b962fbcfa7971ece3068740a9ae1481818
Author: ***
Date:   Fri Feb 21 15:37:22 2020 +0900

    third commit

commit 56c6b311d907e4eb1585f2ca85c022cd1b0de421
Author: ***
Date:   Fri Feb 21 15:37:00 2020 +0900

    second commit

commit fa3d3504fef972519721c40fec8fbf709d183dc2
Author: ***
Date:   Fri Feb 21 14:47:48 2020 +0900

first commit
```

3. 최신 커밋인 71d부터 되돌려봅니다.

```bash
git revert 71d52f8c331cce4be7dbb183956f9877c783c70a
```

or

```bash
git revert HEAD
```

HEAD가 최신 커밋을 바라보고 있으니 둘은 똑같은 task를 수행하겠죠?

revert를 누르면 자동으로 vi editor가 실행됩니다. revert는 해당 커밋을 되돌리는 커밋(!)을 수행하기 때문인데요. 에디터에서 커밋 메세지를 수정할 수 있습니다. 이런 되돌리기 커밋을 작성하고 싶지않으시면 --no-commit 옵션을 주시면됩니다.

보시면 third commit을 되돌리는 Revert &quot;third commit&quot;이 생겼습니다.


만약 -n 옵션을 주고 같은 동작을 수행하면

```bash
git revert --no-commit HEAD
```

위와같이 커밋이 남지않습니다

```bash
❯ git status
On branch master
Your branch is up to date with 'origin/master'.

You are currently reverting commit 71d52f8.
  (all conflicts fixed: run "git revert --continue")
  (use "git revert --abort" to cancel the revert operation)

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   README.md
```

그리고 git status 출력시 다음과 같이 출력됩니다.

4. 만약에 한꺼번에 여러개의 커밋을 되돌리고 싶을땐 어떻게 해야할까요?

```bash
git revert --abort
```

를 통해 revert를 취소하고 다시 해보겠습니다

한꺼번에 여러개의 커밋을 취소하고 싶을 땐 <rev2>..<rev1> 이 문법을 쓰면됩니다.</rev1></rev2>

이 표현법은 rev1부터 rev2&nbsp;**전(중요!)**&nbsp;커밋까지를 포함하는데요. 우리 실습예제는 first commit(HEAD~2), second commit(HEAD~1), third commit(HEAD)&nbsp;중 뒤에 두개를&nbsp;삭제하고 싶은 것이므로, <rev2>에는 second commit 의 parent인 HEAD~2(first commit)을 넣고, rev1에는 HEAD를 넣어주었습니다!</rev2>

```bash
git revert --no-commit HEAD~2..HEAD
```

또는 부모를 뜻하는 ^를 넣어서 HEAD~1^를 넣어줘봅시다

```bash
git revert --no-commit HEAD~1^..HEAD
```

요론식으로 해도 똑같겠죠!^^

그리고 git status를 출력해보면 이렇습니다!

```bash
❯ git status
On branch master
Your branch is up to date with 'origin/master'.

You are currently reverting commit 56c6b31.
  (all conflicts fixed: run "git revert --continue")
  (use "git revert --abort" to cancel the revert operation)

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	deleted:    1.txt
	deleted:    2.txt
```

revert를 했다는 커밋을 남겨주고!

```bash
git commit -m "reverted second and third commit"
```

push!

```bash
git push origin master
```

짠! 초기상태로 돌아왔습니다.


커밋또한 예상한대로 잘 들어가있습니다.

* * *

여기까지 실습을 마무리했습니다! 최대한 git revert를 쓸일을 만들지 말아야겠지만, 불가피하게 발생했을 때 어떻게 해야할지 알게 되었습니당~!