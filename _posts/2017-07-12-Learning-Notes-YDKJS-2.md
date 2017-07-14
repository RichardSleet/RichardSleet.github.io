---
layout: post
title: 你不懂的JS学习笔记-第二部分
category: LearingNote-YDKJS
---
# You don't KnowJS
> 引语：十分感谢React中文社区开源群的一位管理员大大给我推荐的[你不懂的JS](https://github.com/getify/You-Dont-Know-JS)这本书，网上有许多开源人士都参与了翻译这本书，再次十分感谢。这里记录一下学习书的笔记，***个人理解仅供参考，真正的学习还需要读原著，如有改正和探讨谢谢指出***

# 第二部分:关于this和对象原型

## 第一章 关于this

### 1. this是什么
* 我的理解this就是上下文的含义比如说之所有再任何的地方书写setTimeout()函数,就是因为实际上是golobal.setTimeout()前面的是上下文,因为在上下文的范围内所以可以不用写.
* 下面这种方式就是显示的传递上下文,但是并不常用
```js
function identify(context) {
return context.name.toUpperCase();
}
function speak(context) {
var greeting = "Hello, I'm " + identify( context ); console.log( greeting );
}
identify( you ); // READER
speak( me ); //hello, 我是 KYLE
```
### 2. 对this的误解
* 首先this并不是函数本身
```js
function foo(num) {
console.log( "foo: " + num );
// 记录 foo 被调用的次数
this.count++; 
}
foo.count = 0;
var i;
for (i=0; i<10; i++) { 
if(i>5){
    foo( i ); 
    }
}
     // foo: 6
     // foo: 7
     // foo: 8
     // foo: 9
// foo 被调用了多少次?
console.log( foo.count ); // 0 -- WTF?
```
* 看到上面的代码确实晕了一下,但是仔细想想确实是对.一个最主要的因素是 ***谁调用,this指就向谁*** 所以this的在这里面就是指向全局变量,为全局变量添加了一个count的变里 这个变量和foo.count搞迷糊了

### 3. 对this的作用域得误解
* 结合后面的例子肯能会比较好,但是值得注意的是this 在任何情况下都不指向函数的词法作用域意思就是
```js
function foo(){
    let a = 1
    console.log(this.a)
}
foo();
var a = 2;//将会输出2
```
## 第二章 this全面解析
### 1.调用位置
* 确定this的绑定首先要搞懂谁在调用,当我们调试程序的时候很容易接触到这个调用栈的概念,就是为了帮助我们查看是谁调用了这个函数
* 下面的代码文章中已经很详细了
```js
function baz() {
// 当前调用栈是:baz
// 因此，当前调用位置是全局作用域:baz的上一级
console.log( "baz" );
bar(); // <-- bar 的调用位置 
}
function bar() {
// 当前调用栈是 baz -> bar
// 因此，当前调用位置在 baz 中
         console.log( "bar" );
foo(); // <-- foo 的调用位置 
}
function foo() {
// 当前调用栈是 baz -> bar -> foo // 因此，当前调用位置在 bar 中
         console.log( "foo" );
}
baz(); // <-- baz 的调用位置
```
### this的四种绑定方式
#### 1.默认绑定:
* 下面的方式就是默认的绑定方式其中this执行的是全局对象
```js
function foo() { 
    console.log( this.a );
}
var a=2; 
foo(); // 2
```
* 像上面一样的通过func()绑定的方式为默认绑定.在严格模式下是不会绑定全局变量的
```js
function foo() { 
"use strict";
console.log( this.a );
}
var a = 2;
foo(); // TypeError: this is undefined
```
#### 2.隐式绑定:
* 下面的代码就是隐式绑定,他会获取调用者的上下文  

```js
function foo() { 
console.log( this.a );
}
var obj=
{ 
a: 2,
foo: foo 
};
obj.foo(); // 2
~~~~~~~~~~我是分割线~~~~~~~~

function foo() { 
    console.log( this.a );
}

var obj2={ 
a: 42,
foo: foo 
};

var obj1={ 
a: 2,
obj2: obj2 
};

obj1.obj2.foo(); // 42
```  

* 在这样得引用链当中,上下文是不会被传递的所以只会成为调用他的人的上下文
* 隐式丢失:很明想就是不小心绑定到了全局对象或者是一个undefinded对象上  

```js
function foo() 
{ console.log( this.a );
}
var obj=
{ a: 2,
foo: foo };
var bar = obj.foo; // 函数别名!
var a = "oops, global"; // a 是全局对象的属性 bar(); // "oops, global"
```
* 上面这段代码需要好好揣摩一下,首先说说bar,虽然函数函数中写了var bar = obj.foo,但是值得注意的是bar应该和obj.foo指向的是同一个地方,这个 = 是一个深拷贝而并非浅拷贝(我也不知道这么说是不是很规范)
* 下面还有更晕的地方....
```js
function foo() { 
    console.log( this.a );
}
function doFoo(fn) {
// fn 其实引用的是 foo fn(); // <-- 调用位置!
}
var obj={ 
a: 2,
foo: foo 
};
var a = "oops, global"; // a 是全局对象的属性 
doFoo( obj.foo ); // "oops,global"
```
* 这里面要具体说明一下首先是一个obj.foo是会进行一个前面提到的LHS的操作找到 他应该赋值给谁赋值給fn,这同样是一个深拷贝.所以其实传入的是一个指向foo()函数的指针.当执行最后一行代码的时候.首先会去找doFoo的然后这个作用域没有就会再次查找顶层作用域得到a输出.

#### 3.显示绑定
* 简单点来说就是通过特有函数改变this的指向eg.call,apply 他们的第一个参数通常都是this需要绑定上的对象
```js
function(){
    console.log(this.a)
}
var obg = {
    a:2
}
foo.call(obg);
```
* 笔者在这里并没有说call,apply的区别,简单说明一下就是call(this,arg1,arg2),apply(this,qrguments)
* 硬绑定,像下面这种方式无论怎么调用bar()foo的this始终的绑定在obj上无法改变这就是硬绑定,下面这种bar就相当于包裹函数
```js
fucntion foo(){
    console.log(this.a)
}

var obj={
    a:2
}
var bar = function(){
    foo.call(obj)
}
bar() //2
setTimeout(bar,100);//2

bar.call(window)///2
```
还有一种方法就是创建一个可以循环重复使用的函数像笔者下面给的代码
```js
function foo(something){
    console.log(this.a,something);
    return this.a + something;
}
//简单的绑定函数
function bind(fn,obj){
    return function(){
        return fn.apply(obj,arguments);
    }
}
var obj ={
    a : 2
}
var bar = bind(foo,obj);
var b = bar(3);
console.log(b)
```
* 在硬绑定的条件下,es5就提供了这样的一种方法 Function.prototype.bind,用法如下
```js
function foo(something){
    console.log(this.a + something)
    return this.a + something
}
var obj = {
    a:2
}
var bar = foo.bind(obj)//字面意思就是把foo函数的this对象绑定在obj上
var b = bar(3);
console.log(b)
```
#### 4.new绑定
* 首先说一说js中的new,当然这个java,c,c++的new都是不一样的.因为js实际上没有类的概念虽说函数的原型比较像但是毕竟还是有很大的不同的.下面是笔者对js中的构造函数的解释
> 首先我们重新定义一下 JavaScript 中的“构造函数”。在 JavaScript 中，构造函数只是一些 使用 new 操作符时被调用的函数。它们并不会属于某个类，也不会实例化一个类。实际上， 它们甚至都不能说是一种特殊的函数类型，它们只是被 new 操作符调用的普通函数而已。  
举例来说，思考一下 Number(..) 作为构造函数时的行为，ES5.1 中这样描述它:
15.7.2 Number 构造函数
当 Number 在 new 表达式中被调用时，它是一个构造函数:它会初始化新创建的 对象。
所以，包括内置对象函数(比如 Number(..)，详情请查看第 3 章)在内的所有函数都可 以用 new 来调用，这种函数调用被称为构造函数调用。这里有一个重要但是非常细微的区 别:实际上并不存在所谓的“构造函数”，只有对于函数的“构造调用”。 
* 使用 new 来调用函数，或者说发生构造函数调用时，会自动执行下面的操作。(第三部很重要)
1. 创建(或者说构造)一个全新的对象。
2. 这个新对象会被执行[[原型]]连接。
3. 这个新对象会绑定到函数调用的this。
4. 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。
* 没错构造函数就是普通的函数!
* 笔者给出了例子代码
```js
function foo(a) { 
    this.a = a;
}
var bar = new foo(2); 
console.log( bar.a ); // 2
```
1.用new调用返回一个全新的对象,你可以理解成为对象的自面量,但是原型是不同的就是对象里面就是
{
    a:2
}
2.对象的this会绑定在bar上所以bar.a =this

### 优先级
* 简单的来说找到this的作用域就是找到调用位置,判断规则.因为规则太多的冲突就会引发冲突,自然就会有优先级这么一说了
> 默认绑定<隐式绑定<显示绑定<new绑定  
* 接下来笔者给的es5中bind的实现我就没有看懂了,简单来说es5中的bind会判断是不是new的调用,如果是bind的this将会全部改变.需要做这样判断的原因就是new绑定和硬绑定实际上是经常使用的绑定.举例
```js
//一个即将被构造函数调用的函数
function foo(p1,p2){
    this.val = p1 * p2;
}
//null就是任意绑定,写了没有用的,总会被new干掉的
var bar = foo.bind(null,'p1');
var baz = new bar('p2');
bac.val //p1p2
```
* 下面是笔者总结的规则
> 1. 函数是否在new中调用(new绑定)?如果是的话this绑定的是新创建的对象。
     var bar = new foo()
> 2. 函数是否通过call、apply(显式绑定)或者硬绑定调用?如果是的话，this绑定的是 指定的对象。
     var bar = foo.call(obj2)
> 3. 函数是否在某个上下文对象中调用(隐式绑定)?如果是的话，this绑定的是那个上 下文对象。
     var bar = obj1.foo()
> 4. 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到undefined，否则绑定到 全局对象。
     var bar = foo()

### 会被忽略的情况
* 1. 当显示绑定传入的是this为null或者为undefined的时候 等价于默认绑定
```js
function foo() { 
    console.log( this.a );
}
var a = 2;
foo.call( null ); // 2 = foo()
```
* 这就不必多说了,但是为什么会出现这样的情况呢,这样的话可以用来展开数组传入函数的参数当中eg:
```js
function foo(a,b) {
console.log( "a:" + a + ", b:" + b );
}
// 把数组“展开”成参数
foo.apply( null, [2, 3] ); // a:2, b:3
// 使用 bind(..) 进行柯里化 var bar = foo.bind( null, 2 ); bar( 3 ); // a:2, b:3
//当然在es6 当中加入了 ...的语法可以替代这样的一个功能,然而翻译大大说这样并不能实现柯里化的相关语法
```
* 2. 间接引用  

```js
function foo() { 
    console.log( this.a );
}
var a = 2; 
var o = {
    a:3,
    foo:foo
}; 
var p = { 
    a : 4 
};
o.foo(); // 3
(p.foo = o.foo)(); // 2
```
* 这里我会写一些有趣的代码探讨一下

### 软绑定
* 因为经过硬绑定的函数是没法通过隐式和显式绑定绑定对象的所以说就出现了软绑定的一些概念.
```js
//如果软绑定不存在
if(!Function.prototype.softBind){
    Function.proptype.softBind = function(obj){
        var fn = this;
        //就是把
        var curried = [].slice.call(arguments,1)
        var bound = function(){
            return fn.apply{
                (!this || this === (window || global)) ?
                obj : this
                curried.concat.apply( curried, arguments )
            };
        };
        bound.prototype = Object.create(fn.prototype)
        return bound;
    }
}
```
* 我承认我看不懂了
* 这里我会写一些有趣的代码探讨一下

### 特殊的箭头函数
* 箭头函数最大的特例就是不属于 this的四种绑定的任何一种,而是根据所在位置的外层作用域所决定的.
```js
function foo(){
    return(a)=>{
        console.log(this.a)
    }
}
var obj1 = {
    a:2
};
var obj2 ={
    a:3
};
var var = foo.call(obj1)
bar.call(obj2);//2
```
* 简单来说箭头函数()=>{}在foo()内部,所有他的this始终是和foo()的this是一样的
,因为经过硬绑定绑定道了obj1,所以只能执行obj1

