# Solidity知识点

[TOC]

## 函数可见性


- external:外部函数，可以在其他合约中调用。外部函数`f`不能在内部调用，但是可以通过`this.f()`调用
- public:公共函数，可以在内部调用，也可以通过消息调用。对于公共状态变量，会生成getter函数
- internal:这些函数和状态变量只能在内部访问，不能使用`this`调用
- private:私有函数。只能在定义的合约中可见，其他合约都不可见(包括继承合约)

区块链外部的所有观察者都可以看到合同中的所有内容，`private`只是阻止其他合约访问和修改信息

```
pragma solidity >=0.4.0 < 0.6.0;

contract C {
    uint private data;
    // private:只能在当前C合约内调用
    function f(uint a) private pure returns (uint b) {
        return a + 1;
    }
    
    // public:公共函数,都可以调用
    function setData(uint a) public {
        data = a;
    }
    
    function getData() public view returns (uint) {
        return data;
    }
    // internal:内部函数，只能在当前C合约中或者继承的子合约中调用
    function compute(uint a, uint b) internal pure returns(uint) {
        return a + b;
    }
    
}


contract D {
    function readData() public {
        C c = new C();
        // uint local = c.f(7);  // 错误:f是private函数，只能在C合约内部调用
        c.setData(3);
        uint local = c.getData();  
        // local = c.compute(3,5); // 错误:compute是internal函数只能在内部或者继承的子合约内部调用
    }
}

contract E is C {
    function g() public {
        C c = new C();
        uint val = compute(3,5);  // E继承自C，可以调用compute方法
    }
}
```


