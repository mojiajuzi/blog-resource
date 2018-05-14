---
title: "Caddyfile"
date: 2018-01-31T23:07:05+08:00
draft: true
description: Caddy服务器的CaddyFile文件详情
tags: ["Caddy", "Go"]
---

该文章将会向你展示使用`Caddyfile`配置`Caddy`是一件十分简单的事情

Caddyfile是一个文本文件，用来配置Caddy如何运行

该文件的第一行永远是服务站点的地址，比如:
```
localhost:8080
```

当你保存以后，一旦启动caddy服务器，那么将会自动查找Caddyfile文件，并加载其中的配置
默认情况会在当前的目录下面查找Caddyfile文件，如果将配置文件放置在其他的地方，那么在
启动的时候，需要指明Caddyfile文件所在的路径

```
caddy -conf ../path/to/Caddyfile
```

紧跟着站点地址的下一行则是指令，Caddy提供了丰富的[指令](https://caddyserver.com/docs)
比如: [gzip](https://caddyserver.com/docs/gzip)则是一个HTTP指令

```
localhost:8080
gzip
log ../access.log
```

有一些指令需要使用多行进行配置，这个时候需要使用`{}`进行配置，而且`{`必须在指令的行尾
```
localhost:8080
gzip
log ..access.log
markwodn /blog {
    css /blog.css
    js /scripts.js
}
```
如果`{}`里面不进行设置，那么则应该省略
如果配置参数的值包含空白符，那么则应该使用`""`进行包裹

当然caddyfile文件中也可以以`#`开头，添加注释

```
# 注释可以单独作为一行
foobar #也可以放在配置的末尾
```

如果需要在一个caddyfile文件中对多个站点进行配置，那么则必须使用`{}`对每一个站点进行分割
```
mysite.com {
    root /www/mysite.com
}

sub.mysite.com {
    root /www/sub.mysite.com
    gzip
    log ../access.log
}
```
`{}`的使用规则，与多个参数的指令规则一样，多余多站点，所有的配置都必须包含在站点的`{}`之内，不允许嵌套

对于多个站点使用共同的配置，可以使用如下的方式
```
localhost:8080, https://site.com, http://mysite.com {
    ...
}
```

另外站点的地址可以是特殊的地址或以及使用通配符
```
example.com/static, *.example.com {
    ...
}
```
需要注意的是对于使用路径地址作为站点，则路由请求规则使用最长匹配原则，如果基础路劲是一个目录，那么
则需要在路径站点地址前面加上斜杠`/`

对于站点地址以及参数而言，也可以使用变量，这些变量必须使用`{}`进行包裹
```
localhost:{$PORT}
root {%SITE_ROOT%}
```
如上的每一种方式，都可以运行在任何的平台，单一的环境变量并不会扩展到多个参数/值

与nginx不同的是，caddy中并没有继承和脚本的概念，某些情况下，你可能需要在caddyfile的
多个地方对同一个站点进行配置，在这种情况下，推荐使用`import`指令，这样可以避免重复

关于更多的详情，请查看[caddy文档](https://caddyserver.com/docs)


