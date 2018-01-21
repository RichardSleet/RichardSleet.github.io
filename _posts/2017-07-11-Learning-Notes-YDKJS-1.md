---
layout: post
title: 你不知道的JS学习笔记-第一部分(闭包和作用域)
category: LearingNote-YDKJS
---
|修改时间|修改原因| 
|---|---|
|2017.7.11|创建读书笔记|
|2018.1.13|又来重新读一遍,解决解决一些遗留的问题|
# You don't KnowJS
> [你不知道的JS](https://github.com/getify/You-Dont-Know-JS)这本书github上已经有了7w的star最近也是张野大佬给我推荐了一波，阅读过之后感觉对js的基础又有了更好的理解,醍醐灌顶啊有木有!本来我是从来不写这种读书笔记的,但是这本书的内容实在是太多,哪哪都是重点。于是也不敢偷懒,就写了这个读书笔记.笔记是基于原著的基础加上了一些自己的扩展理解**如果有错误的话，十分感谢您的指出**

# 第一部分:作用域和闭包
### js在编译的时候三个大步骤
编译器干的事情其实就是将一个高级语言转换成为操作系统所能够识别的低级语言,而对于js来说可能编译器干的事情也比较类似,就是把前端的js脚本代码通过编译解析成为语法树,然后丢给浏览器的js引擎来执行处理,比如chrom的v8(其实我只知道这一个引擎),编译的步骤也就分成下面三个步骤

1. **分词/词法分析**  
通俗来说就是编译器首先会做的事情会将我们写的代码首先拆分成可以进行编译的代码 eg：var a = 2；可以被编译器分割为 var,a,=,2,;  
2. **解析/语法分析**  
对于**抽象语法树**的理解可以参考美团前端的博客[传送门](https://tech.meituan.com/abstract-syntax-tree.html)概要说来就是用了UglifyJS(来跟我一起读:a-gv-fai-JS)根据项目的需要通过更改底层的js的全局ast来完成一些全局的重构  
这里再安利一个在线看AST的[小网站](https://astexplorer.net/)还有一个参考[博客](http://www.zcfy.cc/article/understanding-asts-by-building-your-own-babel-plugin)
3. **代码生成**  
最后呢编译器会将这样的AST语法树编译为可执行的底层代码(可能是浏览器的也可能是操作系统的),编译器就会对应的分配相应的内存等等具体可以参考mdn文档[传送门](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_Management)这里面说了js的一些内存分配上的问题~  
上面这几条基本上就是js编译器干的绝大部分事情了,然后编译器就会把产生的代码交给js引擎,js引擎叮咣叮咣的优化执行最终执行出来对应的效果.  

### JS为什么是一种解释型语言呢?
那么为什么说js是一种解释型语言呢,这种编译过程基本上是和java差不多的呀,可java确是编译型语言?  

其实我的理解是这个样子的,js对应的编译器的预编译和引擎的执行往往是不可分割的,也正如书上所说编译器前脚编译过引擎后脚就去执行.而不会像是java一样首先javac编译然后java执行,完完整整的分割开来.  

至于为什么会这样子呢?  

纯粹是因为JS最初被设计出来的时候就是一种轻量的交互语言,更多的职责是交给java.所以设计者当时也并没有把它做成编译型语言的打算,不过现在也出现了WebAssembly字节码技术,具体相关可移步[某乎](https://www.zhihu.com/question/31415286/answer/58022648)

### 聊聊作用域(通常指的是词法作用域或者也可以叫做静态作用)  
直接总结一下:首先是**编译器**由上面的步骤编译代码这时候一些变量的声明会在编译期间交给**作用域**然后**作用域**就会在内存当中组成一个树一样的结构，全局作用域下面会有嵌套的函数作用域。最后JS引擎执行代码,每次get到识别的变量就会根据作用域查找，大概就是这样的一个流程.容易理解的是为声明的变量分配存储空间这件事是在预编译的时候做的,可是真正向变量当中写入值的时候确实引擎执行的时候做的.也就不难理解js为什么会有一些变量提升的问题了

##### 下面是定义四连~:  
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

所以一个常见的的错误就是**ReferenceError**：你询问了一个变量,但是作用域他祖宗都不认识这个变量,就会出现这样的错误.  

同样有一个特别神奇的事情就是在Chrome的控制台当中直接输入代码 `a = 2;` 将2赋值给一个没有声明的变量竟然没有错误,这作用域时瞎了么.其实书中也给出了这个现象的解释  
当进行LHS查询的时候需要特别注意的是 ***如果LHS在全局作用域当中都无法找到变量就会创建一个变量(非严格模式)***

## 第二章: 词法的作用域
在第 1 章中，我们将“作用域”定义为一套规则，这套规则用来管理引擎如何在当前作用 域以及嵌套的子作用域中根据标识符名称进行变量查找。 

作用域共有两种主要的工作模型。第一种是最为普遍的，被大多数编程语言所采用的词法 作用域，我们会对这种作用域进行深入讨论。另外一种叫作动态作用域，仍有一些编程语 言在使用(比如 Bash 脚本、Perl 中的一些模式等)。

### 确定变量所在的作用域
那么什么样的变量在什么样的作用域块中是如何来确定的呢?  

答案是非常的简单最普遍的一种处理方法就是,变量在哪一个作用域块中声明,那么他就位于哪一个作用域中.这种处理方法也就是词法作用域.  

这里需要补充一波书上没有说提到的知识点:
![执行环境和作用域的概念](https://raw.githubusercontent.com/RichardSleet/Materials/master/RichardSleet'sBlog/YDKJS/Scope%20and%20Execution%20Context.jpg)

上面的图片展示了一些必要的概念.其中我们所知道的Window对象就是全局执行上下文(GEC)的**this对象**,打开Chrome控制台输入下面的代码
```js
var foo = 'Hello World';
console.log(window.foo);
```
你会神奇的发现这里面的foo变到了window里面.这其实时还是因为JS早期设计语言的诟病.这是因为js的创造者在创建这个语言的时候为了考虑浏览器的内存所以他会自动的把var声明的变量放在window当中(这点是从阮大大的博客当中提到的)
window对象也就是浏览器对象提供了一些操作html,封装一些事件机制的全局对象等等.所以一般来说一个window对应一个窗口.(使用Frame就另外来说了)

#### 小总结
1.一个词法不可能同时在两个作用域中,作用域查找会在找到第一个匹配的标识符时停止  
2.全局变量会自动成为全局对象的属性(据阮老师的博客上说这是由于js的设计这为了减少内存了留下的历史问题)  
3.无论函数在哪里被调用，也无论它如何被调用，它的词法作用域都只由函数被声明时所处的位置决定。(词法作用域是静态作用域和动态的没有关系)

### 欺骗词法作用域
上面我们曾经说到过,最常用的作用域划分方法就是在声明的位置确定声明的作用域.但是JS依然提供了一些奇葩的方法来改变这个规则(有时可能影响执行效率,而且不利于阅读.所以不被推荐使用)  

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

## 第三章:函数作用域和块作用域

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
foo函数创建了一个当执行到这里的时候会出现的变量环境时a,b,c和bar.very easy~  

什么是隐藏内部实现呢?  

通常我们把函数看成一段代码的集合这段代码和其他代码拥有不一样的执行上下文,所以反转一下思路.其实隐藏内部实现就是把一段代码包裹一个函数让他和当前的执行环境上下文产生一个隔离.模块话就用到了这样的一个思想.

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
所以经常使用function(){}用来隐藏代码解决冲突(这是因为js在es5当中只有函数作用域并没有块作用域),经常用来解决一些全局变量的冲突问题.So Easy~

如果是为了单纯的隐藏一些变量,使其不能够在外面访问到.直接声明函数进行调用还是一个十分麻烦的办法.就像下面一样
```js
//不理想的方法
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
这样做有两个缺点
1. 必须声明一个具名函数foo(),意味着 foo 这个名称本身“污染”了所在作用域(在这个例子中是全局作用域)。
2. 必须显式地通过函数名foo()调用这个函数才能运行其中的代码。

然而使用了自执行函数以后完全规避了上面个两个问题,简单说一下自执行函数,其实就是**欺骗编译器对于function声明函数的捕捉通过添加()或者+-*等等一些方法欺骗了编译器的检查(后面会提到)**所以忽略了function的声明语句.而当引擎执行到这里的时候会检测到这里出现一个**结果值**(不是返回值)为函数的一个引用.在这个引用的后面又会出现()对函数进行调用.所以也就变成了自执行函数.一般这种函数并不是在编译的时候识别的所以自然而然也不会出现在作用域当中.

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
### 结果值
因为之前反复提到了结果值,书中也没有明确的说这个概念.所以这里也就扩展一下

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



