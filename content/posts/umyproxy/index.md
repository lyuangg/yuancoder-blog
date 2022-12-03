---
title: "PHP MYSQL 连接池解决方案 - umyproxy"
date: 2022-12-03T18:24:33+08:00
draft: false
tags:
- mysql
- php
- umyproxy
- go
categories:
- 开源
---

<!--more-->

传统的 PHP 项目在高并发场景下有一个痛点：没有 mysql 连接池。 

### 为什么连接池这么重要

我们看看在没有连接池的情况下会发生什么事情: 

1. 大量的短连接会占用端口，而计算机只有256*256（65536）个端口, 实际更少，高并发情况下会导致无端口可用。
2. 当服务器处理完请求后主动关闭连接, 这个场景下会出现大量 socket 处于 TIME_WAIT 状态, 这个状态是 TCP 协议规定的，主要是为了保证连接的可靠性。

### PHP 为什么没有连接池

这是由于 PHP-FPM 的运行机制，导致了 PHP 没办法使用连接池，因为每次请求结束，PHP 脚本都会被释放， 无法保持一个连接池。

### PHP 使用连接池的方案

1. 基于 workman 或者 swoole 这样的常驻内存的框架是可以实现连接池的， 但是这种运行模式与 php-fpm 完全不一样了，需要对原来的项目进行大量的改造，传统项目实现成本较高，新项目可以考虑使用。
2. 对于传统的 php-fpm 项目可以使用本地代理的方式来实现，每个项目本地建立一个代理服务， 代理服务与 mysql 建立长连接和连接池，项目访问代理服务来与 mysql 通信。

我用 go 语言写了一个 mysql 代理服务[umyproxy](https://github.com/lyuangg/umyproxy)，代理服务支持连接池，不需要对项目进行大量的改动，仅仅修改配置即可。 

### umyproxy 实现原理

- 代理服务通过 `unix domain socket` 与 php 脚本进行通信，这样不占用端口，也不需要真正的经过网络协议栈，有更高的通信效率。
- 代理服务与 mysql 建立连接实现连接池。
- 代理服务可以识别 php 和 mysql 的握手，认证，发送命令等协议。
- 当 php 程序使用完成发送 close 指令时，代理服务把 mysql 连接放入连接池，等待下次请求使用。
- PHP 程序只需要把连接 mysql 的方式改成 unix socket 的方式即可。

项目地址: [github.com/lyuangg/umyproxy](https://github.com/lyuangg/umyproxy)