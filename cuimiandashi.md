### Solidity中的变量类型
 **数值类型(Value Type)**：包括布尔型，整数型等等





局部变量只有在调用函数时候产生 在函数内部生效

把不需要修改的值用constant定义为常量可以节省gas费

三元运算符



for循环:

continue会不执行后面的代码，直接i++

break会跳出循环

i不能太大，會消耗太多gas



报错控制

require 第一个参数为判断条件，为表达式，如果表达式为false就报错

revert 不能写表达式 

assert 断言



函数修改器

modifier

_表示函数运行的地方，会先执行函数修改器中的代码再执行其他代码

sandwich三明治写法： 将函数的代码插入到函数修改器的代码中间执行



构造函数：只会在部署的时候被调用1次，之后再也不能被调用，一般用于初始化变量



Owner合约

功能：设置合约的owner（方法：setOwner) 只有owner可以调用这个方法

在函数修改器里判断地址是否为合约所有者 在setOwner使用这个函数修改器

```
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.4;
contract Ownable {
    address public owner;

    constructor(){
        owner = msg.sender;
    }

    modifier onlyOwner(){
        require(msg.sender == owner,"mot owner");
        _;
    }

    function setOwner(address _newOwner) external onlyOwner{
        require(_newOwner != address(0),"invalid address");
        owner = _newOwner;
    }
}
```





函数返回值

```
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.4;

contract FunctionOutputs {
	//返回多个值
    function returnMany() public pure returns (uint,bool){
        return (1,true);
    }
    
    function named() public pure returns (uint x, bool b){
        x = 1;
        b = true;
    }

	//命名式返回
	function  named2() public pure returns (uint x, bool b){
        return(2,true)
    }

    function destructingAssignments() public pure {
        (uint x, bool b) = returnMany();
        (,bool _b) = returnMany();
    }
}
```





数组

- 动长数组 长度可以变化
- 定长数组

数组初始化

数组方法：delete、push、pop、length

delete只是将元素替换为0

pop是删除最后一个元素

在内存中只能定义定长数组 所以不能使用pop、删除和push

访问数组的某个元素用下标来访问

如果需要返回数组的所有元素 就需要在函数中返回该数组



删除数组元素的两种方法

法1：消耗gas较多

通过移动位置删除数组元素

```js
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.4;

contract ArrayShift {
    uint[] public arr;

    function example() public {
        arr = [1,2,3];
        delete arr[1];
    }

    function remove(uint _index) public {
        require(_index <arr.length,"index out of bound");
        for(uint i = _index;i<arr.length-1;i++){
            arr[i] = arr[i+1];
        }
        arr.pop();
    }

    function test() external {
        arr =[1,2,3,4,5];
        remove(2);//[1,2,4,5];
        assert(arr[0] == 1);
        assert(arr[1] == 2);
        assert(arr[2] == 4);
        assert(arr[3] == 5);
        assert(arr.length == 4);

        arr =[1];
        remove(0);
        assert(arr.length == 0);
    }
}
```

法2：数组中元素的顺序被打乱

```
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.4;

contract ArrayReplaceLast {
    uint256[] public arr;

    function remove(uint256 _index) public {
        arr[_index] = arr[arr.length - 1];
        arr.pop();
    }

    function test() external {
        arr = [1, 2, 3, 4];
        remove(1);

        assert(arr.length == 3);

        remove(2);
        assert(arr.length == 2);
    }
}
```



映射

如何声明一个映射

- 简单映射
- 嵌套型映射

```
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.4;

contract Mapping {
  mapping(address => uint) public balances;
  mapping(address => mapping(address => bool))  public isFriend;

  function examples() external {
    //   设置balances的值
      balances[msg.sender] = 123;
    //   读取
    uint bal = balances[msg.sender];
    uint bal2 = balances[address(1)];//0

    balances[msg.sender] += 456;
    delete balances[msg.sender];//0删除值 默认为0

    isFriend[msg.sender][address(this)] = true;
  }
}

```

可迭代的映射

```js
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.4;

contract IterableMapping {
    mapping(address => uint) public balances;
    mapping(address => bool) public inserted;
    address[] public keys;

    function set(address _key,uint _val) external {
         balances[_key] = _val;

         if(!inserted[_key]){
              inserted[_key] = true;
              keys.push(_key);
         }
    }

    function getSize() external view returns (uint){
        return keys.length;
    }

    function first() external view returns (uint) {
        return balances[keys[0]];
    }

    function last() external view returns (uint) {
        return balances[keys[keys.length -1]];
    }
}
```





结构体

```js
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.4;

contract Structs {
  //结构体是一种类型
  //类型的名字要大写
    struct Car {
        string model;
        uint year;
        address owner;
    }

    Car public car;
    Car[] public cars;
    mapping(address => Car[]) public carsByOwner;

    function examples() external {
      //3种设置元素的方式
        Car memory toyota = Car("Toyota",1900,msg.sender);
        Car memory lambo = Car({year:1980,model:"Lamborghini",owner:msg.sender});
        Car memory tesla;
        tesla.model = "Tesla";
        tesla.year = 2010;
        tesla.owner = msg.sender;

        cars.push(toyota);
        cars.push(lambo);
        cars.push(tesla);

        cars.push(Car('Ferrari',2020,msg.sender));
		//放在存储中的变量才可以修改
        Car storage _car = cars[0];
        _car.year =1999;
        delete _car.owner;//可以删除元素中的某个值
      
        delete cars[1];//可以删除整个元素
    }
}
```



枚举

```js
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.4;

contract Enum {
    enum Status {
          // status的默认值就是None
        None,
        Pending,
        Shipped,
        Completed,
        Rejected,
        Canceled
    }

    Status public status;

    struct Order{
        address buyer;
        Status status;//类型里嵌套类型
    }

    Order[] public orders;

    function get() view external returns (Status) {
        return status;
    }

    function set(Status _status) external {
        status = _status;
    }

    function ship()  external{
        status = Status.Shipped;//用shipped的值替换status的值
    }
    function reset() external {
      
        delete status;
    }
}
```





#### 继承

- 简单继承

- 多重继承

  - 调用父合约示例

  ```
  contract Yeye{
      event Log(string msg);

      function hip() public virtual{
          emit Log("Yeye");
      }

      function pop() public virtual{
          emit Log("Yeye");
      }

      function yeye() public virtual{
          emit Log("Yeye");
      }
  }

  contract Baba is Yeye{
      // 重写
      function hip() public virtual override{
          emit Log("Baba");
      }

       function pop() public virtual override{
          emit Log("Baba");
      }
  }

  contract Erzi is Yeye,Baba{
      function hip() public virtual override(Yeye,Baba){
          emit Log("Erzi");
      }
      function pop() public virtual override(Yeye,Baba){
          emit Log("Erzi");
      }
      // 调用父合约的函数
      // 直接调用：
      function callParent() public {
          Yeye.pop();
      }
      // super关键字：
      function callParentSuper() public {
          super.pop();
      }
  }
  ```

  ​

  - 菱形继承

    ```js
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;

    /* 继承树：
      God
     /  \
    Adam Eve
     \  /
    people
    */
    contract God{
        event Log(string message);

        function foo() public virtual{
            emit Log("God.foo called");
        }
        function bar() public virtual{
            emit Log("God.bar called");
        }
    }

    contract Adam is God{
        function foo() public virtual override{
            emit Log("Adam.foo called");
            Adam.foo();
        }

        function bar() public virtual override{
            emit Log("Adam.bar called");
            super.bar();
        }
    }
    contract Eve is God {
        function foo() public virtual override {
            emit Log("Eve.foo called");
            Eve.foo();
        }

        function bar() public virtual override {
            emit Log("Eve.bar called");
            super.bar();
        }
    }

    contract people is Adam,Eve{
        function foo() public override(Adam,Eve){
            super.foo();
        }

        function bar() public override(Adam,Eve){
            super.bar();
        }
    }

    ```

- 修饰器继承

  ```js
  // SPDX-License-Identifier: GPL-3.0
  pragma solidity ^0.8.4;

  contract Base1 {
      modifier exactDividedBy2And3(uint _a) virtual {
          require(_a % 2 == 0 &&_a %3 == 0);
          _;
      } 
  }

  contract  Identifier is Base1 {
      function getExactDividedBy2And3(uint _dividend) public exactDividedBy2And3(_dividend) pure returns(uint,uint){
          return getExactDividedBy2And3WithoutModifier(_dividend);
      }

      function getExactDividedBy2And3WithoutModifier(uint _dividend) public pure returns(uint,uint){
          uint div2 = _dividend /2;
          uint div3 = _dividend /3;
          return (div2,div3);
      }
  }
  ```

  ​

- 构造函数继承

  ```js
  // SPDX-License-Identifier: GPL-3.0
  pragma solidity ^0.8.4;

  abstract contract A {
      uint public a;

      constructor(uint _a){
          a=_a;//100
      }
  }

  contract C is A {
      //传入_c=10
      constructor(uint _c) A(_c * _c){}
  }
  ```

  ​

