# BSCnode
币安智能链节点搭建,节点搭建教程,全节点教程BSC节点

## 一、服务器配置要求

 -  [全节点建议配置](https://docs.binance.org/smart-chain/developer/fullnode.html)
```javascript
系统：Mac & Linux
CPU：16核
内存：64 GB 内存
带宽：50M以上
硬盘：大于2T固态SSD可用空间数据盘
```
 - 本次搭建使用配置
```javascript
系统：Ubuntu
CPU：32核心64线程
内存：128GB 内存
带宽：1000M上下对等
硬盘：固态8T
```
---
 - BSC官方文档：[https://docs.bnbchain.org/docs/validator/fullnode/](https://docs.bnbchain.org/docs/validator/fullnode/)
 - BSC快照github：[https://github.com/binance-chain/bsc-snapshots](https://github.com/binance-chain/bsc-snapshots)
 - BSC github地址：[https://github.com/binance-chain/bsc/releases](https://github.com/binance-chain/bsc/releases)

---

## 二、系统环境准备

1、ubuntu一键更新：
```powershell
apt-get -y update wget git screen gcc automake autoconf libtool make unzip aria2 vim
```
2、安装Goland（这里不要用一键安装，要手动安装Goland，不然版本太低）安装 Go 主要是为了去编译 go-ethereum 源码

 - 下载最新goland安装包，去这里查看Linux最新版本下载链接，右键复制下载链接。https://golang.google.cn/dl/
 

```powershell
# 下载最新安装包（下边链接是发文时最新版本：go1.20.2）
wget https://golang.google.cn/dl/go1.20.2.linux-amd64.tar.gz

#将下载的二进制包解压至 /usr/local目录
tar -C /usr/local -xzf go1.20.2.linux-amd64.tar.gz
```

 - 将 /usr/local/go/bin 目录添加至 PATH
   环境变量，编辑/etc/profile文件，把下边命令添加到profile文件末尾保存即可。

```powershell
vim /etc/profile
#把下列一行写到最后边然后 :wq 保存退出
export PATH=$PATH:/usr/local/go/bin
#然后使命令生效
source /etc/profile
```
使用go version确认安装正确

如下显示则安装正确。

```powershell
[root@localhost ~]# go version
go version go1.20.2 linux/amd64
```

## 三、节点安装部署
在根目录创建`jiedian`文件夹用来存放节点程序，并在同时在jiedian里边创建一个`kuaizhao`文件夹，下载的快照数据

> 1.务必使用固态硬盘并且可使用空间大于2T
> 2.如果固态不够使用，可以把快照压缩包下载到机械硬盘里边，解压的时候解压到固态硬盘

 - **安装BSC版本的geth**

```powershell
#进入根目录
cd /
 
#创建jiedian及kuaizhao文件夹
mkdir -p jiedian/kuaizhao
 
#进入jiedian文件夹
cd /jiedian
 
#git部署bsc节点程序
git clone https://github.com/binance-chain/bsc
 
#进入下载好的bsc程序文件夹
cd bsc/
 
#执行编译安装geth
make geth
```
将 /jiedian/bsc/build/bin 目录添加至 PATH 环境变量，编辑/etc/profile文件，把下边命令添加到profile文件末尾保存即可。
```powershell
vim /etc/profile
#把下列一行写到最后边然后 :wq 保存退出
export PATH=$PATH:/jiedian/bsc/build/bin
#然后使命令生效
source /etc/profile
 
#使用geth version确认安装正确
```

 - **配置创世块**

```powershell
wget   $(curl -s https://api.github.com/repos/bnb-chain/bsc/releases/latest |grep browser_ |grep mainnet |cut -d\" -f4)
unzip mainnet.zip
geth --datadir node init genesis.json
```

 - **下载BSC快照**

创建一个用来下载快照的screen窗口

```powershell
screen -S xiazai
```

> *注意1：使用screen窗口期间可以退出或者关闭命令行对话框。
> *注意2：退出当前窗口时用ctrl+ad（顺序按a和d字母即可），绝对不要用exit或ctrl+d退出会话。
> *注意2：退出会话后，可以用screen -x xiazai重新连接到会话。这样可以保持在shell下运行，网络中断不会影响。

开始下载快照

> 这里使用的是BNB48提供的快照
> https://github.com/48Club/bsc-snapshots

```powershell
aria2c -s14 -x14 -k100M 最新下载地址 -o geth.tar.lz4
```
下载完成后解压到当前文件夹
```powershell
lz4 -cd geth.tar.lz4 | tar xf -
```
移动 下载的快照数据文件夹chaindata到./jiedian/bsc/node/geth/ 文件夹下
```powershell
#首先删除我们自己生成的chaindata文件夹
rm -rf /jiedian/bsc/node/geth/chaindata
#然后移动下载好的过去
mv /jiedian/kuaizhao/geth/chaindata /jiedian/bsc/node/geth
```
移动完毕以后退出screen的xiazai窗口，并创建bsc窗口并开始运行节点。

> 移动完毕以后退出screen的xiazai窗口，并创建bsc窗口并开始运行节点。
>退出当前窗口时用ctrl+ad（顺序按a和d字母即可），绝对不要用exit或ctrl+d退出会话

```powershell
#退出xiazai窗口
ctrl+ad
```
启动节点之前要先修改配置文件参数，修改BSC主网配置文件

 - **修改BSC主网配置文件**

```powershell
#编辑配置文件
vim /jiedian/bsc/config.toml
```
参数说明：
> TrieTimeout：这意味着geth将不会将状态持久化到数据库中，直到达到这个时间阈值，如果节点已经被强制关闭，它将从最后一个状态开始同步，这可能需要很长时间,可设置为：TrieTimeout
> = 200000000000
> 注意：当TrieTimeout值设置的越大，系统崩溃后，节点恢复的时间越长
> HTTPHost: HTTP-RPC服务连接白名单，此参数的值默认为 "localhost"，仅允许本地可访问，**如果需要外网访问节点**请设置为："0.0.0.0"
> HTTPVirtualHosts：HTTP-RPC服务监听接口,此参数的值默认为["localhost"],可设置为：HTTPVirtualHosts = ["*"]
> HTTPPort：http协议rpc端口
> WSPort：websocket协议rpc端口
> WSHost：websocket服务连接白名单，此参数的值默认为 "localhost"，仅允许本地可访问，可设置为："0.0.0.0"
> WSOrigins：websocket服务监听接口,可设置为：WSOrigins = ["*"]

## 四、启动BSC智能全节点

```powershell
#创建bsc节点启动窗口
screen -S bsc
#进入到bsc文件夹
cd /jiedian/bsc/
```
**启动命令（ --cache 86016 这里这个参数一定按照你的服务器内存大小修改，我这里是128G内存，所以我给了80G）**
```powershell
geth --config ./config.toml --datadir ./node --txlookuplimit 0 --cache 86016 --rpc.allow-unprotected-txs --syncmode full --tries-verify-mode none --diffblock 5000 --rpc.txfeecap 0 --rpc.gascap 0 --http --http.corsdomain "*" --pruneancient=true --diffsync=true
```

**然后按ctrl+ad回到主会话即可**

参数说明：

> --config：指定BSC节点配置文件
> --datadir：指定BSC节点数据库和密钥存储库的数据目录(默认即可)
> --cache：设置最大分配给内部缓存的内存，默认:1024（设置越大，每次同步的数据越多，消耗的内存也越大）
> --rpc.allow-unprotected-txs：允许通过RPC提交不受保护的（非 EIP155 签名）交易
> --txlookuplimit 0 : 禁用删除事务索引
> --diffsync：启用差异同步协议来帮助节点更快地同步
> --rpc.txfeecap：无上限gas费用
> --rpc.gascap：无交易费用上限

## 五、节点状态监听

```powershell
geth attach http://127.0.0.1:8545
#这里的端口如果修改配置文件了，就填写配置文件的端口即可
```

```powershell
> eth.syncing	#查看当前块情况，结果为false为同步完成
> net.peerCount	#查看当前连接同步节点数量
> eth.blockNumber #当前同步区块高度
```
说明：
> currentBlock: 14290861, #当前同步到块高度
> highestBlock: 14297354,   #主网当前高度
> knownStates:297473485,   
> pulledStates: 297473485,   
> startingBlock: 14270385 

退出请按 ctrl+d 回到主会话。

 - **停止节点**
 
 打开bsc窗口

```powershell
screen -x bsc
#然后按 ctrl+c 即可
```
 
 **引导节点将在不久的将来得到增强。到目前为止，发现 http 服务将提供一些稳定的公共 p2p 对等点进行同步。请访问[https://api.binance.org/v1/discovery/peers](https://api.binance.org/v1/discovery/peers)获取动态对等信息。您可以将对等信息附加到StaticNodesconfig.toml 中以增强完整节点的网络。为了避免拥挤的网络，发现服务会不时更改节点信息，如果全节点连接的节点太少，请尝试获取新节点。**

## 六、需要购节点、服务器、脚本的联系我
https://t.me/dingsi
vx：dingsii
![image](https://user-images.githubusercontent.com/68468901/228177211-bb4c8772-7e58-49c2-a28b-4af0730b8302.png)

