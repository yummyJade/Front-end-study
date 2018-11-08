[TOC]





## 第一部分 作用域和闭包

### 第1章 作用域



js代码编译执行中涉及到两个成员，一个是引擎，一个是编译器。

* 引擎 你可以把它视为一个总导演，控制着整个过程
* 编译器 负责分词词法分析（两者区别在于生成器的识别状态）解析等活，比如说把var a = 2分解成 var / a/ =/2，转换成对机器来说有意义的词法单元，再转成一个树结构，再把代码生成为可执行代码，也就是机器指令。

例：对于var a = 2;

分词词法解析生成树结构依然相同，但是在进行代码转换的时候。



引擎会将其视为var a; 和 a =2;

var a，若不存在，即声明一个新的变量，由编译器在当前作用域创建添加一个新变量

a = 2，由编译器生成可执行代码，引擎会在当前作用域之中查找，或者找不到以后向上查找，最后找到变量并赋值。



引擎的查找分为：LHS RHS

直观理解为左右

LHS：也就是查找容器

RHS：也就是找到对应的变量值



```javascript
function foo(a){
    var b = a;
    return a + b;
}
var c = foo(2);

//3 LHS  c =   a = 2(隐式)  b =  
//4 RHS  foo 、 = a  a·· ··b  
```



> 作用域是一套规则，规定在何处以及如何查找变量

作用域嵌套作用域链



LHR RHS在查询失败的情况下处理方式是不同的

LHR查找不到变量，在非严格模式下，会在全局作用域中创建一个新的变量，并将其返回给引擎。而在严格模式下，会抛出ReferenceError错误

RHS查找不到，会抛出ReferenceError错误，如果是对变量进行不合理或者说是非法操作，比如对一个变量进行函数调用，会抛出TypeError

### 第2章 词法作用域

词法作用域的分析是基于代码最初所属的静态结构，而作用域是由书写代码时函数声明的位置决定的。  编译的时候进行分词和词法分析，所以编译器就知道了变量大概所处的位置，从而可以对程序的执行进行预测。



欺骗词法作用域

！！不推荐使用，会降低性能



* eval() 实现代码块的动态插入
* with() 实现词法作用域的添加



关于js的性能

一部分依赖于静态分析，因此加入了eval等就相当于之前的分析都是无效的



### 第4章 提升



```javascript
a = 2;
var a;
console.log(a); //2

//事实上它们等价于

var a;
a = 2;
console.log(a);
```



js代码不是逐行执行的

```javascript
console.log(a);
var a = 2;


//事实上它们等价于

var a;
console.log(a); //undefined
a = 2;

```

```javascript
foo();
function foo(){
    console.log(a); // undefined;
    var a = 2;
}

//实际上它们等价于

function foo(){
    var a;
    console.log(a);
    a = 2;
}
foo();
```





**包括变量和函数在内的所有声明都会在任何代码被执行前首先被处理**

也就是说，定义声明在编译阶段进行，而赋值声明会原地等待执行阶段



每个作用域都会发生提升，包括函数内部。

但是函数表达式不会提升

```javascript
foo();	//TypeError
var foo = function bar(){
    
};

//实际上它们等于

var foo;
foo();
foo = function bar(){
    
};
```



函数优先



```javascript
foo(); //1
var foo;
function foo(){
    console.log(1);
}
foo = function(){
    console.log(2);
};

//实际上等价于

function foo(){
    console.log(1);
}
foo(); //1
foo = function(){
    console.log(2);
}
```







## 第二部分 this和对象原型

### 第1章 关于this

我们为什么要使用this？

实际上它可以隐式的“传递“对象引用



!!callee已经被弃用，现在一般采用具名函数来调用自身



！那么this到底是什么，**this是在运行时被绑定的，取决于函数的调用方式**



### 第2章 this全面解析

理解调用位置

最重要的是分析调用栈（也就是到达当前位置所调用的所有函数）



```javascript
function baz(){
    //当前调用栈是baz
    //当前调用位置是全局作用域
    console.log("baz");
    bar(); //bar的调用位置
    
}
function bar(){
    //当前调用栈是baz->bar
    //调用位置在baz
    console.log("bar");
    foo();  //foo的调用位置
}
function foo(){
    //当前调用栈是baz->bar->foo
    //调用位置在bar
    console.log("foo");
}
baz(); //baz的调用位置

```



#### 绑定规则

1.默认绑定

独立函数调用

·

```javascript
function foo(){
    console.log(this.a);
}
var a = 2;
foo(); //2
```

a是在全局作用域中的变量，因此this.a就被解析成了全局变量a



2.隐式绑定

```javascript
function foo(){
    console.log(this.a);
}
var obj = {
    a:2,
    foo:foo
};
obj.foo(); //2
```

隐式绑定依赖于上下文对象

上述例子，通过obj来调用函数，从某种意义上来说，函数被添加进了obj对象的属性

因此this.a等价于obj.a



容易出现丢失的情况？可能会应用默认绑定，从而把this绑定到全局对象或者undefined上



```javascript
function foo(){
    console.log(this.a);
}
var obj={
    a:2,
    foo:foo
};
var bar = obj.foo;  //函数别名
var a = "oops,global";
bar(); //"oops,global"

//这是因为bar虽然看起来是obj.foo，但实际上引用的是foo本身

//同样还发生在回调函数之中

function foo(){
    console.log(this.a);
}
function doFoo(fn){
    //fn其实引用的是foo
    fn();
}
var obj={
    a:2,
    foo:foo
};
var a="oops,global";
doFoo(obj.foo);  //"oops,blobal"
```





* 函数别名引用

* 函数回调，隐式赋值



3.显示绑定

call

apply



```javascript
function foo(){
    console.log(this.a);
}
var obj={
    a:2
};
foo.call(obj);  //2

```



硬绑定

典型的为创建一个包裹函数，负责接收参数并返回值

```javascript
function foo(sth){
    console.log(this.a,something);
    return this.a+sth;
}
var obj = {
    a:2
};
var bar = function(){
    return foo.apply(obj,arguments);
};
var b = bar(3);  //2 3
console.log(b);  //5
```

或者创建一个可以重复使用的辅助函数



```
function foo(sth){
    console.log(this.a,sth);
    return this.a+sth;
}
function bind(fn,obj){
    return function(){
        return fn.apply(obj,arguments);
    }
}

var obj = {
    a:2
};
var bar = bind(foo,obj);
var b = bar(3);  //2 3
console.log(b); //5
```



4.new绑定

我们用new操作符实际上完成了对函数的构造调用

```javascript
function foo(a){
    this.a = a;
}
var bar = new foo(2);
console.log(bar.a);  //2
```

那么在对函数进行构造调用的时候，默认会有this的绑定，或者说会影响到this的绑定



#### 优先级

默认规则最低







