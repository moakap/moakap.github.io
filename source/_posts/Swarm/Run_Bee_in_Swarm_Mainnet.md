---
title: Swarm主网运行Bee
categories: 
  - 蜻蜓点水
  - Swarm
tags: 
  - Swarm
  - Mainnet
date: 2021-06-24 10:19:55
---

Swarm主网与6月21日上线，根据[官方通知](https://medium.com/ethereum-swarm/swarm-airdrop-is-finishing-on-21-june-2021-important-notice-to-all-participants-6a58f29017a2)，所有测试网的活动都会结束，之前参与空投的节点需要在7月12号之前把之前兑现。
## 空投安排
- 6月21日: 主网上线，空投截至
- 6月21日 - 7月12日: 支票兑现，收到支票但没有兑现的节点不能参与空投。
- 7月15日: Swarm基金会公布信任节点(trusted nodes)，并部署空投BZZ (aBZZ - airdrop BZZ)到Goerli测试网。根据官方说法，到时候会发布一个专门的应用。通过空投应用可以查看自己的节点是否会收到空投以及对应的aBZZ币数量。
- 7月15日 - 21日: 所有人可以在官网查看自己的aBZZ账户，参与者可以将aBZZ集中的某一个账户，用来在主网接收BZZ。
- 7月22日: aBZZ转移截止。
- 8月2日: 派发BZZ，测试网Goerli的aBZZ会转化成xDai中的BZZ。

<!-- more -->

## Swarm主网运行Bee
同时，从6月21日开始，所有节点都可以切到主网来专区BZZ。根据官方的说法，从测试网到主网没有任何继承关系，不存在所谓的升级操作(upgrade)。除了ethereum地址不变，节点在主网里会是一个全新的节点。

具体步骤请参考官方声明 

- [Install Bee | Swarm Bee Client (ethswarm.org)](https://docs.ethswarm.org/docs/installation/install)

- [Releases · ethersphere/bee (github.com)](https://github.com/ethersphere/bee/releases)

### 基本配置
几个要修改的地方，其它保存不变即可。

```yaml
## ID of the Swarm network (default 1)
network-id: 1 # 主网ID为1，测试网10
## cause the node to start in full mode
full-node: true
## enable swap (default true)
swap-enable: true
## swap ethereum blockchain endpoint (default "ws://localhost:8546")
swap-endpoint: https://stake.getblock.io/mainnet/?api_key={API_KEY} # 替换成自己注册的key
## initial deposit if deploying a new chequebook (default 10000000000000000)
swap-initial-deposit: 0
## triggers connection to main network
mainnet: true 
```

### 常见问题

这里说一下几个常见的问题

#### chequebook is not deployed by factory 

> INFO[2021-06-23T10:34:54+08:00] using existing chequebook xxxx
> Error: chequebook init: chequebook not deployed by factory

这个是由于使用了测试网的chequebook地址，所以提示没有在主网部署。解决方法是删除bee目录下的```localstore```文件夹。不放心的化可以重命名一下，然后重启bee，会自动创建新的```localstore```. 

```shell
mv localstore localstore_testnet
```



#### Transaction contains data, but the ABI signature could not be found

> INFO[2021-06-23T13:23:34+08:00] no chequebook found, deploying new one.      
> Error: chequebook init: validation failed: Transaction contains data, but the ABI signature could not be found: signature 15efd8a7 not found

bee-clef版本和Bee版本不匹配，bee-clef没有使用最新的[v0.4.13](https://github.com/ethersphere/bee-clef/releases/tag/v0.4.13) 直接下载[最新的bee-clef版本](https://github.com/ethersphere/bee-clef/releases)按照这里[安装](https://docs.ethswarm.org/docs/installation/install)并重启bee-clef. 然后再启动Bee即可。



#### Invalid chain id. 

> INFO[2021-06-23T13:46:48+08:00] no chequebook found, deploying new one.      
> Error: chequebook init: Invalid chain id.

这个是因为主网使用了xDai的chain ID 100，但是bee-clef还在使用默认的chain-id 5. Discord给出的回复是你要告诉bee-clef使用xDai的chain ID 100. 

解决办法是直接修改/usr/bin/bee-clef-service中的CHIANID=5改为CHAINID=100. 

```shell
sudo nano /usr/bin/bee-clef-service -> CHAINID=5 to CHAINID=100
```



#### chequebook init: Insufficient funds. 

> INFO[2021-06-23T13:51:28+08:00] no chequebook found, deploying new one.      
> Error: chequebook init: Insufficient funds. The account you tried to send transaction from does not have enough funds. Required 54770624999825000 and got: 0.

账户xDai不足，下边这个连接领取xDai. 

- [Swarm以太坊挖矿BZZ如何领取xDAI](https://www.163.com/dy/article/GD45HLQC05119I7P.html)



#### failed syncing event listener, shutting down node err: 503 Service Unavailable: upstream connect error or disconnect/reset before headers. reset reason: overflow

>  ="2021-06-23T14:15:35+08:00" level=error msg="failed syncing event listener, shutting down node err: 503 Service Unavailable: upstream connect error or disconnect/reset before headers. reset reason: overflow"
> time="2021-06-23T14:15:35+08:00" level=info msg="api shutting down"
> time="2021-06-23T14:15:36+08:00" level=warning msg="peer not reachable when attempting to connect"
> time="2021-06-23T14:15:36+08:00" level=info msg="puller shutting down"
> time="2021-06-23T14:15:36+08:00" level=info msg="pusher shutting down"
> time="2021-06-23T14:15:36+08:00" level=info msg="pull syncer shutting down"
> time="2021-06-23T14:15:36+08:00" level=info msg="kademlia shutting down"
> time="2021-06-23T14:15:36+08:00" level=warning msg="peer not reachable when attempting to connect"
> time="2021-06-23T14:15:36+08:00" level=warning msg="peer not reachable when attempting to connect"
> time="2021-06-23T14:15:36+08:00" level=warning msg="peer not reachable when attempting to connect"
> time="2021-06-23T14:15:36+08:00" level=info msg="kademlia persisting peer metrics"

运行起来以后跑一阵子会自动重启，原因有很多，多半是因为RPC endpoint不稳定导致的。这里使用的是官方推荐的getblock.io, 估计是用的人太多，经常会overflow。

- Run your own xDAI node or utilize a public RPC endpoint such as [getblock.io](https://getblock.io/)

除了自己运行xDai节点，也可以参考[这里](https://www.xdaichain.com/for-developers/developer-resources)，试了一下，使用 swap-endpoint: https://rpc.xdaichain.com/ 或者 https://xdai.poanetwork.dev 会稍微稳定一些，但是还是会有偶尔的不同原因的重启。:( 

还有论坛里推荐的这个，

> swap-endpoint: https://dai.poa.network/

##### Too many concurrent requests

> time="2021-06-23T14:43:59+08:00" level=error msg="failed syncing event listener, shutting down node err: 503 Service Unavailable: {\"jsonrpc\":\"2.0\",\"error\":{\"code\":-32017,\"message\":\"Timeout\",\"data\":\"Nethermind.JsonRpc.Modules.ModuleRentalTimeoutException: Unable to rent an instance of IEthRpcModule. Too many concurrent requests.\\n   at Nethermind.JsonRpc.Modules.BoundedModulePool`1.SlowPath() in /src/Nethermind/Nethermind.JsonRpc/Modules/BoundedModulePool.cs:line 58\\n   at Nethermind.JsonRpc.Modules.RpcModuleProvider.<>c__DisplayClass15_0`1.<<Register>b__0>d.MoveNext() in /src/Nethermind/Nethermind.JsonRpc/Modules/RpcModuleProvider.cs:line 74\\n--- End of stack trace from previous location ---\\n   at Nethermind.JsonRpc.JsonRpcService.ExecuteAsync(JsonRpcRequest request, String methodName, ValueTuple`2 method, JsonRpcContext context) in /src/Nethermind/Nethermind.JsonRpc/JsonRpcService.cs:line 162\\n   at Nethermind.JsonRpc.JsonRpcService.ExecuteRequestAsync(JsonRpcRequest rpcRequest, JsonRpcContext context) in /src/Nethermind/Nethermind.JsonRpc/JsonRpcService.cs:line 115\\n   at Nethermind.JsonRpc.JsonRpcService.SendRequestAsync(JsonRpcRequest rpcRequest, JsonRpcContext context) in /src/Nethermind/Nethermind.JsonRpc/JsonRpcService.cs:line 105\"},\"id\":485}"
> time="2021-06-23T14:43:59+08:00" level=info msg="api shutting down"
> time="2021-06-23T14:43:59+08:00" level=warning msg="peer not reachable when attempting to connect"
> time="2021-06-23T14:44:00+08:00" level=info msg="pusher shutting down"
> time="2021-06-23T14:44:00+08:00" level=info msg="puller shutting down"
> time="2021-06-23T14:44:00+08:00" level=error msg="stream handler: handshake: unable to handshake with peer id 16Uiu2HAkvjcdFia9eSwujbvBpq7PBCMJWWGPqLMfVnW2VVFUKp4f"
> time="2021-06-23T14:44:00+08:00" level=error msg="stream handler: handshake: unable to handshake with peer id 16Uiu2HAm9KGs5VeTn1rR8R5ZztG2F3qNCpY9iiDb7GQ2nB3PUHAF"
> time="2021-06-23T14:44:00+08:00" level=info msg="pull syncer shutting down"
> time="2021-06-23T14:44:00+08:00" level=info msg="kademlia shutting down"
> time="2021-06-23T14:44:00+08:00" level=info msg="kademlia persisting peer metrics"



