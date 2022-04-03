## 前言
本文讲述了详细的基于win10+geth的智能合约编译+部署+调用的全流程，如有疑问请在评论区多多交流！

==编译过程：==
首先打开[remix-solidity-ide-中文版](http://remix.hubwiz.com/)，这里可以进行智能合约的编写及编译，如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210326114829112.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
这里我们新建一个hello.sol的简单智能合约，其功能就是将输入的字符串直接输出，代码如下：

```c
pragma solidity ^0.5.1;

contract Hello{
    function echo(string memory text) public pure returns(string memory) {
        return text;
    }
}
```
这里注意的是，第一行的版本号是必须写的，pragma是编译器识别的一个关键字。而且，在编译的时候，也要选择与版本号对应的编译器，如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210326115120632.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
点击开始编译并编译成功后，下方会出现一个绿色的框框里面写着你的合约名。此外，在该界面中也附带测试功能，我们不妨试试：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021032611550673.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
如上图所示，首先点击红色的部署。然后在已部署合约中点击echo函数，并写入第一个字符串（记住带""），随后便会得到对应的输出。

==部署过程：==
虽然在remix上可以进行不同节点的智能合约测试，但是还是得在geth上真正部署起来才比较真实。

这里，我们需要把remix中编译的abi以及web3deploy复制到一个txt文件上，以便后续的使用。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210326115927146.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
如上图，abi在编译图的右下角，而web3deploy需要打开右下角的”详情“
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210326120211130.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
打开后，看到WEB3DEPLOY，复制下来就好了。

这里需要注意的是，abi还有一些后续处理，其需要压缩转义，这时候我们要借助[JSON压缩转义网站](http://www.bejson.com/zhuanyi/)。首先，我们把之前合约的abi内容粘贴到该网页上，如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021032612044941.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
点击压缩并转义后我们得到如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210326135213923.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
这里的abi应该是在EVM（以太坊虚拟机）中合约的接口，后续的调用中必须用到。下面我们分别给出转义后的abi以及原合约的WEB3DEPLOY

压缩转义abi:

```c
[	{		\"constant\":true,		\"inputs\":[			{				\"name\":\"text\",				\"type\":\"string\"			}		],		\"name\":\"echo\",		\"outputs\":[			{				\"name\":\"\",				\"type\":\"string\"			}		],		\"payable\":false,		\"stateMutability\":\"pure\",		\"type\":\"function\"	}]
```

wen3deploy：

```c
var helloContract = web3.eth.contract([{"constant":true,"inputs":[{"name":"text","type":"string"}],"name":"echo","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"pure","type":"function"}]);
var hello = helloContract.new(
   {
     from: web3.eth.accounts[0], 
     data: '0x608060405234801561001057600080fd5b506101b7806100206000396000f3fe60806040526004361061003b576000357c010000000000000000000000000000000000000000000000000000000090048063f15da72914610040575b600080fd5b34801561004c57600080fd5b506101066004803603602081101561006357600080fd5b810190808035906020019064010000000081111561008057600080fd5b82018360208201111561009257600080fd5b803590602001918460018302840111640100000000831117156100b457600080fd5b91908080601f016020809104026020016040519081016040528093929190818152602001838380828437600081840152601f19601f820116905080830192505050505050509192919290505050610181565b6040518080602001828103825283818151815260200191508051906020019080838360005b8381101561014657808201518184015260208101905061012b565b50505050905090810190601f1680156101735780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b606081905091905056fea165627a7a723058204b93968b3a970c6c7974216c1add930ee57348fe6eeb4b8a8cd31276563267e70029', 
     gas: '4700000'
   }, function (e, contract){
    console.log(e, contract);
    if (typeof contract.address !== 'undefined') {
         console.log('Contract mined! address: ' + contract.address + ' transactionHash: ' + contract.transactionHash);
    }
 })
```

接下来我们可以进入geth客户端进行部署了，我们首先打开node1节点，并将web3deploy内容复制上去。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210326135919518.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
如上图，刚开始第一次运行的时候被拒绝了，是因为我们没有解锁账户。因为在智能合约发布的时候，需要严格指明发布人的信息。因此，我们unlock以下eth.accounts[0]后就可以发布合约了，这时我们看看交易池里的信息。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210326140109935.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
此时可以发现，合约已经被放进交易池里了

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210326140238943.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
我们直接输入合约名称:hello，可以看到其abi信息，以及交易的hash。但由于未曾记录到区块中，所以暂时还没有addr。我们可以通过查询transactionhash的方式，查看一下这个合约：

```c
eth.getTransaction("对应的transactionhash")
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210326140532991.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
如上图，可以发现的是，该合约的blocknumber为null，所以它是未记录上链的。此时，也是无法调用其中的echo函数的。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210326140705436.png)
因此，我们使其上链。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210326141442519.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
我们看到contract mined说明上链成功！这时候务必记下它的地址，后续会用到。这时候我们可以再看看transactionhash的信息：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210326142034375.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
可以发现，它已经被记录在第48个区块上。

==调用过程：==
接下来，我们输入abi：

```c
var abi = JSON.parse('压缩转义的abi')
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210326142235790.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
如上图，abi已经存进去了，接下来，我们存入之前保存的合约addr.
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021032614270887.png)
最后，调用abi和addr结合的命令召唤出深藏在链上的hello合约：

```c
hellocontract = eth.contract(abi).at(helloaddr)
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210326142825278.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk4NjQ5MA==,size_16,color_FFFFFF,t_70)
nice，我们发现我们找到了hello函数，其中里面包含echo方法。我们尝试用一下echo方法：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210326142925271.png)
至此为止，一个合约的编译、部署、调用过程全部结束。

==总结：==
1.可以调用node2节点，同时更新链后，调用上述部署的合约（方法同上）做到一合约多节点调用
2.没有转账的合约不算真正的合约，下回我们会讲解一下有关转账功能的合约的编写。
3.产生的dag会大大压榨c盘的空间，记得及时清理ethash中的垃圾！
4.一个合约的abi和addr一定要记住！不然就找不到合约了。