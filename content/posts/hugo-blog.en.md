---
title: "Blogging with Hugo"
date: 2022-12-01T15:22:38+08:00
draft: false
---

<!--more-->

## Install

- macos

```bash
brew install hugo
```
- linux

```
download： https://github.com/gohugoio/hugo/releases/latest
```

## Create a site

```bash
hugo new site yuancoder-hugo
cd yuancoder-hugo 
git init
hugo server -D
```

## Install theme

```bash
git submodule add https://github.com/dillonzq/LoveIt.git themes/LoveIt
echo "theme = 'LoveIt'" >> config.toml
```

## Directory Structure

```text
yuancoder/
├── archetypes/                
│   └── default.md              
├── assets/                     
├── content/                   
├── data/                      
├── layouts/                  
├── public/                   
├── static/                  
├── themes/                 
└── config.toml            
```

## Add content

```bash
hugo new posts/my-first-post.md
```

## Configure the site

```text
example: ./themes/LoveIt/exampleSite/config.toml 
config: ./config.toml
```

## Publish the site

```bash
hugo
```


## Deployment with Rsync

```
rsync -avz --delete public/ www-data@yourserver:/www/
```

