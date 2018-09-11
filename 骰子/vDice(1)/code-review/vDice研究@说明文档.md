
#vDice：
    1.合约文件
    2.合约结构
    3.合约解读

##1.游戏按照赔率不同7个不同的合约文件 
> 每一个合约相互独立，唯一的区别是玩家获胜的概率不同。

 uint constant pwin 玩家获胜的可能性 总可能性10000。
新合约
--5% 0x4e646A576917a6A47D5B0896c3E207693870869D

--10% 0xE8A51bE86ad96447D45DDEdDc55013f25157688c

--25% 0xe642B6f79041C60d8447679B3a499F18D8B03b81

--40% 0x1E2Fbe6BE9Eb39Fc894d38be976111F332172d83

--50% 0xdd98b423dc61A756e1070De151b1485425505954

--75% 0x49fDdEae0b521dAb8d0c4b77E7161094F971320D

--90% 0x7DA90089A73edD14c75B0C827cb54f4248D47eCc

旧合约
--5％ 0x869DB2680600fbf9913DF90eF0fD057935c248b6

--10％ 0x7da247644C9f5BC0Ea4E2C789eBF69569cDd3f1a

--10％ 0x83Cfd2d33bb9ED552A1E4B6AF1bc3ea73BdEE4f0

--10％ 0xc76e765aa9D5cc462C17F8860fEd3cdaB37ADFe6

--10％ 0x78F6A1d4b94d2e6c4cE3c5CC858b9dfB6e98B50f

--10％ 0x4B92a948Ced9D457b4655aBf62ed930a090f8566

--10％ 0xEB927A56588D17d551E72ef0a2bE460A4dDB7060

新的合约增加了赔率的类型， 对于一般玩家，10%的概率基本上等于有去无回。
新合约可以让一些文件的投资人也参与进来。

## 2.合约结构
+-- ORACLIZE_API (预言机)

| +-- OraclizeI.sol(查询)

| +-- OraclizeAddrResolverI.sol(地址解析)

| +-- usingOraclize.sol(api) 

+-- Dice.sol

| +--SECTION I MODIFIERS AND HELPER FUNCTIONS （辅助方法）

	| +-- CONSTANT HELPER FUNCTIONS 
	
		| +--getBankroll()
	
		| +--getMinInvestment()
	
	    | +--getStatus()
	
		| +--getBet(uint id)
	
		| +--getMinBetAmount()
	
	    | +--getMaxBetAmount()
	
		| +--getLossesShare(address currentInvestor)
	
	    | +--getProfitShare(address currentInvestor)
	
		| +--getBalance(address currentInvestor)
	
	    | +--searchSmallestInvestor()
	
		| +--changeOraclizeProofType(byte _proofType)
	
	    | +--changeOraclizeConfig(bytes32 _config)
	
	| +-- PRIVATE HELPERS FUNCTION
	
		| +--safeSend(address addr, uint value)
	
	    | +--addInvestorAtID(uint id)
	
		| +--profitDistribution()

| +-- SECTION II: BET & BET PROCESSING

	| +-- function()
	
	| +-- bet()
	
	| +-- __callback(bytes32 myid, string result, bytes proof)
	
	| +-- isWinningBet(Bet thisBet, uint numberRolled)
	
	| +-- isLosingBet(Bet thisBet, uint numberRolled)

| +-- SECTION III: INVEST & DIVEST

	| +-- increaseInvestment()
	
	| +-- newInvestor()
	
	| +-- divest()
	
	| +-- divest(address currentInvestor)
	
	| +-- forceDivestOfAllInvestors()
	
	| +-- forceDivestOfOneInvestor(address currentInvestor)

| +--SECTION IV: CONTRACT MANAGEMENT

	| +-- stopContract()
	
	| +-- resumeContract()
	
	| +-- changeHouseAddress(address newHouse)
	
	| +-- changeOwnerAddress(address newOwner)
	
	| +-- changeGasLimitOfSafeSend(uint newGasLimit)

| +-- SECTION V: EMERGENCY WITHDRAWAL

	| +-- voteEmergencyWithdrawal(bool vote)
	
	| +--proposeEmergencyWithdrawal(address withdrawalAddress)
	
	| +-- executeEmergencyWithdrawal()


+-- 隐藏合约，测试网

  	| +-- OraclizeAddrResolver.sol  mainnet

	| +--OraclizeAddrResolver.sol  ropsten testnet
	
	| +-- OraclizeAddrResolver.sol  browser-solidity

游戏的核心是随机数生成。


这里截取赢赔率90%()的合约举例子分析：
SECTION I

###首先是一些常量：
//玩家有9000次获取成功 获胜率90%
    uint constant pwin = 9000; //玩家获胜概率 
    uint constant **edge** = 190; //edge percentage (10000 = 100%)赢面 
    uint constant maxWin = 100; //max win (before edge is taken) as percentage of bankroll (10000 = 100%) 赢面被拿走之前的资金资助
    uint constant minBet = 200 finney;//1milliether(finney) = 1e15wei=0.2eth

    uint constant maxInvestors = 10; //投资者人数最多
    
    uint constant houseEdge = 90; //edge percentage (10000 = 100%)//庄家赢面
    uint constant divestFee = 50; //拿走费用的百分比例 divest fee percentage (10000 = 100%)
    uint constant emergencyWithdrawalRatio = 10; //紧急提现率 ratio percentage (100 = 100%)
    //gas安全值
    uint safeGas = 2300;
    uint constant ORACLIZE_GAS_LIMIT = 175000;
    //无效赌注标识
    uint constant INVALID_BET_MARKER = 99999;
    //紧急超时3天
    uint constant EMERGENCY_TIMEOUT = 3 days;


这些常量保存了最基本的投注信息。
	##自由设置赢面

 ###如何保证我们一直盈利？ 

  凯利公式 
  	

![](https://i.imgur.com/BREb6y3.jpg)

>edge 在赌博中可以理解为赢面：即 获胜的概率＊赔率 - 失败的概率，也就是上文提到的赢面。当edge的数字为正的时候，这就是值得下注的比赛，而edge为0或者负数的情况说明赌徒不具备edge, 不应该下注。 赌场要做的就是使投资人的预期收益是负的。这样赌场就会永远赚钱。


###结构体和其他常量


    WithdrawalProposal 提现收益 地址和时间
    Investor 投资者的信息 投资地址 投资数额 是否支持紧急提现 
    mapping(address => uint) public investorIDs; 投资者Id
    mapping(uint => Investor) public investors; 为投资者的编号
    
    //投资者数量
    uint public numInvestors = 0;
    //被投资
    uint public invested = 0;
    //合约管理者
    address public owner;
    //庄家
    address public houseAddress;
    //是否停止游戏
    bool public isStopped;
    //提现提案
    WithdrawalProposal public proposedWithdrawal;
    Bet  //赌注包含的信息 玩家地址 堵住数额 回滚数额
    //保存赌注信息
    mapping (bytes32 => Bet) public bets;
    bytes32[] public betsKeys;
    //投资收益 投资损失 利益分配
    uint public investorsProfit = 0;
    uint public investorsLosses = 0;
    bool profitDistributed;

###系统日志
  //系统日志

    event LOG_NewBet(address playerAddress, uint amount);
    event LOG_BetWon(address playerAddress, uint numberRolled, uint amountWon);
    event LOG_BetLost(address playerAddress, uint numberRolled);
    event LOG_EmergencyWithdrawalProposed();
    event LOG_EmergencyWithdrawalFailed(address withdrawalAddress);
    event LOG_EmergencyWithdrawalSucceeded(address withdrawalAddress, uint amountWithdrawn);
    event LOG_FailedSend(address receiver, uint amount);
    event LOG_ZeroSend();
    event LOG_InvestorEntrance(address investor, uint amount);
    event LOG_InvestorCapitalUpdate(address investor, int amount);
    event LOG_InvestorExit(address investor, uint amount);
    event LOG_ContractStopped();
    event LOG_ContractResumed();
    event LOG_OwnerAddressChanged(address oldAddr, address newOwnerAddress);
    event LOG_HouseAddressChanged(address oldAddr, address newHouseAddress);
    event LOG_GasLimitChanged(uint oldGasLimit, uint newGasLimit);
    event LOG_EmergencyAutoStop();
    event LOG_EmergencyWithdrawalVote(address investor, bool vote);
    event LOG_ValueIsTooBig();
    event LOG_SuccessfulSend(address addr, uint amount);
 用来记录合约中主要发生的事件。 

####修改器
	 onlyIfNotStopped 停止
	 onlyIfStopped  已经停止
	 onlyInvestors 投资者
	 onlyNotInvestors 不是投资者 
	 onlyOwner 管理员 
	 onlyOraclize oraclize内部调用 
	 onlyMoreThanMinInvestment 满足最低投资额
	 onlyMoreThanZero 非零
	 onlyIfBetExist 下注的人存在
	 onlyIfBetSizeIsStillCorrect 只有一种情况可以投注
     onlyIfValidRoll 检测赌注有效
     onlyWinningBets 赌注有效
     onlyLosingBets
     onlyAfterProposed 提议
     onlyIfProfitNotDistributed
     onlyIfValidGas 有效的gas 
	 onlyIfNotProcessed 校验
     onlyIfEmergencyTimeOutHasPassed 紧急提案超时
     investorsInvariant

###帮助工具
	####公共方法
    getBankroll() 获取投资利润
  投资收益加获取的利润减去投资损失

   getMinInvestment()  最小投资额 实际上就是遍历数组 取出mapping中最小的值。 后面方法是searchSmallestInvestor() 这里面实现

   getBalance获取的指定地址余额


   getMinBetAmount()最少赌注额的人

   getMinInvestment() 最小投资额

   getStatus() 检查vDice状态

   getBet() 返回赌注信息	

   numBets()

   getMinBetAmount()

   getMaxBetAmount()

   getLossesShare(address currentInvestor)

   getProfitShare(address currentInvestor)

   searchSmallestInvestor()

   changeOraclizeProofType(byte _proofType)

   changeOraclizeConfig(bytes32 _config) 

以上的公共方法 主要用来管理投资人和查询相关信息

####   私有方法： 内部调用 
safeSend(address addr, uint value)
addInvestorAtID(uint id)
profitDistribution() 遍历所有的地址  给符合条件的地址增加数额



###II 下注

bet()
这里通过Oraclize，访问random.org获取真随机数，数据用椭圆曲线加密，
使用官方的python工具。

格式如
>oraclize_query(
                "nested",
                "[URL]['json(https://api.random.org/json-rpc/1/invoke).result.random.data.0', '\\n{\"jsonrpc\":\"2.0\",\"method\":\"generateSignedIntegers\",\"params\":{\"apiKey\":${[decrypt] BIYkzb1GQzRZFNsTzF7fh+n8VmT8GEyW3mHYlrU8It5O6/bam6/LVVxqkury8YZDJPjm0mWQeqQGebGAVSFrFw16/VHJ65QMFBfIHN2frhav/d10ARqECjoOvse5v4/DIT3LQUHPEx0Z/5UdtqYTQydW/pbC5BM=},\"n\":1,\"min\":1,\"max\":10000${[identity] \"}\"},\"id\":1${[identity] \"}\"}']",
                ORACLIZE_GAS_LIMIT + safeGas
            );


__callback（）  回调函数 防止重入攻击


isWinningBet（） 获胜分配


isLosingBet  失败分配



####IIII取消资格 管理员可以强迫某些作弊的玩家退出
increaseInvestment()
newInvestor()
divest()
divest(address currentInvestor)
forceDivestOfAllInvestors()
forceDivestOfOneInvestor

###Iv合约管理 激活合约 更改管理员地址

stopContract()

resumeContract()

changeHouseAddress(address newHouse)


changeOwnerAddress(address newOwner) 

changeGasLimitOfSafeSend(uint newGasLimit)

###紧急提现 需得到百分之十的人的支持 就可以进行提现 
voteEmergencyWithdrawal(bool vote)

proposeEmergencyWithdrawal()

executeEmergencyWithdrawal()

整个过程 玩家投注eth 如果产生的随机数在范围内 获胜eth自动回来 失败的话返回1wei.






