---
title: Swarm Bee配置
default_category: Swarm
tags: 
  - Swarm
  - Bee
  - BZZ
---

每个Bee节点可以在启动的时候添加命令行参数来配置，具体参数可以通过```bee start --help```查看。

例如下边这个例子，我们打开了Bee的debug API，并且设置debug API的监听端口为6666.

```shell
bee start \
  --api-addr=:8888 \
  --debug-api-enable=true \
  --debug-api-addr=:6666
```

## 配置文件

Bee节点也可以通过提供```--config```加yaml配置文件的方式来配置。

```shell
bee start --config /home/<user>/bee-config.yaml 
```

提示： 可以通过命令```bee printconfig &> bee-default.yaml```导出当前配置文件作为默认修改版本。



## 配置通过软件包管理器安装的Bee

如果你的Bee是通过```apt-get```或```yum```这样的软件包管理器安装的，在安装过程中会自动生成一个配置文件，Bee启动的时候默认使用这个配置文件。

要修改Bee配置，可以直接修改对应的配置文件，然后重启Bee生效。

**Linux**

```shell
sudo vi /etc/bee/bee.yaml
sudo systemctl restart bee
```

**Mac OS**

```shell
vi /usr/local/etc/swarm-bee/bee.yaml
brew services restart swarm-bee
```

## 自动产生配置文件

通过命令行启动Bee时候，用```printconfig```替换```start```可以很简单的生成Bee的配置文件。

**例子**

```shell
bee printconfig \
  --verbosity=5 \
  &> bee-config.yaml
```

上述命令会生成类似下边的文件内容，其中包含了Bee的默认配置。

## 配置选项

### 全局选项

#### --config 

设置Bee配置文件，默认为```/home/<user>/.bee.yaml```.



### 启动选项

#### --api-addr

默认 ```:1633```.

Bee API地址和端口。忽略IP地址，会监听所有地址的对应端口。

#### --bootnode

默认```/dnsaddr/bootnode.ethswarm.org```.

设置Bee启动节点的地址（要启动或加入的网络），可以是多个地址。

默认情况下节点会连接到Swarm主网。如果要使用私有或者测试网络，必须配置启动节点。

#### --clef-signer-enable

默认```false```.

如果要使用以太坊的```Clef```作为独立的交易签名器，设置为```true```。 Clef是以太坊的新功能，需要相应的规则文件或者运行在高级模式下，用来实现对交易和支票的自动签名。

#### --clef-signer-endpoint

默认为当前操作系统下clef的默认路径。

你也可以为你的Clef签名器指定自定义ipc文件。

#### --clef-signer-ethernum-address

默认会选择clef下的第一个地址（index 0）。

如果你在Bee Clef中导入了多个key，可以使用这个参数来指定要使用的地址。

**注意**：如果在Bee Clef中导入了多个地址，必须为每个Bee节点指定对应的签名器地址，第一个地址也需要配置，因为这些地址在导入的时候可能会重新排序。

#### --cors-allowed-origins

默认为```[]```.

允许的Http/WS API响应地址域。API只会接受对应地址域的请求，例如：

```shell
bee start --cors-allowed-origins="*"
bee start --cors-allowed-origins="https://website.ethswarm.org"
```

#### --data-dir

默认```/home/<user>/.bee```. 

Bee数据存储路径。包含以下三种类型的数据。

1. 区块数据

   具体会包含本地pin过的区块数据和文件、缓存的区块数据、或当前节点职责范围内需要提供给对端节点的区块数据。

2. 状态数据

   本地Bee节点的状态数据，需要备份。

3. 密钥库（keystore）数据

   包含加密过的私钥，需要备份并且保证私密性。

**注意：**在秘钥库(keystore)中的文件要保证绝对的安全！它们是你的网络身份的加密标记，一旦丢失无法恢复。

#### --db-capacity

默认```5000000```.

区块数据容量。区块数据已4086byte作为基础单位，所以总的数据库容量（kb）大概为```db-capacity * 4096```. 默认值5,000,000个区块大概有20.5gb。为了能够在网络中有效工作，我们建议一个Bee节点的容量至少要2.5gb。一些不参与数据存储的节点可以设置更小的容量。

下边4个参数对应更低层级的配置，是针对LevelDB的Openfile函数的配置。

##### --db-block-cache-capacity

默认```33554432```。

对应LevelDB的```BlcokCacheCapacity```.

##### --db-disable-seeks-compaction

默认```false```. 

对应LevelDB的```DisableSeeksCompaction```.

##### --db-open-files-limit

默认```200```.

**注意：**为了兼容低性能的硬件和操作系统，```--db-open-files-limit```的默认值设置的比较低。建议如果可能的话，增大到接近```10000```或者更多。

对应LevelDB的```OpenFileCacheCapacity```.

##### --db-write-buffer-size

默认```33554432```.

对应LevelDB的```WriteBuffer```. 

#### --debug-api-addr

默认```:1635```.

Debug API的地址和端口。忽略地址会监听主机的所有接口。

```--debug-api-enable```必须设置才会生效。

#### --debug-api-enable

默认```false```. 

设置为```true```来打开debug api。

#### --gateway-mode

默认```false```.

设置为```true```会关闭API中的一些敏感功能，可以使api地址在公共网络中更安全。

#### --global-pinning-enable

默认```false```.

设置为```true```来打开全局pin功能。

#### --nat-addr

默认```""```.

设置对应的NAT公网IP。一般情况下会自动生成，一些特殊情况可能需要手工设置。

#### --network-id

默认```1```.

接受新连接的网络ID。主网设置```1```, 测试网```2```.

#### --p2p-addr

默认```:1634```.

p2p协议消息的IP地址和端口。

#### --p2p-quic-enable

默认```false```.

#### --p2p-ws-enable

默认```false```.

配置是否打开p2p通信的web-socket传输。

#### --password

默认```""```.

解密Swarm标识密钥的密码。

**注意：**在命令行中设置密码是不安全的。在生成环境中请使用密码文件或环境变量。

#### --password-file

默认```""```.

指向解密用的密码文件路径。空字符串密码不需要设置。

#### --payment-early

*默认* `1000000000000`.

触发Bee结算的最低结算限制。

#### --payment-threshold

*默认* `10000000000000`.

从对等节点获得付款的最低支付门限，超过这个值才会触发支付。

#### --payment-tolerance

*默认* `50000000000000`

可容忍的最大欠债，超过这个门限后Bee会从这个节点断开。

#### --resolver-options

*默认* ```eth:0x00000000000C2E074eC69A0dFb2997BA6C7d2e1e@localhost:8545```.

#### --standalone

*默认* `false`

如果不想让Bee节点连接网络，设置```true```. 开发模式下使用。

#### --swap-enable

*默认* `true`

#### --swap-endpoint

*默认* `http://localhost:8545`

以太坊区块链交换节点。

#### --swap-factory-address

*默认* `anointed contract for the current blockchain id`

#### --swap-initial-deposit

*默认* `100000000000000000`

#### --tracing-enable

*默认* `false`

#### --tracing-endpoint

*默认* `127.0.0.1:6831`

#### --tracing-service-name

*默认* `bee`

#### --verbosity

*默认* `info`

Bee日志等级：0=silent, 1=error, 2=warn, 3=info, 4=debug, 5=trace

#### --welcome-message

*默认* `""`

连接到对端节点时显示的欢迎信息。

### 环境变量

可以使用环境变量定义Bee的配置。

在环境变量中定义好对应的值以后，直接在命令行参数中使用。

例如：```bee start --api-addr $VARIABLE_NAME```。

### 配置优先级

Bee配置按照以下优先级（从高到低）加载处理，高优先级中的配置会覆盖低优先级中的值。

1. 命令行参数
2. 环境变量
3. 配置文件

