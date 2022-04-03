## 前言

为了成功部署智能合约并完成多节点的交互，构建私链是首要的步骤。网上的资料也比较齐全，但每个人在部署的时候都会遇到一些始料不及的问题。比如我在部署期间，json文件的格式错误，enode复制粘贴不到cmd中导致要手打100+的hash码还打错了直接崩溃，诸如此类等等等等。虽然过程很艰辛，但最终实现跨节点转账，以及node1偷偷后续node2再addpeer实现同步，体现了最长链原理时，这一切艰辛都值得了。

都说接触一样新软件，环境部署是最要命的，果不其然！万世开头难，希望后续基于remix的solidity在线编程的智能合约设计以及部署会比这个环境搭建来的友好些吧！

长风破浪会有时，直挂云帆济沧海！

==1.==
许多博客上说https://geth.ethereum.org/downloads/上下载最新版本的geth，可是在官网上永远都是打开资源失败。因此，我建议大家在csdn上的资源中寻找相关的geth版本并自行下载。
此外，一般会将EthereumWallet和geth配套使用，用图形化界面方便操作。但是目前的EthereumWallet普遍有个bug就是它只认定1.8.23版本的geth，而这在全网（不翻墙）的情况下是很难找到的。因此，本文只安装了geth，并在geth中完成多节点的互联部署。

==2.==
开始安装（其他版本亦可）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210324230600401.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210324230642751.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210324230659283.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
这里如果是希望做一些开发测试的话，建议选上development tools。后续选定安装路径一路next即可。最后我们得到如下exe文件。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210324230852527.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
注意：下载的是不含genesis的，genesis是为了搭建私链而创建的创世区块的配置。

==3.==
关于genesis的配置

```cpp
{
  "config": {
        "chainId": 8,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
  "alloc"      : {"0x0be5196a5503e4051b21e713f245b3ab24bcfeaf": {"balance":"2000000000000000000000000"}},
  "coinbase"   : "0x0be5196a5503e4051b21e713f245b3ab24bcfeaf",
  "difficulty" : "0x0200",
  "extraData"  : "0x74686520726f73657320696e206865722068616e642c2074686520666c61766f7220696e206d696e65",
  "gasLimit"   : "0xffffffff",
  "nonce"      : "0x00000000c000ff59",
  "mixhash"    : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp"  : "0x599DA33A"
}
```
这里主要手动调整的是difficulty和gaslimt，尽量将difficulty调小一点，这样就很快（我一开始调了4w，结果半个钟都没完。。。）。其次是gaslimit，在智能合约部署时，需要消费一定的gas，为了使得智能合约都部署成功，我们这里的gaslimit设置的足够大。

==4.==
由于我已建立好node1和node2两个节点，现在我从node3建立起，大家依葫芦画瓢可以建立起node1和node2。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210324231412880.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
首先通过cmd进入geth的安装路径下。其次，进行node 3的部署和启动。
代码如下：

```cpp
geth --datadir node3 init genesis.json //用genesis初始化node3
geth --datadir "node3"  --rpccorsdomain "*"  --networkid 981204 --rpc console --port 30305  --rpcapi "db,eth,net,web3" --rpcaddr "localhost"  --rpcport 8548  --ipcdisable//初次启动node3

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210324231804599.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
至此，我们成功将node3进入web3界面，可以进行相关的操作。

node1 和 node2的初始化代码只需要将node3的“node3"改掉就好了，但node1和node2的初次启动我认为还是很有必要说一下的。话不多说，先上命令：

```c
geth --datadir "node1"  --rpccorsdomain "*"  --networkid 981204 --rpc console --port 30303  --rpcapi "db,eth,net,web3" --rpcaddr "localhost"  --rpcport 8546 --ipcdisable

geth --datadir "node2"  --rpccorsdomain "*"  --networkid 981204 --rpc console --port 30304  --rpcapi "db,eth,net,web3" --rpcaddr "localhost"  --rpcport 8547 --ipcdisable

geth --datadir "node3"  --rpccorsdomain "*"  --networkid 981204 --rpc console --port 30305  --rpcapi "db,eth,net,web3" --rpcaddr "localhost"  --rpcport 8548  --ipcdisable
```

观察一下，networkid必须相同，因为要用同一个网络连一个私链。而port和rpcport必须不同，因为我是单机模式，开放的port就不能重叠。最后ipcdisable保证进程间不能通信（至于为什么我也不知道hhh）

==5.==
web3相关命令

```go
常用命令
    退出：exit
    查询账户：eth.accounts  （eth.accounts[0]）
    
    创建账户：personal.newAccount()
             输入密码
             确认密码   

    查询账户余额
    eth.getBalance(user1)

    当前区块
    eth.blockNumber

    开始 miner.start()
    结束 miner.stop()

    转账
    eth.sendTransaction({from:user1, to:user2, value:web3.toWei(3, "ether")})

    解锁账号
    personal.unlockAccount(user1, password)

    查看自己节点的信息
    admin.nodeInfo
    
    添加节点
    admin.addPeer(‘enode://1e3c1727cd3bee9f25edeb5dbb3b880e03e41f8eec99566557f3ee0422734a8fcad17c161aa93d61bdbfb28ed152c143c7eb501db58bc63502a104a84b62d742@192.168.1.101:30303’)

    查看添加新节点的信息
    admin.peers


    #将wei转换为ether
    web3.fromWei(21000000000000, 'ether')


    #检查交易池
    txpool.status

    #查看正在交易的数据
    eth.getBlock("pending",true).transactions

    #获取某个区块的信息
    eth.getBlock(294)

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210324232503122.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
这里我们用node1进行了一段时间，可以看到区块数从12个到达了14个，注意，要先建立用户。

接下来，我们通过给node3添加node1的地址，实现两条私链互通。首先通过node1查看自身的enode
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210324232926979.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
接着，在node3中加入node1的enode
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210324233032534.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210324233121170.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
这时我们可以看到在node1和node3中的peers中都出现了对方，这说明两者同步了，根据最长链原理，node3中原来长度为0的链应该呗node1中的长度为14的链取代，我们去看看。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210324233246334.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
针不戳，果然同步过来了。

==6.==
这时，我们尝试让node1中的account[0]给node3中的account[0]转1个以太币。我们首先可以验证一下node3中的account[0]是个穷光蛋。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210324233448502.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
我们再看看node1的钱，因为之前已经挖了14个区块，所以有一定的积蓄。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210324233700288.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
Wei是eth的一个小单位，实际上node1的account[0]有69个以太币，中间出了点小错误忽略掉hhh。好，我们现在让node1给node3转10个以太币！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210324234039177.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
每次交易前都要解锁交易发起人的账户，就是相当于要重新输入密码激活。这里的from是账户发起人，而to的地址则是node3的account[0]的地址，这里我们转10个以太币。转完帐后，需要挖进行交易的确认，我们接下里让node1进行确认。在之前，我们可以验证一下node3现在账上的钱。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210324234238151.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
真可惜，还是0。我们的node3快等不及了。我们还是赶紧让node1工作吧。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210324234347358.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
node1二话不说，得到了15、16、17、18号。我们看看node3中的情况。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210324234450911.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
果然，node3中的块号也到了18，而且它的账上也有了10个以太币。node3笑出了猪叫声哈哈哈哈哈啊哈哈哈哈。

==坑点：==
巨坑：cmd在复制enode或者账户地址时，一定要先按一下**右键**！！！！一定！！一定！！不然它是复制不成功的，enode的100多位就要手打！

==总结：==
环境的部署实属不易。自己写环境部署的blog难免会漏掉一些细枝末节。这样我也可以理解别人写环境部署时的疏漏了。为了方便更多的人成功部署geth，希望我的文章会有帮助。如有疑问，欢迎在评论区下提问共同探讨！

附：[我的geth安装包](https://download.csdn.net/download/weixin_40986490/16084306)