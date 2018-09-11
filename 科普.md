从SatoshiDice到EOSDice--上帝的存在不需要证明

介绍随机数之前，我们来看两个随机数生成器的例子;

![](https://i.imgur.com/HOjvh5r.png)

主要缺点是 ：技术难度高 制造难度大 造价昂贵。

![](https://i.imgur.com/HR1btli.jpg)

金融机构加密机，内置自毁装置，不能拆开。



##java.util.Random生成随机数以及原理

    `   Random ran1 = new Random(10);
        System.out.println("使用种子为10的Random对象生成[0,10)内随机整数序列: ");
        for (int i = 0; i < 10; i++) {
            System.out.print(ran1.nextInt(10) + " ");
        }
        System.out.println();
        Random ran2 = new Random(10);
        System.out.println("使用另一个种子为10的Random对象生成[0,10)内随机整数序列: ");
        for (int i = 0; i < 10; i++) {
            System.out.print(ran2.nextInt(10) + " ");
        }`
>两地的结果完全相同。


我们都知道java里面有一个Random()函数，可以随机生成0-1之间的随机数,这种生成的随机数实在给定的seed区间内随机生成数字。而相同种子数的Random对象，相同次数生成的随机数子完全相同。 **那么如果有人拿到了你的seed 那么就可以轻易的推算出你的随机数。**

###为什么random不能生成真随机数？

实际上，seed是一个48比特的种子，就像我们的钱包地址，是一个128位的种子根据bipp44协议生成。 我们只需要对这个种子调用线性同余方程，就能够产生一个随机数。而后续每一次调用Random函数，都会采用上一次产生的随机值来生成新的随机值，所有的取值都均匀分布方程曲线上。
 计算公式是这样样子：
Xi = (Xi-1 * A + C ) mod M (其中ACM是常数 一般取三个质数)

这样的问题是:如果知道了计算机seed泄露，那么随机数将变得极其有规律。

##fomo3d空投攻击

![](https://i.imgur.com/lDLdVz3.jpg)

	   `/**
		 * @dev **generates a random number between 0-99 and checks to see if thats**  生成一个随机数
		*  resulted in an airdrop win
		* @return do we have a winner? 是都有一个获胜者
	*/
		function airdrop()
		private
		view
		returns(bool)
		{
		  uint256 seed = uint256(keccak256(abi.encodePacked(
		  (block.**timestamp**).add
		  (block.**difficulty**).add
		  ((uint256(**keccak256(abi.encodePacked(block.coinbase)))) / (now)**).add
		  (block.**gaslimit**).add
		  ((uint256(keccak256(abi.encodePacked(msg.sender)))) / (now)).add
		  (**block.number**)
		  )));
	
	  if((seed - ((seed / 1000) * 1000)) < airDropTracker_)
	    return(true);
	  else
	    return(false);
	}
	address(fomo3d).call.value(0.1 ether)();
    fomo3d.withdraw();
    selfdestruct(msg.sender);
	
	`

结合airdrop发现，只要我们能够不断制造出符合条件的随机数，降低其随机性，那么我们就能够轻易获取eth,只要我们的收益大于0.1eth(每次空投成本0.1eth). 

###一些可能的些解决方案是
 
1.降低空投奖励，让潜在的攻击者无利可图。

2.生成真随机数。 
我们可以改变随机数的生成方式，采用random.org网站的生成随机数.


##vDice
 vDice之前著名的以太坊的骰子游戏，该网站游戏的玩法如下：
 游戏规则:
Step 1
带着足够的gas发送eth到网站指定的合约地址，你可以选择7合约中的一个。
Step 2
一个随机数被选中，然后和压中的概率对比，如果满足条件断定为获胜者。
Step 3
获胜者将会自动被返还奖励。 失败者将自动返还1gwei的安慰奖。

里面最核心的部分便是生成随机数，随机数的生成方式是有如下的流程： 用户投注，提交json格式参数，调用oraclize获取随机数，然后链上调用callback接收返回结果，随之判断是否获胜，获胜就将获胜者的信息写到区块上，服务器监听事件，给获胜者地址发放奖励。   

我们可以用图表示这个过程
![](https://i.imgur.com/eZCMkyr.jpg)  
		
	    function bet()
	    payable
	    onlyIfNotStopped {
        uint oraclizeFee = OraclizeI(OAR.getAddress()).getPrice("URL", ORACLIZE_GAS_LIMIT + safeGas);
        if (oraclizeFee >= msg.value) throw;
        uint betValue = msg.value - oraclizeFee;
        if ((((betValue * ((10000 - edge) - pwin)) / pwin ) <= (maxWin * getBankroll()) / 10000) && (betValue >= minBet)) {
            LOG_NewBet(msg.sender, betValue);
            bytes32 myid =
            **oraclize_query**(
                "nested",
                "[URL] ['json(https://api.random.org/json-rpc/1/invoke).result.random.data.0', '\\n{\"jsonrpc\":\"2.0\",\"method\":\"generateSignedIntegers\",\"params\":{\"apiKey\":${[decrypt] BMmkEKUccgE4/k+IKb650CLwQ/ACeowJ6a0erQDDqXn1XoTiKRXkw0T3ddPc5l6yUJ8/lEUd3DVG7nwvC/N9jY5NGgeNU4Xvi6HpWjqrevinSkadL3RL0v2w9fr87hd/sURn77W7W8WPoxVH+K8E74+0XHf5vak=},\"n\":1,\"min\":1,\"max\":10000${[identity] \"}\"},\"id\":1${[identity] \"}\"}']",
                ORACLIZE_GAS_LIMIT + safeGas
            );
            bets[myid] = Bet(msg.sender, betValue, 0);
            betsKeys.push(myid);
        }
        else {
            throw;
        }
    }

>这里需要注意的是我们和oraclize通信是采用的椭圆曲线加密算法，使用的时候要采用roacle官方提供的python加密工具将使用公钥对apiKey加密。
>
如果我们要验证是否随机数，是否真随机数关键问题是random.org网站生成随机数否作弊。

打开; https://www.random.org/lists/

我们可以看到官方随机数来源的介绍：

This form allows you to generate random integers. The randomness comes from atmospheric noise, which for many purposes is better than the pseudo-random number algorithms typically used in computer programs.

可以得知，随机数的来源是噪声。 这种噪声生成的随机数要明显优于电脑中生成的伪随机数，已经非常接近真随机数，在经典力学的范畴里的真随机数。如果要生成真正意义上的随机数，就要使用量子学相关知识来解决了，就像开头图片中的额那台量子随机数生成器。 

###链上随机数和链下随机数
那我们也可以深入思考以下，为什么能不能在区块链上获取随机数？
1.区块链是基于共识的系统，**只有在每个交易和区块处理过后，并且每个节点达到相同状态，智能合约才能正常运行，所有事情必须是精确一致**。

2.假设这个区块链有100个节点，那么就会有100条获取数据的请求从每个节点发送到期货交易场所，但是因为这个数据来源于区块链外部，价格是实时波动的，由于网络延迟、节点处理速度等各种原因，每个节点获取的并不是同一时刻的价格，输入到智能合约的价格数据也就不同，因此对应的各节点智能合约输出也会不同，在这种情况下，**整个区块链的信任基础就会崩溃，无法达成共识。**

>如果我们不通过智能合约发出外部数据获取指令，而是由第三方发送一笔区块链交易，在交易中附加需要的数据，交易会将数据嵌入区块，并同步到每个节点，从而保证数据的完全一致。

>为了解决区块链的信任问题，引入了预言机。预言机可以分为单一模型和多重模型。  单一预言机最常用的就是Oraclize。
###Oraclize的TSL可信机制

TLS包含三个基本阶段：1.对等协商支援的密钥算法，2.基于私钥加密交换公钥、基于PKI证书的身份认证，3.基于公钥加密的保密数据传输。在整个传输中，TLS的master key可以分成三个部分：服务器方、受审核方和审核方。

在整个流程中，**互联网数据源作为服务器方，Oraclize作为受审核方，一个专门设计的，部署在亚马逊云上的开源实例作为审核方，每个人都可以通过这个审计方服务对Oraclize过去提供的数据进行审查和检验，以保证数据的完整性和安全性。**

##拓展：
类似的骰子游戏还有中本聪骰子，V2Dice，EOSDice.可以对一些随机样本进行统计，进而生成的随机数，由于样本数的不同，可靠性也不相同。


##为什么赌场能够持续盈利？
在生成随机数之后，结合赌场抽水，玩家便可以得到一个获胜的概率，只要这个赌客一直赌下去，那么这个玩家一定会亏钱亏到零。作为赌场，我们只需要防范具有巨量玩家和赌场对赌。

![](https://i.imgur.com/aI2zuzJ.jpg)

>凯利公式的启示：庄家获胜的关键在于使玩家获得了负的期望值。

现在假定在一个骰子(类vDice)中，在赌场抽水相同的情况下，赌场游戏规则的设置可以分为几种情况：

a.“小博大”：胜率20%，赔率是5，输了全光。b*p - q = 5 *20% -80% = 20%

b.“中博中”：胜率60%，1赔1。b*p - q = 1 *60% - 40% = 20%

c.“大博小”：胜率80%，1赔0.5。b*p - q = 0.5 *80% - 20% = 20%


这个计算收益的公式里，无论是哪种情况，玩家获胜的期望值都是百分之二十，每玩一次玩家大小期望值：100 x 20% - 100 x 80% = -60 。

>只要庄家能够制造使玩家的期望值为负，那么这个赌场就能够持续运行下去。 


	


 