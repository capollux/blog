---
layout:   post
title:    "Recover Git Branch"
date:     2016-10-03 14:26:18 +0900
author:   junwoo
---

```console
$ git reflog
```

* git의 액션 히스토리가 나온다.
* 여기에서 나오는 HEAD@{숫자}를 이용하여 돌아가고자 하는 상태, 브랜치로 돌아간다.

```console
$ git checkout -b [DELETED BRANCH] HEAD@{NUMBER}
```

* 복구 완료.
