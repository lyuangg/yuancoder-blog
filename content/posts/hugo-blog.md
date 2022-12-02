---
title: "使用 Hugo 搭建博客"
subtitle: ""
date: 2022-11-30T18:30:05+08:00
draft: false
tags:
- hugo 
- blog
categories:
- 开发
---

<!--more-->

## 安装

- macos

```bash
brew install hugo
```
- linux

```
下载： https://github.com/gohugoio/hugo/releases/latest
```

## 创建站点

```bash
hugo new site yuancoder-hugo
cd yuancoder-hugo 
git init
hugo server -D
```

## 安装主题

```bash
git submodule add https://github.com/dillonzq/LoveIt.git themes/LoveIt
echo "theme = 'LoveIt'" >> config.toml
```

## 目录结构

```text
yuancoder/
├── archetypes/                 # 内容模版目录
│   └── default.md              # 默认的内容模板
├── assets/                     # 存储所有需要 Hugo Pipes 处理的文件。
├── content/                    # 写的 markdown 文件都在这里
├── data/                       # 数据模板目录
├── layouts/                    # 布局模板文件目录，存放 .html 布局模板文件
├── public/                     # 生成静态站点的文件输出目录
├── static/                     # 静态资源存放目录, 其它脚本、图像、CSS 等
├── themes/                     # 主题目录
└── config.toml                 # 站点配置
```

## 添加文章

```bash
hugo new posts/my-first-post.md
```

## 站点配置

```text
参考配置文件: ./themes/LoveIt/exampleSite/config.toml 
修改配置文件: ./config.toml
```

## 发布站点

```bash
hugo
```

`hugo` 命令执行完后会在 `./public` 目录下生成所有静态的页面和资源。
直接部署 `public` 目录即可。


## 百度统计 

直接在 `./config.toml` 配置文件页面底部信息 `custom` 中增加统计代码即可

```toml
 # Footer config
  # 页面底部信息配置
  [params.footer]
    enable = true
    # Custom content (HTML format is supported)
    # 自定义内容 (支持 HTML 格式)
    custom = "这里可以添加统计代码"
```

## 使用 Rsync 部署

```
rsync -avz --delete public/ www-data@yourserver:/www/
```