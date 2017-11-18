---
layout: post
title: 你不知道的JS学习笔记-第一部分(闭包和作用域)
category: LearingNote-YDKJS
---
# You don't KnowJS
> 引语：[你不懂的JS](https://github.com/getify/You-Dont-Know-JS)这本书github上已经有了7w的star最近也是张野大大给我推荐了一波，阅读过之后感觉对js的基础又有了更好的理解。本来我是从来不这种读书笔记的，但是这本书的内容实在是太多太多哪哪都是重点。所以
也就决定记录以下重要的地方便于以后复习方便**如果有错误，感谢指出**

# 第一部分:作用域和闭包
## 第一章
### 编译的三个步骤(当然也就是编译器干的事情了)
1. 分词/词法分析
通俗来说就是编译器会将我们写的代码首先拆分成可以进行编译的代码 eg：var a = 2；可以被编译器分割为var,a,=,2,; 空格是否会被当作词法单元，取决于空格在这门语言中是否具有意义。   
2. 解析/语法分析
AST：[抽象语法树](https://tech.meituan.com/abstract-syntax-tree.html)的概念他会把上述分割好的代码进组装成为一个语法树m,=,var,a,2 都回变成语法树的各个节点，从而为编译做准备。  
3. 代码生成
编译器最终会将这样的AST语法树编译为可执行的底层代码。特别要强调的是JS的引擎在编译器执行是会帮助编译器做代码优化，同通常来说他不会编译的过程就发生在引擎执行代码的前很短的时间，并不是像执行C/C++等这些代码需要先build完整个文件再进行run这样的方式。

### 理解作用域 (通常指的是词法作用域或者也可以叫做静态作用域)  
首先说一下基本的执行顺序首先是编译器由上面的步骤编译代码然后对于一些变量的声明会在编译期间交给作用域然后作用域就会组成一个
像是一个树的结构，全局作用域下面会有嵌套的函数作用域。最后JS引擎根据作用域去执行代码，大概就是这样的一个流程。
介绍以下三个关键的概念:  
1. 编译器: 用来在引擎执行代码前提供给引擎代码并且向作用域提供组成“树”的节点
2. 引擎：用来负责执行和编译的环境 配合作用域组成自己的上下文  
3. 作用域：负责收集并维护由所有声明的标识符(变量)组成的一系列查询，并实施一套非常严格的规则，确定当前执行的代码对这些标识符的访问权限。  
LHS和RHS：“赋值操作的目标是谁(LHS)”以及“谁是赋值操作的源头(RHS)”。PS：rhs参考对象为语句中的常量例如：console.log(1)就是谁来对于1进行操作,lh参考对象为语句中的常量例如：a = 22应该赋值给谁如果是 console.log（a）应该就是RHS和LHS一起

### 引擎和作用域的关系
下面是我写的书上的题
测验的答案：LHS：foo->c，2->a，a->b  
ps：1. 可以理解为foo需要知道自己应该赋值给谁所以LHR  
ps：2. 可以理解为2需要知道自己赋值给谁这里是foo的参数a  
ps：3. 可以理解为a需要给谁赋值  
测试的答案：RHS：2->foo，a->foo，b->a，a+b->return  
ps: 1. 可以理解为是谁调用2，所以是foo  
ps：2. 同上可以知道后续的三个答案

### 作用域的嵌套
作用域的嵌套：作用域是个家族，爸爸认识儿子的人，爷爷认识爸爸认识的人，每次问儿子有没有有认识的人,如果没有再问爸爸。  也就是
上文提到的树结构  
ReferenceError：你找了作用域整个家族都不认识的人就会出错，并没有申明这个引用了没有声明的变量的错误.
LHS查询的时候需要特别注意的是 ***如果LHS在全局作用域当中都无法找到变量就会创建一个变量(非严格模式)***
如果查找的目的是对变量进行赋值，那么就会使用 LHS 查询;如果目的是获取变量的值，就会使用 RHS 查询

## 第二章: 词法的作用域
### 词法阶段 
1.一个词法不可能同时在两个作用域中,作用域查找会在找到第一个匹配的标识符时停止  
2.全局变量会自动成为全局对象的属性(据阮老师的博客上说这是由于js的设计这为了减少内存了留下的历史问题)  
3.无论函数在哪里被调用，也无论它如何被调用，它的词法作用域都只由函数被声明时所处的位置决定。(词法作用域是静态作用域和动态的没有关系)
### 欺骗词法作用域
1.eval函数:接受字符串代码他会在编译器执行在引擎快要执行的时候将这段代码写在他位于的位置,不推荐使用.不过可以解决var出现的变量死区的问题  
2.with函数:简单来说with函数{}以内的语句在当前的位置以内创建了一个作用域而且自动放入了吧obj对象当中的属性放了进去，这就有点想是在Chrome中的命令行写global.a = 0然后a=1进行赋值时一样的，依然不推荐使用
``` js
//with的用法
var obj = {  
    a: 1,
    b: 2,
    c: 3 
};
// 单调乏味的重复 "obj"
obj.a = 2;
obj.b = 3;
obj.c = 4;
// 简单但是不快捷的方式 
with (obj) {
    a = 3;
    b = 4;
    c = 5;
}
//obj.a = 3
/***************我是分界线*****************/
function foo(obj) { 
    with (obj) {
        a = 2; 
    }
}
var o1 = { 
    a: 3
};
var o2 = { 
    b: 3
};
foo( o1 );
console.log( o1.a ); // 2
foo( o2 );
console.log( o2.a ); // undefined
console.log( a ); // 2——不好，a 被泄漏到全局作用域上了!
```
这就是前面说的o2.a会进行LHS查询当查询到顶级时就会给全局变量赋值  
eval和with我认为并不是作为词法作用域的范围,因为词法作用域是在编译前就做好的,所以这个叫做欺骗词法作用域,当然因为是动态的所以会消耗性能

## 第三章:函数作用域和块作用域

### 函数中的作用域 
先看下面代码函数bar(..) 拥有自己的作用域范围，全局作用域也有自己的作用域范围，它只包含了一个标识符:foo。而foo可以理解为一层楼进入一个房间的门，是一个入口.函数作用域
主要提供函数**变量**的访问,找不到一个变量就会去上一个作用域找,而之后所提到的原型链是一个对象的原型链是在这个对象的内部找属性找不到的时候就会去查找.(自己在看书的时候不小心弄混了)
```js
function foo(a) { 
    var b = 2;
    // 一些代码
    function bar() { 
        // ...
    }
    var c = 3;
}
```

### 隐藏内部实现
在下面的代码当中doSomethingElse的调用并不是最安全的，因为其他函数都可以调用
```js
function doSomething(a) {
    b = a + doSomethingElse( a * 2 );
    console.log( b * 3 );
}
function doSomethingElse(a) { 
    return a - 1;
}
var b;
doSomething( 2 ); // 15
```

而下面的函数则是比较安全的
```js
function doSomething(a) { 
    function doSomethingElse(a) {
        return a - 1; 
    }
    var b;
    b = a + doSomethingElse( a * 2 );
    console.log( b * 3 );
}
doSomething( 2 ); // 15
```
所以function(){}用来隐藏代码解决冲突(这是因为js在es5当中只有函数作用域并没有块作用域)  

### 函数作用域
js当中为了能够模仿块级作用域,边有人想到了用函数作用域模仿的概念.先来看看下面代码
```js
//并不理想
function foo() { 
    var a = 3; 
    console.log( a ); 
} 
foo(); 
//方法1：
(function foo(){ 
    console.log( a ); // 3 
})(); 
```
>虽然这种技术可以解决一些问题，但是它并不理想，因为会导致一些额外的问题。首先,必须声明一个具名函数 foo()，意味着 foo 这个名称本身“污染”了所在作用域(在这个 例子中是全局作用域)。其次，必须显式地通过函数名foo()调用这个函数才能运行其中的代码。然而使用了自执行函数以后欺骗编译器对于通过()或者+-*等等欺骗了编译器的检查(后面会提到)
所以忽略了function的声明语句.而这个语句的结果值就是这个函数调用以后就会执行  

### 匿名函数和立即执行函数还有函数的声明和函数表达式
编写带有名字的函数便于理解
```js
setTimeout( function timeoutHandler() { 
    // <-- 快看，我有名字了! 
    console.log( "I waited 1 second!" );
}, 1000 );
```
function 和 var 编译时存在函数的提升，也就是说var 和 function会优先的被编译器识别交给作用域。然后引擎在访问代码的时候就能够查询到编译器交过来的变量了  
下面说的方法其实是通过一些其他的表达式干扰编译器的判断,让编译器认为这并不是一个声明,对于函数的表达式和函数的声明还有立即执行函数可以看看这两个博主的文章[看看我](https://my.oschina.net/u/2331760/blog/468672)[还有我](https://my.oschina.net/u/2331760/blog/468672)  
1.编译器在编译代码的时候发现一行代码如果第一个出现的是funtion则会被理解为函数的声明语句(var也是而let的机制可能就不同),编译器就会自动把它叫给作用域.函数的声明是不会有结果值的  
 2.当一个有function的函数的声明加入其他的东西时(例如括号+或者-等)编译器会把他认为是一个非声明的语句,而这些语句是需要引擎来执行的  
 3.当引擎执行代码的时候会发现这里面藏着一个非声明的语句于是就执行他这时候是有结果值的,所以可以对他进行调用  

下面的代码就是例子(感受一下js的黑暗吧)
```js
//下面对可以对返回的结果值进行调用,括号的位置并不影响因为function(){}被作为表达式执行完以后会就会返回函数所以两个都行
( function foo(){} )()//ƒ foo(){}
( function foo(){} () )//ƒ foo(){}
//我自己又写了一下感受邪恶吧
(function(){return (a)=>{ console.log(a); return (b)=>{console.log(b)}}})()(1)(2)
//下面没有返回的结果值就不可以所以报错
function foo(){}()
//所以你可以依靠this和作用域来实现let,如果没有声明就会出错
function foo(){console.log(a); (function(){this.a = 10})(); console.log(a); }
function foo(){console.log(a); let a = 10; console.log(a); }
```

### 块作用域
{}无法创建块作用域因为js并不支持块作用域，但是try{}catch{}却可以,和function(){
}我认为他们创建的其实是函数作用域,其实他们一直是在用函数作用域模拟块作用域

## 第四章:提升
### 函数优先的原则
看下面的代码
```js
foo(); // 1
var foo;
function foo() { 
    console.log( 1 );
}
foo = function() { 
    console.log( 2 );
};
```
上面的代码会执行1和下面的代码是等价的,这说明函数的声明是要比var提前的,我认为可能编译器在发现有function生命的时候会把var替换掉
```js
function foo() { 
    console.log( 1 );
}
foo(); // 1
foo = function() { console.log( 2 );
};
```  

# 第五章: 闭包
### 什么是闭包
其实我把闭包想象为一个被保存的作用域,而实现方式通常使用function(){}创建这样一个函数作用域的方式(当然也有其他的方式)  
```js
function foo() { 
    var a = 2;
    function bar() { 
        console.log( a );
    }
    return bar; 
}
var baz = foo();
baz(); // 2 —— 朋友，这就是闭包的效果。
```
看起来你并不觉得这有什么牛逼的地方,但是其实js当中闭包是十分常用的功能(比如所有的回调函数其实都是闭包)
```js
//当你把闭包的返回进入另一个函数内部的时候,你就可以在另一个函数内部访问他的变量!!!!!!!
function foo() { 
    var a=2;
    function baz() { 
        console.log( a ); // 2
    }
    bar( baz ); 
}
function bar(fn) {
    fn(); // 妈妈快看呀，这就是闭包!
}
//通过作用域访问的方式进行传递闭包
var fn; 
function foo() {
    var a=2;
    function baz() { 
        console.log( a );
    }   
    fn = baz; // 将 baz 分配给全局变量 
}
function bar() {
    fn(); // 妈妈快看呀，这就是闭包!
}
foo();
bar(); // 2
```
作者也告诉我们不仅仅如此,闭包之所用重要是因为 在定时器、事件监听器、 Ajax请求、跨窗口通信、Web Workers或者任何其他的异步(或者同步)任务中，只要使用了回调函数，本质都是在使用闭包!
```js
function wait() {
    let a = 1;
    function test(){
        console.log(this.a)
        console.log(a)
    }
    return test;
}
var a = 2;
wait()();
// 定时器
function wait(message) {
    //这就是闭包
    function timer() {
        console.log( message );
    }
    //下面的就可以理解为引擎会在1s内调用一个函数,而这个函数就是闭包,他会访问message的作用域
    setTimeout( timer, 1000 ); 
}
wait( "Hello, closure!" );
//事件监听器
function setupBot(name, selector) {
$( selector ).click( function activator() {
console.log( "Activating: " + name ); });
}
     setupBot( "Closure Bot 1", "#bot_1" );
     setupBot( "Closure Bot 2", "#bot_2" );
//触发的activator函数也可以看做是一个闭包
```

### 循环和闭包
先看看代码
```js
for (var i=1; i<=5; i++) { 
setTimeout( function timer() {
    console.log( i );
}, i*1000 );
}
```
其实以上的输出结果并不会是1,2,3,4,5.反而回是6,6,6,6,6.这是因为setTimeout闭包并不是立即执行的,而是延迟执行的.所以第一步会先把for循环走完,当延迟执行的函数重新回到这个作用域的时候,这里的变量已经面目全非了,所以为了能够维护闭包调用的作用域我们会才去一些措施(我记得大搜车的笔试题就有这个)
```js
//这样是不行的,虽然我们确实创建了一个供闭包将来回头查看的作用域,但是这个作用域里面什么都没有
for (var i=1; i<=5; i++) { 
    (function() { 
        setTimeout( function timer() { console.log( i );}, i*1000 );})();
}
//所以像下面这样的才能够运行,因为这里面维护的作用域就不再是空了,当然也是因为这里面是一个值变量
for (var i=1; i<=5; i++) { 
(function() {
    var j = i;
    setTimeout( function timer() {
                 console.log( j );
             }, j*1000 );
})(); }
```

### 模块
在前端方面最早的模块机制的实现其实就是闭包,开头我说通常使用function(){}来维持一个特定的作用域,而下面的返回object的对象将各个维持特定作用域的function(){}组合起来,也能够实现闭包.
```js
function CoolModule() { 
var something = "cool";
var another = [1, 2, 3];
function doSomething() { 
    console.log( something );
}
function doAnother() {
console.log( another.join( " ! " ) );
}
return {    
doSomething: doSomething, 
doAnother: doAnother
};}
var foo = CoolModule(); 
foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```
> 首先，CoolModule() 只是一个函数，必须要通过调用它来创建一个模块实例。如果不执行外部函数，内部作用域和闭包都无法被创建。  
其次，CoolModule()返回一个用对象字面量语法{ key: value, ... }来表示的对象。这个返回的对象中含有对内部函数而不是内部数据变量的引用。我们需要保持内部数据变量是隐藏且私有的状态。可以将这个对象类型的返回值看作本质上是模块的公共 API。  
从模块中返回一个实际的对象并不是必须的，也可以直接返回一个内部函数。jQuery 就是一个很好的例子。jQuery 和 $ 标识符就是 juery 模块的公共 API但它们本身都是函数(使用jq的时候其实是调用了他的构造函数创造了一个jq的节点)
这样就实现了访问API中的方法但是却又不会使变量污染,但是你必须使用它然后自己赋值一个变量  
闭包的形成必须有两个条件:1.必须有像上面一CoolModule()一样的封闭函数,也就是闭包所能保留的作用域范围.2.封闭函数至少要返回一个函数去作为探测这个作用域的闭包

### 现代和未来的模块机制
看看下面代码
```js
//这里的模块定义就像上面的那样返回的是由闭包函数组成的对象
var MyModules = (
    function Manager() {
    var modules = {};
    //这个函数是将创建定义,模块的名字,和制定自己所依赖的模块当你需要依赖其他模块的时候就会在这里进行加载,传入实现函数进行加载,最后是这个模块的实现
    function define(name, deps, impl) { 
        for (var i=0; i<deps.length; i++) {
            deps[i] = modules[deps[i]];
        }
        modules[name] = impl.apply( impl, deps );
    }
    function get(name) { return modules[name];}
    return {
        define: define,
        get: get 
    }
})();
//首先定义一个自己的模块bar,用来分装一个说你好的方式
MyModules.define( "bar", [], function() { 
    function hello(who) {return "Let me introduce: " + who; }
    return {
    hello: hello
};});
//foo依赖于
MyModules.define( "foo", ["bar"], function(bar) {
    var hungry = "hippo";
    function awesome(){
        console.log(bar.hello(hungry).toUpperCase())
    }
    return {
        awesome: awesome
    }; 
});
var bar = MyModules.get( "bar" );
var foo = MyModules.get( "foo" );
console.log(bar.hello( "hippo" )); // Let me introduce: hippo 
foo.awesome(); // LET ME INTRODUCE: HIPPO
```
实际上在foo中得到的闭包bar闭包和 var bar = MyModules.get( "bar" );得到的闭包是一样的

### 未来的模块机制
1. 在es6中会把一个文件当做一个模块,我个人理解就是用一个(function 文件名(导入的其他文件){})将整个文件代码括起来  
2. es6的模块是比较稳定的,在之前的模块机制用函数来分装模块会导致只有在引擎执行代码的时候才会知道为什么错,但是es6的模块机制会被编译器识别也就会在执行知道将会有什么错误.  
3. 这里对应的应该是require和import的区别因为在webstorm编写代码的时候,require即便是路径写的不对也不会在webstorm出现错误,但是使用import导入的时候如果不存在webstorm会提示你没办法导入,我想着就是webstorm后台为你编译进行提示错误吧  
4. 还有一点好处是如果b块里面用了c,当b导入到a中c也就会自己导入  
5. module和import的区别
> import 可以将一个模块中的一个或多个 API 导入到当前作用域中，并分别绑定在一个变量 上(在我们的例子里是 hello)。module 会将整个模块的 API 导入并绑定到一个变量上(在 我们的例子里是 foo 和 bar)。export 会将当前模块的一个标识符(变量、函数)导出为公 共 API。这些操作可以在模块定义中根据需要使用任意多次。

## 附录的内容
### 动态作用域
通过前面的学习我们知道静态作用域也就是词法作用域,也就是词法作用域是由编译器提前执行代码的时候构造出来的一个作用域,我觉得他一定是采用树进行存储的.而动态的作用域实际上更多的是指的this指针,也就是说在引擎执行代码的过程中进行变化的.(大部分的作用域应该是词法作用域,但是难免的要使用一些在执行过程中变化的作用域)
```js
function foo(){
    console.log(a)//2
}
function bar(){
    var a = 3;
    foo(); 
}
var a = 2;
bar();
```
上面的代码当中foo()作用域中没有a变量,也就是说要执行RHS引用(当然也没有console变量他会执行LHS引用)所以会找到2  


### 块作用域
之前说过js中是没有块级作用域的但是这其实并不是一个正常的编程语言的行为,所以模拟块级作用域是非常重要的.其实在一些语法中就已经有了块级作用域  
比如with 和 catch 
```js
try{throw 2;}catch(a){ 
    console.log( a ); // 2
}
console.log( a ); // ReferenceError
```

### this词法
这里主要提到了箭头函数的用法，比如说
```
var obj = {
    id:"awesome",
    cool:function coolFn(){
        console.log(this.id);
    }
}
var id = "not awesome"
obj.cool();//"awesome"
setTimeout(obj.cool,100);//"not awesome"
```
obj.cool()固然是隐式绑定但是当放在函数当中的时候其实这个隐式绑定会被断开因为他把这个函数
的指针赋给了setTimeout的参数变量所以调用的时候其实是cool()这种方式。除了文章中提到的self保存住this的方法，就是使用箭头函数的绑定可以写成这个样子
```
var obj = {
    count: 0,
    cool: function coolFn() {
        if (this.count < 1) {
            setTimeout( () => {  // 箭头函数是什么鬼东西？
            this.count++;
            console.log( "awesome?" );
            }, 100 );
        }
    }
};
obj.cool(); // "awesome?" 
```
箭头函数的笔记在后面还会详细的学习记录下。

## 遗留问题
1. 最后我还是没有弄懂，附录中动态作用域的问题，作者也说了动态作用域关心的是这个调用的位置而不是声明的位置，所以如果按照作者的动态作用域的观点会输出this可是
作者自己又否定了说this的实现原理并不是一个纯粹的动态作用域。那他到底是个什么？
```js
function foo(){
    console.log(a)//2
}
function bar(){
    var a = 3;
    foo(); 
}
var a = 2;
bar();
```


