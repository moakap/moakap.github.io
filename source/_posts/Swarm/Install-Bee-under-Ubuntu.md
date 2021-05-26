---
title: Ubuntu下运行Swarm Bee节点
default_category: Swarm
tags: 
  - Swarm
  - Bee
  - Ubuntu
---

## 1. 安装

基本安装步骤比较简单，可以直接[Swarm Bee参考官方文档]([Quick Start | Swarm Bee Client (ethswarm.org)](https://docs.ethswarm.org/docs/installation/quick-start))。



### 1.1 Bee Clef安装

在安装Bee之前，建议首先[安装Bee Clef软件包]([Clef External Signer | Swarm Bee Client (ethswarm.org)](https://docs.ethswarm.org/docs/installation/bee-clef/))。[Clef]([go-ethereum/cmd/clef at master · ethereum/go-ethereum (github.com)](https://github.com/ethereum/go-ethereum/tree/master/cmd/clef))是Go以太坊客户端使用的独立签名器（单独的一个对以太坊交易进行签名认证的服务），能够管理基于key-file的账户和硬钱包账户。[Bee-Clef]([ethersphere/bee-clef: bee-clef is official ethereum clef binary wrapped and preconfigured for bee as a service (github.com)](https://github.com/ethersphere/bee-clef))则在clef基础上针对bee做了定制化配置。Bee节点可以通过bee-clef来访问Swarm测试网或以太坊主网。当然，你也可以直接在本地运行原生的[clef]([go-ethereum/cmd/clef at master · ethereum/go-ethereum (github.com)](https://github.com/ethereum/go-ethereum/tree/master/cmd/clef))，然后增加自己的配置在运行Bee。**如果确定要使用Bee-clef，必须在Bee之前先安装Bee Clef。**

#### 选择最新版本

- 打开[Bee-clef下载页面]([Releases · ethersphere/bee-clef (github.com)](https://github.com/ethersphere/bee-clef/releases/))，查看最新版本。写当前文档时的最新版本为v0.4.12。 



#### 安装Bee-Clef

- 下载Bee-clef

  ```shell
  wget https://github.com/ethersphere/bee-clef/releases/download/v0.4.12/bee-clef_0.4.12_arm64.deb
  sudo dpkg -i bee-clef_0.4.12_arm64.deb
  ```



#### 配置Bee-clef

安装以后的默认配置文件在```/etc/bee-clef```目录下，包含```4byte.json```和```rules.s```两个文件，已经针对Bee做了基本配置，这里不需要特别修改。

```shell
# ll /etc/bee-clef/
-rw-r--r--  1 root root  163 Feb 25 01:48 4byte.json
-rw-r--r--  1 root root  249 Feb 25 01:48 rules.js
```



#### 启动Bee-Clef

在使用后台启动之前，建议先通过手工启动查看bee-clef是否正确安装。正常启动后会有一下的日志打印，其中会出现三个文件路径

- ```/etc/bee-clef/4byte.json```

  4byte数据库文件，包含了默认的Bee需要的一些方法。

- ```/etc/bee-clef/rules.js``` 

  签名器规则文件，定义了最基本的签名规则。比如一般来说，我们的很多交易或操作需要用户手工确认才能进行。但是也有一些操作，可以在不提示用户的情况下直接允许，比如规则文件里提到的启动、列出账户等。

  ```
  function OnSignerStartup() {
      return "Approve"
  }
  function OnApprovedTx() {
      return "Approve"
  }
  function ApproveListing() {
      return "Approve"
  }
  function ApproveTx() {
      return "Approve"
  }
  function ApproveSignData() {
      return "Approve"
  }
  ```

  

- ```/var/lib/bee-clef/clef.ipc``` 

  这个是bee-clef签名器与Bee节点之间进行通信的IPC通信管道，bee的所有请求都是通过这个管道发送到bee-clef。在接下来Bee的配置文件中也会用到。

  bee-clef正常启动后，一旦```bee```节点开始跟```bee-clef```交互，日志中每隔几秒或几分钟就会打印```INFO [05-11|23:41:53.365] Op approved```. 

  

```shell
# bee-clef-service start
bee-clef-service /var/lib/bee-clef /var/lib/bee-clef/password /etc/bee-clef
Waiting for the clef.ipc file to show up at /var/lib/bee-clef/clef.ipc

WARNING!

Clef is an account management tool. It may, like any software, contain bugs.

Please take care to
- backup your keystore files,
- verify that the keystore(s) can be opened with your password.

Clef is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY;
without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR
PURPOSE. See the GNU General Public License for more details.
INFO [05-11|23:35:49.197] Using stdin/stdout as UI-channel
INFO [05-11|23:35:49.925] Loaded 4byte database                    embeds=146841 locals=4 local=/etc/bee-clef/4byte.json
{"jsonrpc":"2.0","id":1,"method":"ui_onInputRequired","params":[{"title":"Master Password","prompt":"Please enter the password to decrypt the master seed","isPassword":true}]}
Waiting for the clef.ipc file to show up at /var/lib/bee-clef/clef.ipc
INFO [05-11|23:35:51.134] Rule engine configured                   file=/etc/bee-clef/rules.js
INFO [05-11|23:35:51.134] Starting signer                          chainid=5 keystore=/var/lib/bee-clef/keystore light-kdf=true advanced=false
INFO [05-11|23:35:51.135] IPC endpoint opened                      url=/var/lib/bee-clef/clef.ipc
{"jsonrpc":"2.0","method":"ui_onSignerStartup","params":[{"info":{"extapi_http":"n/a","extapi_ipc":"/var/lib/bee-clef/clef.ipc","extapi_version":"6.1.0","intapi_version":"7.0.1"}}]}
INFO [05-11|23:36:12.193] Op approved
INFO [05-11|23:36:12.203] Op approved
INFO [05-11|23:41:53.365] Op approved
INFO [05-11|23:45:04.948] Op approved
INFO [05-11|23:45:08.734] Op approved
```



#### Bee-Clef数据存储

bee-clef的密钥及其它数据默认存储在```/var/lib/bee-clef/```. 

- ```1e4b791d7af0cf7a42a8/ ``` - bee-clef账户保险库，里边的config.json文件中会保存加密过的key/value配置文件。
- ```clef.ipc``` - 刚才提到的IPC通信管道。
- ```keystore/``` - bee-clef的账户地址。bee通过这个地址接入测试网或主网。
- ```masterseed.json``` - 存放bee-clef密码
- ```password```账户的密钥文件 

```shell
# ll /var/lib/bee-clef/
total 24
drwxr-x---  4 bee-clef bee-clef 4096 May 11 19:07 ./
drwxr-xr-x 50 root     root     4096 May 11 00:26 ../
drwx------  2 bee-clef bee-clef 4096 May 11 00:24 1e4b791d7af0cf7a42a8/
srw-rw----  1 root     root        0 May 11 19:07 clef.ipc=
drwx------  2 bee-clef bee-clef 4096 May 11 00:24 keystore/
-r--------  1 bee-clef bee-clef  868 May 11 00:24 masterseed.json
-rw-------  1 bee-clef bee-clef   32 May 11 00:24 password
```



### 1.2 Bee安装

安装完bee-clef并确保其正常启动后，我们来安装Bee。

#### 查看最新版本

先到[Bee版本下载]([Releases · ethersphere/bee (github.com)](https://github.com/ethersphere/bee/releases/))查看最新版本，当前最新版本为v0.5.3. 



#### 下载并安装Bee

```sh
wget https://github.com/ethersphere/bee/releases/download/v0.5.3/bee_0.5.3_amd64.deb
sudo dpkg -i bee_0.5.3_amd64.deb
```



#### 配置Bee

安装完毕以后默认配置文件在```/etc/bee/bee.yaml```。也可以运行```bee printconfig &> bee-default.yaml```打印当前的Bee配置作为模板来自己修改。其中几个地方需要注意。

- clef配置 - 需要根据Bee-Clef的配置设置对应的IPC路径。

  ```
  ## enable clef signer
  clef-signer-enable: true
  ## clef signer endpoint
  clef-signer-endpoint: /var/lib/bee-clef/clef.ipc
  ```

- debug API地址 - 如果需要运行bee-dashboard，必须设置。默认为本机的1635端口，也可以自己修改。

  ```
  ## debug HTTP API listen address (default ":1635")
  debug-api-addr: 127.0.0.1:1635
  ## enable debug HTTP API
  debug-api-enable: true
  ```

- Swap节点 - 以太坊交换节点，也就是接入以太坊的入口节点。必须设置，默认为本机的:8545端口，如果你在本机运行geth （go的以太坊客户端）。否则，推荐使用Warm官方公共节点```https://rpc.slock.it/goerli```。也可以自己到[Ethereum API | IPFS API & Gateway | ETH Nodes as a Service | Infura](https://infura.io/) 注册一个，使用自己注册的节点来减少拥堵。

  ```
  ## enable swap (default true)swap-enable: true## swap ethereum blockchain endpoint (default "http://localhost:8545")swap-endpoint: https://rpc.slock.it/goerli
  ```

其它配置请参考[配置文件 – MoDocs (lovpia.com)](https://www.lovpia.com/Swarm/BeeConfiguration)。



#### 启动运行Bee

更新完配置文件以后，可以直接通过命令行启动来检测配置是否正确。第一次启动会需要输入密码，同时需要账户有至少10gBZZ作为启动资金。

##### 领取启动资金

先通过```bee-clef-keys```命令获取节点的ethernum地址（在产生的```bee-clef-jey-xxxx.json```文件中```address```字段）。

- [faucet-request (discord.com)](https://discord.com/channels/799027393297514537/813744618776428594) - 通过faucet机器人领取，是好是坏，碰运气。
- [Goerli: Authenticated Faucet (mudit.blog)](https://faucet.goerli.mudit.blog/) 需要先在Twitter上发布条信息，然后把twitter链接粘贴到页面里领取，可以一次领取37.5gETH，然后到[Bzzaar (ethswarm.org)](https://bzz.ethswarm.org/)兑换即可，成功率很高，而且可以兑换不止10个gBZZ. 



##### 运行Bee

可以直接执行```bee start```来启动，然后查看运行日志是否正常。如果使用自定义配置文件，就加上```--config <path to bee.yaml>```。

Bee正常启动以后，会默认监听:1633端口，当日志中出现```api address: xxxx```时，Bee已经启动。如果打开了debug api，也会在日志中看到```debug api address: xxx:1633```. 



```shell
# bee start --config bee.yaml Welcome to the Swarm.... Bzzz Bzzzz Bzzzz                \     /            \    o ^ o    /              \ (     ) /   ____________(%%%%%%%)____________  (     /   /  )%%%%%%%(  \   \     )  (___/___/__/           \__\___\___)     (     /  /(%%%%%%%)\  \     )      (__/___/ (%%%%%%%) \___\__)              /(       )\            /   (%%%%%)   \                 (%%%)                   !                   INFO[2021-05-18T13:55:09+08:00] version: 0.5.3-acbd0e2                       INFO[2021-05-18T13:55:09+08:00] using swarm network address through clef: <xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx> INFO[2021-05-18T13:55:09+08:00] swarm public key <xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx>  INFO[2021-05-18T13:55:09+08:00] pss public key <xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx> INFO[2021-05-18T13:55:09+08:00] using ethereum address <xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx>INFO[2021-05-18T13:55:09+08:00] debug api address: 127.0.0.1:1635            INFO[2021-05-18T13:55:11+08:00] using default factory address for chain id 5: <xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx> INFO[2021-05-18T13:55:12+08:00] using existing chequebook <xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx> INFO[2021-05-18T13:55:25+08:00] database capacity: 2000000 chunks (approximately 7.8GB) INFO[2021-05-18T13:55:33+08:00] name resolver: no name resolution service provided INFO[2021-05-18T13:55:33+08:00] api address: [::]:1633...# 注意：其中的xxx为你的Bee节点对应的key和地址信息.
```

确保Bee可以正常启动并运行以后，就可以把Bee加入到系统服务后台运行就可以了。



#### 查看Bee基本信息

最基本的就是在浏览器直接查看```localhost:1633```, 也可以在命令行用curl命令查看，如果显示```Ethereum Swarm Bee```就说明已经在运行了。

```
# curl -s localhost:1633Ethereum Swarm Bee
```

其它信息可以通过[bee debug api]([Bee Debug API (ethswarm.org)](https://docs.ethswarm.org/debug-api/))查看。

比如自己做一个简单的bee_health_check.sh脚本，可以快速输出当前Bee节点的基本信息。

```shell
#!/usr/bin/env bash# get addresses of current nodeecho '====> 获取Bee节点地址...'echo '> curl -s localhost:1635/addresses'curl -s localhost:1635/addresses | jq# check connected peersecho '====> 获取已经建立连接的节点列表...'echo '> curl -s localhost:1635/peers'curl -s localhost:1635/peers | jq# check balance# echo '====> 获取账户基本信息...'# echo 'curl -s localhost:1635/balances'# curl -s localhost:1635/balances | jq# get chequebook balanceecho '====> 获取支票账户信息...'echo '> curl -s localhost:1635/chequebook/balance'curl -s localhost:1635/chequebook/balance | jqecho '====> 获取支票信息...'echo '> curl -s localhost:1635/chequebook/cheque'curl -s localhost:1635/chequebook/cheque | jq                             
```

另外一种方式是直接使用bee-dashboard查看，就是把bee debug api获取的状态通过图形界面可视化。需要提前打开debug api，具体参考[bee-dashboard教程]([ethersphere/bee-dashboard: An app which helps users to setup their Bee node and do actions like cash out cheques (github.com)](https://github.com/ethersphere/bee-dashboard))。



## 2. 其它注意事项

### 问题1：```could not connect to backend at xxx```

```shell
could not connect to backend at https://rpc.slock.it/goerli. In a swap-enabled network a working blockchain node (for goerli network in production) is required. Check your node or specify another node using --swap-endpoint. Error: get chain id: Post "https://rpc.slock.it/goerli": dial tcp 185.20.210.132:443: i/o timeout
```

因为Swap节点不可用，或者节点拥堵，建议到[Ethereum API | IPFS API & Gateway | ETH Nodes as a Service | Infura](https://infura.io/) 注册一个自己的节点，避免官方节点拥堵。




