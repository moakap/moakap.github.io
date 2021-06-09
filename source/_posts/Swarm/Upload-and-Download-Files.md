---
title: 上传和下载文件
categories: 
  - Technology
  - Swarm
tags: 
  - Swarm上传
  - Swarm下载
date: 2021-06-06 12:02:45

---

## 上传和下载文件

当我们把文件上传到Swarm，这些文件会被分解成大小为4kb的数据块，然后分布存储到网络中负责存储和分发节点。每个数据块上粘贴了一个特定面值的邮票（单位为gBZZ），邮票金额会随着数据的存储不断消耗，这些都是在我们购买邮票组的时候已经设定了。对于网络中的存储节点来说，邮票面值的大小代表了数据的重要程度和存储持续性，也就是数据要保持DISC（分布式不可更改存储数据块）。

<!-- more -->

### Overview

按照下边的步骤上传数据到Swarm：

1. 给Bee节点钱包充值
2. 使用gBZZ购买邮票包
3. 等待票据包分发到网络
4. 上传内容，需要绑定邮票包ID，BEE节点会在对应的数据块上粘贴邮票
5. 使用hash下载内容

## 购买邮票包

要上传数据到Swarm，必须消耗一定量的gBZZ，以此来告诉网络中的存储和分发节点你的数据是重要的。所以，在开始接下来的步骤前，你必须先购买一些邮票。参考[购买邮票包](https://docs.ethswarm.org/docs/access-the-swarm/keep-your-data-alive)。

### 上传

Bee节点运行起来以后，会打开对应的HTTP API，你可以通过这些API来控制Bee。使用[curl](https://ec.haxx.se/http/http-multipart)命令行可以方便的调用这些api。

首先，确保API正常运行。

```shell
curl http://localhost:1633
```

```shell
Ethereum Swarm Bee
```

Bee API正常运行后，通过POST请求直接上传文件。这里你必须在头部指定```batch id```。

```shell
curl -H "Swarm-Postage-Batch-Id: 78a26be9b42317fe6f0cbea3e47cbd0cf34f533db4e9c91cf92be40eb2968264" -F file=@bee.jpg http://localhost:1633/bzz
```

我们也可以通过```Content-Type```来设置MIME类型，通过```name```来参数设置文件名。

```shell
curl --data-binary @bee.jpg  -H "Swarm-Postage-Batch-Id: 78a26be9b42317fe6f0cbea3e47cbd0cf34f533db4e9c91cf92be40eb2968264" -H "Content-Type: video/jpg" "http://localhost:1633/bzz?name=bee.jpg"
```

注意：上传到Swarm的数据默认是公开的。对于敏感文件，必须在上传之前对文件内容进行加密来保证数据的隐私性。

上传成功以后，结果会已json形式返回，其中包含swarm引用ID（hash），也就是上传文件的swarm地址，例如

```json
{"reference":"042d4fe94b946e2cb51196a8c136b8cc335156525bf1ad7e86356c2402291dd4"}
```

妥善保存这个地址，接下来我们需要使用这个地址获取文件。

在Swarm中，每个数据文件会有一个唯一的地址。如果同一个文件上传两次，会得到相同的hash值。这也确保了Swarm中数据的安全性和真实性。

如果想在上传过程中获取上传状态，参考[获取上传状态](https://docs.ethswarm.org/docs/access-the-swarm/syncing)。一旦文件完全同步到网络，网络中的其它节点会负责存储和分发数据。

## 下载

文件上传到Swarm以后，可以通过HTTP GET直接下载。

命令行下载

```shell
curl -OJ http://localhost:1633/bzz/042d4fe94b946e2cb51196a8c136b8cc335156525bf1ad7e86356c2402291dd4
```

你也可以直接在浏览器中输入连接来下载文件。

[http://localhost:1633/bzz/042d4fe...2291dd4](http://localhost:1633/bzz/042d4fe94b946e2cb51196a8c136b8cc335156525bf1ad7e86356c2402291dd4)

## 公共网关

要分享文件给没有运行Bee节点的人，你只要简单替换链接中的主机地址为Swarm公共网关。这样他们也能够下载这个文件了。

[https://gateway.ethswarm.org/bzz/042d4fe...2291dd4](https://gateway.ethswarm.org/bzzd/042d4fe94b946e2cb51196a8c136b8cc335156525bf1ad7e86356c2402291dd4)
