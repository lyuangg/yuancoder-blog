---
title: "博客增加评论"
date: 2022-12-01T19:40:26+08:00
draft: false
tags:
- hugo
---

<!--more-->

一开始使用 `gitalk` 作为评论系统，发现配置完有一些问题， 后面选用了 `giscus`。
`giscus` 的配置还是非常简单的。   
跟着 giscus页面 [https://giscus.app/zh-CN](https://giscus.app/zh-CN) 一步步操作，最后会生成配置。 

根据生成的配置修改 `./config.toml` 配置文件 

```toml
[params.page.comment.giscus]
```
    

看看效果吧 ！ 
