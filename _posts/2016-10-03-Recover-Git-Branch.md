---
layout:   post
title:    "Recover Git Branch"
date:     2016-10-03 14:26:18 +0900
---

```console
$ git reflog
```

With this command, you can see the git action history. Then find the commit what you want to recover and checkout new branch using **HEAD@{NUMBER}** next to commit checksum.

```console
$ git checkout -b [DELETED BRANCH] HEAD@{NUMBER}
```
