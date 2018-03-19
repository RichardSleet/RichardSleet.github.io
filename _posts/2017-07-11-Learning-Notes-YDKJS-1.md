---
layout: post
title: 你不懂的Javascript的小记(作用域)
category: LearingNote-YDKJS
---

* TOC
{:toc}

修改时间|修改原因
---|---
2017.7.11|创建读书笔记
2018.1.13|又来重新读一遍,解决解决一些遗留的问题

## 引言
> [你不知道的Javascript](https://github.com/getify/You-Dont-Know-JS)这本书在github上已经有了7w多的star.不同于大犀牛,望远镜等一些Javscript的书籍,这本书更注重于对JS原理和基础的理解,所以再次记录一些读书时的一些学习知识和自己的理解**如果有错误的话，十分感谢您的指出**

## 作用域和闭包

### Javascript的编译
编译器干的事情其实就是将一个高级语言转换成为操作系统所能够识别的低级语言,js也不例外就是把前端的js脚本代码通过编译解析成对应的语法树(AST),再丢给浏览器的js引擎来执行处理.
1. **分词/词法分析**  
编译器会将我们写的代码拆分成各个有意义的代码块,例如:`var a = 2;`可以被编译器分割为`var`,`a`,`=`,`2`,`;`这五个部分.这个过程就是**词法分析**
2. **解析/语法分析**  
将上面的代码块组装成为一个AST语法树的阶段称为**语法分析**  
对于**抽象语法树**的理解可以参考[美团前端的博客](https://tech.meituan.com/abstract-syntax-tree.html)这篇博客概要说来就是用了UglifyJS更改底层的js的AST语法树来完成一些重构.  
这里再安利一个在线看AST的[小网站](https://astexplorer.net/)还有一个参考[博客](http://www.zcfy.cc/article/understanding-asts-by-building-your-own-babel-plugin)
3. **代码生成**  
最后一步,编译器会将这样的AST语法树编译为可执行的底层代码,编译器会在这时对一些变量进行声明,开辟内存空间等等具体可以参考mdn文档[传送门](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_Management)这里面说了js的一些内存分配上的问题~

### JS为什么是一种解释型语言呢?
这纯粹是因为JS最初被设计出来的时候就是一种轻量的交互语言,更多的职责是交给java.所以设计者当时也并没有把它做成编译型语言的打算,不过现在也出现了WebAssembly字节码技术加快了大大加快了JS引擎的速度,具体相关可移步[某乎](https://www.zhihu.com/question/31415286/answer/58022648)

### 聊聊作用域(通常指的是词法作用域或者也可以叫做静态作用)  
编译器由上面的步骤编译代码的时候会有一些变量的声明会在编译期间交给作用域,而作用域就会在内存当中组成一个树一样的结构，全局作用域下面会有嵌套的函数作用域。最后JS引擎执行代码,每次识别到变量就会去作用域查找.所以为声明的变量分配存储空间这件事情是在预编译的时候就完成了,可是真正向变量当中写入值的时候确实引擎执行的时候做的.也就不难理解js为什么会有一些变量提升的问题了.
1. 编译器: 用来在引擎执行代码前提供给引擎代码并且向作用域提供组成“树”的节点(上节有提)
2. 引擎：用来负责执行和编译的环境 配合作用域组成自己的上下文  
3. 作用域：负责收集并维护由所有声明的标识符(变量)组成的一系列查询，并实施一套非常严格的规则，确定当前执行的代码对这些标识符的访问权限。  
4. LHS和RHS：LHS是对容器的赋值而RHS是对容器的查找

**那么问题来了,那么console.log(a)是一个RHS操作还是一个LHS操作呢?**  
其实正确的理解应该是两个RHS的操作,一个RHS是对console实例的log方法的寻找,第二个RHS是对变量a的寻找.当然书上还给出了一些特别的例子
```js
function foo(a) { 
    console.log( a ); // 2
}
foo( 2 );
```
上面时照搬书上的例子,其中console.log(a)如上面我说的.还有一点需要注意的是.变量a会有一个默认的LHS的操作.因为参数的操作本质上也就是把传过来的值,赋值给这个函数作用域内的参数变量,比如参数是a,调用函数时传入的变量是2,就会有一个`a = 2;`

### 作用域的嵌套
**什么是作用域**:作用域是根据名称查找变量的一套规则  

**什么是作用域的嵌套**：作用域是个家族，儿子认识一部分人，爸爸认识一部分人, 爷爷认识一部分人.引擎的在某一时刻可能和某一个儿子玩的来,经常问儿子认不认识这个人(变量),有时候儿子不认识了,于是儿子就去找爸爸,爸爸再不认识就会去找爷爷直到找到他祖宗为止~(拟人化一下)  

所以一个常见的的错误就是**ReferenceError**:你通过LHS询问了了一个变量,但是作用域他祖宗都不认识这个变量,就会出现这样的错误. 另一个和RHS相关的错误时**TypeError**:你通过RHS询问了一个变量,但是你却对变量进行了不合法的操作比如调用了一个非函数式的变量.

同样有一个特别神奇的事情就是在Chrome的控制台当中直接输入代码 `a = 2;` 将2赋值给一个没有声明的变量竟然没有错误,这作用域时瞎了么.其实书中也给出了这个现象的解释  
当进行LHS查询的时候需要特别注意的是 ***如果LHS在全局作用域当中都无法找到变量就会创建一个变量(非严格模式)***

## 词法的作用域
在第 1 章中，我们将“作用域”定义为一套规则，这套规则用来管理引擎如何在当前作用域以及嵌套的子作用域中根据标识符名称进行变量查找。 

作用域共有两种主要的工作模型。第一种是最为普遍的，被大多数编程语言所采用的词法 作用域，我们会对这种作用域进行深入讨论。另外一种叫作动态作用域,仍有一些编程语言在使用(比如Bash脚本、Perl中的一些模式等)。

### 确定变量所在的作用域
那么什么样的变量在什么样的作用域块中是如何来确定的呢?  

最普遍的一种处理方法就是,变量在哪一个作用域块中声明,那么他就位于哪一个作用域中.这种处理方法也就是词法作用域.  

这里需要补充一波书上没有说提到的知识点:
![执行环境和作用域的概念](https://raw.githubusercontent.com/RichardSleet/Materials/master/RichardSleet'sBlog/YDKJS/Scope%20and%20Execution%20Context.jpg)

上面的图片展示了一些必要的概念.其中我们所知道的Window对象就是全局执行上下文(GEC)的**this对象**,打开Chrome控制台输入下面的代码
```js
var foo = 'Hello World';
console.log(window.foo);
```
你会神奇的发现这里面的foo变到了window里面.这其实时还是因为JS早期设计语言的诟病.js的创造者在创建这个语言的时候为了考虑浏览器的内存所以他会自动的把var声明的变量放在window当中(这点是从阮大大的博客当中提到的)
window对象也就是浏览器对象提供了一些操作html,封装一些事件机制的全局对象等等.所以一般来说一个window对应一个窗口.(使用Frame就另外来说了)

#### 小总结
1.一个词法不可能同时在两个作用域中,作用域查找会在找到第一个匹配的标识符时停止  
2.全局变量会自动成为全局对象的属性(据阮老师的博客上说这是由于js的设计这为了减少内存了留下的历史问题)  
3.无论函数在哪里被调用，也无论它如何被调用，它的词法作用域都只由函数被声明时所处的位置决定。(词法作用域是静态作用域和动态的没有关系)

### 欺骗词法作用域
上面我们曾经说到过,最常用的作用域划分方法就是在声明的位置确定声明的作用域.但是JS依然提供了一些奇葩的方法来改变这个规则(而且不利于阅读.所以不被推荐使用)因为变量的生命会在编译的时候确定,也对于变量的查找做出了一些优化,因为使用这个规则会跳过编译,反而降低执行的效率.  

1.**eval函数**: 接受代码字符串他会在编译器执行而引擎快要执行的时候将这段代码写在它应该位于的位置,其实只是对当前的执行上下文做了一些修改,不过可以解决var出现的变量死区的问题  
2.**with函数**: with函数创建了一个全新的执行上下文并且把参数中的对象,放进了执行环境上下文的变量区域当中.
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
这就是前面说的o2.a会进行LHS查询当查询到顶级时就会给全局变量赋值.就会导致上面的泄露到的全局变量  

这里加一个小话题,经常会在面试的时候问到的一个问题就是,如何实现一个用var模拟一个let,不会产生变量的跃迁呢.意志没有找到合适的方法去完成.但是却想了一个比较烂的方法
```js
function foo () {
    //如果加上这个输出,就会因为找不到num变量引发RefrenceError的错误,去掉就正常了
    console.log(num);
    eval('var num = 0;');
    console.log(num)
}
```
不过当然并不推荐这样使用,至于有没有好方法还请大神留言

## 函数作用域和块作用域

### 函数作用域和隐藏内部实现
JS当中最常用的产生作用域的方法莫过于声明函数,也就是函数作用域就像下面一样.
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
foo这个执行环境上下文出于活跃的时候,它的**变量对象**对象应该包含a,b,c和bar.  

**什么是隐藏内部实现**  
通常我们把函数看成一段代码的集合这段代码和其他代码拥有不一样的执行上下文(ec),所以反转一下思路.其实隐藏内部实现就是把一段代码包裹一个函数让他和当前的执行环境上下文产生一个隔离,从而避免变量泄露到全局.模块化就用到了这样的一个思想.

在下面的代码当中doSomethingElse的调用并不是最安全的，因为其他模块函数都可以调用
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
所以经常使用function(){}用来隐藏代码解决冲突(这是因为js在es5当中只有函数作用域并没有块作用域),经常用来解决一些全局变量的冲突问题.

如果是为了单纯的隐藏一些变量,使其不能够在外面访问到.直接声明函数进行调用还是一个十分麻烦的办法.就像下面一样
```js
//不理想的方法
function foo() { 
    var a = 3; 
    console.log( a ); 
} 
foo(); 

//改善的方法
(function foo(){ 
    console.log( a ); // 3 
})(); 
```
这样做有两个缺点
1. 必须声明一个具名函数foo(),意味着 foo 这个名称本身“污染”了所在作用域(在这个例子中是全局作用域)。
2. 必须显式地通过函数名foo()调用这个函数才能运行其中的代码。

然而使用了自执行函数以后完全规避了上面个两个问题,简单说一下自执行函数,其实就是**欺骗编译器对于function声明函数的捕捉通过添加()或者+-*等等一些方法欺骗了编译器的检查(后面会提到)**所以忽略了function的声明语句.而当引擎执行到这里的时候会检测到这里出现一个**结果值**(不是返回值,在读到语句的部分会仔细说明)为函数的一个引用.在这个引用的后面又会出现()对函数进行调用.所以也就变成了自执行函数.一般这种函数并不是在编译的时候识别的所以自然而然也不会出现在作用域当中.

### 匿名函数,函数的声明,立即执行函数和函数表达式
**具名函数**顾名思义就是有具体名字的函数,声明这样的函数方便对于函数作用的理解,同理**匿名函数**就是一个没有名字的函数,简简单单~.通常因为一个函数是具名函数,会对于调试,编码过程都会更加人性化一些.比如如果一个函数没有名字,除了使用arguments.callee.你还有其他的办法进行递归么?所以具名函数还是比较推荐的写法.
```js
setTimeout( function timeoutHandler() { 
    // <-- 快看，我有名字了! 
    console.log( "I waited 1 second!" );
}, 1000 );
```

**函数的声明**无非是使用function关键字生命函数变量,谈到变量的生命在es5当中也就两种方式
1. function
2. var
在编译时编译器会优先对**var**和**function**产生的变量识别并且交给作用域(函数也可以看错是一个变量),这也就是为什么会存在变量的提升这样的问题。然后引擎在访问代码的时候就能够查询到编译器交过来的变量了.

下面着重的说一下之前提到的**自执行函数**  

原理之前也说过了,是通过一些其他的表达式干扰编译器的判断,让编译器认为这并不是一个声明,对于函数的表达式和函数的声明还有立即执行函数可以看看这两个博主的文章[看看我](https://my.oschina.net/u/2331760/blog/468672)  [还有我](https://my.oschina.net/u/2331760/blog/468672)  

 1. 编译器在编译代码的时候发现一行代码如果第一个出现的是funtion则会被理解为函数的声明语句,编译器就会自动把它交给作用域.**而函数的声明是不会有结果值的** 这一点可以在Chrome的控制台打入`function func(){}`然后就会发现,返回的是一个undefined. 
 2. 当一个有function的函数的声明加入其他的东西时(例如括号+或者-等)编译器会把他认为是一个非声明的语句,而这些语句是需要引擎来执行的  
 3. 当引擎执行代码的时候会发现这里面藏着一个非声明的语句于是就执行他这时候是有结果值的,所以可以对他进行调用  

下面的代码就是自执行函数的例子(感受一下js的黑暗吧!!!)
```js
//下面可以对返回的结果值进行调用,括号的位置并不影响因为function(){}被作为表达式执行完以后会就会返回函数所以两个都行
(function foo(){

})()
//ƒ foo(){}

(function foo(){

}())
//ƒ foo(){}

//我自己又写了一下感受邪恶吧,应该很简单就可以看懂了~
(function(){
    return (a)=>{ 
        console.log(a); 
        return (b)=>{
            console.log(b)
        }
    }
})()(1)(2)

//这个例子是会报错的,因为首先编译器声明了通过function声明了foo,而恰巧声明是没有结果值的.所以调用的时候其实就好比undefined().也就会报错了
function foo(){

}()
```
### 块作用域
{}无法创建块作用域因为js并不支持块作用域,面试的时候可能会问你一些块级作用域的实现方法,比如我司的经典面试题
```js
//这段代码有什么问题?
function foo(){
    i = 3;
    console.log(i);
}
foo();
for (var i = 0; i < 10; i++) { 
    console.log( i );
}
console.log(i)

//可能写成这个样子会比较好,虽然很丑但是却有效的遏制了这个现象\
function foo(){
    i = 3;
    console.log(i);
}
foo();
(function(a,b,c){
    for (var i = 0; i < 10; i++) { 
        console.log( i );
        console.log( a + i);
    }
})(a,b,c)
console.log(a);
eval('var a = 1');
```
很明显for循环中的i并不是只有在循环体内部才可以访问的到,他在外部也可以访问.在实际开发当中经常会有这些问题

其实Javascript中还是有块级作用域的存在的:
1. 使用with(){}情况,因为之前对with的描述其实是创建了一个全新的执行上下文,所以里面执行的代码,以及变量的声明自然也就保存在了这个作用域当中
2. try{}catch(){},因为try{}中的代码可能会发生异常,一但发生了异常.自然而然当前的执行上下文也会被替换掉换成处理异常的执行上下文也就是catch{},所以惊奇的就是这个catch{}当中的部分产生了一个块级作用域

es6当中引入的let就是为了解决这样一个问题,let是可以识别块级作用域的.
```js
var flag = true;
if (flag) {
    let num = 1
    console.log(num);
}
//refrenceError
console.log(num);
```

### 提一下垃圾回收机制
因为书中这里也是简单的提及了一下,我所知道的对于内存的管理方法主要有类一类是**手动管理内存**,另一类是**自动管理内存**,手动管理内存的方法就不用说了基本上就是`malloc()`或者`free()`,而对于自动管理内存来说,比较麻烦的事情是你很难知道这一块内存是不是应该被释放.我所知道的两种自动管理内存的方法分别是**引用计数**和**标记清除**  

**引用计数**:学过iOS的同学多多少少对这个会有一些了解,简单来说就是如果一个内存当中没有指针指向他的话,这块内存就应该被释放,但这样也就会产生循环引用的问题,里面具体的学问还是去google吧

**标记清楚**:标记清除这个东西其实就是垃圾回收机制,引擎每过一段时间就会从JS的全局对象进行查找,就像遍历树一样.如过有些内存并没有在这个过程当中被查找到,那么这些内存就会被认为是不再需要的.  

作者这里是想说明的是对于一些闭包的优化,如下面代码.因为这个作用域产生了一个闭包导致,闭包能够窥探到的区域中间存在大量的内存无法释放,通过添加一个块级作用域/函数作用域把大量占据内存的数据包裹起来.让闭包不会对这块作用域进行窥探也就不会存留大量的内存数据了.
```js
//优化前
function process(data) { 
    // 在这里做点有趣的事情
}
var someReallyBigData = { .. };
process( someReallyBigData );
var btn = document.getElementById( "my_button" );
btn.addEventListener( "click", function click(evt) {
          console.log("button clicked");
}, /*capturingPhase=*/false );

//优化后

function process(data) { 
    // 在这里做点有趣的事情
}
// 在这个块中定义的内容可以销毁了! 
{
let someReallyBigData = { .. }; 
process( someReallyBigData );
}
var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
         console.log("button clicked");
}, /*capturingPhase=*/false );
```

## 提升
### 函数优先的原则
**函数的声明会提前于变量的声明**,如下
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
尽管同时声明了变量但是,发现调用的时候并不会抛出`TypeError`,所以编译器会优先采用函数的声明.转换过来的写法就是下面这个样子

```js
function foo() { 
    console.log( 1 );
}
foo(); // 1
foo = function() { 
    console.log( 2 );
};
```  
你可能会以为这是在非严格模式下面,后声明的变量会覆盖之前生命的变量导致的,但是其实将var和function的声明反过来也一样

## 闭包
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
作者也告诉我们不仅仅如此,闭包之所用重要是因为在定时器、事件监听器、Ajax请求、跨窗口通信、Web Workers或者任何其他的异步(或者同步)任务中，只要使用了回调函数，本质都是在使用闭包!
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

这里简单说一下闭包里面的变量会被保留,最开头的图片当中有说到为什么 EC 执行堆栈包括三个部分
1. this对象
2. 变量对象
3. 作用域链  

当调用闭包返回出来的函数的时候,这个函数会被压入 ECS 执行环境堆栈,这时这个EC的作用域链的上一层恰恰就是闭包的引用,所以可以通过这个词法作用域的引用了访问闭包里面的内容.

### 循环和闭包
首先是一个我司经典的面试题
```js
for (var i=1; i<=5; i++) { 
setTimeout( function timer() {
    console.log( i );
}, i*1000 );
}
```
其实以上的输出结果并不会是1,2,3,4,5.反而是6,6,6,6,6.这是因为setTimeout闭包并不是立即执行的,而是延迟执行的.所以第一步会先把for循环走完,当延迟执行的函数重新回到这个作用域的时候,这里的变量已经面目全非了,所以为了能够维护闭包调用的作用域我们会才去一些措施
```js
//很明显我们要做的是让回调函数回来的时候能够重新的看到当时出现的变量,于是我们就在外面创建了一个函数作用用域
for (var i=1; i<=5; i++) { 
    (function() { 
        setTimeout( function timer() { 
            console.log( i );
        }, i*1000 );
    })();
}
//不过这样是失败的,因为闭包回到这个函数作用域发现什么也没有,于是接着遵循作用域的查找.向上查找于是又重新得到了之前面目全非的作用域,所以会输出6
//像下面这样的才能够运行,因为这里面维护的作用域就不再是空了,当然也是因为这里面是一个值变量
for (var i=1; i<=5; i++) { 
    (function() {
        setTimeout( function timer() {
            console.log( j );
        }, j*1000 );
    })(i); 
}

//当我也以为这就完了的时候笔者又给出了一个例子
for(let i=1;i<=5;i++) {
    setTimeout( function timer() {
            console.log( i );
    }, i*1000 );
}
//其实
```

### 模块
除了回调,另一个常用的地方就是模块机制了,通过一个函数作用域将一个module中的内容封闭起来,然后返回一个闭包使得外面的作用域能够通过闭包进行对暴露出来的函数和变量进行操作
```js
//一个module
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
    };
}
var foo = CoolModule(); 
foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```
1. CoolModule()只是一个函数，必须要通过调用它来创建一个模块实例。如果不执行外部函数，内部作用域和闭包都无法被创建。(照搬的书上的原话,我觉得这样字还是蛮符合懒加载的思想的)  
2. CoolModule()返回一个用{ key: value, ... }来表示的对象。这个返回的对象中是对内部函数的引用。

从模块中返回一个实际的对象并不是必须的，也可以直接返回一个内部函数。  
jQuery 就是一个很好的例子。jQuery 和 $ 标识符就是 juery 模块的公共 API但它们本身都是函数(使用jq的时候其实是调用了他的构造函数创造了一个jq的节点)  
这样就实现了访问API中的方法但是却又不会使变量污染,但是你必须使用它然后自己赋值一个变量  
闭包的形成必须有两个条件:
1. 必须有像上面一CoolModule()一样的封闭函数,也就是闭包所能保留的作用域范围.
2. 封闭函数至少要返回一个函数去作为探测这个作用域的闭包

### 现代和未来的模块机制
上面的实现仅仅是,一个模块机制最基础的部分,实际上随着引用模块的越来越多,模块当中会出现相互依赖的一些复杂逻辑.其实很多所谓的文件模块机制,其实就是在这个模块的最外层包上一个function函数.
```js
/**
* 这个模块相当于一个模块的管理器,用来定义和获取模块
*/
var MyModules = (
    function Manager() {
        //内部加载的模块
        var modules = {};
        //定义一个模块参数分别时模块的名字,依赖的模块数组,模块的函数实现
        function define(name, deps, impl) { 
            for (var i=0; i<deps.length; i++) {
                deps[i] = modules[deps[i]];
            }
            modules[name] = impl.apply( impl, deps );
        }
        //获取某个模块
        function get(name) { 
            return modules[name];
        }
        return {
            define: define,
            get: get 
        }
    }
)();
//首先定义一个自己的模块bar,用来封装一个说你好的方式
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
在这里总结一下整个JS常用的模块机制,首先附上查看的文档  
[官网对commonJS的说明](https://nodejs.org/docs/latest/api/modules.html)  
[阮大对commonJS说明](http://javascript.ruanyifeng.com/nodejs/module.html)  
[国外一大佬对commonJS和ESM的说明](https://hackernoon.com/node-js-tc-39-and-modules-a1118aecf95e)  

#### node采用的commonJS模块机制
首先自然而然是理清一下commonJS的机制,盗用博客上的图  

![commonJS机制](https://cdn-images-1.medium.com/max/1600/1*Rn5xTqjKdPZuG7VnqMzN1w.png)

1. **解析** :根据`require(xx)`命令中的一些参数对模块进行解析,使其能够被解析成为一个node识别的模块固件.当然这一部分解析是有很多查找步骤的,他会遍历一些本地的文件系统.**node会有自己的查找模块的优先策略,比如说当你的路径是一个文件夹的时候,他会去读取这个文件夹中的index文件.具体的顺序就去google吧**当然这一步也会给加载什么样的模块提出一个关键性质的指向.(其实就是不同的模块分类,例如文件模块,核心模块)
2. **加载** :根据上面返回的解析固件指向,会决定node会用一个什么样的方式来加载.如果是一个核心模块,他将会把这个核心模块(node的核心模块都是已经二进制编译好的文件,读取速度快)的链接引入,如果是一个文件模块则他会把这个文件模块读入内存.(读入JS文件到内存和执行JS文件并不是一回事,前者仅仅是把字符串放入了内存当中而已,而后者是确实交给了JS引擎进行编译和执行)
3. **封装** :这是最关键的一步.如果一个文件是文件模块的话,在读取内存中的JS代码并且交给JS引擎的前一段事情,会对这段代码进行一个用function的封装(也就是像上一节说的一样)
```js
/** 封装前 */
const m = 1;
module.exports.m = m;
/** 封装后 */
function (exports, require, module, __filename, __dirname) {
  const m = 1;
  module.exports.m = m;
}
//这样子写是不是就有点和上面定义模块的define函数比较相似了~
```
4. **解析**接下来就就是把这块代码交给JS引擎进行解析,从上文我们也可以看到我们在写一个js文件的代码的时候经常会使用require,exports这些变量,一个非常大的**误解**就是很多人都认为这是一个global的全局变量,其实不然这个变量只是node在进行封装的时候将文件以function(){}包裹起来而这些**所谓的全局变量其实只不过是一些函数参数**  

    **有一个和ES6module本质上的区别**就是在这个JS文件被包装解析之前,是不可能得到export的对象的.换句话说就是Node的module的运行机制是读取文件代码然后进行封装.而ESM的做法是把这个module进行序列化,在他被**编译之后,执行之前**去执行的.(这个原因也就是为什么ESM并不会产生循环引用,而commonJS却会有循环引用的问题)
    ```js
    export const m = 1;
    ```

    这里着重说一下**ESM**的处理方式,在**编译之后,执行之前**的这个关键的时间里面,ESM会有产生叫做`Module Record`的内部结构.原文是这个样子说的
    
    >Within this Module Record, among other key bits of information, is a static listing of the symbols that are exported by the module.

    翻译一下就是这个`ModuleRecord`的一个功能就是会存储一个静态列表,里面记录了在module中export的信息.这有什么用呢?

    打个比方在使用了像下面一样的import导入了一个包的时候,基本上可以理解为我需要用一下foo提供的m变量
    ```js
    import {m} from “foo”;
    ```
    这个语句其实是和在编译的时候识别的,也就是说这个感觉和var还有function变量的声明是一样的,他会建立起来某种链接在你引用这个module和这个module被创建的地方,而这个过程是发生在任何代码被执行以前的.这种感觉会和强类型语言的`inteface`特别相似.(这里面因为nodeJS本身的commonJS和ESM规范冲突做了很多的处理具体可以看[国外一大佬对commonJS和ESM的说明](https://hackernoon.com/node-js-tc-39-and-modules-a1118aecf95e))

5. **调用**,回到我们的commonJS这边来,最后一步就是通过得到module交给使用方来使用

#### ESM模块机制(ESMAscript module)
其实大部分都和上面的commonJS没有什么区别(指上面的步骤,具体细节还是很多不一样的),除去那个解析的关键的点.这个关键的区别,直接使得ESM的模块机制并不是通过执行代码动态确定的.而是将模块机制转换为了一个静态的方式

1. 在es6中会把一个文件当做一个模块,我个人理解就是用一个(function 文件名(导入的其他文件){})将整个文件代码括起来  
2. es6的模块是比较稳定的,在之前的CommonJS模块机制用函数来封装模块会导致只有在引擎执行代码的时候才会知道为什么错,但是es6的模块机制会被编译器识别提前报错.(运行module时循环引用的问题解决方法),比如你使用require即便是路径写的不对也不会在webstorm出现错误,但是使用import导入的时候如果不存在webstorm会提示你没办法导入,这可能就是webstorm后台为你编译进行提示错误吧.  
3. 还有一点好处是如果b块里面用了c块,当b块导入到a块中时,c块也就会自己导入进a块当中.  
4. 因为 ESM 使用了一个链接的形式挂载到了对应的`Module Record`的静态列表上面,可以将一个模块中的一个或多个模块通过这样的链接形式导入到当前作用域中。而commonJS 是需要运行加载这个代码的,自然也就会执行这个module的全部代码,。但是多次使用 require 导入变量是不会重复加载的,这是因为第一次加载后,这个 module 会被放入内存当中,而不用重新被加载.

## 附录的内容
### 动态作用域
通过前面的学习我们知道静态作用域也就是词法作用域,也就是词法作用域是由编译器提前执行代码的时候构造出来的一个作用域.为了方便理解可以去看看执行环境的讲解.一个执行环境上下文会存储三个东西1.作用域链(静态作用域)2.this(动态作用域)3.(这个静态作用域的变量).**我个人的理解词法作用域更普遍于是个函数,动态作用域普遍于是个对象**
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
这里做一个自问自答,首先全局上下文发现一个bar()函数被调用会去push压栈一个bar()的上下文,同理foo()上下文也会被压入.这样一来难道不是foo()中的变量找不到,去上一个**上下文**找么?
并不是的!!!!执行上下文虽然是这样的入栈顺序,但是每个执行上下文都会有一个作用域链,查找变量是根据作用域链确定的并不是根据执行上下文确定的!!!!  
那么作用域链又是根据什么确定的呢?就是我们最开始讨论的方式他在写在哪里,作用域就在哪里.这个作用域只会识别函数的声明,所以自然也不用理会函数调用的相关事情了.所以这里面的作用域很简单
```js
{
    //全局作用域
    foo {
        //foo作用域
    }
    bar {
        //bar作用域
    }
}
```

### 块作用域
之前说过js中是没有块级作用域的但是这其实并不是一个正常的编程语言的行为,所以模拟块级作用域是非常重要的.其实在一些语法中就已经有了块级作用域  
比如with 和 catch 
```js
try{throw 2;}catch(a){ 
    console.log( a ); // 2
}
console.log( a ); // ReferenceError
```



