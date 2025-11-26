---
title: EdgeOne Pages Functions 中 Request Header Host 奇奇怪怪的问题
publishDate: 2025-11-22
description: EdgeOne Pages 中 Node Functions 无法使用相对路径导航
tags:
  - EdgeOne
  - EdgeOne Pages
language: 'Chinese'
---

如题，EdgeOne Pages 中 Edge 和 Node Functions 中 Request Header Host 并不一样，

- `Edge Functions`: 指向的是自己的域名
- `Node Functions`: 指向的是CDN的域名

因此，Node Functions 中使用相对路径导航大概率会出错，如 `fetch('/xxx')`

## 问题排查

在 Edge Functions 和 Node Functions 均部署以下代码，

``` js
export async function onRequest({request}) {
  // 获取request的详细信息
  const requestInfo = {
    url: request.url,
    method: request.method,
    headers: {},
    redirect: request.redirect,
    referrer: request.referrer,
    referrerPolicy: request.referrerPolicy,
  };

  // 获取所有的请求头
  for (const [key, value] of request.headers.entries()) {
    requestInfo.headers[key] = value;
  }

  // 将request信息转换为格式化的JSON字符串
  const requestInfoString = JSON.stringify(requestInfo, null, 2);

  return new Response(
    `hi from <edge/node>\nRequest Info:\n${requestInfoString}`,
    {
      headers: {
        'content-type': 'application/json; charset=UTF-8',
        'Access-Control-Allow-Origin': '*',
      },
    }
  );
}
```

### Deploy 环境

#### Edge Functions
在 [Edge Functions](https://test.eo.zinc233.top/edge) 下访问，得到的结果如下，其 url 和 Host 都是指向访问的域名的。

``` json
hi from edge
Request Info:
{
  "url": "https://test.eo.zinc233.top/edge",  // [\!code highlight]
  "method": "GET",
  "headers": {
    "sec-ch-ua": "\"Not_A Brand\";v=\"99\", \"Chromium\";v=\"142\"",
    "sec-ch-ua-mobile": "?0",
    "sec-ch-ua-platform": "\"Linux\"",
    "upgrade-insecure-requests": "1",
    "user-agent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36",
    "accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
    "sec-fetch-site": "none",
    "sec-fetch-mode": "navigate",
    "sec-fetch-user": "?1",
    "sec-fetch-dest": "document",
    "accept-language": "zh,en;q=0.9,zh-CN;q=0.8",
    "priority": "u=0, i",
    "Cookie": "...",
    "CDN-Loop": "TencentEdgeOne; loops=1",
    "eo-pages-language": "zh",
    "eo-pages-dataset": "mode=watermark;lang=zh",
    "Accept-Encoding": "gzip,deflate,br",
    "X-NWS-LOG-UUID": "3716365549362329524",
    "Host": "test.eo.zinc233.top",  // [\!code highlight]
    "Content-Length": "0"
  },
  "redirect": "follow",
  "referrer": "about:client",
  "referrerPolicy": ""
}
```


#### Node Functions
在 Node Functions 下访问，之前测试的结果 url 和 Host 似乎指向的是CDN的域名。

不知道为什么，后面再次部署这个函数生产环境一直超时崩溃，无法复现了😂

``` log
START RequestId: 087ad250-c763-11f0-b008-5254003756e9 

Unhandled Rejection: TypeError: crypto.createHash is not a function 
    at validateRequestHeaders (file:///var/user/index.mjs:571:6) 
    at Server.<anonymous> (file:///var/user/index.mjs:591:34) 
    at Server.emit (node:events:524:28) 
    at parserOnIncoming (node:_http_server:1141:12) 
    at HTTPParser.parserOnHeadersComplete (node:_http_common:118:17) 

ERROR RequestId: 087ad250-c763-11f0-b008-5254003756e9 Result: Invoking task timed out after 30 seconds
END RequestId: 087ad250-c763-11f0-b008-5254003756e9
Report RequestId: 087ad250-c763-11f0-b008-5254003756e9 Duration: 30000ms Memory: 1024MB MemUsage: 7.058594MB
```

### Dev 环境

奇怪的是，在本地，结论和上面是相反的。

#### Edge Functions
访问本地 `http://localhost:8088/edge`，这边的 url 和 Host 指向了一个临时的域名

``` json
hi from edge
Request Info:
{
  "url": "https://9278dd5c-ed88-4434-9bb9-b3912c10f972.edgeone.site/edge",  // [\!code highlight]
  "method": "GET",
  "headers": {
    "x-forwarded-host": "localhost:8088",
    "x-forwarded-proto": "http",
    "x-forwarded-port": "8088",
    "x-forwarded-for": "::1",
    "accept-language": "zh,en;q=0.9,zh-CN;q=0.8",
    "sec-fetch-dest": "document",
    "sec-fetch-user": "?1",
    "sec-fetch-mode": "navigate",
    "sec-fetch-site": "none",
    "accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
    "user-agent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36",
    "upgrade-insecure-requests": "1",
    "sec-ch-ua-platform": "\"Linux\"",
    "sec-ch-ua-mobile": "?0",
    "sec-ch-ua": "\"Not_A Brand\";v=\"99\", \"Chromium\";v=\"142\"",
    "connection": "close",
    "Accept-Encoding": "gzip,deflate,br",
    "X-NWS-LOG-UUID": "11649271619795003927",
    "Host": "9278dd5c-ed88-4434-9bb9-b3912c10f972.edgeone.site",  // [\!code highlight]
    "Content-Length": "0"
  },
  "redirect": "follow",
  "referrer": "about:client",
  "referrerPolicy": ""
}
```

#### Node Functions
访问本地 `http://localhost:8088/node`，这边 url 和 Host 会指向本地

``` json
hi from node
Request Info:
{
  "url": "http://localhost:9000/node",  // [\!code highlight]
  "method": "GET",
  "headers": {
    "accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
    "accept-encoding": "gzip, deflate, br, zstd",
    "accept-language": "zh,en;q=0.9,zh-CN;q=0.8",
    "connection": "close",
    "eo-connecting-geo": "[object Object]",
    "functions-request-id": "",
    "host": "localhost:9000",  // [\!code highlight]
    "sec-ch-ua": "\"Not_A Brand\";v=\"99\", \"Chromium\";v=\"142\"",
    "sec-ch-ua-mobile": "?0",
    "sec-ch-ua-platform": "\"Linux\"",
    "sec-fetch-dest": "document",
    "sec-fetch-mode": "navigate",
    "sec-fetch-site": "none",
    "sec-fetch-user": "?1",
    "upgrade-insecure-requests": "1",
    "user-agent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36",
    "x-forwarded-for": "::1",
    "x-forwarded-host": "localhost:8088",  // [\!code highlight]
    "x-forwarded-port": "8088",
    "x-forwarded-proto": "http"
  },
  "redirect": "follow",
  "referrer": "about:client",
  "referrerPolicy": ""
}
```