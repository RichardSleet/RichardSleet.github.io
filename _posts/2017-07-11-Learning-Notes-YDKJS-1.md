---
layout: post
title: 你不懂的JS学习笔记-第1部分
category: LearingNote-YDKJS
---
# You don't KnowJS
> 引语：十分感谢React中文社区开源群的一位管理员大大给我推荐的[你不懂的JS](https://github.com/getify/You-Dont-Know-JS)这本书，网上有许多开源人士都参与了翻译这本书，再次十分感谢。这里记录一下学习书的笔记，***个人理解仅供参考，真正的学习还需要读原著，如有改正和探讨谢谢指出***

# 第一部分:作用域和闭包
## 第一章
### 1. 编译的三个步骤 P5
1. 分词/词法分析:通俗来说就是编译器会将程序猿写的代码首先拆分成可以进行编译的代码 
* eg：var a = 2；可以被编译器分割为var,a,=,2,;  
 格是否会被当作词法单元，取决于空格在 这门语言中是否具有意义。
2. 解析/语法分析  
* AST：[抽象语法树](https://tech.meituan.com/abstract-syntax-tree.html) 的概念，他会上述分割好的代码进组装成为一个语法树，而=，var ，a，2 都回变成语法树的节点。
3. 代码生成
* 编译器最终会将这样的AST语法树编译为可执行的底层代码。
* 特别要强调的是JS的引擎在编译器执行是会帮助编译器做代码优化，同通常来说他不会编译的过程就发生在引擎执行代码的前很短的时间，并不是像执行C/C++等这些代码需要先build完整个文件再进行run  

### 2. 理解作用域 P7
1. 介绍以下三个关键的概念
* 编译器: 用来在引擎执行代码前提供给引擎代码 PS：我自己理解编译器就是一个给作用域介绍人的人 
* 引擎：用来负责执行和编译的环境 PS：我自己理解引擎就是一个想作用域查询认识谁的人
* 作用域：负责收集并维护由所有声明的标识符(变量)组成的一系列查询，并实施一套非常严格的规则，确定当前执行的代码对这些标识符的访问权限。PS：我自己理解作用域就是一个人是很多变量的东西
* LHS和RHS：“赋值操作的目标是谁(LHS)”以及“谁是赋值操作的源头
(RHS)”。PS：理解为引擎的动作  
一般来说函数的调用是RHS理解为 谁用他： 例如：console.log(1)就是谁用1  
LHS可以理解为 他给谁 例如：a = 2； 2应该给谁   
如果是 console.log（a）应该就是RHS和LHS一起  

### 3. 引擎和作用域的对话 P9
* 对话中console是个内置对象很关键 ps:我理解的就是他是全局作用域
* 测验的答案：LHS：foo->c，2->a，a->b  
* ps：1. 可以理解为foo需要知道自己应该赋值给谁所以LHR
* ps：2. 可以理解为2需要知道自己赋值给谁这里是foo的参数a
* ps：3. 可以理解为a需要给谁赋值
* 测试的答案：RHS：2->foo，a->foo，b->a，a+b->return 
* ps: 1. 可以理解为是谁调用2，所以是foo
* ps：2. 同上可以知道后续的三个答案

### 4. 作用域的嵌套P10
* 作用域的嵌套：ps：可以理解为作用域是个家族，爸爸认识儿子的人，爷爷认识爸爸认识的人，每次问儿子没有问爸爸。
* ReferenceError：你找了作用域整个家族都不认识的人就会出错，并没有生命这个引用了没有声明的变量的错误.

### 5. 异常 P12
* LHS查询的时候需要特别注意的是 ***如果LHS在全局作用域当中都无法找到变量就会创建一个变量(非严格模式)***

### 5. 小结 P12
* 如果查找的目的是对
变量进行赋值，那么就会使用 LHS 查询;如果目的是获取变量的值，就会使用 RHS 查询

# 第二章: 词法的作用域
### 1.词法阶段 P14
* 一个词法不可能同时在两个作用域中
* 作用域查找会在找到第一个匹配的标识符时停止
* 全局变量会自动成为全局对象(比如浏览器中的 window 对象，node的global)的属性
* 无论函数在哪里被调用，也无论它如何被调用，它的词法作用域都只由函数被声明时所处 的位置决定。

* 对于闭包很常见的代码,但是对此要深刻的解析一下
### 2. 欺骗词法 P18
* eval函数:接受字符串代码他会在编译器执行在引擎快要执行的时候将这段代码写在他位于的位置,在严格模式的程序中,eval(..) 在运行时有其自己的词法作用域，意味着其中的声明无法修改所在的作用域。
* with函数:
``` js
//with的用法
var obj = {  
a: 1,
b: 2,
c: 3 
};
// 单调乏味的重复 "obj" obj.a = 2;
obj.b = 3;
obj.c = 4;
// 简单的快捷方式 with (obj) {
         a = 3;
         b = 4;
         c = 5;
}
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
* 这就是前面说明的o2.a会进行LHS查询当查询到顶级时就会给全局变量赋值
* 一般并不会这样使用代码，因为这样重新创建的作用域会影响JS的性能

# 第三章:函数作用域和块作用域

### 1. 函数中的作用域 P22
* bar(..) 拥有自己的作用域气泡。全局作用域也有自己的作用域气泡，它只包含了一个标识符:foo。而foo可以理解为一层楼进入一个房间的门，是一个入口

### 2. 隐藏内部实现
* 在下面的代码当中doSomethingElse的调用并不是最安全的，其他函数都可以调用
```js
function doSomething(a) {
b = a + doSomethingElse( a * 2 );
         console.log( b * 3 );
     }
function doSomethingElse(a) { return a - 1;
}
var b;
doSomething( 2 ); // 15
```
* 而下面的函数则是比较安全的

```js
function doSomething(a) { function doSomethingElse(a) {
return a - 1; }
var b;
         b = a + doSomethingElse( a * 2 );
         console.log( b * 3 );
     }
     doSomething( 2 ); // 15

```
* 所以function(){}用来隐藏代码解决冲突
* 文章中还提到了在加载第三方管理的问题P25

### 3. 函数作用域
```js
//并不理想
function foo() { // <-- 添加这一行
var a = 3; console.log( a ); // 3
} // <-- 以及这一行 foo(); // <-- 以及这一行
//方法1：
(function foo(){ // <-- 添加这一行 var a = 3;
console.log( a ); // 3 })(); // <-- 以及这一行
```
>虽然这种技术可以解决一些问题，但是它并不理想，因为会导致一些额外的问题。首先， 必须声明一个具名函数 foo()，意味着 foo 这个名称本身“污染”了所在作用域(在这个 例子中是全局作用域)。其次，必须显式地通过函数名(foo())调用这个函数才能运行其中的代码。

* P27顶部的立即函数在后面进行笔记

### 4.匿名函数和立即执行函数还有函数的声明和函数表达式
* 编写带有名字的函数便于理解
```js
setTimeout( function timeoutHandler() { // <-- 快看，我有名字了! console.log( "I waited 1 second!" );
}, 1000 );
```
* function 和 var 编译时存在函数的提升，也就是说var 和 function会优先的被编译器识别交给作用域。
* 对于函数的表达式和函数的声明还有立即执行函数可以看看这两个博主的文章[惦记我](https://my.oschina.net/u/2331760/blog/468672)[还有我](https://my.oschina.net/u/2331760/blog/468672)
* ***这一段理解是我认真的看了上面的blog和后面的文章所总结的***
> 1.一行代码如果第一个出现的是funtion则会被理解为函数的声明  
2.当一个有function的函数的声明并不是一个声明而是一个表达式的时候,function就可以理解为一个有独立作用域的表达式块  
3.最后需要说的是编译器首先是会把一个作用域中的变量进行采集这通常是 var,function 的声明，而这时引擎并不会执行表达式有关的代码，当声明完成就会吃杏表达式的代码
4.()有两层意思第一层是分组操作符 就像 (a+b)+c一样，还有一层意思就是函数的调用 就像 var a = funtion(){}; a();当做为函数的调用前面必须是一个表达式  

* 这里我会写一些有趣的代码探讨一下

### 5.块作用域

* {}无法创建块作用域，但是try{}catch{}却可以,和function(){
}却可以

* 这里我会写一些有趣的代码探讨一下

# 第四章:提升
### 1.提升的概念
* 上文已经总结了
### 2. 函数优先的原则

```js
foo(); // 1
var foo;
function foo() { console.log( 1 );
}
foo = function() { console.log( 2 );
};
```
* 上面的代码会执行1和下面的代码是等价的
```js
function foo() { console.log( 1 );
}
foo(); // 1
foo = function() { console.log( 2 );
};
```
* 这里我会写一些有趣的代码探讨一下

# 第五章: 闭包
### 1.什么是闭包
```js
function foo() { vara=2;
function bar() { console.log( a );
}
return bar; }
var baz = foo();
baz(); // 2 —— 朋友，这就是闭包的效果。

```
* 对于闭包很常见的代码,许多书上都有这段代码但是仅仅看着这段代码并不知道什么意思,有什么作用
* 于是出现一个问题,但是作者却告诉了我们许多的用处
```js
//在另一个函数中访问其他函数的变量
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

//还有另一种办法
var fn; 
function foo() {
vara=2;
function baz() { 
    console.log( a );
}
fn = baz; // 将 baz 分配给全局变量 }
function bar() {
        fn(); // 妈妈快看呀，这就是闭包!
     }
foo();
bar(); // 2
```
* 作者也告诉我们不仅仅如此,闭包之所用重要是因为
> 在定时器、事件监听器、 Ajax请求、跨窗口通信、Web Workers或者任何其他的异步(或者同步)任务中，只要使 用了回调函数，实际上就是在使用闭包!
```js
// 定时器
function wait(message) {
         setTimeout( function timer() {
             console.log( message );
}, 1000 ); }
wait( "Hello, closure!" );
// timer实际上就是一个具有闭包功能的函数
//事件监听器
function setupBot(name, selector) {
$( selector ).click( function activator() {
console.log( "Activating: " + name ); });
}
     setupBot( "Closure Bot 1", "#bot_1" );
     setupBot( "Closure Bot 2", "#bot_2" );
//触发的 activator函数也可以看做是一个闭包
```
# 2.循环和闭包 p49
*这一部分作者真的是讲的很详细,不得不说我看懂了,如何维持var 的一个内部作用域是很重要的
# 3.模块
* 模块机制的实现其实就是闭包  

```js
function CoolModule() { 
var something = "cool";
var another = [1, 2, 3];
function doSomething() { 
    console.log( something );
}
function doAnother() {
console.log( 
    another.join( " ! " ) 
    );
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
> 首先，CoolModule() 只是一个函数，必须要通过调用它来创建一个模块实例。如果不执行 外部函数，内部作用域和闭包都无法被创建。  
其次，CoolModule()返回一个用对象字面量语法{ key: value, ... }来表示的对象。这 个返回的对象中含有对内部函数而不是内部数据变量的引用。我们保持内部数据变量是隐 藏且私有的状态。可以将这个对象类型的返回值看作本质上是模块的公共 API。  
从模块中返回一个实际的对象并不是必须的，也可以直接返回一个内部函 数。jQuery 就是一个很好的例子。jQuery 和 $ 标识符就是 jQuery 模块的公 共 API，但它们本身都是函数(由于函数也是对象，它们本身也可以拥有属 性)。
* 这样就实现了访问API中的方法但是却又不会使变量污染
* 这里我会写一些有趣的代码探讨一下(返回函数的性质)  


# 4.现代和未来的模块机制
* 首先方便理解下面的代码我先总结一下js当中的四个调用模式

```js
var MyModules = (function Manager() {
var modules = {};
function define(name, deps, impl) { 
    for (var i=0; i<deps.length; i++) {
                 deps[i] = modules[deps[i]];
             }
             modules[name] = impl.apply( impl, deps );
         }
function get(name) { return modules[name];
}
return {
define: define,
get: get };
}
)();

//使用和定义
MyModules.define( "bar", [], function() { function hello(who) {
return "Let me introduce: " + who; }
return {
hello: hello
}; });
                  MyModules.define( "foo", ["bar"], function(bar) {
                      var hungry = "hippo";
   }
return {
awesome: awesome
}; });
                  var bar = MyModules.get( "bar" );
                  var foo = MyModules.get( "foo" );
                  console.log(
                      bar.hello( "hippo" )
); // Let me introduce: hippo foo.awesome(); // LET ME INTRODUCE: HIPPO
```
* 这段代码的意思是通过返回闭包来封装一个模块,并将闭包的名字加入到列表当中,上文中的代码还加入了一个功能就是当这个闭包依赖于其他闭包的时候,需要加载其他闭包

# 未来的模块机制
* 在es6中会把一个文件当做一个模块,我个人理解就是用一个(function 文件名(导入的其他文件){})将整个文件代码括起来
* es6的模块是比较稳定的,在之前的模块机制用函数来分装模块会导致只有在引擎执行代码的时候才会知道为什么错,但是es6的模块机制会被编译器识别也就会在执行知道将会有什么错误.
* 这里对应的应该是require和import的区别因为在webstorm编写代码的时候,require即便是路径写的不对也不会在webstorm出现错误,但是使用import导入的时候如果不存在webstorm会提示你没办法导入,我想着就是webstorm后台为你编译进行提示错误吧
* 还有一点好处是如果b块里面用了c,当b导入到a中c也就会自己导入
* module和import的区别
> import 可以将一个模块中的一个或多个 API 导入到当前作用域中，并分别绑定在一个变量 上(在我们的例子里是 hello)。module 会将整个模块的 API 导入并绑定到一个变量上(在 我们的例子里是 foo 和 bar)。export 会将当前模块的一个标识符(变量、函数)导出为公 共 API。这些操作可以在模块定义中根据需要使用任意多次。
* 这里我会写一些有趣的代码探讨一下

