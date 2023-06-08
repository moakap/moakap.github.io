---
title: ubuntu下安装和部署RAP2
date: 2023-06-08 15:15:51
categories: 
  - 蜻蜓点水
  - RAP2
tags:
  - RAP2
  - Ubuntu
---

# ubuntu下安装和部署RAP2

## 1. 后台部署
### 1.1 安装mysql和redis

```
$ sudo apt update
$ sudo apt install mysql-server
$ sudo apt install redis-server
```

### 1.2 安装pandoc
1.2.1 从[pandoc release下载页面](https://github.com/jgm/pandoc/releases)下载最新版本
```
$ wget https://github.com/jgm/pandoc/releases/download/3.1/pandoc-3.1-1-amd64.deb
```

1.2.2 安装
```
$ sudo dpkg -i pandoc-3.1-1-amd64.deb
$ pandoc -v
pandoc 3.1
Features: +server +lua
Scripting engine: Lua 5.4
User data directory: /home/ubuntu/.local/share/pandoc
Copyright (C) 2006-2023 John MacFarlane. Web:  https://pandoc.org
This is free software; see the source for copying conditions. There is no
warranty, not even for merchantability or fitness for a particular purpose.

```

### 1.3 安装nodejs和npm

```
$  sudo apt install nodejs npm
```

### 1.4 创建数据库

```
$ mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
...

mysql> CREATE DATABASE IF NOT EXISTS RAP2_DELOS_APP DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
Query OK, 1 row affected, 2 warnings (0.07 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| RAP2_DELOS_APP     |
...
```

### 1.5 初始化
1.5.1 下载rap2-delos并安装依赖软件包
```
$ git clone https://github.com/thx/rap2-delos.git
...
$ cd rap2-delos
$ npm install

```

1.5.2 修改项目配置
根据需要修改`src/config/config.{ENV}.ts`中对应的数据库配置信息。

1.5.3 安装typescript

```
sudo npm install -g typescript
npm run build
```

1.5.4 初始化数据库
```
$ npm run create-db

> rap2-delos@2.9.0 create-db /home/ubuntu/mysites/rap/rap2-delos
> cross-env NODE_ENV=development node dist/scripts/initSchema

----------------------------------------
DATABASE √
    HOST     localhost
    PORT     3306
    DATABASE RAP2_DELOS_APP
----------------------------------------
成功初始化 DB Schema
```

### 1.6 启动后台服务
1.6.1 开发模式下启动
```
$ npm run dev
> rap2-delos@2.9.0 dev /home/ubuntu/mysites/rap/rap2-delos
> cross-env NODE_ENV=development nodemon --watch scripts --watch dist dist/scripts/dev.js

[nodemon] 2.0.21
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): scripts dist/**/*
[nodemon] watching extensions: js,mjs,json
[nodemon] starting `node dist/scripts/dev.js`
Starting task: locker check
----------------------------------------
rap2-delos is running as http://localhost:8080
----------------------------------------
DATABASE √
    HOST     localhost
    PORT     3306
    DATABASE RAP2_DELOS_APP
----------------------------------------

```

1.6.2 生产模式下启动
```
$ npm run build
$ npm start
```

## 2. 前端部署

### 2.1 下载前端代码库
```
$ git clone https://github.com/thx/rap2-dolores.git
...
$ cd rap2-delores
```

### 2.2 安装依赖
```
$ npm install
```

### 2.3 配置后端api地址并生成静态站点
配置文件路径：/src/config/config.prod.js
```
module.exports = {
  serve: 'http://127.0.0.1:8080',
  keys: ['some secret hurr'],
  session: {
    key: 'koa:sess'
  }
}
```
更改`serve`字段，改成我们的后端访问地址，访问地址直接使用ip，不要使用127.0.0.1。 注意加 http://


```
$ npm run build
```

#### “0308010c:digital envelope routines::unsupported”错误
参考[如何解决 “0308010c:digital envelope routines::unsupported” 的错误](https://www.freecodecamp.org/chinese/news/error-error-0308010c-digital-envelope-routines-unsupported-node-error-solved/)，修改rap2-dolores下package.json中的`react-scripts start`命令，添加`--openssl-legacy-provider`选项。

``` package.json
...
"scripts": {
    "build-css": "node ./node_modules/sass/sass src/:src/ --quiet",
    "watch-css": "npm run build-css && node ./node_modules/sass/sass src/:src/ --watch",
    "start-js": "react-scripts start --openssl-legacy-provider",
    "start": "npm-run-all -p watch-css start-js --max-old-space-size=4096",
    "build": "npm run lint && npm run build-css && react-scripts --openssl-legacy-provider build",
    "test-backup": "npm run lint && react-scripts test --env=jsdom",
    "test": "npm run lint",
    "eject": "react-scripts eject",
    "lint": "tslint --project ./ -c tslint.json --fix",
    "build-docker": "docker build --rm -f \"Dockerfile\" -t rapteam/rap2-dolores:latest ."
  },
...

```

### 2.4 部署`/build`到服务器

#### 2.4.1 ngnix配置文件

```
#
server {
        listen 80;
        listen [::]:80;

        server_name rap2.example.com;

        root ${PATH_TO_rap2-dolores};
        index index.html index.php;

        location / {
                try_files $uri $uri/ /index.html;
        }

}
```

#### 404 not found
参考[这里](https://blog.csdn.net/waterdemo/article/details/81354973)，只需要确保nginx配置文件下的localtion路径为`try_files $uri $uri/ /index.html;`即可。

如果使用了下边的配置，就会出现404错误
```
        location / {
                try_files $uri $uri/ =404;
        }
```



