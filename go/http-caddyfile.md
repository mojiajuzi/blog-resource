---
title: "The HTTP Caddyfile"
date: 2018-02-06T23:54:27+08:00
draft: true
tags: ["Caddy", "Go"]
---

#### 站点地址(Site Addresses)
HTTP服务使用站点地址作为labels层，地址的形式为`scheme://host:port/path`,当你填写地址的时候，可以选择其中的
一部分进行使用

`host:port`通常是`localhost`或者是站点的域名，如果使用默认的https模式(此时端口号为：443),那么默认的端口号是`2015`
在某种意义上来说，协议代表着所使用的端口号，比如：http代表使用80端口， https代表使用443端口，如果协议和端口号都进行
指定，那么指定的优先级将会高于协议所默认的端口号。如下表格假设使用了自动使用https模式下，指定不同的格式所使用的端口号
```bash
:2015                    # Host: (any), Port: 2015
localhost                # Host: localhost; Port: 2015
localhost:8080           # Host: localhost; Port: 8080
example.com              # Host: example.com; Ports: 80->443
http://example.com       # Host: example.com; Port: 80
https://example.com      # Host: example.com; Ports: 80->443
http://example.com:1234  # Host: example.com; Port: 1234
https://example.com:80   # Error! HTTPS on port 80
*.example.com            # Hosts: *.example.com; Port: 2015
example.com/foo/         # Host: example.com; Ports: 80, 443; Path: /foo/
/foo/                    # Host: (any), Port: 2015, Path: /foo/
```
需要注意的是，站点域名地址必须唯一，对同一个域名的配置，必须放在相同的语句块内


#### 路径匹配(Path Matching)
某些指令接受`base path`地址参数的匹配，将该地址参数作为前缀进行比配，如果一个地址以`base path`开始，那么将会进行匹配
比如：某个`base path`为`/foo`，那么`/foo, /foo.html, /foobar, /foo/bar.html`将都会被匹配到,如果仅仅需要匹配到
某一个目录，那么只需要在`base path`后面添加斜杠就可以了`/foo/`


#### 指令(Directives)
绝大多数指令能够想中间件一样被调用。每一个中间件在应用中的功能是非常单一的用来处理HTTP请求,而且有且只有中间件能够被链式调用
由于每一个中间件又是非常的小，因此保证`Caddyfile`足够的小而且非常的快


语法中的的参数变量能够在指令中传递，具体指令中的参数，请查看具体的指令文档

caddy中的指令将会作为一个插件注册，文档页面的将会显示指令的前缀以及服务的类型
比如:`http.realip`或者`dns.dnssec`当在Caddyfile中使用时，需要去掉前缀`http.`,
前缀的作用仅仅用来确定名称的唯一性，但是在caddyFile中并没有使用到

#### 占位符(Placeholders)
在某些情况下，指令将会需要使用占位符以便用来放置那些在运行时才能确定的值，占位符使用`{}`进行包裹
比如:`{query}`或者`{>Referer}`将`query`和`>Referer`当做变量，占位符与在Caddyfile中所使用的环境变量
没有任何的关系