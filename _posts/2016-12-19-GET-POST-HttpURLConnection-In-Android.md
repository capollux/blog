---
layout:   post
title:    "GET / POST HttpURLConnection in Android"
date:     2016-12-19 17:26:11 +0900
---

When you send GET or POST request using `HttpURLConnection`, the request sometimes send POST request even if you set GET parameter like `HttpURLConnection.setRequestMethod("GET")`.

Even if you set the GET method, `HttpURLConnection.getOutputStream()` will change to POST method.

So even if you set the GET method using `conn.setRequestMethod("GET")`, the method will be changed to POST with the following codes.

```java
conn.setDoOutput(true);
OutputStream out = conn.getOutputStream();
```
