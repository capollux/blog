---
layout:   post
title:    "rbenv Build System On Ubuntu Sublime Text 3"
date:     2016-11-11 22:11:22 +0900
tags:     ubuntu rbenv ruby sublimetext
author:   junwoo
comments: true
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
