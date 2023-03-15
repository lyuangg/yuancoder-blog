---
title: "Laravel umyproxy package"
date: 2023-03-15T10:43:49+08:00
draft: false
tags:
- mysql
- php
- umyproxy
- laravel
- composer
categories:
- 开源
---

<!--more-->

为了方便 umyproxy 在 laravel 项目中使用，写了一个 laravel 的 package。

[https://github.com/lyuangg/laravel-umyproxy](https://github.com/lyuangg/laravel-umyproxy)

**使用很简单**

1. 使用 composer 安装 

```
composer require 'lyuangg/laravel-umyproxy'
```

2. 修改 laravel 的 env 文件，增加 DB_SOCKET

```
DB_SOCKET=/tmp/umyproxy.socket
```

> 注意 socket 路径权限。   
> 路径也可以指定 `/dev/shm/umyproxy.socket`   
> 这个目录实际会在内存中，可以提升性能

3. 启动服务

```bash
php artisan umyproxy:serve
```

> 服务启动会自动读取 `config/database.php` 里面的 mysql 配置

自定义参数, 查看帮助:

```bash
php artisan umyproxy:serve -h
```

**umyproxy**

> [https://github.com/lyuangg/umyproxy](https://github.com/lyuangg/umyproxy)
