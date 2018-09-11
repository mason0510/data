Bch赌博游戏SatoshiDice公平性证明

筛子所采用的算法sha256哈希算法
1.投注一个随机数 假定投注20mbch

2.网站进行哈希运算  输了 网站拿走了这笔钱

3.查看交易的游戏id
主要界面
![](https://i.imgur.com/n7xU6h0.jpg)

>未公开的seed seed将会于一周后公布
>这里可以使用已经公布的seed

4.random number seed 使用这个种子生成随机数
算法是： seed 交易id 和hex生成 
numberSeed = sha256(hexStringToByte(serverSeed + txid + voutHex)；

serverSeed是预先生成好的，txid是你押注时产生的区块链交易id，这个本身就是不可能预知的

voutHex 为了增加随机性 使游戏更加公平 类似于加盐 
slighting mocker obstinate state donation catacomb overexert astrology doorbell bulgur angriness choosy

5.将得到的三个数字进行拼接
sha256( hexStringToByte ( "e4f1f9e74e9f548d4c866c627e8093275a37a8a956bf24c2bead7de3f4e7bacc8e75f71737d4b5a1c2328ab41a47f650937aec38cfd4cb6a259d813aaa08a88600000000 ") )
计算结果
81828f636d8c13b66db23e80ea850061cc0ec4bf26aaf21969d941c1a5015101

6.
相关算法：取8位作为种子散列
function getRandomNumber(sha256) {
	var seed = new Array();
	for (var s = 0; s < 8; s++) {
	    var intText = sha256.substring(s * 8, (s + 1) * 8);
	    seed.push(parseInt(intText, 16));
	    mersenneTwister.seedArray(seed);
	}
	return mersenneTwister.short();
}

7.mersenneTwister 
通过梅森螺旋算法计算幸运数字

测试地址：
https://satoshidice.com/verifygame.html?id=267609