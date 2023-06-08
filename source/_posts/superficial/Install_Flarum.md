---
title: 安装Flarum
categories: 
  - 蜻蜓点水
tags: 
  - Flarum
  - nginx
date: 2023-02-24 10:19:55
---

## 1. 安装Composer
```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '55ce33d7678c5a611085589f1f3ddf8b3c52d662cd01d4ba75c0ee0459970c2200a51f492d557530c71c15d8dba01eae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```

## 2. 安装Flarum
```
composer create-project flarum/flarum . 
```

## 3. 设置nginx和URL重写
在nginx站点配置文件中添加root和flarum 配置文件。
```
root /<Flarum路径>/public
include /<Flarum 路径>/.nginx.conf;
```

## 4. 安装Flarum
