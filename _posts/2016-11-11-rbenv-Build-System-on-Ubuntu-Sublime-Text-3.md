---
layout:   post
title:    "rbenv Build System On Ubuntu Sublime Text 3"
date:     2016-03-04 17:30:12 +0900
tags:     ubuntu atom
author:   junwoo
---

When you install `ruby` using `rbenv` on ubuntu, sometimes `build` doesn't work. Then you should make custom build system for `ruby`

* rbenv.sublime-build

```json
{
  "cmd": ["/home/[user]/.rbenv/shims/ruby", "$file"],
  "file_regex": "^(...*?):([0-9]*):?([0-9]*)",
  "selector": "source.ruby"
}
```
