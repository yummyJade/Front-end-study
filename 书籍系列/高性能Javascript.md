# 高性能Javascript

## 第1章 加载和执行

尽可能合并脚本



defer标签 但是兼容性并不好

**动态加载脚本元素**<script>不会阻塞页面加载，且放进<head>标签更加的保险

XHR获取脚本并注入，但有同域的限制，不被大型项目所用



推荐无阻塞方案：

先添加动态加载所需要的代码（也就是这个函数），再加载初始化页面所需的剩下的代码

```javascript
function loadScript(url,callback){
    var script = document.createELement("script");
    script.type = "text/javascript";
    if(script.readyState){ //IE
        script.onreadystatechange = function(){
            if(script.readyState == "loaded" || script.readyState == "complete"){
                script.onreadystatechange = null;
                callback();
            }
        }
    }else{ //其他浏览器
        script.onload = function(){
            callback();
        };
    }
    script.src = url;
    document.getElementByTagName("head")[0].appendChild(script);
    
}

loadScript("the-rest.js",function(){
    Application.init();
});

//the-rest.js就是剩余的js

```

使用LazyLoad类库，可以下载多个文件并确定顺序，但是注意要减少文件个数，因为每一次都是一个http请求

LABjs 链式方式引用



## 第2章 数据存取

共有四种情况

* 字面量
* 本地变量
* 数组元素
* 对象成员

字面量和局部变量总体来说速度较快



**标识符解析**

标识符搜索从局部变量开始，沿作用域链进行

标识符解析，标识符越深，读写速度越慢

故，如果变量需要被访问一次以上，请**使用局部变量存储**

**作用域链改变**

with会将一个可变对象推入作用域顶端，使得该对象的访问速度变快，但是与其同时，对其他变量的访问变慢

try catch会将异常对象推入一个变量对象并置于作用域的顶端，可以委托给一个错误处理器函数，将影响最小化？？

```javascript
try{
    //..
}catch{
    handleError(ex); //委托给错误处理函数
}
```



同样的还有eval()

总之要尽量减少动态作用域的使用，这样js进行结构分析并优化的结果毫无作用。



**闭包、作用域、内存**

闭包：允许函数访问局部作用域之外的数据

```javascript
function assignEvents(){
    var id = "xdi9";
    document.getElementById("save-btn").onclick = function(event){
        saveDocument(id);
    };
}
```



assignEvents的作用域链指向当前活动对象和全局对象，同样，对于闭包，因为存在id的引用，所以闭包的作用域链中也指向了前者的两个对象

当正常函数对象销毁的时候，闭包仍存在引用，故激活对象无法销毁，造成了更大的内存开销

对象成员 点操作符（注：每一次使用点操作符都要遍历它的所有成员）

嵌套的对象成员会影响性能，尽量少用

包括原型链，越深就开销越大

对以上情况的解决方案都是用一个局部变量进行储存



## 第3章 DOM编程



DOM和JS就相当于两座孤岛，我们应该尽量减少其中的关联

减少DOM访问次数，把运算尽量多的留在ECMAScript这一端处理



innerHtml在某某些老版本浏览器中比DOM操作更出色，不过需要进行评估 

节点克隆 但提升也不算特别明显

**HTML集合**

例如document.getElemensByName()，这些集合类似数组却并不是数组。重要的是，正如DOM标准中所定义的，HTML集以一种“假定实时态”实时存在，这意味着当底层文档更新的时候，它也会自动更新。



集合比数组慢 集合指的是，比如说a标签集合，因为它反映的是一种底层的变化，比较慢

可以将数组长度储存在一个len变量中

可以先把集合元素拷贝到数组中，方法：toArray()

```javascript
var coll = document.getElementsByTagName('div');
var ar = toArray(coll);
```



如果集合小，那么len就够了，集合大，那么就先存数组里，但是要注意拷贝数组也会带来一定的消耗，所以需要自己权衡一下

**访问集合元素时使用局部变量**

对于任何类型的DOM访问，如果需要多次访问，最好使用一个局部变量缓存此成员

```javascript
var coll = document.getElementsByTagName('div');

for(...){
    //...
    el = coll(count);
    name = el.nodeName;
}
```



**遍历DOM**

childNodes/nextSibling

后者在IE表现更优异

使用API代替节点的循环选择分类

**重绘与重排**

DOM树 渲染树



重排何时发生？

* 添加或删除可见的DOM元素
* 元素位置改变
* 元素尺寸改变（padding margin border height width···）
* 内容改变 （例如文本改变或者被另一个不同尺寸的图片代替）
* 页面渲染器初始化
* 浏览器窗口尺寸改变

大多数浏览器通过队列化修改并批量执行来优化重排过程，但是有些操作会导致队列强行刷新

比如：

* offsetTop,offsetLeft,offsetWidth,offsetHeight
* scrollTop,scrollLeft,scrollWidth,scrollHeight
* clientTop,clientLeft··
* getComputedStyle()

以上属性和方法需要返回最新的布局信息，所以浏览器不得不执行所有的待处理任务

合并样式修改，将查询放在最后，将样式改变用class代替



**批量修改DOM**

1.使元素脱离文档流

2.对其应用多重改变

3.把元素带回文档中



有三种基本方法可以使DOM脱离文档

* 隐藏元素，应用修改，重新显示
* 使用文档片段（document fragment）在当前DOM之外构建一个子树，再把它拷贝进文档
* 将原始元素拷贝到一个脱离文档的节点中，修改副本，完成后再替换原始元素



你可先把display属性设置为none然后执行完dom操作再把它加进去

**缓存布局信息**

比如说把一个元素沿某一个方向移动（这么说起来我想到了当时那个椭圆）

现在先来看一个低效的方法

```javascript
//低效的
myElement.style.left = 1 + myElement.offsetLeft + 'px';
myElement.style.top = 1 + myElement.offsetTop + 'px';
if(myElement.offsetLeft >= 500){
    stopAnimation;
}
```

但这种方法并不好，因为元素每一次移动都要查询一次偏移量

让我们看一下更加高效的方法

```javascript
//先获取一个起始位置
var current = myElement.offsetLeft;

current++;
myElement.style.left = current + 'px';
myElement.style.top = current + 'px';
if(current >= 500){
    stop···；
}
```



或者让动画元素脱离文档流也是一个好方法



:hover会明显降低运行速度？？

**事件委托**

使用事件委托防止大量无谓的绑定，或是说添加大量的事件处理器（比如说用户根本不点击，这一类的情况发生；事件处理器的绑定多在onload过程中，容易造成堵塞）

重点在于判断事件源，并根据需求选择是否取消冒泡或者是默认动作

```java
document.getElementById('menu').onclick = function(e){
    //浏览器target
    e = e || window.event;
    var target = e.target || e.srcElement;
    
    if(target.nodeName !== 'A'){
        
    }
}
```



## 第4章 算法和流程控制

4种循环

* for
* while
* do···while
* for in



for in会查找原型链，因此最慢

倒序循环

Duff's Device

switch总体来说比if else表现的快

可以将出现可能性大的条件放在前面

## 第5章 字符串和正则表达式



```javascript
str += "one" + "two";
//这会导致临时内存的分配，我们可以这样写···

str += "one";
str += "two";

str = str + "one" +"two";
```

以上方法并不适用于IE



**正则表达式工作原理**

1. 编译
2. 设置起始位置
3. 匹配每一个正则表达式字元
4. 匹配成功或失败



**回溯**

遇到分支 | 或者{2，}一类，进行匹配失败后需要回溯到最后一个决策点

在搜索字面字符串的时候并不是一个好方法？？



## 第6章 快速响应的用户界面

使用定时器放弃UI线程的控制权

setTimeout指的是多少秒以后被加入任务队列

可以考虑分割成原子任务，由定时器调用

运行任何js任务都不应该超过100ms

**web worker**

是UI线程之外的，独立出来的线程，但是局限性在于它无法访问DOM等一系列UI相关的资源

但是可以用于处理：

* 编码解码大字符串
* 复杂数学运算
* 大数组排序



## 第7章 ajax



有5种技术常用于向服务器请求数据

* XHR
* Dynamic script tag insertion
* iframes
* Comet
* Mutipart XHR



XHR

readyState为3的时候，说明正在与传输中的服务器响应进行交互，响应信息还在传输过程中，也就是所谓的流 streaming

为4的时候说明接受完毕



动态脚本注入

用动态的script插入，可以突破跨域问题，不过方法单一，且无法判断当前成功与否



MXHR

允许客户端只用一个HTTP请求就可以从服务端向客户端传送多个资源



ps：后来发现这书可能是老古董了，就不仔细看了