---
layout:   post
title:    "GET / POST HttpURLConnection in Android"
date:     2016-12-19 17:26:11 +0900
author:   junwoo
---

`HttpURLConnection` 을 이용하여 GET 또는 POST 방식으로 특정 URL을 호출하는 경우,
`HttpURLConnection.setRequestMethod("GET")` 이라고 명시적으로 GET을 지정하여도 특정한 경우에는 내부적으로 POST로 처리함.

`HttpURLConnection.getOutputStream()` 메소드 내부에서 현재 지정된 method 가 GET인 경우, 강제로 POST로 변경됨.

따라서 `conn.setRequestMethod("GET")` 로 GET으로 설정했다고 하더라도, 아래의 라인이 추가된다면 내부적으로 POST 방식으로 변경됨.

```java
conn.setDoOutput(true);
OutputStream out = conn.getOutputStream();
```
