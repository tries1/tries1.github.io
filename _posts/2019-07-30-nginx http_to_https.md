---
layout: post
title:  "Nginx http to https"
date:   2019-07-30 17:00:00
author: Glenn
categories: Nginx
---

## Nginx HTTP 요청을 HTTPS로 Redirect하기 

**Nginx에서 아주 간단하게 적용이 가능하다**

nginx/nginx.conf 또는 nginx/conf.d/*.conf에서 설정
```nginx
server {
    listen 80;
    charset utf-8;
    server_name sample.com;

    # redirect http to https
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    charset utf-8;
    server_name sample.com;
}
```

*간단한 설정으로 모든 http(80 port) 요청을  https(443 port) 보내도록 설정할수있다.*
