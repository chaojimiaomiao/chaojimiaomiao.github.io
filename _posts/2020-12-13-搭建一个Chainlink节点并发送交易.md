---
layout: post
title:  "搭建一个Chainlink节点并发送交易（上）"
modified:   2020-12-13
tags: [区块链, 技术, link]
comments: true
---

### 前言
> **Chainlink**是一个去中心化的预言机网络，它可以让区块链中的智能合约安全地访问外部世界的数据。在这个教程中，你将学习到如何最快自建Chainlink的节点，并发送一笔交易请求。

* 目录
{:toc}


#### 什么是Chainlink节点
Chainlink 基于ETH 链，但并不是ERC20，而是 ERC677 合约。
它是 ERC20 合约的一个扩展，兼容 ERC20 协议标准，可以在转账时携带数据，并触发接收合约的业务逻辑，扩大了智能合约的应用场景。
目前，Chainlink已经能实现的包括但不限于：
1. 分布式价格推送
2. 随机函数，链上随机数
3. 外部适配器连接链外资源
4. 其他预言机网络服务
丰富的[节点与合约市场](https://market.link/)使得Chainlink成为了各大DEX交易所接入预言机的首选。

#### 搭建Chainlink节点有什么好处
成为Chainlink认证的节点提供商，将会获得节点运营奖励。

#### 怎么搭建
我们先从Chainlink的测试网kovan上开始，你需要：
1. 一台linux云服务器，以ubuntu为例 
2. 一个postgresql数据库
3. 一个eth节点

<!--more-->

本教程以[google cloud](https://cloud.google.com/)为例，你也可以选用国内的阿里云，两者都支持postgresql数据库。笔者一开始选择的是[vultr云服务器](http://vultr.com/)，后台并没有自带数据库，需要自己搭建，比较复杂就不赘述。
由于国内用户对GCloud并不熟悉，所以说得详细一些。

##### 第一步 配置GCloud  

登陆 https://cloud.google.com/ 注册一下，绑定一张visa信用卡。新用户会获得300美元的奖励金，用完以前是免费的。  
进入控制台：  
![进入控制台](/img/link/1.png)
创建虚拟机实例：  
![创建虚拟机实例](/img/link/2.png)

设置服务器名称，默认美区，选择标准型双核8GB内存：  
![设置服务器名称](/img/link/4.png)

选择ubuntu18.04系统，启动磁盘为SSD 10G:  
![选择启动磁盘](/img/link/5.png)

勾选允许HTTP流量，点击创建：  
![创建虚机](/img/link/6.png)

等待5分钟以后，创建成功！  
![创建成功](/img/link/7.png)  

接下来先配置虚拟机网络，然后再配置数据库。因为数据库会用到前面的网络实例。  
配置虚机网络，如图所示：  
![配置网络](/img/link/10.png)

选择库，在跳出的搜索框里输入 **`networking`**，选择列表下方的第一个并应用。  
![networkingapi](/img/link/10_1.png)  

创建SQL实例：  
![创建SQL](/img/link/9.png)

选择PostgreSQL 数据库。  
PostgreSQL是目前最强大的开源对象关系型数据库，Chainlink用的是这个数据库。`（注：ETH 使用的是开源NOSQL数据库，leveldb）`  
![选择数据库](/img/link/11.png)
创建 PostgreSQL实例，如图所示：  
![创建 PostgreSQL实例](/img/link/12.png)

记得在**连接**一栏把专用IP也勾上。  
<br>
创建的过程需要等待5分钟左右，喝杯咖啡休息一下：）

成功以后添加用户：  
![添加用户](/img/link/15.png)
系统默认创建了名为postgres的管理员，你也可以在更多(三个点)里面修改默认密码，并在后面使用这个用户。  
![用户实例](/img/link/16.png)

同时，你也可以自己再创建一个数据库实例，比如命名为 `chainlink-kovan-db`:  
![数据库实例](/img/link/17.png)

大功告成！  
![配置完成](/img/link/17_1.png)

至此，服务器部分已经配置完了。  

* * *
##### 第二步 运行Chainlink节点

此处参考 [官方教程：运行一个节点](https://docs.chain.link/docs/running-a-chainlink-node)，是英文版的。
首先，我们需要登陆这台服务器。
![登陆服务器](/img/link/21_0.png)
google提供了5种办法登陆，你可以图省事直接点击**`在浏览器中打开`**，并使用内置的客户端直连。
但是由于我们之后需要在本地的浏览器里直接输入`localhost`访问节点配置页面，所以我们选择用 gcloud去登陆它。详细文档请参考： [gcloud文档](https://cloud.google.com/compute/docs/gcloud-compute) 。  

安装 Cloud SDK，windows mac linux都有。
![安装SDK](/img/link/21.png)

安装过程中会弹出浏览器，让你登陆google 账号。
安装完成后，复制gcloud 命令
![gcloud命令](/img/link/21_1.png)

```
gcloud compute ssh --zone "us-central1-a" "chainlink-kovan-icy" --project "molten-snowfall-298315"
```
由于我们需要通过ssh tunnel连上6688这个端口，所以我们在这个命令后加上一点东西
```
gcloud compute --project "molten-snowfall-298315" ssh --zone "us-central1-a" "chainlink-kovan-icy" -- -L 6688:localhost:6688
```
![ssh tunnel](/img/link/21_2.png)

连上以后敲以下命令：
* 安装docker，将当前用户添加至docker用户组。
```
curl -sSL https://get.docker.com/ | sh
sudo usermod -aG docker $USER
```
* 创建kovan目录并配置环境变量
```
mkdir ~/.chainlink-kovan
echo "ROOT=/chainlink
LOG_LEVEL=debug
ETH_CHAIN_ID=42
MIN_OUTGOING_CONFIRMATIONS=2
LINK_CONTRACT_ADDRESS=0xa36085F69e2889c224210F603D836748e7dC0088
CHAINLINK_TLS_PORT=0
SECURE_COOKIES=false
GAS_UPDATER_ENABLED=true
ALLOW_ORIGINS=*" > ~/.chainlink-kovan/.env
```

* 配置以太坊节点

```
echo "ETH_URL=wss://kovan.infura.io/ws/v3/0ce89c2fce5443f58de18b0c8e1b1f6d" >> ~/.chainlink-kovan/.env
```

* 配置外部数据库
```
echo "DATABASE_URL=postgresql://chainlink-db-user:icy@10.13.128.2:5432/postgres" >> ~/.chainlink-kovan/.env
```

* 启动节点
```
cd ~/.chainlink-kovan && sudo docker run -p 6688:6688 -v ~/.chainlink-kovan:/chainlink -it --env-file=.env smartcontract/chainlink local n
```
![下载chainlink](/img/link/28.png)



#### 
#### 总结：
1. gcloud auth login
2. ssh tunnel
3. start node command


* * *


#### 发送交易


为什么要使用gcloud auth login，因为可以看到UI页面。

&nbsp;&nbsp;以太坊的全部区块数据已经高达2.8T，三种模式
*   –syncmode  "fast"         _Enable fast syncing through state downloads_
*   –syncmode  "light"        _Enable light client mode_
* –syncmode  "full"


[\<三种数据同步模式\>](https://www.cnblogs.com/bizzan/p/11341713.html)
https://www.cnblogs.com/bizzan/p/11341713.html
可以选择自己搭建，也可以使用外部提供的。

chainlink是基于以太坊开发的
发送交易