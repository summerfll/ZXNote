# fabric1.0多机部署e2e_cli示例（三虚拟机）

## 一、准备工作

### 1.1环境准备：

首先要安装好go、docker、docker compose等。具体请参考https://www.cnblogs.com/studyzy/p/7237287.html

我们要部署的是2Peer+1Orderer的架构。为此我们需要准备3台机器。我们可以开3台虚拟机，而且安装相同的系统，我们用的是Ubuntu 16.04版。为了方便，我建议先启用1台虚拟机，在其中把准备工作做完，然后基于这台虚拟机，再复制出2台即可。这里是我用到3台Server的主机名（角色）和IP：

| 角色                   | IP              |
| ---------------------- | --------------- |
| orderer.example.com    | 192.168.242.218 |
| peer0.org1.example.com | 192.168.242.216 |
| peer0.org2.example.com | 192.168.242.217 |

## 二.docker-compose 配置文件准备

### 2.1单机运行4+1 Fabric实例，确保脚本和镜像正常：

然后需要确保每台机子上e2e__cli的单机环境能跑起来。 
进入e2e_cli文件夹，运行

```
./network_setup.sh up
```

这个命令可以在本机启动4+1的Fabric网络并且进行测试，跑Example02这个ChainCode。我们可以看到每一步的操作，最后确认单机没有问题。确认我们的镜像和脚本都是正常的，我们就可以关闭Fabric网络，继续我们的多机Fabric网络设置工作。关闭Fabric命令：

```
./network_setup.sh down
```

**注意：**测试完单机一定要关闭，否则docker容器等会有残留，影响下一步工作。 

### 2.2生成公私钥、证书、创世区块等

公私钥和证书是用于Server和Server之间的安全通信，另外要创建Channel并让其他节点加入Channel就需要创世区块，这些必备文件都可以一个命令生成，官方已经给出了脚本：

```
./generateArtifacts.sh mychannel
```

运行这个命令后，系统会创建channel-artifacts文件夹，里面包含了mychannel这个通道相关的文件，另外还有一个crypto-config文件夹，里面包含了各个节点的公私钥和证书的信息。其中channel-artifacts文件夹为：

channel-artifacts/ 
├── channel.tx 
├── genesis.block 
├── Org1MSPanchors.tx 
└── Org2MSPanchors.tx

**注意：**三个主机中的crypto-config和channel-artifacts文件夹中的内容必须一致

### 2.3设置peer节点的docker-compose文件

### 2.4设置orderer节点的docker-compose文件

### 2.5分发配置文件

2.3-2.5参考https://www.cnblogs.com/studyzy/p/7237287.html进行配置

## 三.启动Fabric

现在所有文件都已经准备完毕，我们可以启动我们的Fabric网络了。

### 3.1启动orderer

让我们首先来启动orderer节点，在orderer服务器上运行：

```
docker-compose -f docker-compose-orderer.yaml up –d
```

运行完毕后我们可以使用docker ps看到运行了一个名字为orderer.example.com的节点。

### 3.2启动peer

然后我们切换到peer0.org1.example.com服务器，启动本服务器的peer节点和cli，命令为：

```
docker-compose -f docker-compose-peer.yaml up –d
```

运行完毕后我们使用docker ps应该可以看到2个正在运行的容器。

然后我们切换到peer0.org2.example.com服务器，启动本服务器的peer节点和cli，命令为：

```
docker-compose -f docker-compose-peer.yaml up –d
```

现在我们整个Fabric2+1服务器网络已经成型，接下来是创建channel和运行ChainCode。

### 3.3创建Channel

由于官方提供的一键部署命令./scripts/script.sh mychannel问题较多，所以直接采用手动部署

我们切换到peer0.org1.example.com服务器上，使用该服务器上的cli来运行创建Channel操作。首先进入cli容器：

```
docker exec -it cli bash
```

进入容器后我们可以看到命令提示变为：

[root@b41e67d40583:/opt/gopath/src/github.com/hyperledger/fabric/peer](mailto:root@b41e67d40583:/opt/gopath/src/github.com/hyperledger/fabric/peer)#

输入命令创建channel：

~~~c
ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
peer channel create -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/channel.tx --tls true --cafile $ORDERER_CA
~~~

### 3.4节点加入channel

当前我们在peer0.org1.example.com服务器上，直接使用命令：

~~~
peer channel join -b mychannel.block
~~~

我们切换到peer0.org2.example.com服务器上，进入cli容器，使用命令加入通道：

~~~c
CORE_PEER_LOCALMSPID="Org2MSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
CORE_PEER_ADDRESS=peer0.org2.example.com:7051
peer channel join -b mychannel.block
~~~

### 3.5更新锚节点

不同组织间只能通过锚节点（anchor peers）进行通信，peer1节点和peer2节点为不同的组织，需要更新锚节点才能通信。每个组织含有一个锚节点，org1组织的锚节点为peer0.org1.example.com，org2组织的锚节点为peer0.org2.example.com,这两个组织下可以挂载多个节点，本项目中两个节点属于不同组织，所以两个节点都需要更新锚节点。

在peer0.org1.example.com服务器上，使用命令更新锚节点：

~~~c
CORE_PEER_LOCALMSPID="Org1MSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
peer channel update -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/Org1MSPanchors.tx --tls true --cafile $ORDERER_CA
~~~

切换到peer0.org2.example.com服务器上，进入cli容器，使用命令更新锚节点：

~~~c
CORE_PEER_LOCALMSPID="Org2MSP"
ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
CORE_PEER_ADDRESS=peer0.org2.example.com:7051
peer channel update -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/Org2MSPanchors.tx --tls true --cafile $ORDERER_CA
~~~

### 3.6 节点链码安装

链码的安装需要在各个相关的Peer上进行，对于我们现在这种Fabric网络，如果2个Peer都想对Example02进行操作，那么就需要安装4次。

仍然是保持在CLI的命令行下，我们先切换到peer0.org1这个节点：

~~~
CORE_PEER_LOCALMSPID="Org1MSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
CORE_PEER_ADDRESS=peer0.org1.example.com:7051
peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02
~~~

安装的过程其实就是对CLI中指定的代码进行编译打包，并把打包好的文件发送到Peer，等待接下来的实例化。

使用命令docker ps可看到此时peer0.org1这个节点上含有3个容器：

![2](F:\笔记\pic\2.png)

切换到peer0.org2.example.com服务器上，进入cli容器，使用命令安装链码：

~~~c
CORE_PEER_LOCALMSPID="Org2MSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
CORE_PEER_ADDRESS=peer0.org2.example.com:7051
peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02
~~~

运行docker ps命令，可以看到本来是2个容器，现在已经变成了3个容器，因为ChainCode会创建一个容器，说明fabric网络多机部署成功

![3](F:\笔记\pic\3.png)

### 3.7实例化链码

实例化链码主要是在Peer所在的机器上对前面安装好的链上代码进行包装，生成对应Channel的Docker镜像和Docker容器。并且在实例化时我们可以指定背书策略。我们运行以下命令完成实例化：

~~~c
peer chaincode instantiate -o orderer.example.com:7050 --tls true --cafile $ORDERER_CA -C mychannel -n mycc -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "OR      ('Org1MSP.member','Org2MSP.member')"
~~~

### 3.7在一个Peer上查询并发起交易

现在链上代码的实例也有了，并且在实例化的时候指定了a账户100，b账户200，我们可以试着调用ChainCode的查询代码，验证一下，在cli容器内执行：

~~~c
peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'
~~~

返回结果：Query Result: 100

接下来我们可以试着把a账户的10元转给b。对应的代码：

```c
peer chaincode invoke -o orderer.example.com:7050  --tls true --cafile $ORDERER_CA -C mychannel -n mycc -c '{"Args":["invoke","a","b","10"]}'
```

### 3.8在另一个节点上查询交易

由于mycc已经在前面org1的时候实例化了，也就是说对应的区块已经生成了，所以在org2不能再次初始化。我们直接运行查询命令：

```c
peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'
```

返回结果：Query Result: 90