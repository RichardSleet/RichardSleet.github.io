---
layout: post
title: 你不懂的JS学习笔记-第二部分
category: LearingNote-YDKJS
---
# 第二部分:关于this和对象原型

## 第一章 关于this

### 1. this是什么
* 下面这种方式就是显示的传递上下文,但是并不常用
```js
function identify(context) {
return context.name.toUpperCase();
}
function speak(context) {
var greeting = "Hello, I'm " + identify( context ); 
console.log( greeting );
}
identify( you ); // READER
speak( me ); //hello, 我是 KYLE
```

### 2. 对this的误解  

* 十分需要注意的是是this永远都不会指向他所在的函数自己也就是说你在函数当中创建的变量,你是无法通过this的方式去访问的,所以下面的代码就是很好的例子
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
* 需要注意的原则就是 ***那个object调用了函数,this指就向谁*** 即使object的函数还是会调用函数但是她们只会指向object调用的函数,因为调用堆栈当中永远不会是别他的词法作用域(也就是前面提到的静态作用域,这是完全分割的),调用堆栈当中存储的一定是一个正儿八经的对象,而不是function.就和下面说的一样

### 3. 对this的作用域得误解
* 下面就是一个最典型的例子foo当中调用了bar输出了this.a和this.b很明显结果的输出并不是123而是最外层this指向的window对象这是因为this查找的调用堆栈当中根本不存在函数的词法作用域.
```js
function foo(){
    foo.a = 123;
    let b = 123;
    function bar(){
        console.log(this.a);
        console.log(this.b);
    }
    bar();
}
foo();
//惊奇发现的是如果使用的var变量就会输出对应的值
var a = 0;
var b = 1;
//如果使用的是let变量则就输出undefine
let a = 0;
let b = 1;
~~~~我是分割线~~~~~
function foo() {    
    var a = 2;
    this.bar(); 
}
function bar() { 
    console.log( this.a );
}
foo(); // ReferenceError: a is not defined
```  
* 我可以理解成为如果var在this指向嘴顶层的作用域也就是window或者global时就会自动创建window对象. 

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
* 下面的方式就是默认的绑定方式其中this执行的是全局对象,不要简简单单的认为默认绑定就是隐式绑定自动默认的值而已,使用默认绑定我感觉就像对于执行了function的.
* 说白了默认绑定就是null.foo()没有绑定任何一个对象那就绑定全局对象
```js
function foo() { 
    console.log( this.a );
}
var  = 2; 
foo(); // 2
~~~~分割线~~~~~~
var obj = {
    a : 2,
    foo:function(){
        console.log(this.a)//2 this指向foo

        function bar(){
            console.log(this.a)//3 而采用这个this采用默认绑定指向了window
        } 
        bar()
    }
}
var a = 3;
```  

* 像上面一样的通过foo()绑定的方式为默认绑定,简单来说就是用当前的作用域的this来调用foo()在严格模式下是不会绑定全局变量的.ß
```js  
function foo() { 
"use strict";
console.log( this.a );
}
var a = 2;
foo(); // TypeError: this is undefined
```  
#### 2.隐式绑定:
* 下面的代码就是隐式绑定,他会获取调用者的上下文,切记不要简简单单的认为默认绑定就是隐式绑定的一种
```js
function foo() { 
    console.log( this.a );
}
var obj={ 
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

* 静态绑定的this只能进行一次绑定,并不能传递.
* 在这样得引用链当中,上下文是不会被传递的所以只会成为调用他的人的上下文
* 隐式丢失:很明想就是不小心绑定到了全局对象或者是一个undefinded对象上  
```js
function foo() 
{ 
    console.log( this.a );
}
var obj=
{ 
    a: 2,
    foo: foo 
};
var bar = obj.foo; // 函数别名!
var a = "oops, global"; // a 是全局对象的属性 
bar(); // "oops, global"
```
* 上面这段代码需要好好揣摩一下,首先说说bar,虽然函数中写了var bar = obj.foo,但是值得注意的是bar应该和obj.foo指向的是同一个地方,这个 = 是一个浅拷贝.所以这时bar启用的实际上是一个默认绑定
```js
function foo() { 
    console.log( this.a );
}
function doFoo(fn) {
// fn 其实引用的是 foo 
    fn(); // <-- 调用位置!
}
var obj={ 
a: 2,
foo: foo 
};
var a = "oops, global"; // a 是全局对象的属性 
doFoo( obj.foo ); // "oops,global"
```
* 这里面要具体说明一下首先是一个obj.foo是会进行一个前面提到的LHS的操作(因为这里会进行参数的赋值)找到他应该赋值给谁赋值給fn,这同样是一个浅拷贝.所以其实传入的是一个指向foo()函数的指针.当执行最后一行代码的时候.首先会去找doFoo的然后这个作用域没有就会再次查找顶层作用域得到a输出.也就是说看是什么绑定一定要看最后的()调用是什么绑定

#### 3.显示绑定
* 简单点来说就是通过特有函数改变this的指向(eg.call,apply) 他们的第一个参数通常都是this需要绑定上的对象  
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
* 还有一种方法就是创建一个可以循环重复使用的函数像笔者下面给的代码  
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
* 首先说一说js中的new,当然这个和java,c,c++的new都是不一样的.因为js实际上没有类的概念虽说函数的原型比较像但是毕竟还是有很大的不同的.下面是笔者对js中的构造函数的解释
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

* 用new调用返回一个全新的对象,你可以理解成为对象的自面量,但是原型是不同的就是对象里面就是
{
    a:2
}
2.对象的this会绑定在bar上所以bar.a = this

### 优先级
* 简单的来说找到this的作用域就是找到调用位置,判断规则.因为规则太多的冲突就会引发冲突,自然就会有优先级这么一说了
> 默认绑定<隐式绑定<显示绑定<new绑定  
* 接下来笔者给的es5中bind的实现,简单来说es5中的bind会判断是不是new的调用,如果是bind的this将会全部改变.需要做这样判断的原因就是new绑定和硬绑定实际上是经常使用的绑定ß
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
* 当显示绑定传入的是this为null或者为undefined的时候 等价于默认绑定  
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
* 间接引用:如果你总是拿着函数的指针进行赋值,那么就会有一些像下面的状况(p.foo = o.foo)();想这种奇葩的调用其实并不是调用了p.foo(),因为执行p.foo = o.foo 会返回一个结果值,而这个结果值恰恰就是就是函数的引用,调用这个引用并没有并没有进行任何绑定所以还是使用默认绑定  
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
p.foo()//4
```
* `1

### 软绑定
* 因为经过硬绑定的函数是没法通过隐式和显式绑定绑定对象的所以说就出现了软绑定的一些概念. 他可以在this被绑定以后还能够更改this的指向,下面代码就是
* 下面就是硬绑定和软绑定的一些实现,在进行硬绑定和软绑定的时候会先
* `1下面的代码还是云里雾里的
```js
//如果软绑定不存在
if(!Function.prototype.softBind){
    Function.proptype.softBind = function(obj){
        var fn = this;
        //就是把
        var curried = [].slice.call(arguments,1)
        var bound = function(){
            return fn.apply(
                (!this || this === (window || global)) ?
                obj : this
                curried.concat.apply( curried, arguments )
            );
        };
        bound.prototype = Object.create(fn.prototype)
        return bound;
    }
}
```  
### 特殊的箭头函数
* 箭头函数最大的特例就是不属于 this的四种绑定的任何一种,而是根据所在位置的外层作用域所决定的.所以下面的代码  
```js
function foo(){
    return(a)=>{
        //继承来自foo的绑定
        console.log(this.a)
    }
}
var obj1 = {
    a:2
};
var obj2 ={
    a:3
};
var bar = foo.call(obj1)
bar.call(obj2);//2
```  

* 也就是说返回的this都会指向所在的函数的this
* 简单来说箭头函数()=>{}在foo()内部,所有他的this始终是和foo()的this是一样的
,因为经过硬绑定绑定道了obj1,所以只能执行obj1

## 第三章:对象  

### 对象的语法
* 对象的定义可以有两种类型  
* 文字语法形式即对象字面量  
```js
var myObj = {
    key : value
    //....
}
typeof myObj //object
```  
* 构造型形式即是通过new
```js
var myObj = new Obj();
myObj.key = value
typeof myObj //object
```  
* 以上两种方法基本上是一样的都回创建一个object(基本类型的对象).并且你可以通过浏览在浏览器中的公有链接_proto_去访问他的原型.当然实际上就是私有的[[prototype]].因为在es3之前并没有一个方法可以访问到[[prototype]].而在浏览器当中会为你提供一个_proto_的方法去访问原型链.
* 强烈的提醒这个[[prototype]]和prototype不是一个东西  

### js的类型  

#### js在es5当中的一些基本类型
1. string
2. number
3. boolean
4. null
5. undefined
6. object
* 笔者虽然没有提到,但是如果按照typeof的返回值来看null是属于object.但是我们并不能使用typeof来看这个问题.因为typeof在这个语法设计的时候也是有缺陷的.但是null是不是object的一种特殊类型是一个很模糊的边缘.笔者也并没有提到这里.从严格意义上说null自己就是一个数据类型.
* 其次再来说说function,严格意义上讲function并不是基本类型之一.但是function确实拥有object所有的类型,所以spec并没有将它作为一个基本的function类型.而是把它作为object,他是一个特殊的object,你很名显得可以知道fucntion类型都会有[[prototype]] 和可以被调用的[[call]]属性
* 当然在es6还加入了一个新的基本类型 Symbol 
* 切记上面的前五种类型并不是对象.对象只是js的一个基本类型之一
* 上面的基本类型的字面量是没有任何的方法的,之所以可以使用(str.length)是因为上面的类型默认被转换为下面的内置对象
* 还有一点要说的是 前五种是值类型而第六种是引用类型.--来自望远镜的支援
* string -> String, number->Number , boolean-> Boolean
* null 和 undefined 没有构造形   

#### 内置对象
js语音提供了object类型的内置对象(当然使用typeof的时候会返回function,这是因为实际上他们就是一堆构造函数)
1. String (大写的呦)
2. Number
3. Boolean
4. Object
5. Function
6. Array
7. Date
8. RegExp
9. Error
* 上述的内置对象确实是object的数据类型但是使用typeof返回的时候得到的是function,所以处于更好的理解.将这些理解为function函数更为好.
* 你可能会好奇Fuction和Object会是什么关系[具体可以看这里](https://stackoverflow.com/questions/9959727/proto-vs-prototype-in-javascript)简单来说你会发现原型链的最顶层就是Object.prototype.就是上述内置对象Object函数的prototpye对象属性.这也可能就是js当中唯一一个_proto_是null的了.你可以输入下面代码看看真伪
```js
console.log(Object.prototype.__proto__)
```
* 而Function和Object的区别在stackoverflow里面的图示已经很明白了.所以总结一下:一个函数例如函数foo的原型是指向Fuction.prototype的,但是foo.prototype._proto_是指向Object.prototype.而Fuction.prototype是指向Object.prototyp的.所以区分义object是不是一个function我想应该就是他有没有prototype的属性(当然有时候经过Function.prototype.bind进行绑定也是没有prototype属性的)
* 醉醉重要的是_proto_永远只会指向纯纯粹粹的对象,而不是fucntion.这也迎合了null算成Object的一个类型会更好.  

### 内容
* 简单的来说就是写在一个对象里面值并不是存在对象内部而是保留了这个指针(从技术角度 来说就是引用)指向真正存储的位置因为function就是值类型呀.
* . 和 []其实只是一个指向同一块内存的不同指针而已
* 在对象中属性的名称永远都是字符串,使用其他类型写入都会被转为字符串  
```js
var myObject = { };
myObject[true] = "foo"; 
myObject[3] = "bar"; 
myObject[myObject] = "baz";
myObject["true"]; // "foo"
myObject["3"]; // "bar"
myObject["[object Object]"]; // "baz"
```

#### 1.可计算的属性名
* 所以使用[]访问属性自然而然的就是可以用表达式进行访问  
```js
var prefix = "foo";
var myObject = {
[prefix + "bar"]:"hello", 
[prefix + "baz"]: "world"
};
myObject["foobar"]; // hello
myObject["foobaz"]; // world
```  

##### Symbol
* 这里简单的插入介绍一下symbol,当你像上面一样使用属性的时候,你会发现属性的名字会经常的重复,这将会是一个很大的问题.所以在es6当中就引入了Symbol这样一个**基本类型**,同时提供了Symbol的内置函数或对象来创建symbol类型  
```js
var a = Symbol();
var b = Symbol();
//传入参数可以用来描述symbol类型
var c = Symbol("I am peter");
var d = Symbol("I am peter");
a === b //false
c === d //false
```
* 现在用他来解决我们实际的问题,加入到我们任何一个认为他有可能出现属性重名的对象中 
```js
var mySymbol = new Symbol();
var a = {
};
a[mySymbol] = 'hello!'
var a = {
    [mySymbol]:'hello!'
}
~~~~~或者~~~~
var a = { }
Object.defineProperty(a,mySymbol,{
    value:'Hello!'
})
```
* 上面间的介绍了一下用symbol作为属性名的三种方法,这来自于阮一峰大大的es6一书上,但让还有之前的symbol,iterator例子当中的创建symbol的方法.
* 切记的是symbol是不能够使用.运算符,因为后面会默认为字符串,而并非symbol类型.  

#### 2.属性和方法
* 在JAVA和其他的语言当中,当访问的属性是一个函数的时候,大家会把他叫做方法.比如人的一个方法就是学习,而这个方法是局限于人内部的
* 但是! 在js中这样做是错的因为从技术角度来说，函数永远不会“属于”一个对象，所以把对象内部引用的函数称为“方法”似乎有点不妥。  
```js
function foo(){
    console.log('foo');
}
var  someFoo = foo;//对foo变量的引用
var myObject = {
    someFoo:foo
}
foo; // function foo(){..}
someFoo; // function foo(){..} 
myObject.someFoo; // function foo(){..}
```  
* someFoo和myObject.someFoo 只不过是对于同一个对象的不同引用 

#### 3. 数组
* 当然js的数组也不和java和c中的数组一样
* js中数组更关心的是数组的下标就像指针一样指向一个存储内存,并不是和C一样直接存储连续的存储空间(这个不知道对不对)
* js 完全没有其他语言数组该有的样子,你甚至完全可以给这个数组添加 下标不是int型的,但是这并不改变内置方法的计算比如.length  
```js
var myArray = [ "foo", 42, "bar" ]; 
myArray.baz = "baz";
myArray.length; // 3
myArray.baz; // "baz"
```
* 但是这样其实是很危险的行为

#### 4.对象的copy  
* 像java这样的函数就会有一个copy()的方法用来提供深拷贝和浅拷贝,但是js并不同js的深copy还是很复杂的.一般来说最长用的就是直接转换json进行深拷贝
```js
function anotherFunction() { /*..*/ }
var anotherObject = { 
    c: true
};
var anotherArray = [];
var myObject = { 
    a: 2,
    b: anotherObject, // 引用，不是复本! 
    c: anotherArray, // 另一个引用!
    d: anotherFunction
};
anotherArray.push( anotherObject, myObject );  
```  

* 上述的代码就是一个简单的浅赋值, 对于b,c,d来说就是三个同样的指针引用
* 当我们想把这样的浅拷贝转换成深拷贝的时候,也就是说需要复制对应的内存里面的内容.所以这就会有一个死循环,比如在复制anotherObject和anotherArray的时候
anotherArray引用了anotherObject和myObject复制myObject的时候又需要赋值anotherArray就产生了这样的死循环
* 所以js的深拷贝一直没有明确的方法
* 然而笔者给了一个特别巧妙的深复制的方法通过js序列化来进行深复制  
```js
     var newObj = JSON.parse( JSON.stringify( someObj ) );
```  

* 对于比较容易的浅复制js提供了Object.assign()方法  
```js
    var newObj = Object.assign( {}, myObject );
     newObj.a; // 2
     newObj.b === anotherObject; // true
     newObj.c === anotherArray; // true
     newObj.d === anotherFunction; // true
```  

#### 5.属性描述符
* 一个对象的的属性不仅仅会存储属性的值而且还会存储属性的描述符
```js
var myObject = { 
    a:2
};
Object.getOwnPropertyDescriptor( myObject, "a" );
// {
// value: 2,
// writable: true,
// enumerable: true,
// con gurable: true 
// }
```
* 他会包含三个特性:writable(可写)、 enumerable(可枚举)和 configurable(可配置)
当然还会有set,get和put
* 当需要给一个属性访属性描述符的时候,可以使用Object.defineProperty()方法  
```js
var myObject = {};
object.defineProperty( myObject, "a", {
        value: 2,
        writable: true, 
        configurable: true, 
        enumerable: true
} );
myObject.a; // 2
```  

* writable
* 这个属性代表着是否可以修改属性的  
```js
var myObject = {};
Object.defineProperty( myObject, "a", {
        value: 2,
        writable: false, // 不可写! configurable: true, enumerable: true
     } );
myObject.a = 3;
myObject.a; // 2
```  

* configurable
* 属性可以配置即可以使用defineproperty()方法修改属性描述符  
```js
var myObject = { a:2
};
myObject.a = 3;
myObject.a; // 3
Object.defineProperty( myObject, "a", {
value: 4,
writable: true,
configurable: false, // 不可配置!
enumerable: true 
} );
myObject.a; // 4
myObject.a = 5;
myObject.a; // 5
Object.defineProperty( myObject, "a", {
         value: 6,
        writable: true, 
        configurable: true, 
        enumerable: true
} ); // TypeError
```  

* 特别要强调的是更改这个属性以后就再也没法改动属性了,即单项操作.但是值得注意的是 我们还可以把 writable 的状态由 true 改为 false，但是无法由 false 改为 true。  

* Enumerable
* 这个属性是指该属性是否可以被遍历比如for in 循环 还有es6当中的iterator的迭代器

#### 6. 不变性
* 如果你希望一个对象的所有直接属性都不可以被修改比如设置writeable为false.可是这这个属性的引用没有更改但是依然可以对这个属性的引用的内存进行修改,那么你还需要禁止这个属性的引用(但是这样是不提倡的)  
```js
myImmutableObject.foo; // [1,2,3]
myImmutableObject.foo.push( 4 );
myImmutableObject.foo; // [1,2,3,4]
```
* 对象常量:
* writable:false 和 configurable:false 就可以创建一个对象常量  

* 不可扩展:
* 如果禁止向对象添加属性可以使用 object.preventExtensions()方法.当你向这个对象添加属性的时候就会抛出异常  
```js
var myObject = { 
    a:2
};
Object.preventExtensions( myObject );
myObject.b = 3;
myObject.b; // undefined
```
* 密封:
* Object.seal(..) 会创建一个“密封”的对象，这个方法实际上会在一个现有对象上调用 Object.preventExtensions(..) 并把所有现有属性标记为 configurable:false。
但是可以修改属性的值.也就是不可以添加属性也不可以更改属性的描述符.

* 冻结:
* Object.freeze(..) 会创建一个冻结对象，这个方法实际上会在一个现有对象上调用 Object.seal(..) 并把所有“数据访问”属性标记为 writable:false，这样就无法修改它们的值。也就是不可以添加属性也不可以更改属性的描述符还不可以重新的设置对象
* 以上方法都会不会对对象内部引用的其他引用起作用

#### 7. [[Get]]
* 下面的代码就是get操作
```js
var myObject = { 
a: 2
};
myObject.a; // 2
```
* 执行上面的代码的时候其实就是调用了get函数,首先查找是不是有同名的属性有就返回当没有的时候会查询原型链的方法

#### 8. [[Put]]
* 字面理解就是存放的意思相对于给对象赋值(就是赋予一个没有的属性)他的调用需要下面的条件
> 1. 属性是否是访问描述符(参见3.3.9节)也就是这个对象的属性存在不存在如果是并且存在setter就调用setter。 
> 2. 属性的数据描述符中writable是否是false?如果是，在非严格模式下静默失败，在
严格模式下抛出 TypeError 异常。
> 3. 如果都不是，将该值设置为属性的值。
* 而当没有属性同样需要考虑原型链机制

#### 9. Getter和Setter  
* get的理解就是获取一个值,set的理解就是设置一个值.put的理解就是存放一个值么,如果这个值的访问修饰符已经存在的话,他就会执行这个值的set方法,如果这个值的访问修饰符不存在就会执行put方法
```js
var myObject = {
// 给 a 定义一个 
getter get a() {
    return 2; 
}
};
Object.defineProperty( 
    myObject, // 目标对象 
    "b", // 属性名
{
// 描述符
// 给 b 设置一个 getter
get: function()
{ 
    return this.a * 2 
},
// 确保 b 会出现在对象的属性列表中
enumerable: true
}
);
myObject.a; // 2
myObject.b; // 4
```
* 上面加入get关键词是属于隐式定义,defineProperty(..)属于显示定义,他们都创建了一个没有值的属性,但是返回值却被当做了这个值
* 所以就会有一个问题这样创建的只有getter的属性无论给什么值都是get()方法的return的返回值(所以 set 操作是没有意义的),并且set如果没有被定义(put)操作就会失效,因为像上面的一样[[put]]会查看是否有描述符,当然我们定义了get所以就会存在描述符,然后再次调用set方法,但是并没有set方法就会失效
```js
var myObject = {
    //给a定义一个getter
    get a(){
        return 2;
    }
}
myObject.a = 3;
myObject.a; //2
console.log(myObject)//{}空对象
```
* 定义setter是很必要的,setter方法会覆盖属性默认的put()方法,由上面的情况可以知道getter和setter应该是成对出现的  
```js
var myObject = {
    get a(){
        return this._a_
    },
    set a(val){
        this._a_ = val * 2
    }
}
myObject.a = 2;
myObject.a; // 4
```
* 上述的_a_其实就是put默认存储的变量的位置  

#### 10.存在性
* 当一个属性被赋值为undefined后,我们对他再进行获取但是如何区分是值不存在的undefined还是我们设置的undefined呢
```js
var myObject = {
    a:2
}
("a" in myObject)
("b" in myObject)//false
myObject.hasOwnProperty("a")//true
```
* js为我们提供了in 和 hasOwnProperty() 两种方法
* 他们的区别是使用in的时候也会对原型链上的值进行检索,但是hasOwnProperty()就像名字一样只会检索自己的属性
* 在使用hasOwnProperty()的时候特别需要注意的一种失败的情况就是hasOwnProperty是来自Object的一种委托但是,如果一个对象并没有得到Object的原型就会出现失败的情况.所以一种更加强硬的判断方法就是Object.prototype.hasOwnProperty. call(myObject,"a") 借助显示绑定.
* in检查是否有某个值的时候实际是检查的属性名是不是存在,所以 4 in [1,2,4]这个代码得到的答案false

* 枚举:
* 当一个属性被设置为不可枚举的时候虽然可以访问到他的值,但是却不会出现在for...in..当中  
* 需要特别注意的是 for...in 还可以遍历数值的索引,所以这个方法尽量只应用在对象上
* propertyIsEnumerable(..) 方法相当于在上述hasOwnProperty(..)的基础上面继续判断是否满足 enumerable:true
* Object.keys(..)会返回一个可以枚举属性的数组
* Object.getOwnPropertyNames(..) 会返回一个数组包含的所有属性不管是不是可以枚举
* Oject.keys(..),Object.getOwnPropertyNames(..),Object.hasOwnProperty(..)一样只会直接查找对象直接包含的属性

#### 11.遍历
* for...in 可以遍历原型链上的所有的属性名,但是当我们需要遍历值的时候会用到迭代器
forEach(...),every(...),和some(...)
* 使用for循环的方式遍历数组其实本质上并不是指的遍历是通过下标指针去遍历值数组当中的  
* 下面就是对应的回调函数处理不同的地方
* forEach(...): 会遍历数组中的所有值并忽略回调函数的返回值
* every(..) : 会一直运行直到回调函数返回 false(或者“假”值)
* some(..) 会一直运行直到回调函数返回 true(或者 “真”值)。
* 特别强调的是数组的遍历是有序的但是对象的遍历是无序的,不同引擎for...in的遍历可能会不一样
* 对于数组直接遍历像for一样直接遍历值可以使用es6的for....of方法,for...of会吊用迭代器对象的next()方法返回所有的值,因为数组有内置的 iterator迭代器我们可以手动遍历
```js
var myarray = [1,2,3]
var it = myArray[Symbol.iterator]()//调用返回地带起的方法,得到迭代器
it.next(); // { value:1, done:false } 
it.next(); // { value:2, done:false } 
it.next(); // { value:3, done:false } 
it.next(); // { done:true }
```
* symbol 是es6加入的第七种数据类型切记不是内置对象 [关于symbol](http://www.infoq.com/cn/articles/es6-in-depth-symbols) 简单的来说就是为了防止属性重名使用的一种特殊标识符
* 和数组不一样普通的对象是没有iterator的所以没有for...of遍历,但是我们依然可以在一个对象中加入iterator
```js
var myObject = {
    a : 2,
    b : 3 
}
//这里加入的Symbol.iterator斌不是一个string类型而是es6加入的新的类型symbol.确保这事一个唯一的属性字段.
Object.defineProperty(
    myObject,
    Symbol.iterator,
    {
    enumerable: false,
    writable: false,
    configurable: true,
    //value 就是当前属性的值,这是一个会返回iterator的函数
    value:function(){
        var o = this;
        var idx = 0;
        var ks = Object.keys(o)
        return {
            next: function(){
                return {
                    value: o[ks[idx++]],
                    done : (idx>ks.length)
                }
            }
        }
    }
    }
)
// 手动遍历
var it = myObject[Symbol.iterator]();
it.next();//{value:2,done:false}
it.next();//{value:3,done:false}
it.next();//{value:undefined,done:true}
// 用for...of遍历
for (var v of myObject){
    console.log(v);
}
```

## 第四章:混合对象类
* 这一整个章节都是一些理论的东西,所以摘要一些关键的地方

### 类理论
* 在其他的编程语言当中一般子类都是覆写父类的方法,但在js当中这么写可能会降低代码的可读性和健壮性
* js中的类
* 虽然es6中的各种属性都在为js模拟一个相似的基于类的设计模式但是实际上两者完全不同的

### 类的机制
* 类的构造:
* 笔者在这里把类和对象比作家住的蓝图和建筑,就像我在大一的时候理解的就是造钱的模具和钱的关系  
* 构造函数:
* 想上面的举例决定每个大楼不一样的属性或者是钱的唯一的编号就是构造函数需要赋予的不同的东西,构造函数是属于类的

### 类的继承
* 就如儿子和爸爸一样,子类会继承父类的一些特性.但是这些特性是相互分开的,即使父类再次改变也不会影响子类的但是父类和子类斌不是上述的蓝图和建筑的关系.

* 1.类的多态
* 多态的另一个方面是，在继承链的不同层次中一个方法名可以被多次定义，当调用方法时 会自动选择合适的定义。
* 特别注意的是像别的语言一样使用super调用父类的的构造函数是没有问题的因为构造函数是属于类的,但是在js当中恰恰相反'类'(注意引号)是属于构造函数的.而js当中每一个构造函数都会有一个.prototype 的对象,因此并没有什么关系(es6可以通过super(调用)而且是必须的)

* 2.多重继承
* 对于java等一些语言只支持单继承,但是js本身是没有提供"多重继承"的功能的,因为多重继承更为复杂,但是却可以使用其他方法来实现多继承

### 混入(其实就是为了模拟其他语言继承时的赋值)
* 值得注意的是js当中只有对象并不存在可以实例化的'类',所以自然不能像蓝图对建筑一样赋值重新建造一份.但是js确可以模拟这样的复制行为即是混入.分为显示混入和隐式混入
* 在大多数框架中被称为extend()
```js
function extend(sourceObj,targetObj){
    for(var key in sourceObj){
        //只赋值存在的
        if(!(key in targetObj)){
            targetObj[key] = sourceObj[key];
        }
    }
    return targetObj;
}
var Vehicle = {
    engines:1,
    iginition: function(){
        console.log("Turning on my engine");
    },
    drive: function(){
        this.iginition();
        console.log("Steering and moving forward!");
    }
}
var Car = extend(Vehicle,{
    wheels:4,
    drive: function() { 
        Vehicle.drive.call( this ); 
        console.log("Rolling on all " + this.wheels + " wheels!");}
})
```
* js中只有对象没有类,而对象就是通过函数得到的.但是上述的这样的做法是一个浅复制的操作.函数只是获取了一个iginition()的引用,而driver没有被复制实现了了子类对父类的重写.
* 混合复制就不会像上面一样,他会像下面一样创建一个空对象然后赋值a对返回的对象再赋值b
```js
//另一种混合函数,但是肯能会被重写,原理就是先进行复制
function mixin(sourceObj,targetObj){
    for(var key in sourceObj){
        targetObj[key] = sourceObj[key];
    }
    return targetObj
}
var Vehicle={
    //...
}
//先把值进行复制
var Car = mixin(Vehicle, { } )
mixin({
    wheels:4,
    drive:function(){

    }
},Car)
```

#### 3. 寄生继承
* `1先做个标记回来看看
```js
function Vehicle(){
    this.engines = 1
}
Vhicle.prototype.igition = function(){
    console.log("Turning on my engine.")
}
Veicle.prototype.drive = function(){
    this.ignition();
    console.log("Steering and moving forward!")
}
// "寄生"Car
fucntion Car(){
    var car = new Vehicle();
    /// 对car进行定制
    car.wheels = 4
    //保存到Vehicle::drive() 的特殊引用
    var vehDrive = car;
    // 重写 Vehicle::drive()
}   
```

## 第五章:原型
### [[Prototype]]
* js的所有的对象即object都会有一个特殊的内置属性就是[[Prototype]],在我的理解这个[[Prototype]]指向他的原型对象(纯纯粹粹的object),绝大部分对象在创建的时候会被赋予一个值.
* 这里可能会联想到_proto_和prototype到底有什么区别,[点我看看](https://stackoverflow.com/questions/9959727/proto-vs-prototype-in-javascript)
```js
var myObject = {
    a:2
}
myObject.a//
```
* 梳理一下使用[[Get]]操作的整个过程
> 1. Get首先会检查对象本身有没有这个属性有的话就是使用他
> 2. 当没有的时候Get就会获取原型链上的值进行遍历,这会查找整条[[Prototype]]链
,如果还没有的话就会返回undefined

* 对于for...in操作: 同样是和Get查找原型链是一样的但是,只能查找enumerable可枚举的类型
* 对于 in 操作:同样是会遍历所有的原型链上的属性名

* Object.prototype
* 所有的普通的[[Prototype]] 链最终都会指向内置的Object.prototype,也就是内置对象Object的prototype属性.这赶脚就像是java中的顶级父类一样Object,即便js没有提供类的概念.但是Object.prototype会像java的父类一样提供许多的功能函数比如 toString(),valvueOf()等等

* 属性的设置和屏蔽
* 这里我们梳理一下使用[[Put]]添加一个属性的情况
> 1. 比如myObject.foo = "bar";这条语句会首先的查找myObject中的属性是否有"bar"属性名如果有,就进行setter操作  
> 2. 当没有的时候噗就会查找原型链如果原型链的上层出现了这个属性就会产生三种情况 1.上层foo具有普通数据的访问属性并且 writable:true那么他会在myObject中添加foo.而这个下层的属性会屏蔽上层.切记屏蔽属性!!因为没有设置setter无法调用他的函数!2.上层foo具有普通数据的访问属性并且只读读writable:false则这个赋值就是失败的并且如果在严格模式就会报错3.上层foo是一个setter,则这个setter一定会被调用.(注意在defineProperty中不能够同事设置vaule,writable和get,set方法)
> 3. 最后当一直查询原型链最头为止还是没有就会创建一个myObject里面的foo

* 所以并不是设置上层存在的属性,就会设置下层的屏蔽属性
* 尤其来说说只读属性的处理情况,其实会出先这样的情况主要为了模拟'类'属性的继承.可以foo理解为父类的属性,myObject继承了foo的属性(但是他并没有在面向对象语言中的赋值属性,本质来书myObject是不存在foo的)
* 隐式并不是一个很好的操作,甚至你有时候会需要注意到隐式屏蔽的一些状况  
```js
var anotherObject = {
    a:2
};
//下面的Object.create会把myObject关联anotherObject的原型
var myObject = Object.create(anotherObject);
anotherObject.a //2
myObject.a //2
anotherObject.hasOwnProperty("a"); //true
myObject.hasOwnProperty("a");//false
//隐式的属性屏蔽
myObject.a = myObject.a + 1 
//这里很明显两访问的不是上一个原型链的值
anotherObject.a//2
myObject.a //3
myObject.hasOwnProperty( "a" ); // true
```
* 上面myObject.a = myObject.a + 1 出现了隐式屏蔽,流程是首先是右边的myObject.a 会执行get操作查找原型链,然后当赋值给左边的时候会先进行[[Put]],发现父原型中存在"a",所以第一种情况设置屏蔽属性并且设置为3.

### 类
* 某种意义上说js才是一个"面向对象的语言"(因为他连类都没有好么)也就是我们之前提到的工程的蓝图和造钱的模子 
* 类函数:
* 虽然没有类但是js一直在模范类的使用方法.为了这种理念.所有的函数都会有一个的共有不可枚举的属性prototype.**切记是所有的函数,只是函数**  
```js
function foo(){
    //,,,,
}
foo.prototype
```
* 上面的这个函数的prototype原型属性(和[[prototype并不一样]]).我最开始就把原型理解成他的Class.但是这样做还是有很大的缺陷的.其实他只是一个用来关联对象的东西.  
```js
function Foo(){
    // ...
}
var a = new Foo();
Object.getPrototypeOf( a ) === Foo.prototype; // true
```
 * 解释一下上面的代码:当调用new()的时候首先执行的就是new绑定的步骤
 1. 创建(或者说构造)一个全新的对象。
 2. 这个新对象会被执行[[原型]]连接。//[[protutype]]
 3. 这个新对象会绑定到函数调用的this。
 4. 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。
 * 在第二步的时候就是会新对象的[[prototype]]会被赋予引用的链接,也就是关联到那个Foo()函数的prototype上面.
 * 特别注意,js的new和面向对象的new完全不同.因为你不可能创建一个类的多个实例,只能创建一个对象的多个对象,而他们的[[prototype]]关联的是同一个对象,所以他们不像儿子从爸爸得到了特有的基因,当爸爸基因改变的时候儿子则不会变.他们之间还是存在相互的关联的.
 * 生成的a的对象会关联 Foo.[[prototype]]的对象.笔者还提到了调用new的时候,其实本意并不是创建这样的一个关联这个关联只是一个意外的结果
 * 所以js当中实际上不存在类和对象这样一个不是平级的概念,而全部都是对象.所以js中绝对不存在继承!!! 
 * 当你用原型关联了以后,js的委托就可以通过这样的一个关联调用你关联的对象的方法  
 
* 构造函数
 ```js
 function Foo(){
     // ...
 }
 Foo.prototype.constructor === Foo ///true
 var a = new Foo();
 a.constructor === Foo //true
 ```
 * 看上面的代码 Foo.prototype 默认在声明function时有一个公用不可枚举的属性 .constructor 而这个属性就是函数Foo.随时随地的结合[图示](https://stackoverflow.com/questions/9959727/proto-vs-prototype-in-javascript)
 * 其实a中并没有constructeor这个属性.但是a的[[prototype]]指向了Foo.prototype.所以自然而然的会得到原型链上的值
 * 构造函数就是和普通的函数是一样的.构造函数只不过是一个普通的函数调用new以后相对于对象而言的一种函数,而且当前面加上一个new的时候他就会用一个构造对象的形式去调用他.但依然会调用!!!  
 ```js
 function NothingSpecial(){
     console.log("You will see me")
 }
 var a = new NothingSpeacial();
 //"You will see me"
 a;//{}
 ```  
 * 其实js一直在模仿class的形式     
 ```js
 function Foo(name){
        this.name = name
 }
 Foo.prototype.myname = function(){
     return this.name;
 }
 var a = new Foo("a")
 var b = new Foo("b";
 a.myname()//"a"
 b.myName()//"b"
 ```
 * 来看看上面的这些代码,上面的代码展示了两种"面向类的"技巧
 1. this.name = name 是给每一个对象的值也就a和b所以为什么要有this这样一个东西呢.就是在你new的时候需要对你造出来的这个对象操作一些不一样的方式就像每个钱都会有自己的编码一样.而this就在这里绑定代指这样你创建的对象
 2. 而Foo.prototype.myname这样的一个操作很明显是给函数Foo()的prototype赋值.又因为a和b都会超找原型链,所以自然而然的就会调用到 Foo.prototype.myname这样一个函数.
 而这又采取了隐式绑定

## (原型)继承
* 虽然js当中没有绝对的继承,但是我们依然可以模拟出来
```js
function Foo(){
    this.name = name;
}
Foo.prototype.myName = function(){
    return this.name
}
function Bar(name,label){
    Foo.call(this,name);
    this.label = label;
}
Bar.prototype = Object.create(Foo.prototype);
Bar.prototype.myLable = function(){
    return this.label
}
var a = new Bar("a","obj a")
a.myName();//"a"
a.myLabel();//"obj a"
```
* 这里Bar.prototype 会被替换为一个空对象而这个空对象的原型执行Foo.prototype方法
```js
// 和你想要的机制不一样! 
// 这样写的意思是Bar.prototype会直接指向Foo.prototype.但是并不会创建一个关联的对象
Bar.prototype = Foo.prototype;
// 基本上满足你的需求，但是可能会产生一些副作用 :( 
// 这样写Bar.prototype 确实指向了Foo但是有个巨大的问题就是Foo()是会被执行的(比如写日志、修改状态、注 册到其他对象、给 this 添加数据属性，等等)这就会影响到Bar
Bar.prototype = new Foo();
```
* 在es6前后有两个办法可以啊改变这样的一个状态
```js
// ES6之前需要抛弃默认的Bar.prototype
     Bar.ptototype = Object.create( Foo.prototype );
// ES6开始可以直接修改现有的 Bar.prototype 
    Object.setPrototypeOf( Bar.prototype, Foo.prototype );
```
### 六种继承方式
* 1.简单原型链继承:
* 这种继承方式的核心就是将子构造函数的prototype设置为父构造函数的一个对象实例,这里面虽然继承过来的属性是被存放在了prototype的属性里面,也就是说所有的子类属性都回访问这个.
* 由于每一次进行原型绑定的时候内部的[[prototype]]其实会执行一个浅copy对需要拷贝的原型,所以如果你的值是一些引用,这就会导致不同对象实际操作的是一个引用
```js
function Super(){
    this.val = 1;
    this.arr = [1] 
}
function Sub(){}
Sub.prototype = new Super();//核心
var sub1 = new Sub();
var sub2 = new Sub();
sub1.val = 2;
sub1.arr.push(2);
alert(sub1.val)//2
alert(sub2.val)//1
alert(sub1.arr)//1,2
alert(sub2.arr)//1,2
```
* 2.借用构造函数的继承:
* 为了防止上面的情况的出现,所以再考虑不要把父类的对象放入在原型链当中,而是直接在构造函数中引入父类的构造函数,这样一来父类对象需要继承的东西会完全的在子类的对象当中,而不在原型当中.
* 这里其实有很大的弊端就是其实继承的时候如果父类的方法当中有函数,也就是说每一个对象当中都有一个这样的函数,
这也就导致每个对象会随着父类的变多而占用的内存特别多.也就是所谓的函数的复用
```js
function Super(){
    this.val = 1;
    this.arr = [1];
    this.func = function(){
        console.log(this.val)
    }
}
function sub(val){
    Super.Call(this.val);
}
var sub1 = new Sub();
var sub2 = new Sub();
sub2.val = 2;
sub1.arr.push(2);
console.log(sub1.val)//1
console.log(sub2.val)//2
console.log(sub1.arr)//1,2
console.log(sub2.arr)//1
```
* 3.组合继承
* 组合继承就是为了解决上面的问题,对需要公用引用的部分就放在原型上,不需要公用的部分就放在每个对象当中
* 但是非要说这个方法不好的地方就是你会发现在sub1和Sub1._proto_也就是Sub.prototype是一样的其实完全不需要在Sub.prototype中写入这些变量,因为她们已经在子实例当中存在了
```js
function Super(){
    this.val = 1;
    this.arr = [1];
}
Super.prototype.fun1 = function(){}
Super.prototype.fun2 = function(){}
function Sub(){
    Super.call(this)
}
Sub.prototype = new Super();
var sub1 = new Sub(1);
var sub2 = new Sub(2);
console.log(sub1.fun1 === sub2.fun1)//true
```
* 4.寄生组合继承
* 这个名字真是又臭又长,就是为了改善上面的状况.将上面的Sub.prototype不用new Super()的方式创建,而是使用beget()函数创建,这样就不会携带多余的变量.
```js
function beget(obj){
    var F = function(){}
    F.prototype = obj;
    return new F();
}
function Super(){
    this.val = 1;
    this.arr = [1];
}
Super.prototype.fun = function(){}
function Sub(){
    Super.call(this)//核心
}
var proto = beget(Super.prototype);
proto.constructor = Sub //核心
Sub.prototype = proto //核心
var sub = new Sub();
console.log(sub.val);
console.log(sub.arr);
```
* 5.原型式继承
* 再进行改造一下这回将sup作为beget()函数的参数,但是因为更改了构造函数依然没有多余的属性值,而父类的属性也放入了原型上面,不过依然有问题共享引用的问题
```js
function beget(obj){
    var F = function(){}
    F.prototype = obj;
    return new F();
}
function Super(){
    this.val = 1;
    this.arr = [1];
}
var sup = new Super();
var sub = beget(sup)
```

### 检查"类"关系
* 我就是填充一下格式
```js
function Foo(){
}
Foo.prototype.blah =....;
var a = new Foo()
```
* 上面的代码可以通过 a instaceof Foo 来确定左边的对象是不是右边的函数构造出来的.也就是在a的原型链中是否指向Foo.prototype 的对象
* 如果你想比较两个对象之间的关系千万不要使用下面的方法
```js
function isRelatedTo(o1,o2){
    function F(){};
    F.prototype = 02;
    return o1 instanceof F
}
var a = { }
var b = Object.craete(a);
isRelatedTo(b,a)//true
```
* 上面的代码最大的问题就是实际上o1并不是由F构造的!!!所以即使得到了我们想要的但是它还是有弊端的
* `1 没搞懂有什么弊端
* `1 没搞懂有什么是js的反射
* 除了instaceof 还可以利用js的[[prototype]]反射机制的原理.isPrototypeOf(a)实际上就是看看a的整条[[Prototype]]链中有没有Foo.prototype.这一种方法并不需要间接的引用函数(Foo)
```js
Foo.prototype.isPrototypeOf( a ); // true
```

## 第六章:行为委托
* 因为这一章更多的是思想模式和理论的概念,所以就简单的自己总结一下.
### 比较一下类理论和委托理论
#### 1.类理论: 
* 我理解的类理论就是存在一个类和对象两个概念,只有类才能创建对象.是一个上下级别的概念.只有类才能实现继承和多态.比如说如果一部分东西拥有共同的方法则这个公共的方法应该作为基类.而对于不同的部分会有不同的重写.也就是子类的特殊化.这就是类也就是所谓的面向对象的设计.
```js
class Task{
    id;
    // 构造函数 Task()
    Task(ID) { id = ID; } 
    outputTask() { output( id ); }
}
class XYZ inherits Task { 
    label;
    // 构造函数 XYZ()
    XYZ(ID,Label) { super( ID ); label = Label; } 
    outputTask() { super(); output( label ); }
}
class ABC inherits Task { // ...
}
```
* 上面是书中的伪代码,值得强调的一点事.这种类理论,父类的公用的东西实际上是在每个继承类的对象的内部,每一个都是相互独立的.比如由XYZ类构造的对象会赋值一份Task,和XYZ中的行为.

#### 2.委托理论:
* 但是js当中的理论并不是这个样子的.js采用的委托理论.是因为js当中只有对象没有类.js当中创建对象不需要依靠class就像之前所说的那样子直接用{}创建对象字面量.但是js可以依靠对象之间的委托来实现这样的一个继承多态的功能.
* 同理我们可以同样的把一个共同的部分作为一个对象,而其他和他平等的对象实际上是于这个抽出共同行为的对象相互关联.每个对象不同的部分由对象本身来实现.而如果需要执行相同的地方可以在这个对象中调用关联的抽出共同行为的对象的方法.
* 这么说可能会比较绕口,来看一个代码体验一下.

```js
//正如同我们刚才所讲的创建一个对象,抽出共同的行为
Task={
setID: function(ID) { this.id = ID; },
outputID: function() { console.log( this.id ); }
}
// 将一个对象和这个公共行为抽出来的对象相互关联(对象之间是平等的)
XYZ = Object.create( Task );
// 为不同的地方添加不同的对象
XYZ.prepareTask = function(ID,Label) { 
    this.setID( ID );
    this.label = Label;
};
XYZ.outputTaskDetails = function() { 
    this.outputID();
    console.log( this.label );
};
```
* 根据js的[[prototype]]机制如果当需要调用共同的行为也就是Task中的方法,对象中肯定没有实现这样的一个方法,所以就会首先查询关联的对象有没有这个方法.于是查询Task则会检测到共同的方法对其调用.
* 对于上面的代码出现了id和label两个成员变量.他们都是存储在XYZ的内部而并不是Task!如果每一个对象都需要向Task里面存储东西的话.那就amzing了.
* 所以前面就会说为什么你再继承的时候要避免在对象中写重名的方法,这样你就会完全屏蔽掉你关联对象的方法.
* 上述中之所以调用Task的setID方法会在XYZ中出现ID是因为这里使用了前面提到的this的隐式绑定.
* 当你需要更加区分两者的差别的时候[P172也的图会给你答案](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch6.md)

### 更简洁的设计.
* 这里展现一下笔者是如何通过对象的关联来简洁代码的.
* 如果有一个功能需要登录后提交表单进行验证.则在MVC中的C中需要控制两个逻辑流程1,操作form表单得到信息.2.网络请求的控制.按照面向类的理论  

```js
// 从这里面抽取的共同的行为作为一个类
function Controller() {
this.errors = []; 
}
// 在这个共同的类当中加入共同的行为
Controller.prototype.showDialog(title,msg) 
{ // 给用户显示标题和消息

};
Controller.prototype.success = function(msg) {
this.showDialog( "Success", msg ); 
};
Controller.prototype.failure = function(err) { 
this.errors.push( err );
this.showDialog( "Error", err );
};

// 登录功能的对象
function LoginController() {
Controller.call( this ); 
}
// 把子类关联到父类
LoginController.prototype = Object.create( Controller.prototype );

LoginController.prototype.getUser = function() {
    return document.getElementById( "login_username" ).value;
};

LoginController.prototype.getPassword = function() {
    return document.getElementById( "login_password" ).value; 
};

LoginController.prototype.validateEntry = function(user,pw) { 
user = user || this.getUser();
pw = pw || this.getPassword();
if (!(user && pw)) { 
    return this.failure(
           "Please enter a username & password!"
        );
}else if (user.length < 5) {
return this.failure(
"Password must be 5+ characters!"
); }
// 如果执行到这里说明通过验证
return true; };

// 重写基础的failure(),模仿继承的重写.
LoginController.prototype.failure = function(err) {
//“super”调用 
Controller.prototype.failure.call(this,"Login invalid: " + err);
};

// 另一个子类
function AuthController(login) {
Controller.call( this ); // 合成
this.login = login;
}

// 把子类关联到父类 
AuthController.prototype = Object.create( Controller.prototype ); 
// 添加相同的行为
AuthController.prototype.server = function(url,data) {
return $.ajax( { url: url,
        data: data
    } );
};
// 添加相同的行为
AuthController.prototype.checkAuth = function() {
var user = this.login.getUser(); 
var pw = this.login.getPassword();
if (this.login.validateEntry( user, pw )) {
    this.server( "/check-auth",{ user: user,pw: pw })
.then( this.success.bind( this ) )
.fail( this.failure.bind( this ) ); 
}
};

// 重写基础的 success() 
AuthController.prototype.success = function() {
//“super”调用
Controller.prototype.success.call( this, "Authenticated!" ); 
};
// 重写基础的 failure() 
AuthController.prototype.failure = function(err) {
//“super”调用 
Controller.prototype.failure.call(this,"Auth Failed: " + err);
};
var auth = new AuthController(	
	new LoginController()
);
auth.checkAuth();  
```  

> 所有控制器共享的基础行为是success(..)、failure(..) 和 showDialog(..)。 子 类 LoginController 和 AuthController 通过重写 failure(..) 和 success(..) 来扩展默认基础类行为。此外，注意 AuthController 需要一个 LoginController 的实例来和登录表单进行 交互，因此这个实例变成了一个数据属性。

* 接下来上面的代码我们用对象的关联来简化这段代码
* 首先我们可以分析按照面向类的观点这样的一个代码是非常需要抽象出来一个controller从而有不同的实现的,但是在对象关联的设计模式当中并不是这样的.
* 简单的来说 上述面向类的思想就是A继承C,L也继承C.A在使用的时候又需要L中的行为来决定L和A公有C的行为(就是A中的checkAuth方法成功就是执行C的成功失败就是执行C的失败).总结一下就是A的之所以要继承C就是因为L的行为导致的所以你会发现我们完全可以把中间的A继承C和L继承C改为A关联L就行了
* 如果你想要说为什么A不能直接的继承L呢?如果A继承L在这种逻辑当中A需要覆写L的基础行为.但是A还需要根据L的行为来让L显示信息(也就是A需要通过checkAuth得到L的结果决定L当中是不是应该执行成功还是失败).所以这样的继承也就没有什么用处.  

```js
var LoginController = { 
errors: [],
getUser: function() {
return document.getElementById("login_username").value;
},
getPassword: function() {
return document.getElementById( "login_password").value; 
},
validateEntry: function(user,pw) { 
    user = user || this.getUser(); pw = pw || this.getPassword();
if (!(user && pw)) { 
    return this.failure("Please enter a username & password!");
}
else if (user.length < 5) {
return this.failure("Password must be 5+ characters!"); 
}
// 如果执行到这里说明通过验证
return true; 
},
showDialog: function(title,msg) { // 给用户显示标题和消息
},
failure: function(err) {
this.errors.push( err );
this.showDialog( "Error", "Login invalid: " + err ); }
};
// 让 AuthController 委托 LoginController
var AuthController = Object.create( LoginController );
AuthController.errors = []; AuthController.checkAuth = function() {
var user = this.getUser(); var pw = this.getPassword();
if (this.validateEntry( user, pw )) { 
    this.server( "/check-auth",{user: user,pw: pw })
.then( this.accepted.bind( this ) )
.fail( this.rejected.bind( this ) ); }
};
 AuthController.server = function(url,data) {
return $.ajax( { url: url,
              data: data
          } );
};
AuthController.accepted = function() {
this.showDialog( "Success", "Authenticated!" ) 
};
AuthController.rejected = function(err) { 
    this.failure( "Auth Failed: " + err );
};
```
* 所以按对象关联的设计模式来看,这样的代码更加简便AuthController需要使用LoginController的方法也就是将,既然A想要根据L的结果来做你的事情,那我们就让A把这件事情委托给B就好了!!!!! A和L就像兄弟一样,你让我干啥我就干啥
* 这里真的是看了好久了

### 更好的语法  
* 在上面的AuthController中你会发现这样写非常的反人类不像LoginController一样所以es6提供的一种方式就是
```js
// 使用更好的对象字面形式语法和简洁方法 
var AuthController = {
errors: [],
checkAuth() { },
server(url,data) { }
};
// 现在把 AuthController 关联到 LoginController 
Object.setPrototypeOf( AuthController, LoginController );
```
#### 反词法
* es6的语法糖实现了这样的一个功能
```js
var Foo = {
//es6
bar() { /*..*/ },
// 去除语法糖
bar: function() { /*..*/ },
};
```
* 简单来说就是bar(){}会被默认成一个匿名函数赋值给bar()(作者为什么用bar,难道是我邪恶了)但是这会有在当中无法自我引用的缺点当需要回调的时候就会显得比较乏力

### 自省
* 简单的来说就是检查实例的类型,就如之前所说的 a instaceof func;
* 正如之前说的instaceof只能用于一个对象和一个函数的关系,如果是两个函数需要怎么办呢.
#### 鸭子模型
* 如果他走起来像鸭子,那他就是鸭子.
```js
if (a1.walkLikeDuck) { 
    a1.walkLikeDuck();
}
```
* 就像上面只要a1能够调用walkLikeDuck这个方法,我们就认为他是这个类的实例,但是这是脆弱的.比如我们用promise的then来判断方法.但是这个then并不是一定会出现的.所义就会又问题了.




