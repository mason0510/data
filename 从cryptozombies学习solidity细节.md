## 从cryptozombies学习solidity细节

## 第一章

### 合约

- 版本指定

  ```
  pragma solidity ^0.4.19;
  ```

- 创建合约

  ```
  //建立僵尸工厂
  contract ZombieFactory {
  
  }
  ```

### 状态变量和整型

- **状态变量**是被永久地保存在合约中。也就是说它们被写入以太币区块链中. 想象成写入一个数据库

  ```
   // 在合约内,方法外 定义一个状态变量 dna数字
   uint dnaDigits = 16;
  ```

- 整型

### 数学运算

- 乘方操作

  ```
  // 10的dnaDigits次方
  // dna系数
  uint dnaModulus = 10 ** dnaDigits;
  //为了保证我们的僵尸的DNA只含有16个字符，我们先造一个uint数据，让它等于10^16。这样一来以后我们可以用模运算符 % 把一个整数变成16位。
  ```

  

### 结构体

结构体允许你生成一个更复杂的数据类型，它有多个属性。 

```
//僵尸有多个属性
struct Zombie {
     string name;
     uint dna;
 }
```



### 数组

- 如果想建立一个集合，可以用 **数组**这样的数据类型. 
- Solidity 支持两种数组: **静态** 数组(长度是固定的) 和 **动态** 数组(长度不固定)
     - 静态数组 `uint[2] fixedArray; `
     - 动态数组 `uint[] dynamicArray; `: 可以动态添加元素
- 也可以建立一个 **结构体**类型的数组 
- 状态变量被永久保存在区块链中。所以在合约中创建动态数组来保存成结构的数据是非常有意义的。 
- 可以定义 `public` 数组, Solidity 会自动创建 **getter** 方法. 其它的合约可以从这个数组读取数据（但不能写入数据），所以这在合约中是一个有用的保存公共数据的模式。
```
  // Zombie结构体数组  结构体表示僵尸
  Zombie[] public Zombies;
```



### 函数

- 习惯上函数里的变量都是以(`_`)开头 (但不是硬性规定) 以区别全局变量 

  ```
   // 生成僵尸的函数
   function createZombie(string _name,uint _dna) {}
  ```

### 使用结构体和数组

- `array.push()` 在数组的 **尾部** 加入新元素 ，所以元素在数组中的顺序就是我们添加的顺序 

  ```
   function createZombie(string _name, uint _dna) {
          //在僵尸数组中,添加一个僵尸
          zombies.push(Zombie(_name, _dna));
      }
  ```

  

### 私有函数和公有函数

- Solidity 定义的函数的属性默认为`公共`。 这就意味着任何一方 (或其它合约) 都可以调用这个合约里的函数。

  显然，不是什么时候都需要这样，而且这样的合约易于受到攻击。 所以将自己的函数定义为`私有`是一个好的编程习惯，只有当你需要外部世界调用它时才将它设置为`公共`。

- 在函数名字后面使用关键字 `private` 即可。私有函数的名字用(`_`)起始。 只有自己合约中的其他函数才能调用这个函数
### 函数的更多属性

- 返回值:`returns(数据类型)`

- 修饰符:

  - view:如果函数实际上没有改变 Solidity 里的状态，即它没有改变任何值或者写任何东西。

    这种情况下我们可以把函数定义为 **view**, 意味着它只能读取数据不能更改数据

  - pure:Solidity 还支持 **pure** 函数, 表明这个函数甚至都不访问应用里的数据，例如：

    这个函数甚至都不读取应用里的状态 — 它的返回值完全取决于它的输入参数，在这种情况下我们把函数定义为 **pure**.

  ```
 // 产生一个随机的dna 只会读取合约的数据 不会改变 所以view
 function _generateRandomDna(string _str) private view returns(uint) {}
  ```

### Keccak256 和 类型转换

- Keccak256:

  Ethereum 内部有一个散列函数`keccak256`，它用了SHA3版本。一个散列函数基本上就是把一个字符串转换为一个256位的2进制数字。字符串的一个微小变化会引起散列数据极大变化。 

- 类型转换

​       `需要转换成的数据类型(待转换的数据)`

```
 function _generateRandomDna(string _str) private view returns (uint) {
       // 首先得到一个 256位的16进制数组 强转为10进制的数字
        uint rand = uint(keccak256(_str));
        // 得到一个16位的数字
        return rand % dnaModulus;
    }
 //创造一个僵尸   
 function createRandomZombie(string _name) public {
       //得到一个随机的dna
       uint randDna = _generateRandomDna(_name);
       //将名字个dna值传入生成僵尸的函数
        _createZombie(_name,randDna);
    }
```
### 定义事件

- event

    // 定义事件
    event NewZombie(uint zombieId, string name, uint dna);
    
    // 这里触发事件
    //array.push() 返回数组的长度类型是uint 
    uint id = zombies.push(Zombie(_name, _dna)) - 1;
    NewZombie(id, _name, _dna);
    
## 第二章

它得支持多用户，并且采用更加有趣,而不仅仅使用随机的方式，来生成新的僵尸。

如何生成新的僵尸呢？通过让现有的僵尸猎食其他生物！

### Mapping 和 Address

#### Address

- 以太坊区块链由 **account** (账户)组成，你可以把它想象成银行账户。一个帐户的余额是 **以太** （在以太坊区块链上使用的币种），你可以和其他帐户之间支付和接受以太币，就像你的银行帐户可以电汇资金到其他银行帐户一样。
- 每个帐户都有一个“地址”，你可以把它想象成银行账号。这是账户唯一的标识符，它看起来长这样：

`0x0cE446255506E92DF41614C46F1d6df9Cc969183`

- 所以我们可以指定“地址”作为僵尸主人的 ID。当用户通过与我们的应用程序交互来创建新的僵尸时，新僵尸的所有权被设置到调用者的以太坊地址下。

#### Mapping

-  **映射** 是另一种在 Solidity 中存储有组织数据的方法。
- 映射是这样定义的：

```
//对于金融应用程序，将用户的余额保存在一个 uint类型的变量中：
mapping (address => uint) public accountBalance;
//或者可以用来通过userId 存储/查找的用户名
mapping (uint => string) userIdToName;
```

- 映射本质上是存储和查找数据所用的键-值对。在第一个例子中，键是一个 `address`，值是一个 `uint`，在第二个例子中，键是一个`uint`，值是一个 `string`。

```
  // 记录某个id的僵尸属于哪个地址
  mapping (uint => address) public zombieToOwner;
  // 记录一个地址拥有多少个僵尸
  mapping (address => uint) public ownerZombieCount;
```

### Msg.sender

- 在 Solidity 中，有一些全局变量可以被所有函数调用。 其中一个就是 `msg.sender`，它指的是当前调用者（或智能合约）的 `address`。
- 在 Solidity 中，功能执行始终需要从外部调用者开始。 一个合约只会在区块链上什么也不做，除非有人调用其中的函数。所以 `msg.sender`总是存在的。
- 以下是使用 `msg.sender` 来更新 `mapping` 的例子：

```
mapping (address => uint) favoriteNumber;

function setMyNumber(uint _myNumber) public {
  // 更新我们的 `favoriteNumber` 映射来将 `_myNumber`存储在 `msg.sender`名下
  favoriteNumber[msg.sender] = _myNumber;
  // 存储数据至映射的方法和将数据存储在数组相似
}

function whatIsMyNumber() public view returns (uint) {
  // 拿到存储在调用者地址名下的值
  // 若调用者还没调用 setMyNumber， 则值为 `0`
  return favoriteNumber[msg.sender];
}
```

在这个小小的例子中，任何人都可以调用 `setMyNumber` 在我们的合约中存下一个 `uint` 并且与他们的地址相绑定。 然后，他们调用 `whatIsMyNumber` 就会返回他们存储的 `uint`。

使用 `msg.sender` 很安全，因为它具有以太坊区块链的安全保障 —— 除非窃取与以太坊地址相关联的私钥，否则是没有办法修改其他人的数据的。

```
function _createZombie(string _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        // 创建僵尸后 将这个僵尸的id设为发起合约者
        zombieToOwner[id] = msg.sender;
        // 调用者的地址 僵尸数加1
        ownerZombieCount[msg.sender]++;
        NewZombie(id, _name, _dna);
    }
```

### Require

- 在第一课中，我们成功让用户通过调用 `createRandomZombie`函数 并输入一个名字来创建新的僵尸。 但是，如果用户能持续调用这个函数来创建出无限多个僵尸加入他们的军团，这游戏就太没意思了！于是，我们作出限定：每个玩家只能调用一次这个函数。 这样一来，新玩家可以在刚开始玩游戏时通过调用它，为其军团创建初始僵尸。我们怎样才能限定每个玩家只调用一次这个函数呢？
- 答案是使用`require`。 `require`使得函数在执行过程中，当不满足某些条件时抛出错误，并停止执行：

```
function sayHiToVitalik(string _name) public returns (string) {
  // 比较 _name 是否等于 "Vitalik". 如果不成立，抛出异常并终止程序
  // Solidity 并不支持原生的字符串比较, 我们只能通过比较两字符串的 keccak256 哈希值来进行判断)
  require(keccak256(_name) == keccak256("Vitalik"));
  // 如果返回 true, 运行如下语句
  return "Hi!";
}
```

```
    function createRandomZombie(string _name) public {
        // 只有调用者的僵尸数为0的时候才能调用这个函数
        require(ownerZombieCount[msg.sender] == 0);
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }
```



### 继承(Inheritance)

- 我们的游戏代码越来越长。 当代码过于冗长的时候，最好将代码和逻辑分拆到多个不同的合约中，以便于管理。
- 有个让 Solidity 的代码易于管理的功能，就是合约 **inheritance** (继承)：

```
contract Doge {
  function catchphrase() public returns (string) {
    return "So Wow CryptoDoge";
  }
}

contract BabyDoge is Doge {
  function anotherCatchphrase() public returns (string) {
    return "Such Moon BabyDoge";
  }
}
```

- 由于 `BabyDoge` 是从 `Doge` 那里 **inherits** （继承)过来的。 这意味着当你编译和部署了 `BabyDoge`，它将可以访问 `catchphrase()` 和 `anotherCatchphrase()`和其他我们在 `Doge` 中定义的其他公共函数。
- 这可以用于逻辑继承（比如表达子类的时候，`Cat` 是一种 `Animal`）。 但也可以简单地将类似的逻辑组合到不同的合约中以组织代码。

```
//继承僵尸工厂合约
constract ZombieFeeding is ZombieFactory {}
```

### Import

- 代码已经够长了，我们把它分成多个文件以便于管理。 通常情况下，当 Solidity 项目中的代码太长的时候我们就是这么做的。
- 在 Solidity 中，当你有多个文件并且想把一个文件导入另一个文件时，可以使用 `import` 语句：

```
import "./someothercontract.sol";

contract newContract is SomeOtherContract {

}
```

- 这样当我们在合约（contract）目录下有一个名为 `someothercontract.sol` 的文件（ `./` 就是同一目录的意思），它就会被编译器导入。

### Storage与Memory

- 在 Solidity 中，有两个地方可以存储变量 —— `storage` 或 `memory`。
- **Storage** 变量是指永久存储在区块链中的变量。 **Memory** 变量则是临时的，当外部函数对某合约调用完成时，内存型变量即被移除。 你可以把它想象成存储在你电脑的硬盘或是RAM中数据的关系。
- 大多数时候你都用不到这些关键字，默认情况下 Solidity 会自动处理它们。 状态变量（在函数之外声明的变量）默认为“存储”形式，并永久写入区块链；而在函数内部声明的变量是“内存”型的，它们函数调用结束后消失。
- 然而也有一些情况下，你需要手动声明存储类型，主要用于处理函数内的**结构体** 和 **数组** 时：

```
contract SandwichFactory {
  struct Sandwich {
    string name;
    string status;
  }

  Sandwich[] sandwiches;

  function eatSandwich(uint _index) public {
    // Sandwich mySandwich = sandwiches[_index];

    // ^ 看上去很直接，不过 Solidity 将会给出警告
    // 告诉你应该明确在这里定义 `storage` 或者 `memory`。

    // 所以你应该明确定义 `storage`:
    Sandwich storage mySandwich = sandwiches[_index];
    // ...这样 `mySandwich` 是指向 `sandwiches[_index]`的指针
    // 在存储里，另外...
    mySandwich.status = "Eaten!";
    // ...这将永久把 `sandwiches[_index]` 变为区块链上的存储

    // 如果你只想要一个副本，可以使用`memory`:
    Sandwich memory anotherSandwich = sandwiches[_index + 1];
    // ...这样 `anotherSandwich` 就仅仅是一个内存里的副本了
    // 另外
    anotherSandwich.status = "Eaten!";
    // ...将仅仅修改临时变量，对 `sandwiches[_index + 1]` 没有任何影响
    // 不过你可以这样做:
    sandwiches[_index + 1] = anotherSandwich;
    // ...如果你想把副本的改动保存回区块链存储
  }
}
```

```
 function feedAndMultiply(uint _zombieId, uint _targetDna) public{
      // 僵尸的主人才能用这个方法
      require(msg.sender == zombieToOwner[_zombieId]);
      //取到僵尸数组中这个id的僵尸
      Zombie storage myZombie = Zombies[_zombieId];
  } 
```



### 更多函数可见性

#### Internal和external

- 除 `public` 和 `private` 属性之外，Solidity 还使用了另外两个描述函数可见性的修饰词：`internal`（内部） 和 `external`（外部）。
- `internal` 和 `private` 类似，不过， 如果某个合约继承自其父合约，这个合约即可以访问父合约中定义的“内部”函数。
- `external` 与`public` 类似，只不过这些函数只能在合约之外调用 - 它们不能被合约内的其他函数调用。稍后我们将讨论什么时候使用 `external`和 `public`。
- 声明函数 `internal` 或 `external` 类型的语法，与声明 `private` 和 `public`类 型相同：

```

```

### 接口

- 如果我们的合约需要和区块链上的其他的合约会话，则需先定义一个**interface** (接口)。
- 先举一个简单的栗子。 假设在区块链上有这么一个合约：

```
contract LuckyNumber {
  mapping(address => uint) numbers;

  function setNum(uint _num) public {
    numbers[msg.sender] = _num;
  }

  function getNum(address _myAddress) public view returns (uint) {
    return numbers[_myAddress];
  }
}
```

- 这是个很简单的合约，您可以用它存储自己的幸运号码，并将其与您的以太坊地址关联。 这样其他人就可以通过您的地址查找您的幸运号码了。

- 现在假设我们有一个外部合约，使用 `getNum` 函数可读取其中的数据。
- 首先，我们定义 `LuckyNumber` 合约的 **interface** ：

```
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
```

- 请注意，这个过程虽然看起来像在定义一个合约，但其实内里不同：
- 首先，我们只声明了要与之交互的函数 —— 在本例中为 `getNum` —— 在其中我们没有使用到任何其他的函数或状态变量。
- 其次，我们并没有使用大括号（`{` 和 `}`）定义函数体，我们单单用分号（`;`）结束了函数声明。这使它看起来像一个合约框架。
- 编译器就是靠这些特征认出它是一个接口的。
- 在我们的 app 代码中使用这个接口，合约就知道其他合约的函数是怎样的，应该如何调用，以及可期待什么类型的返回值。

###使用接口

- 继续前面 `NumberInterface` 的例子，我们既然将接口定义为：

```
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
```

- 我们可以在合约中这样使用：

```
contract MyContract {
  address NumberInterfaceAddress = 0xab38...;
  // ^ 这是FavoriteNumber合约在以太坊上的地址
  
  NumberInterface numberContract = NumberInterface(NumberInterfaceAddress);
  // 现在变量 `numberContract` 指向另一个合约对象

  function someFunction() public {
    // 现在我们可以调用在那个合约中声明的 `getNum`函数:
    uint num = numberContract.getNum(msg.sender);
    // ...在这儿使用 `num`变量做些什么
  }
}
```

- 通过这种方式，只要将您合约的可见性设置为`public`(公共)或`external`(外部)，它们就可以与以太坊区块链上的任何其他合约进行交互。

### 处理多返回值

- `getKitty` 是我们所看到的第一个返回多个值的函数。我们来看看是如何处理的：

```
function multipleReturns() internal returns(uint a, uint b, uint c) {
  return (1, 2, 3);
}

function processMultipleReturns() external {
  uint a;
  uint b;
  uint c;
  // 这样来做批量赋值:
  (a, b, c) = multipleReturns();
}

// 或者如果我们只想返回其中一个变量:
function getLastReturnValue() external {
  uint c;
  // 可以对其他字段留空:
  (,,c) = multipleReturns();
}
```

## 第三章

### 智能协议的永固性

到现在为止，我们讲的 Solidity 和其他语言没有质的区别，它长得也很像 JavaScript.

但是，在有几点以太坊上的 DApp 跟普通的应用程序有着天壤之别。

- 第一个例子，在你把智能协议传上以太坊之后，它就变得**不可更改**, 这种永固性意味着你的代码永远不能被调整或更新。
- 你编译的程序会一直，永久的，不可更改的，存在以太网上。这就是Solidity代码的安全性如此重要的一个原因。如果你的智能协议有任何漏洞，即使你发现了也无法补救。你只能让你的用户们放弃这个智能协议，然后转移到一个新的修复后的合约上。
- 但这恰好也是智能合约的一大优势。 代码说明一切。 如果你去读智能合约的代码，并验证它，你会发现， 一旦函数被定义下来，每一次的运行，程序都会严格遵照函数中原有的代码逻辑一丝不苟地执行，完全不用担心函数被人篡改而得到意外的结果。

###外部依赖关系

- 在第2课中，我们将加密小猫（CryptoKitties）合约的地址硬编码到DApp中去了。有没有想过，如果加密小猫出了点问题，比方说，集体消失了会怎么样？ 虽然这种事情几乎不可能发生，但是，如果小猫没了，我们的 DApp 也会随之失效 -- 因为我们在 DApp 的代码中用“硬编码”的方式指定了加密小猫的地址，如果这个根据地址找不到小猫，我们的僵尸也就吃不到小猫了，而按照前面的描述，我们却没法修改合约去应付这个变化！
- 因此，我们不能硬编码，而要采用“函数”，以便于 DApp 的关键部分可以以参数形式修改。
- 比方说，我们不再一开始就把猎物地址给写入代码，而是写个函数 `setKittyContractAddress`, 运行时再设定猎物的地址，这样我们就可以随时去锁定新的猎物，也不用担心加密小猫集体消失了。

### Ownable Contracts

- 指定合约的“所有权” - 就是说，给它指定一个主人（没错，就是您），只有主人对它享有特权。

###OpenZeppelin库的`Ownable` 合约

- 下面是一个 `Ownable` 合约的例子： 来自 **OpenZeppelin** Solidity 库的 `Ownable` 合约。 OpenZeppelin 是主打安保和社区审查的智能合约库，您可以在自己的 DApps中引用。

```
/**
 * @title Ownable
 * @dev The Ownable contract has an owner address, and provides basic authorization control
 * functions, this simplifies the implementation of "user permissions".
 */
contract Ownable {
  address public owner;
  event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

  /**
   * @dev The Ownable constructor sets the original `owner` of the contract to the sender
   * account.
   */
  function Ownable() public {
    owner = msg.sender;
  }

  /**
   * @dev Throws if called by any account other than the owner.
   */
  modifier onlyOwner() {
    require(msg.sender == owner);
    _;
  }

  /**
   * @dev Allows the current owner to transfer control of the contract to a newOwner.
   * @param newOwner The address to transfer ownership to.
   */
  function transferOwnership(address newOwner) public onlyOwner {
    require(newOwner != address(0));
    OwnershipTransferred(owner, newOwner);
    owner = newOwner;
  }
}
```

- 构造函数：`function Ownable()`是一个 **constructor** (构造函数)，构造函数不是必须的，它与合约同名，构造函数一生中唯一的一次执行，就是在合约最初被创建的时候。
- 函数修饰符：`modifier onlyOwner()`。 修饰符跟函数很类似，不过是用来修饰其他已有函数用的， 在其他语句执行前，为它检查下先验条件。 在这个例子中，我们就可以写个修饰符 `onlyOwner` 检查下调用者，确保只有合约的主人才能运行本函数。我们下一章中会详细讲述修饰符，以及那个奇怪的`_;`。
- `indexed` 关键字：别担心，我们还用不到它。

所以`Ownable` 合约基本都会这么干：

1. 合约创建，构造函数先行，将其 `owner` 设置为`msg.sender`（其部署者）
2. 为它加上一个修饰符 `onlyOwner`，它会限制陌生人的访问，将访问某些函数的权限锁定在 `owner` 上。
3. 允许将合约所有权转让给他人。

`onlyOwner` 简直人见人爱，大多数人开发自己的 Solidity DApps，都是从复制/粘贴 `Ownable` 开始的，从它再继承出的子类，并在之上进行功能开发。

既然我们想把 `setKittyContractAddress` 限制为 `onlyOwner` ，我们也要做同样的事情。

### onlyOwner 函数修饰符

```
ZombieFeeding 是个 ZombieFactory
ZombieFactory 是个 Ownable
```

因此 `ZombieFeeding` 也是个 `Ownable`, 并可以通过 `Ownable` 接口访问父类中的函数/事件/修饰符。往后，`ZombieFeeding` 的继承者合约们同样也可以这么延续下去。

###函数修饰符

函数修饰符看起来跟函数没什么不同，不过关键字`modifier` 告诉编译器，这是个`modifier(修饰符)`，而不是个`function(函数)`。它不能像函数那样被直接调用，只能被添加到函数定义的末尾，用以改变函数的行为。

咱们仔细读读 `onlyOwner`:

```
/**
 * @dev 调用者不是‘主人’，就会抛出异常
 */
modifier onlyOwner() {
  require(msg.sender == owner);
  _;
}
```

`onlyOwner` 函数修饰符是这么用的：

```
contract MyContract is Ownable {
  event LaughManiacally(string laughter);

  //注意！ `onlyOwner`上场 :
  function likeABoss() external onlyOwner {
    LaughManiacally("Muahahahaha");
  }
}
```

注意 `likeABoss` 函数上的 `onlyOwner` 修饰符。 当你调用 `likeABoss`时，**首先执行** `onlyOwner` 中的代码， 执行到 `onlyOwner` 中的 `_;` 语句时，程序再返回并执行 `likeABoss` 中的代码。

可见，尽管函数修饰符也可以应用到各种场合，但最常见的还是放在函数执行之前添加快速的 `require`检查。

因为给函数添加了修饰符 `onlyOwner`，使得**唯有合约的主人**（也就是部署者）才能调用它。

### Gas

让我们来看另一种使得 Solidity 编程语言与众不同的特征

####Gas - 驱动以太坊DApps的能源

在 Solidity 中，你的用户想要每次执行你的 DApp 都需要支付一定的 **gas**，gas 可以用以太币购买，因此，用户每次跑 DApp 都得花费以太币。

一个 DApp 收取多少 gas 取决于功能逻辑的复杂程度。每个操作背后，都在计算完成这个操作所需要的计算资源，（比如，存储数据就比做个加法运算贵得多）， 一次操作所需要花费的 **gas** 等于这个操作背后的所有运算花销的总和。

由于运行你的程序需要花费用户的真金白银，在以太坊中代码的编程语言，比其他任何编程语言都更强调优化。同样的功能，使用笨拙的代码开发的程序，比起经过精巧优化的代码来，运行花费更高，这显然会给成千上万的用户带来大量不必要的开销。

####为什么要用 **gas** 来驱动？

以太坊就像一个巨大、缓慢、但非常安全的电脑。当你运行一个程序的时候，网络上的每一个节点都在进行相同的运算，以验证它的输出 —— 这就是所谓的”去中心化“ 由于数以千计的节点同时在验证着每个功能的运行，这可以确保它的数据不会被被监控，或者被刻意修改。

可能会有用户用无限循环堵塞网络，抑或用密集运算来占用大量的网络资源，为了防止这种事情的发生，以太坊的创建者为以太坊上的资源制定了价格，想要在以太坊上运算或者存储，你需要先付费。

注意：如果你使用侧链，倒是不一定需要付费，比如咱们在 Loom Network 上构建的 CryptoZombies 就免费。你不会想要在以太坊主网上玩儿“魔兽世界”吧？ - 所需要的 gas 可能会买到你破产。但是你可以找个算法理念不同的侧链来玩它。我们将在以后的课程中咱们会讨论到，什么样的 DApp 应该部署在太坊主链上，什么又最好放在侧链。

####省 gas 的招数：结构封装 （Struct packing）

在第1课中，我们提到除了基本版的 `uint` 外，还有其他变种 `uint`：`uint8`，`uint16`，`uint32`等。

通常情况下我们不会考虑使用 `uint` 变种，因为无论如何定义 `uint`的大小，Solidity 为它保留256位的存储空间。例如，使用 `uint8` 而不是`uint`（`uint256`）不会为你节省任何 gas。

除非，把 `uint` 绑定到 `struct` 里面。

如果一个 `struct` 中有多个 `uint`，则尽可能使用较小的 `uint`, Solidity 会将这些 `uint` 打包在一起，从而占用较少的存储空间。例如：

```
struct NormalStruct {
  uint a;
  uint b;
  uint c;
}

struct MiniMe {
  uint32 a;
  uint32 b;
  uint c;
}

// 因为使用了结构打包，`mini` 比 `normal` 占用的空间更少
NormalStruct normal = NormalStruct(10, 20, 30);
MiniMe mini = MiniMe(10, 20, 30);
```

所以，当 `uint` 定义在一个 `struct` 中的时候，尽量使用最小的整数子类型以节约空间。 并且把同样类型的变量放一起（即在 struct 中将把变量按照类型依次放置），这样 Solidity 可以将存储空间最小化。例如，有两个 `struct`：

`uint c; uint32 a; uint32 b;` 和 `uint32 a; uint c; uint32 b;`

前者比后者需要的gas更少，因为前者把`uint32`放一起了。

### 时间单位

Solidity 使用自己的本地时间单位。

变量 `now` 将返回当前的unix时间戳（自1970年1月1日以来经过的秒数）。我写这句话时 unix 时间是 `1515527488`。

注意：Unix时间传统用一个32位的整数进行存储。这会导致“2038年”问题，当这个32位的unix时间戳不够用，产生溢出，使用这个时间的遗留系统就麻烦了。所以，如果我们想让我们的 DApp 跑够20年，我们可以使用64位整数表示时间，但为此我们的用户又得支付更多的 gas。真是个两难的设计啊！

Solidity 还包含`秒(seconds)`，`分钟(minutes)`，`小时(hours)`，`天(days)`，`周(weeks)` 和 `年(years)` 等时间单位。它们都会转换成对应的秒数放入 `uint` 中。所以 `1分钟` 就是 `60`，`1小时`是 `3600`（60秒×60分钟），`1天`是`86400`（24小时×60分钟×60秒），以此类推。

下面是一些使用时间单位的实用案例：

```
uint lastUpdated;

// 将‘上次更新时间’ 设置为 ‘现在’
function updateTimestamp() public {
  lastUpdated = now;
}

// 如果到上次`updateTimestamp` 超过5分钟，返回 'true'
// 不到5分钟返回 'false'
function fiveMinutesHavePassed() public view returns (bool) {
  return (now >= (lastUpdated + 5 minutes));
}
```

有了这些工具，我们可以为僵尸设定”冷静时间“功能

###将结构体作为参数传入

由于结构体的存储指针可以以参数的方式传递给一个 `private` 或 `internal` 的函数，因此结构体可以在多个函数之间相互传递。

遵循这样的语法：

```
function _doStuff(Zombie storage _zombie) internal {
  // do stuff with _zombie
}
```

这样我们可以将某僵尸的引用直接传递给一个函数，而不用是通过参数传入僵尸ID后，函数再依据ID去查找。

### 公有函数和安全性

现在来修改 `feedAndMultiply` ，实现冷却周期。

回顾一下这个函数，前一课上我们将其可见性设置为`public`。你必须仔细地检查所有声明为 `public` 和 `external`的函数，一个个排除用户滥用它们的可能，谨防安全漏洞。请记住，如果这些函数没有类似 `onlyOwner`这样的函数修饰符，用户能利用各种可能的参数去调用它们。

检查完这个函数，用户就可以直接调用这个它，并传入他们所希望的 `_targetDna` 或 `species` 。打个游戏还得遵循这么多的规则，还能不能愉快地玩耍啊！

仔细观察，这个函数只需被 `feedOnKitty()` 调用，因此，想要防止漏洞，最简单的方法就是设其可见性为 `internal`。

### 进一步了解函数修饰符

相当不错！我们的僵尸现在有了“冷却定时器”功能。

接下来，我们将添加一些辅助方法。我们为您创建了一个名为 `zombiehelper.sol` 的新文件，并且将 `zombiefeeding.sol` 导入其中，这让我们的代码更整洁。

我们打算让僵尸在达到一定水平后，获得特殊能力。但是达到这个小目标，我们还需要学一学什么是“函数修饰符”。

###带参数的函数修饰符

之前我们已经读过一个简单的函数修饰符了：`onlyOwner`。函数修饰符也可以带参数。例如：

```
// 存储用户年龄的映射
mapping (uint => uint) public age;

// 限定用户年龄的修饰符
modifier olderThan(uint _age, uint _userId) {
  require(age[_userId] >= _age);
  _;
}

// 必须年满16周岁才允许开车 (至少在美国是这样的).
// 我们可以用如下参数调用`olderThan` 修饰符:
function driveCar(uint _userId) public olderThan(16, _userId) {
  // 其余的程序逻辑
}
```

看到了吧， `olderThan` 修饰符可以像函数一样接收参数，是“宿主”函数 `driveCar` 把参数传递给它的修饰符的。

来，我们自己生产一个修饰符，通过传入的`level`参数来限制僵尸使用某些特殊功能。

### 利用 'View' 函数节省 Gas

现在需要添加的一个功能是：我们的 DApp 需要一个方法来查看某玩家的整个僵尸军团 - 我们称之为 `getZombiesByOwner`。

实现这个功能只需从区块链中读取数据，所以它可以是一个 `view` 函数。这让我们不得不回顾一下“gas优化”这个重要话题。

###“view” 函数不花 “gas”

当玩家从外部调用一个`view`函数，是不需要支付一分 gas 的。

这是因为 `view` 函数不会真正改变区块链上的任何数据 - 它们只是读取。因此用 `view` 标记一个函数，意味着告诉 `web3.js`，运行这个函数只需要查询你的本地以太坊节点，而不需要在区块链上创建一个事务（事务需要运行在每个节点上，因此花费 gas）。

稍后我们将介绍如何在自己的节点上设置 web3.js。但现在，你关键是要记住，在所能只读的函数上标记上表示“只读”的“`external view` 声明，就能为你的玩家减少在 DApp 中 gas 用量。

注意：如果一个 `view` 函数在另一个函数的内部被调用，而调用函数与 `view` 函数的不属于同一个合约，也会产生调用成本。这是因为如果主调函数在以太坊创建了一个事务，它仍然需要逐个节点去验证。所以标记为 `view` 的函数只有在外部调用时才是免费的。

### 存储非常昂贵

Solidity 使用`storage`(存储)是相当昂贵的，”写入“操作尤其贵。

这是因为，无论是写入还是更改一段数据， 这都将永久性地写入区块链。”永久性“啊！需要在全球数千个节点的硬盘上存入这些数据，随着区块链的增长，拷贝份数更多，存储量也就越大。这是需要成本的！

为了降低成本，不到万不得已，避免将数据写入存储。这也会导致效率低下的编程逻辑 - 比如每次调用一个函数，都需要在 `memory`(内存) 中重建一个数组，而不是简单地将上次计算的数组给存储下来以便快速查找。

在大多数编程语言中，遍历大数据集合都是昂贵的。但是在 Solidity 中，使用一个标记了`external view`的函数，遍历比 `storage` 要便宜太多，因为 `view` 函数不会产生任何花销。 （gas可是真金白银啊！）。

我们将在下一章讨论`for`循环，现在我们来看一下看如何如何在内存中声明数组。

###在内存中声明数组

在数组后面加上 `memory`关键字， 表明这个数组是仅仅在内存中创建，不需要写入外部存储，并且在函数调用结束时它就解散了。与在程序结束时把数据保存进 `storage` 的做法相比，内存运算可以大大节省gas开销 -- 把这数组放在`view`里用，完全不用花钱。

以下是申明一个内存数组的例子：

```
function getArray() external pure returns(uint[]) {
  // 初始化一个长度为3的内存数组
  uint[] memory values = new uint[](3);
  // 赋值
  values.push(1);
  values.push(2);
  values.push(3);
  // 返回数组
  return values;
}
```

这个小例子展示了一些语法规则，下一章中，我们将通过一个实际用例，展示它和 `for` 循环结合的做法。

> 注意：内存数组 **必须** 用长度参数（在本例中为`3`）创建。目前不支持 `array.push()`之类的方法调整数组大小，在未来的版本可能会支持长度修改。

### For 循环

在之前的章节中，我们提到过，函数中使用的数组是运行时在内存中通过 `for` 循环实时构建，而不是预先建立在存储中的。

为什么要这样做呢？

为了实现 `getZombiesByOwner` 函数，一种“无脑式”的解决方案是在 `ZombieFactory` 中存入”主人“和”僵尸军团“的映射。

```
mapping (address => uint[]) public ownerToZombies
```

然后我们每次创建新僵尸时，执行 `ownerToZombies [owner] .push（zombieId）` 将其添加到主人的僵尸数组中。而 `getZombiesByOwner` 函数也非常简单：

```
function getZombiesByOwner(address _owner) external view returns (uint[]) {
  return ownerToZombies[_owner];
}
```

### 这个做法有问题

做法倒是简单。可是如果我们需要一个函数来把一头僵尸转移到另一个主人名下（我们一定会在后面的课程中实现的），又会发生什么？

这个“换主”函数要做到：

1.将僵尸push到新主人的 `ownerToZombies` 数组中， 2.从旧主的 `ownerToZombies` 数组中移除僵尸， 3.将旧主僵尸数组中“换主僵尸”之后的的每头僵尸都往前挪一位，把挪走“换主僵尸”后留下的“空槽”填上， 4.将数组长度减1。

但是第三步实在是太贵了！因为每挪动一头僵尸，我们都要执行一次写操作。如果一个主人有20头僵尸，而第一头被挪走了，那为了保持数组的顺序，我们得做19个写操作。

由于写入存储是 Solidity 中最费 gas 的操作之一，使得换主函数的每次调用都非常昂贵。更糟糕的是，每次调用的时候花费的 gas 都不同！具体还取决于用户在原主军团中的僵尸头数，以及移走的僵尸所在的位置。以至于用户都不知道应该支付多少 gas。

> 注意：当然，我们也可以把数组中最后一个僵尸往前挪来填补空槽，并将数组长度减少一。但这样每做一笔交易，都会改变僵尸军团的秩序。

由于从外部调用一个 `view` 函数是免费的，我们也可以在 `getZombiesByOwner` 函数中用一个for循环遍历整个僵尸数组，把属于某个主人的僵尸挑出来构建出僵尸数组。那么我们的 `transfer` 函数将会便宜得多，因为我们不需要挪动存储里的僵尸数组重新排序，总体上这个方法会更便宜，虽然有点反直觉。

###使用 `for` 循环

`for`循环的语法在 Solidity 和 JavaScript 中类似。

来看一个创建偶数数组的例子：

```
function getEvens() pure external returns(uint[]) {
  uint[] memory evens = new uint[](5);
  // 在新数组中记录序列号
  uint counter = 0;
  // 在循环从1迭代到10：
  for (uint i = 1; i <= 10; i++) {
    // 如果 `i` 是偶数...
    if (i % 2 == 0) {
      // 把它加入偶数数组
      evens[counter] = i;
      //索引加一， 指向下一个空的‘even’
      counter++;
    }
  }
  return evens;
}
```

这个函数将返回一个形为 `[2,4,6,8,10]` 的数组。

### 提现

在上一章，我们学习了如何向合约发送以太，那么在发送之后会发生什么呢？

在你发送以太之后，它将被存储进以合约的以太坊账户中， 并冻结在哪里 —— 除非你添加一个函数来从合约中把以太提现。

你可以写一个函数来从合约中提现以太，类似这样：

```
contract GetPaid is Ownable {
  function withdraw() external onlyOwner {
    owner.transfer(this.balance);
  }
}
```

注意我们使用 `Ownable` 合约中的 `owner` 和 `onlyOwner`，假定它已经被引入了。

你可以通过 `transfer` 函数向一个地址发送以太， 然后 `this.balance`将返回当前合约存储了多少以太。 所以如果100个用户每人向我们支付1以太， `this.balance` 将是100以太。

你可以通过 `transfer` 向任何以太坊地址付钱。 比如，你可以有一个函数在 `msg.sender` 超额付款的时候给他们退钱：

```
uint itemFee = 0.001 ether;
msg.sender.transfer(msg.value - itemFee);
```

或者在一个有卖家和卖家的合约中， 你可以把卖家的地址存储起来， 当有人买了它的东西的时候，把买家支付的钱发送给它 `seller.transfer(msg.value)`。

有很多例子来展示什么让以太坊编程如此之酷 —— 你可以拥有一个不被任何人控制的去中心化市场。

### 随机数

优秀的游戏都需要一些随机元素，那么我们在 Solidity 里如何生成随机数呢？

真正的答案是你不能，或者最起码，你无法安全地做到这一点。

我们来看看为什么

###用 `keccak256` 来制造随机数。

Solidity 中最好的随机数生成器是 `keccak256` 哈希函数.

我们可以这样来生成一些随机数

```
// 生成一个0到100的随机数:
uint randNonce = 0;
uint random = uint(keccak256(now, msg.sender, randNonce)) % 100;
randNonce++;
uint random2 = uint(keccak256(now, msg.sender, randNonce)) % 100;
```

这个方法首先拿到 `now` 的时间戳、 `msg.sender`、 以及一个自增数 `nonce` （一个仅会被使用一次的数，这样我们就不会对相同的输入值调用一次以上哈希函数了）。

然后利用 `keccak` 把输入的值转变为一个哈希值, 再将哈希值转换为 `uint`, 然后利用 `% 100` 来取最后两位, 就生成了一个0到100之间随机数了。

####这个方法很容易被不诚实的节点攻击

在以太坊上, 当你在和一个合约上调用函数的时候, 你会把它广播给一个节点或者在网络上的 **transaction** 节点们。 网络上的节点将收集很多事务, 试着成为第一个解决计算密集型数学问题的人，作为“工作证明”，然后将“工作证明”(Proof of Work, PoW)和事务一起作为一个 **block** 发布在网络上。

一旦一个节点解决了一个PoW, 其他节点就会停止尝试解决这个 PoW, 并验证其他节点的事务列表是有效的，然后接受这个节点转而尝试解决下一个节点。

**这就让我们的随机数函数变得可利用了**

我们假设我们有一个硬币翻转合约——正面你赢双倍钱，反面你输掉所有的钱。假如它使用上面的方法来决定是正面还是反面 (`random >= 50` 算正面, `random < 50` 算反面)。

如果我正运行一个节点，我可以 **只对我自己的节点** 发布一个事务，且不分享它。 我可以运行硬币翻转方法来偷窥我的输赢 — 如果我输了，我就不把这个事务包含进我要解决的下一个区块中去。我可以一直运行这个方法，直到我赢得了硬币翻转并解决了下一个区块，然后获利。

####所以我们该如何在以太坊上安全地生成随机数呢

因为区块链的全部内容对所有参与者来说是透明的， 这就让这个问题变得很难，它的解决方法不在本课程讨论范围，你可以阅读 [这个 StackOverflow 上的讨论](https://ethereum.stackexchange.com/questions/191/how-can-i-securely-generate-a-random-number-in-my-smart-contract) 来获得一些主意。 一个方法是利用 **oracle** 来访问以太坊区块链之外的随机数函数。

当然， 因为网络上成千上万的以太坊节点都在竞争解决下一个区块，我能成功解决下一个区块的几率非常之低。 这将花费我们巨大的计算资源来开发这个获利方法 — 但是如果奖励异常地高(比如我可以在硬币翻转函数中赢得 1个亿)， 那就很值得去攻击了。

所以尽管这个方法在以太坊上不安全，在实际中，除非我们的随机函数有一大笔钱在上面，你游戏的用户一般是没有足够的资源去攻击的。

因为在这个教程中，我们只是在编写一个简单的游戏来做演示，也没有真正的钱在里面，所以我们决定接受这个不足之处，使用这个简单的随机数生成函数。但是要谨记它是不安全的。

### 重构通用逻辑

不管谁调用我们的 `attack` 函数 —— 我们想确保用户的确拥有他们用来攻击的僵尸。如果你能用其他人的僵尸来攻击将是一个很大的安全问题。

你能想一下我们如何添加一个检查步骤来看看调用这个函数的人就是他们传入的 `_zombieId` 的拥有者么？

想一想，看看你能不能自己找到一些答案。

花点时间…… 参考我们前面课程的代码来获得灵感。

答案在下面，在你有一些想法之前不要继续阅读。

#####答案

我们在前面的课程里面已经做过很多次这样的检查了。 在 `changeName()`, `changeDna()`, 和 `feedAndMultiply()`里，我们做过这样的检查：

```
require(msg.sender == zombieToOwner[_zombieId]);
```

这和我们 `attack` 函数将要用到的检查逻辑是相同的。 正因我们要多次调用这个检查逻辑，让我们把它移到它自己的 `modifier` 中来清理代码并避免重复编码。

## 第五章

### 以太坊上的代币

让我们来聊聊 **代币**.

如果你对以太坊的世界有一些了解，你很可能听过人们聊到代币——尤其是 **ERC20 代币**.

一个 **代币** 在以太坊基本上就是一个遵循一些共同规则的智能合约——即它实现了所有其他代币合约共享的一组标准函数，例如 `transfer(address _to, uint256 _value)` 和 `balanceOf(address _owner)`.

在智能合约内部，通常有一个映射， `mapping(address => uint256) balances`，用于追踪每个地址还有多少余额。

所以基本上一个代币只是一个追踪谁拥有多少该代币的合约，和一些可以让那些用户将他们的代币转移到其他地址的函数。

### 它为什么重要呢？

由于所有 ERC20 代币共享具有相同名称的同一组函数，它们都可以以相同的方式进行交互。

这意味着如果你构建的应用程序能够与一个 ERC20 代币进行交互，那么它就也能够与任何 ERC20 代币进行交互。 这样一来，将来你就可以轻松地将更多的代币添加到你的应用中，而无需进行自定义编码。 你可以简单地插入新的代币合约地址，然后哗啦，你的应用程序有另一个它可以使用的代币了。

其中一个例子就是交易所。 当交易所添加一个新的 ERC20 代币时，实际上它只需要添加与之对话的另一个智能合约。 用户可以让那个合约将代币发送到交易所的钱包地址，然后交易所可以让合约在用户要求取款时将代币发送回给他们。

交易所只需要实现这种转移逻辑一次，然后当它想要添加一个新的 ERC20 代币时，只需将新的合约地址添加到它的数据库即可。

### 其他代币标准

对于像货币一样的代币来说，ERC20 代币非常酷。 但是要在我们僵尸游戏中代表僵尸就并不是特别有用。

首先，僵尸不像货币可以分割 —— 我可以发给你 0.237 以太，但是转移给你 0.237 的僵尸听起来就有些搞笑。

其次，并不是所有僵尸都是平等的。 你的2级僵尸"**Steve**"完全不能等同于我732级的僵尸"**H4XF13LD MORRIS 💯💯😎💯💯**"。

有另一个代币标准更适合如 CryptoZombies 这样的加密收藏品——它们被称为**ERC721 代币.**

**ERC721 代币**是**不**能互换的，因为每个代币都被认为是唯一且不可分割的。 你只能以整个单位交易它们，并且每个单位都有唯一的 ID。 这些特性正好让我们的僵尸可以用来交易。

> 请注意，使用像 ERC721 这样的标准的优势就是，我们不必在我们的合约中实现拍卖或托管逻辑，这决定了玩家能够如何交易／出售我们的僵尸。 如果我们符合规范，其他人可以为加密可交易的 ERC721 资产搭建一个交易所平台，我们的 ERC721 僵尸将可以在该平台上使用。 所以使用代币标准相较于使用你自己的交易逻辑有明显的好处。

### ERC721 标准, 多重继承

让我们来看一看 ERC721 标准：

```
contract ERC721 {
  event Transfer(address indexed _from, address indexed _to, uint256 _tokenId);
  event Approval(address indexed _owner, address indexed _approved, uint256 _tokenId);

  function balanceOf(address _owner) public view returns (uint256 _balance);
  function ownerOf(uint256 _tokenId) public view returns (address _owner);
  function transfer(address _to, uint256 _tokenId) public;
  function approve(address _to, uint256 _tokenId) public;
  function takeOwnership(uint256 _tokenId) public;
}
```

这是我们需要实现的方法列表，我们将在接下来的章节中逐个学习。

虽然看起来很多，但不要被吓到了！我们在这里就是准备带着你一步一步了解它们的。

> 注意： ERC721目前是一个 *草稿*，还没有正式商定的实现。在本教程中，我们使用的是 OpenZeppelin 库中的当前版本，但在未来正式发布之前它可能会有更改。 所以把这 **一个** 可能的实现当作考虑，但不要把它作为 ERC721 代币的官方标准。

### 实现一个代币合约

在实现一个代币合约的时候，我们首先要做的是将接口复制到它自己的 Solidity 文件并导入它，`import "./erc721.sol";`。 接着，让我们的合约继承它，然后我们用一个函数定义来重写每个方法。

但等一下—— `ZombieOwnership`已经继承自 `ZombieAttack`了 —— 它如何能够也继承于 `ERC721`呢？

幸运的是在Solidity，你的合约可以继承自多个合约，参考如下：

```
contract SatoshiNakamoto is NickSzabo, HalFinney {
  // 啧啧啧，宇宙的奥秘泄露了
}
```

正如你所见，当使用多重继承的时候，你只需要用逗号 `,` 来隔开几个你想要继承的合约。在上面的例子中，我们的合约继承自 `NickSzabo` 和 `HalFinney`。

### balanceOf 和 ownerOf

我们来深入讨论一下 ERC721 的实现。

我们已经把所有你需要在本课中实现的函数的空壳复制好了。

在本章节，我们将实现头两个方法： `balanceOf` 和 `ownerOf`。

### `balanceOf`

```
  function balanceOf(address _owner) public view returns (uint256 _balance);
```

这个函数只需要一个传入 `address` 参数，然后返回这个 `address` 拥有多少代币。

在我们的例子中，我们的“代币”是僵尸。你还记得在我们 DApp 的哪里存储了一个主人拥有多少只僵尸吗？

### `ownerOf`

```
  function ownerOf(uint256 _tokenId) public view returns (address _owner);
```

这个函数需要传入一个代币 ID 作为参数 (我们的情况就是一个僵尸 ID)，然后返回该代币拥有者的 `address`。

同样的，因为在我们的 DApp 里已经有一个 `mapping` (映射) 存储了这个信息，所以对我们来说这个实现非常直接清晰。我们可以只用一行 `return`语句来实现这个函数。

> 注意：要记得， `uint256` 等同于`uint`。我们从课程的开始一直在代码中使用 `uint`，但从现在开始我们将在这里用 `uint256`，因为我们直接从规范中复制粘贴。

### ERC721: 转移标准

好了，我们将冲突修复了！

现在我们将通过学习把所有权从一个人转移给另一个人来继续我们的 ERC721 规范的实现。

注意 ERC721 规范有两种不同的方法来转移代币：

```
function transfer(address _to, uint256 _tokenId) public;

function approve(address _to, uint256 _tokenId) public;
function takeOwnership(uint256 _tokenId) public;
```

1. 第一种方法是代币的拥有者调用`transfer` 方法，传入他想转移到的 `address` 和他想转移的代币的 `_tokenId`。
2. 第二种方法是代币拥有者首先调用 `approve`，然后传入与以上相同的参数。接着，该合约会存储谁被允许提取代币，通常存储到一个 `mapping (uint256 => address)` 里。然后，当有人调用 `takeOwnership` 时，合约会检查 `msg.sender` 是否得到拥有者的批准来提取代币，如果是，则将代币转移给他。

你注意到了吗，`transfer` 和 `takeOwnership` 都将包含相同的转移逻辑，只是以相反的顺序。 （一种情况是代币的发送者调用函数；另一种情况是代币的接收者调用它）。

所以我们把这个逻辑抽象成它自己的私有函数 `_transfer`，然后由这两个函数来调用它。 这样我们就不用写重复的代码了。

### ERC721: 批准

现在，让我们来实现 `approve`。

记住，使用 `approve` 或者 `takeOwnership` 的时候，转移有2个步骤：

1. 你，作为所有者，用新主人的 `address` 和你希望他获取的 `_tokenId` 来调用 `approve`
2. 新主人用 `_tokenId` 来调用 `takeOwnership`，合约会检查确保他获得了批准，然后把代币转移给他。

因为这发生在2个函数的调用中，所以在函数调用之间，我们需要一个数据结构来存储什么人被批准获取什么。

### ERC721: takeOwnership

太棒了，现在让我们完成最后一个函数来结束 ERC721 的实现。

最后一个函数 `takeOwnership`， 应该只是简单地检查以确保 `msg.sender` 已经被批准来提取这个代币或者僵尸。若确认，就调用 `_transfer`；

### 预防溢出

恭喜你，我们完成了 ERC721 的实现。

并不是很复杂，对吧？很多类似的以太坊概念，当你只听人们谈论它们的时候，会觉得很复杂。所以最简单的理解方式就是你自己来实现它。

不过要记住那只是最简单的实现。还有很多的特性我们也许想加入到我们的实现中来，比如一些额外的检查，来确保用户不会不小心把他们的僵尸转移给`0` 地址（这被称作 “烧币”, 基本上就是把代币转移到一个谁也没有私钥的地址，让这个代币永远也无法恢复）。 或者在 DApp 中加入一些基本的拍卖逻辑。（你能想出一些实现的方法么？）

### 合约安全增强: 溢出和下溢

我们将来学习你在编写智能合约的时候需要注意的一个主要的安全特性：防止溢出和下溢。

什么是 **溢出** (**overflow**)?

假设我们有一个 `uint8`, 只能存储8 bit数据。这意味着我们能存储的最大数字就是二进制 `11111111` (或者说十进制的 2^8 - 1 = 255).

来看看下面的代码。最后 `number` 将会是什么值？

```
uint8 number = 255;
number++;
```

在这个例子中，我们导致了溢出 — 虽然我们加了1， 但是 `number` 出乎意料地等于 `0`了。 (如果你给二进制 `11111111` 加1, 它将被重置为 `00000000`，就像钟表从 `23:59` 走向 `00:00`)。

下溢(`underflow`)也类似，如果你从一个等于 `0` 的 `uint8` 减去 `1`, 它将变成 `255` (因为 `uint` 是无符号的，其不能等于负数)。

虽然我们在这里不使用 `uint8`，而且每次给一个 `uint256` 加 `1` 也不太可能溢出 (2^256 真的是一个很大的数了)，在我们的合约中添加一些保护机制依然是非常有必要的，以防我们的 DApp 以后出现什么异常情况。

### 使用 SafeMath

为了防止这些情况，OpenZeppelin 建立了一个叫做 SafeMath 的 **库**(**library**)，默认情况下可以防止这些问题。

不过在我们使用之前…… 什么叫做库?

一个**库** 是 Solidity 中一种特殊的合约。其中一个有用的功能是给原始数据类型增加一些方法。

比如，使用 SafeMath 库的时候，我们将使用 `using SafeMath for uint256` 这样的语法。 SafeMath 库有四个方法 — `add`， `sub`， `mul`， 以及 `div`。现在我们可以这样来让 `uint256` 调用这些方法：

```
using SafeMath for uint256;

uint256 a = 5;
uint256 b = a.add(3); // 5 + 3 = 8
uint256 c = a.mul(2); // 5 * 2 = 10
```

我们将在下一章来学习这些方法，不过现在我们先将 SafeMath 库添加进我们的合约。

### SafeMath 第二部分

来看看 SafeMath 的部分代码:

```
library SafeMath {

  function mul(uint256 a, uint256 b) internal pure returns (uint256) {
    if (a == 0) {
      return 0;
    }
    uint256 c = a * b;
    assert(c / a == b);
    return c;
  }

  function div(uint256 a, uint256 b) internal pure returns (uint256) {
    // assert(b > 0); // Solidity automatically throws when dividing by 0
    uint256 c = a / b;
    // assert(a == b * c + a % b); // There is no case in which this doesn't hold
    return c;
  }

  function sub(uint256 a, uint256 b) internal pure returns (uint256) {
    assert(b <= a);
    return a - b;
  }

  function add(uint256 a, uint256 b) internal pure returns (uint256) {
    uint256 c = a + b;
    assert(c >= a);
    return c;
  }
}
```

首先我们有了 `library` 关键字 — 库和 `合约`很相似，但是又有一些不同。 就我们的目的而言，库允许我们使用 `using` 关键字，它可以自动把库的所有方法添加给一个数据类型：

```
using SafeMath for uint;
// 这下我们可以为任何 uint 调用这些方法了
uint test = 2;
test = test.mul(3); // test 等于 6 了
test = test.add(5); // test 等于 11 了
```

注意 `mul` 和 `add` 其实都需要两个参数。 在我们声明了 `using SafeMath for uint` 后，我们用来调用这些方法的 `uint` 就自动被作为第一个参数传递进去了(在此例中就是 `test`)

我们来看看 `add` 的源代码看 SafeMath 做了什么:

```
function add(uint256 a, uint256 b) internal pure returns (uint256) {
  uint256 c = a + b;
  assert(c >= a);
  return c;
}
```

基本上 `add` 只是像 `+` 一样对两个 `uint` 相加， 但是它用一个 `assert`语句来确保结果大于 `a`。这样就防止了溢出。

`assert` 和 `require` 相似，若结果为否它就会抛出错误。 `assert` 和 `require` 区别在于，`require` 若失败则会返还给用户剩下的 gas， `assert` 则不会。所以大部分情况下，你写代码的时候会比较喜欢 `require`，`assert` 只在代码可能出现严重错误的时候使用，比如 `uint`溢出。

所以简而言之， SafeMath 的 `add`， `sub`， `mul`， 和 `div` 方法只做简单的四则运算，然后在发生溢出或下溢的时候抛出错误。

### 在我们的代码里使用 SafeMath。

为了防止溢出和下溢，我们可以在我们的代码里找 `+`， `-`， `*`， 或 `/`，然后替换为 `add`, `sub`, `mul`, `div`.

比如，与其这样做:

```
myUint++;
```

我们这样做：

```
myUint = myUint.add(1);
```

### SafeMath 第三部分

太好了，这下我们的 ERC721 实现不会有溢出或者下溢了。

回头看看我们在之前课程写的代码，还有其他几个地方也有可能导致溢出或下溢。

比如， 在 ZombieAttack 里面我们有：

```
myZombie.winCount++;
myZombie.level++;
enemyZombie.lossCount++;
```

我们同样应该在这些地方防止溢出。（通常情况下，总是使用 SafeMath 而不是普通数学运算是个好主意，也许在以后 Solidity 的新版本里这点会被默认实现，但是现在我们得自己在代码里实现这些额外的安全措施）。

不过我们遇到个小问题 — `winCount` 和 `lossCount` 是 `uint16`， 而 `level` 是 `uint32`。 所以如果我们用这些作为参数传入 SafeMath 的 `add` 方法。 它实际上并不会防止溢出，因为它会把这些变量都转换成 `uint256`:

```
function add(uint256 a, uint256 b) internal pure returns (uint256) {
  uint256 c = a + b;
  assert(c >= a);
  return c;
}

// 如果我们在`uint8` 上调用 `.add`。它将会被转换成 `uint256`.
// 所以它不会在 2^8 时溢出，因为 256 是一个有效的 `uint256`.
```

这就意味着，我们需要再实现两个库来防止 `uint16` 和 `uint32` 溢出或下溢。我们可以将其命名为 `SafeMath16` 和 `SafeMath32`。

代码将和 SafeMath 完全相同，除了所有的 `uint256` 实例都将被替换成 `uint32` 或 `uint16`。

我们已经将这些代码帮你写好了，打开 `safemath.sol` 合约看看代码吧。

现在我们需要在 ZombieFactory 里使用它们。

###SafeMath 第4部分

真棒，现在我们已经为我们的 DApp 里面用到的 `uint` 数据类型都实现了 SafeMath 了。

让我们把 `ZombieAttack` 里所有潜在的问题都修复了吧。 （其实在 `ZombieHelper` 里也有一处 `zombies[_zombieId].level++;` 需要修复，不过我们已经帮你做好了，这样我们就不用再来一章了 😉）。

### 注释

僵尸游戏的 Solidity 代码终于完成啦。

在以后的课程中，我们将学习如何将游戏部署到以太坊，以及如何和 Web3.js 交互。

不过在你离开第五课之前，我们来谈谈如何 **给你的代码添加注释**.

###注释语法

Solidity 里的注释和 JavaScript 相同。在我们的课程中你已经看到了不少单行注释了：

```
// 这是一个单行注释，可以理解为给自己或者别人看的笔记
```

只要在任何地方添加一个 `//` 就意味着你在注释。如此简单所以你应该经常这么做。

不过我们也知道你的想法：有时候单行注释是不够的。毕竟你生来话痨。

所以我们有了多行注释：

```
contract CryptoZombies { 
  /* 这是一个多行注释。我想对所有花时间来尝试这个编程课程的人说声谢谢。
  它是免费的，并将永远免费。但是我们依然倾注了我们的心血来让它变得更好。

   要知道这依然只是区块链开发的开始而已，虽然我们已经走了很远，
   仍然有很多种方式来让我们的社区变得更好。
   如果我们在哪个地方出了错，欢迎在我们的 github 提交 PR 或者 issue 来帮助我们改进：
    https://github.com/loomnetwork/cryptozombie-lessons

    或者，如果你有任何的想法、建议甚至仅仅想和我们打声招呼，欢迎来我们的电报群：
     https://t.me/loomnetworkcn
  */
}
```

特别是，最好为你合约中每个方法添加注释来解释它的预期行为。这样其他开发者（或者你自己，在6个月以后再回到这个项目中）可以很快地理解你的代码而不需要逐行阅读所有代码。

Solidity 社区所使用的一个标准是使用一种被称作 **natspec** 的格式，看起来像这样：

```
/// @title 一个简单的基础运算合约
/// @author H4XF13LD MORRIS 💯💯😎💯💯
/// @notice 现在，这个合约只添加一个乘法
contract Math {
  /// @notice 两个数相乘
  /// @param x 第一个 uint
  /// @param y  第二个 uint
  /// @return z  (x * y) 的结果
  /// @dev 现在这个方法不检查溢出
  function multiply(uint x, uint y) returns (uint z) {
    // 这只是个普通的注释，不会被 natspec 解释
    z = x * y;
  }
}
```

`@title`（标题） 和 `@author` （作者）很直接了.

`@notice` （须知）向 **用户** 解释这个方法或者合约是做什么的。 `@dev`（开发者） 是向开发者解释更多的细节。

`@param` （参数）和 `@return` （返回） 用来描述这个方法需要传入什么参数以及返回什么值。

注意你并不需要每次都用上所有的标签，它们都是可选的。不过最少，写下一个 `@dev` 注释来解释每个方法是做什么的。

## 第六章

但是若没有一个互动界面让用户来使用，一个 DApp 就不能称之为完整。

在这一课，我们将来学习如果用一个名为 **Web3.js** 的库来为你的 DApp 创建一个基本的前端界面，和你的智能合约交互。

需要注意的是这个 APP 界面将使用 **JavaScript** 来写，并不是 Solidity。因为我们的课程专注于 Ethereum / Solidity，我们就暂时假定你已经会用HTML, JavaScript(包括 ES6 [promises](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise))，以及 JQuery 写网站了。因此我们不会花时间来介绍这些技术的基础知识。

### 介绍 Web3.js

完成第五课以后，我们的僵尸 DApp 的 Solidity 合约部分就完成了。现在我们来做一个基本的网页好让你的用户能玩它。 要做到这一点，我们将使用以太坊基金发布的 JavaScript 库 —— **Web3.js**.

###什么是 Web3.js?

还记得么？以太坊网络是由节点组成的，每一个节点都包含了区块链的一份拷贝。当你想要调用一份智能合约的一个方法，你需要从其中一个节点中查找并告诉它:

1. 智能合约的地址
2. 你想调用的方法，以及
3. 你想传入那个方法的参数

以太坊节点只能识别一种叫做 **JSON-RPC** 的语言。这种语言直接读起来并不好懂。当你你想调用一个合约的方法的时候，需要发送的查询语句将会是这样的：

```
// 哈……祝你写所有这样的函数调用的时候都一次通过
// 往右边拉…… ==>
{"jsonrpc":"2.0","method":"eth_sendTransaction","params":[{"from":"0xb60e8dd61c5d32be8058bb8eb970870f07233155","to":"0xd46e8dd67c5d32be8058bb8eb970870f07244567","gas":"0x76c0","gasPrice":"0x9184e72a000","value":"0x9184e72a","data":"0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675"}],"id":1}
```

幸运的是 Web3.js 把这些令人讨厌的查询语句都隐藏起来了， 所以你只需要与方便易懂的 JavaScript 界面进行交互即可。

你不需要构建上面的查询语句，在你的代码中调用一个函数看起来将是这样：

```
CryptoZombies.methods.createRandomZombie("Vitalik Nakamoto 🤔")
  .send({ from: "0xb60e8dd61c5d32be8058bb8eb970870f07233155", gas: "3000000" })
```

我们将在接下来的几章详细解释这些语句，不过首先我们来把 Web3.js 环境搭建起来。

###准备好了么？

取决于你的项目工作流程和你的爱好，你可以用一些常用工具把 Web3.js 添加进来：

```
// 用 NPM
npm install web3

// 用 Yarn
yarn add web3

// 用 Bower
bower install web3

// ...或者其他。
```

甚至，你可以从 [github](https://github.com/ethereum/web3.js/blob/1.0/dist/web3.min.js) 直接下载压缩后的 `.js` 文件 然后包含到你的项目文件中：

```
<script language="javascript" type="text/javascript" src="web3.min.js"></script>
```

因为我们不想让你花太多在项目环境搭建上，在本教程中我们将使用上面的 `script` 标签来将 Web3.js 引入。

### Web3 提供者

首先我们需要 **Web3 Provider**.

要记住，以太坊是由共享同一份数据的相同拷贝的 **节点** 构成的。 在 Web3.js 里设置 Web3 的 `Provider`（提供者） 告诉我们的代码应该和 **哪个节点** 交互来处理我们的读写。这就好像在传统的 Web 应用程序中为你的 API 调用设置远程 Web 服务器的网址。

你可以运行你自己的以太坊节点来作为 Provider。 不过，有一个第三方的服务，可以让你的生活变得轻松点，让你不必为了给你的用户提供DApp而维护一个以太坊节点— **Infura**.

###Infura

[Infura](https://infura.io/) 是一个服务，它维护了很多以太坊节点并提供了一个缓存层来实现高速读取。你可以用他们的 API 来免费访问这个服务。 用 Infura 作为节点提供者，你可以不用自己运营节点就能很可靠地向以太坊发送、接收信息。

你可以通过这样把 Infura 作为你的 Web3 节点提供者：

```
var web3 = new Web3(new Web3.providers.WebsocketProvider("wss://mainnet.infura.io/ws"));
```

不过，因为我们的 DApp 将被很多人使用，这些用户不单会从区块链读取信息，还会向区块链 **写** 入信息，我们需要用一个方法让用户可以用他们的私钥给事务签名。

> 注意: 以太坊 (以及通常意义上的 blockchains )使用一个公钥/私钥对来对给事务做数字签名。把它想成一个数字签名的异常安全的密码。这样当我修改区块链上的数据的时候，我可以用我的公钥来 **证明** 我就是签名的那个。但是因为没人知道我的私钥，所以没人能伪造我的事务。

加密学非常复杂，所以除非你是个专家并且的确知道自己在做什么，你最好不要在你应用的前端中管理你用户的私钥。

不过幸运的是，你并不需要，已经有可以帮你处理这件事的服务了： **Metamask**.

###Metamask

[Metamask](https://metamask.io/) 是 Chrome 和 Firefox 的浏览器扩展， 它能让用户安全地维护他们的以太坊账户和私钥， 并用他们的账户和使用 Web3.js 的网站互动（如果你还没用过它，你肯定会想去安装的——这样你的浏览器就能使用 Web3.js 了，然后你就可以和任何与以太坊区块链通信的网站交互了）

作为开发者，如果你想让用户从他们的浏览器里通过网站和你的DApp交互（就像我们在 CryptoZombies 游戏里一样），你肯定会想要兼容 Metamask 的。

> **注意**: Metamask 默认使用 Infura 的服务器做为 web3 提供者。 就像我们上面做的那样。不过它还为用户提供了选择他们自己 Web3 提供者的选项。所以使用 Metamask 的 web3 提供者，你就给了用户选择权，而自己无需操心这一块。

###使用 Metamask 的 web3 提供者

Metamask 把它的 web3 提供者注入到浏览器的全局 JavaScript对象`web3`中。所以你的应用可以检查 `web3` 是否存在。若存在就使用 `web3.currentProvider` 作为它的提供者。

这里是一些 Metamask 提供的示例代码，用来检查用户是否安装了MetaMask，如果没有安装就告诉用户需要安装MetaMask来使用我们的应用。

```
window.addEventListener('load', function() {

  // 检查web3是否已经注入到(Mist/MetaMask)
  if (typeof web3 !== 'undefined') {
    // 使用 Mist/MetaMask 的提供者
    web3js = new Web3(web3.currentProvider);
  } else {
    // 处理用户没安装的情况， 比如显示一个消息
    // 告诉他们要安装 MetaMask 来使用我们的应用
  }

  // 现在你可以启动你的应用并自由访问 Web3.js:
  startApp()

})
```

你可以在你所有的应用中使用这段样板代码，好检查用户是否安装以及告诉用户安装 MetaMask。

> 注意: 除了MetaMask，你的用户也可能在使用其他他的私钥管理应用，比如 **Mist** 浏览器。不过，它们都实现了相同的模式来注入 `web3` 变量。所以我这里描述的方法对两者是通用的。

### 和合约对话

现在，我们已经用 MetaMask 的 Web3 提供者初始化了 Web3.js。接下来就让它和我们的智能合约对话吧。

Web3.js 需要两个东西来和你的合约对话: 它的 **地址** 和它的 **ABI**。

###合约地址

在你写完了你的智能合约后，你需要编译它并把它部署到以太坊。我们将在**下一课**中详述**部署**，因为它和写代码是截然不同的过程，所以我们决定打乱顺序，先来讲 Web3.js。

在你部署智能合约以后，它将获得一个以太坊上的永久地址。如果你还记得第二课，CryptoKitties 在以太坊上的地址是 `0x06012c8cf97BEaD5deAe237070F9587f8E7A266d`。

你需要在部署后复制这个地址以来和你的智能合约对话。

###合约 ABI

另一个 Web3.js 为了要和你的智能合约对话而需要的东西是 **ABI**。

ABI 意为应用二进制接口（Application Binary Interface）。 基本上，它是以 JSON 格式表示合约的方法，告诉 Web3.js 如何以合同理解的方式格式化函数调用。

当你编译你的合约向以太坊部署时(我们将在第七课详述)， Solidity 编译器会给你 ABI，所以除了合约地址，你还需要把这个也复制下来。

因为我们这一课不会讲述部署，所以现在我们已经帮你编译了 ABI 并放在了名为`cryptozombies_abi.js`，文件中，保存在一个名为 `cryptoZombiesABI` 的变量中。

如果我们将`cryptozombies_abi.js` 包含进我们的项目，我们就能通过那个变量访问 CryptoZombies ABI 。

###实例化 Web3.js

一旦你有了合约的地址和 ABI，你可以像这样来实例化 Web3.js。

```
// 实例化 myContract
var myContract = new web3js.eth.Contract(myABI, myContractAddress);
```

### 调用和合约函数

我们的合约配置好了！现在来用 Web3.js 和它对话。

Web3.js 有两个方法来调用我们合约的函数: `call` and `send`.

### Call

`call` 用来调用 `view` 和 `pure` 函数。它只运行在本地节点，不会在区块链上创建事务。

> **复习:** `view` 和 `pure` 函数是只读的并不会改变区块链的状态。它们也不会消耗任何gas。用户也不会被要求用MetaMask对事务签名。

使用 Web3.js，你可以如下 `call` 一个名为`myMethod`的方法并传入一个 `123` 作为参数：

```
myContract.methods.myMethod(123).call()
```

### Send

`send` 将创建一个事务并改变区块链上的数据。你需要用 `send` 来调用任何非 `view` 或者 `pure` 的函数。

> **注意:** `send` 一个事务将要求用户支付gas，并会要求弹出对话框请求用户使用 Metamask 对事务签名。在我们使用 Metamask 作为我们的 web3 提供者的时候，所有这一切都会在我们调用 `send()` 的时候自动发生。而我们自己无需在代码中操心这一切，挺爽的吧。

使用 Web3.js, 你可以像这样 `send` 一个事务调用`myMethod` 并传入 `123`作为参数：

```
myContract.methods.myMethod(123).send()
```

语法几乎 `call()`一模一样。

## 获取僵尸数据

来看一个使用 `call` 读取我们合约数据的真实例子

回忆一下，我们定义我们的僵尸数组为 `公开`(public):

```
Zombie[] public zombies;
```

在 Solidity 里，当你定义一个 `public`变量的时候， 它将自动定义一个公开的 "getter" 同名方法， 所以如果你像要查看 id 为 `15` 的僵尸，你可以像一个函数一样调用它： `zombies(15)`.

这是如何在外面的前端界面中写一个 JavaScript 方法来传入一个僵尸 id，在我们的合同中查询那个僵尸并返回结果

> 注意: 本课中所有的示例代码都使用 Web3.js 的 **1.0 版**，此版本使用的是 Promises 而不是回调函数。你在线上看到的其他教程可能还在使用老版的 Web3.js。在1.0版中，语法改变了不少。如果你从其他教程中复制代码，先确保你们使用的是相同版本的Web3.js。

```
function getZombieDetails(id) {
  return cryptoZombies.methods.zombies(id).call()
}

// 调用函数并做一些其他事情
getZombieDetails(15)
.then(function(result) {
  console.log("Zombie 15: " + JSON.stringify(result));
});
```

我们来看看这里都做了什么

`cryptoZombies.methods.zombies(id).call()` 将和 Web3 提供者节点通信，告诉它返回从我们的合约中的 `Zombie[] public zombies`，`id`为传入参数的僵尸信息。

注意这是 **异步的**，就像从外部服务器中调用API。所以 Web3 在这里返回了一个 Promises. (如果你对 JavaScript的 Promises 不了解，最好先去学习一下这方面知识再继续)。

一旦那个 `promise` 被 `resolve`, (意味着我们从 Web3 提供者那里获得了响应)，我们的例子代码将执行 `then` 语句中的代码，在控制台打出 `result`。

`result` 是一个像这样的 JavaScript 对象：

```
{
  "name": "H4XF13LD MORRIS'S COOLER OLDER BROTHER",
  "dna": "1337133713371337",
  "level": "9999",
  "readyTime": "1522498671",
  "winCount": "999999999",
  "lossCount": "0" // Obviously.
}
```

我们可以用一些前端逻辑代码来解析这个对象并在前端界面友好展示。