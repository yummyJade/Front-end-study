# ECMAScript6入门

[TOC]



## 1.let和const

**暂时性死区TDZ**

`let`和`const` 会绑定一块变量死区，在声明之前都不可赋值

```javascript
var temp = "2333";
if(true){
    temp = "666";	//reference
    let temp ;
}
```

这叫做“暂时性死区TDZ”，也就是说，进入这个作用域下，变量已经存在了，但是不可获取



**不可重复声明**

1.let

**多层嵌套**

```javascript
{{{let a = "hello world"}}}
```



2.const

必须立即赋值，赋值后值不可变



const不可变的是所指向的内存地址中不可变，所以在给对象声明为常量的时候要特别小心

这意味着对象的属性依然是可操作可更改的，如果想彻底冻结对象

```javascript
var constantize = (obj) => {
  Object.freeze(obj);
  Object.keys(obj).forEach( (key, i) => {
    if ( typeof obj[key] === 'object' ) {
      constantize( obj[key] );
    }
  });
};

```





## 2.变量的解构赋值

1.数组解构

```javascript
let [x,y,z] = [1,2,3];
```

其实是一种模式匹配，只要存在Iterator接口即可，即可迭代的

> 模式匹配：1.let x = 10;
>
> ​		    2.对象模式匹配
>
> ​                    3.数组模式匹配

默认值

严格等于undefined的时候生效

```javascript
let [x=4] = [];

```



2.对象的解构

对象的解构不需要顺序对应，只要变量与属性同名即可

```javascript
let {foo,bar} = {bar:"guys",foo:"hey"};
```

对象的解构赋值实际上发生的是

```javascript
let { foo: foo, bar: bar } = { foo: "aaa", bar: "bbb" };
//也就是说被赋值的实际上是后者，它是找到对应的属性然后赋值，foo只是匹配的模式，baz才是真正的变量
let { foo: baz } = { foo: "aaa", bar: "bbb" };
baz // "aaa"
foo // error: foo is not defined
```

必须理解这种模式的匹配

```javascript
let obj = {
    p:[
        "hello",
        {y:"gus"}
    ]
}
let {foo,foo:[]} = obj;
let obj = {
  p: [
    'Hello',
    { y: 'World' }
  ]
};

let { p, p: [x, { y }] } = obj;
```





默认值

严格等于undefined的时候生效

可以方便的把对象的属性赋给变量

```javascript
let Math={
    cos:"aaa",
    sin:"bbb"
}
let {cos,sin} = Math;
```

3.字符串的解构赋值

字符串被转成了一个类似数组的对象

```javascript
let [a,b,c,d] = "lent";
```



5.函数的解构赋值



不能使用圆括号的情况

* 变量声明语句
* 函数声明语句
* 赋值声明



7.用途

（1）交互变量的值

```javascript
let x = 2;
let y = 3;
[x,y] = [y,x];
```

（2）函数返回多个值，取出方便

函数返回多个值，我们需要使用对象的形式，那么，如果用解构的方式我们就能很容易的获取到了

```javascript
function test(){
    return {
        age:8,
        pro:"manong"
    }
}

let {age,pro} = test();
console.log(age);//8
console.log(pro);//manong
```



（3）函数参数，没有次序的可以用对象，有次序的可以用数组

```javascript
// 参数是一组有次序的值
function f([x, y, z]) { ... }
f([1, 2, 3]);

// 参数是一组无次序的值
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});
```

（4）提取Json数据

```javascript
function test(){
    return {
        status:1,
        data:{
            age:8,
            pro:"manong"
        }
    }
}

let {status,data:{age,pro}} = test();
console.log(age);
console.log(pro);

let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};

let { id, status, data: number } = jsonData;

console.log(id, status, number);
// 42, "OK", [867, 5309]
```



（5）函数参数的默认值

（6）遍历Map （for of）

map 结合 for of使用

```javascript
let hashmap = new Map();
hashmap.set('first','hello');
hashmap.set('second','world');

for(let [key,value] of hashmap){
    console.log(key + "is"+value);
}
```

或者仅仅获取一种也是非常的方便

```javascript
// 获取键名
for (let [key] of map) {
  // ...
}

// 获取键值
for (let [,value] of map) {
  // ...
}
```

（7）输入模块的制定方法





## 4.字符串的扩展

repeat() 计算重复的字符个数

padStart(),padEnd()自动补全

标签模板过滤恶意字符串

```javascript
let message =
  SaferHTML`<p>${sender} has sent you a message.</p>`;

function SaferHTML(templateData) {
  let s = templateData[0];
  for (let i = 1; i < arguments.length; i++) {
    let arg = String(arguments[i]);

    // Escape special characters in the substitution.
    s += arg.replace(/&/g, "&amp;")
            .replace(/</g, "&lt;")
            .replace(/>/g, "&gt;");

    // Don't escape special characters in the template.
    s += templateData[i];
  }
  return s;
}
```

String.raw方法返回原生的string，也就是会在转义字符串之前再加一个\，便于模板字符串的处理等

## 5.正则的扩展

## 6.数值的扩展

3.全局变量方法parseInt()和parseFloat()移植到Number对象上面，减少全局性方法，有利于模块化

5.Number.EPSLION 

这是一个评定误差范围的东东，其实就是能接受的最小误差，毕竟浮点数计算是不准确的，而误差小于一定范围我们就可以认为误差可以忽略不计了，也就是没有意义了

7.Math对象扩展

Math.trunc()去除小数部分，返回整数部分

Math.sign()判断是正数负数还是0

## 7.函数的扩展

默认参数

注：参数默认值是惰性求值的，每次用的时候都要重新计算表达式的值

```javascript
let x = 99;
function foo(p = x + 1){
    console.log(p);
}
foo(); //100
x = 100;
foo(); //101
```

与解构赋值默认值结合使用

```javascript
function foo({x,y = 5} = {}){
    console.log(x,y);
}

//这样就避免了foo()报错的情况，因为只有对象存在的时候才会发生解构
```

双重默认值

```javascript
//写法一
function m1({x=0,y=0}={}){
    return [x,y];
}
//写法二
function m2({x,y} = {x:0,y:0}){
    return [x,y];
}
```

写法一函数参数的默认值是空对象，但是设置了对象解构赋值的默认值，写法二函数参数的默认值是一个有具体属性的对象，但是没有设置对象解构赋值的默认值

undefined触发默认值，null没有这种作用

函数的length属性，返回没有默认值的参数个数，其意义是传入预期的参数个数

**作用域**

一旦设置了参数的默认值，在函数初始化的时候就会形成一个单独的作用域

```javascript
var x = 2;
function test(x,y=x){
    console.log(y);
}

test(5);//5
```

2.rest参数

用...用于获取函数的多余参数，而无需使用arguments对象

...items

4.name

xxx.name返回这个函数的名字

5.箭头函数

箭头函数让表达更加的简洁

```javascript
var f = v => v;
//等同于
var f = function(v){
return v;
}

```

箭头函数没有自己的this，所以它的this实际上是外部this的一个复制品

但是这种恒定不变的this其实简化了this这个东西

**尾递归、尾调用**

 减少栈帧开辟

如何改写成尾递归？

尾递归的好处，以一个计算阶乘的递归函数为例

```javascript
function func(n){
    if(n == 1) return 1;
    else return n*func(n-1);
}
```

这是最原始的写法，其实也没什么问题，也的确很符合逻辑，那么问题出在哪呢，其实是在这：

我们要得到最后的值，显然我们需要得到func(n-1)的值，这不是重点，重点是我们需要先得到n-1的返回值然后和上一阶段的变量结合起来计算，这就意味着A函数在调用B函数之前必须先对A中的进行保存，这就意味着A的栈帧无法释放，毕竟它依赖于n-1，所以就很容易爆栈

那么现在，我们对它进行一下改进

```javascript
function func(n,total){
    if(n==1) return total;
    else return func(n-1,n*total);
}
```

那么在这里，有什么不一样呢，是显而易见的，我们可以看到，这个函数与原函数其中一个不同是引入了一个间接的变量total，现在，当我们需要获取最终的值的时候，虽然也要计算func对应n-1的情况，但由于将计算放在了参数中，所以我们不需要保留func对应n的值，因此此时上一个函数的栈可以销毁，所以相比较而言不会那么容易溢出

当然，具体会不会销毁，就看编译器怎么做了

**柯里化**：指的是一个函数，它接收函数A作为参数，运行之后能返回一个新的函数，并且这个心的函数能够处理函数A中的剩余参数

```javascript
//对于一个加法函数
function add(a,b,c){
    return a+b+c;
}
//add函数的柯里化函数_add（人肉推测）
function _add(a){
    return function(b){
        return function(c){
            return a+b+c;
        }
    }
}
//以下运算等价
add(1,2,3);
_add(1)(2)(3);
```



//柯里化的封装过程

```javascript
// 简单实现，参数只能从右到左传递
function createCurry(func, args) {

    var arity = func.length;
    var args = args || [];

    return function() {
        var _args = [].slice.call(arguments);
        [].push.apply(_args, args);

        // 如果参数个数小于最初的func.length，则递归调用，继续收集参数
        if (_args.length < arity) {
            return createCurry.call(this, func, _args);
        }

        // 参数收集完毕，则执行func
        return func.apply(this, _args);
    }
}
```

柯里化看起来是一种复杂化，但实际上是对思维逻辑的一种简化

## 9.对象的扩展

属性 方法的书写简化

```javascript
//属性可以直接使用变量
const name = "zhangsan";
const Person = {
    name,
    age:"8",
    method(){
    
}
}
```

name

获得属性方法的名称

对于get和set方法

```javascript
const obj = {
    get foo() {},
    set foo(x) {}
}
descriptor.get.name //"get foo"
```

解构赋值

```javascript
let {x,y,...z} = {x:2,y:3,b:4,c:5};
console.log(z); //{ b: 4, c: 5 }
```

可以把不需要的参数提取出来传给下一个函数



...可以提取出参数对象所有可遍历属性，并拷贝到当前对象之中

也可以用于数组提取（数组是特殊的对象）

```javascript
let foo = {...['a','b','c']};
console.log(foo); //{ '0': 'a', '1': 'b', '2': 'c' }
let foo = {...'hello'};
console.log(foo); // {0: "h", 1: "e", 2: "l", 3: "l", 4: "o"} {0: "h", 1: "e", 2: "l", 3: "l", 4: "o"}
```

可以实现对象的拷贝，复刻

```javascript
let newVersion = {
  ...previousVersion,
  name: 'New Name' // Override the name property
};

```

## 10.对象的新增方法

解决严格相等 Object.is() 解决了NaN和+0 -0的问题

对象的合并Object.assign()

```javascript
const target = {a:1};
const source1 = {b:2};
const source2 = {c:3};
Object.assign(target,source1,source2);
```

可用于为对象添加属性

## Set和Map数据结构

Set 不允许重复的数据集

可以结合...用来去重

```javascript
[...new Set(array)]
[...new Set('ababbc')].join('')
```

set的遍历顺序就是插入顺序

set没有key value之说，两个都是相同的

weakset 只能存放对象，且不可预测

## 20.Class的基本语法









对象的扩展

数组的扩展

···



## 关于有关的迭代方式

```javascript
const s = new Set();
[2,3,4,5].forEach(x => s.add(x));
for(let i of s){
    console.log(i);
}
```

```javascript
let set = new Set([1, 2, 3]);
set = new Set([...set].map(x => x * 2));
// 返回Set结构：{2, 4, 6}

let set = new Set([1, 2, 3, 4, 5]);
set = new Set([...set].filter(x => (x % 2) == 0));
// 返回Set结构：{2, 4}
```