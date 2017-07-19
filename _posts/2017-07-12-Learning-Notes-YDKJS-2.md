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
* 上面这段代码需要好好揣摩一下,首先说说bar,虽然函数中写了var bar = obj.foo,但是值得注意的是bar应该和obj.foo指向的是同一个地方,这个 = 是一个深拷贝而并非浅拷贝(我也不知道这么说是不是很规范)
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
#### 1. 当显示绑定传入的是this为null或者为undefined的时候 等价于默认绑定
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
#### 2.间接引用  
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


## 第三章:对象
### 对象的语法
对象的定义可以有两种形
#### 1.文字语法形式即对象字面量
```js
var myObj = {
    key : value
    //....
}
```
### 2.构造型形式即是通过new
```js
var myObj = new Obj();
myObj.key = value
```
* 以上两种方法基本上是一样的
### js的类型
#### js在es5当中的一些基本类型
1. string
2. number
3. boolean
4. null
5. undefined
6. object
* 切记上面的前五种类型并不是对象.对象只是js的一个基本类型之一
* 上面的基本类型的字面量是没有任何的方法的,之所以可以使用(str.length)是因为上面的类型默认被转换为下面的内置对象
* string -> String, number->Number , boolean-> Boolean
* null 和 undefined 没有构造形 
#### 内置对象
js语音提供了object类型的内置对象
1. String (大写的呦)
2. Number
3. Boolean
4. Object
5. Function
6. Array
7. Date
8. RegExp
9. Error
* 当然在es6当中不仅仅对已有的对象方法进行扩展而且加入了一些新的对象 eg Symbol 这些并不确定算不算是内置对象(Promise,Set,Map)
* 笔者特别提到了这些对象虽然很想其他语言的Class但是她们其实并不算是严格意义上的对象,而只是普通函数而这些普通的函数可以通过调用new来进行构造化.

### 内容
* 简单的来说就是写在一个内容里面值并不是存在对象内部而是保留了这个指针(从技术角度 来说就是引用)指向真正存储的位置
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
* 笔者还简单的说了一下es6的symbol这里做下记录
* 这里我会写一些有趣的代码探讨一下

#### 2.属性和方法
* 在JAVA和其他的语言当中,当访问的属性是一个函数的时候,大家会把他叫做方法.比如人的一个方法就是学习
* 但是!在js中这样做是错的因为从技术角度来说，函数永远不会“属于”一个对象，所以把对象内部引用的函数称为“方法”似乎有点不妥。
```js
function foo(){
    console.log('foo');
}
var  someFoo = foo;
var myObject = {
    someFoo:foo
}
foo; // function foo(){..}
someFoo; // function foo(){..} 
myObject.someFoo; // function foo(){..}
```
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
* 像java这样的函数就会有一个copy()的方法用来提供深拷贝和浅拷贝,但是js并不同js的深copy还是很复杂的
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

var myObject = { a:2
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
* 但需要给一个属性访属性描述符的时候,可以使用Object.defineProperty方法
```js
var myObject = {};
bject.defineProperty( myObject, "a", {
        value: 2,
w       ritable: true, 
        configurable: true, 
        enumerable: true
} );
myObject.a; // 2
```
##### 1.writable
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
##### 2.configurable
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
##### 3.Enumerable
* 这个属性是指该属性是否可以被遍历比如for in 循环 还有es6当中的iterator的迭代器

#### 6. 不变性
* 如果你希望一个对象的所有直接属性都不可以被修改,那么你还需要禁止这个属性的引用(但是这样是不提倡的)
```js
myImmutableObject.foo; // [1,2,3]
myImmutableObject.foo.push( 4 );
myImmutableObject.foo; // [1,2,3,4]
```
##### 1.对象常量
* writable:false 和 configurable:false 就可以创建一个对象常量
##### 2.不可扩展
* 如果禁止向对象添加属性可以使用 bject.prevent Extensions(..):
```js
var myObject = { 
    a:2
};
Object.preventExtensions( myObject );
myObject.b = 3;
myObject.b; // undefined
```
##### 3.密封
* Object.seal(..) 会创建一个“密封”的对象，这个方法实际上会在一个现有对象上调用 Object.preventExtensions(..) 并把所有现有属性标记为 configurable:false。
但是可以修改属性的值

##### 4.冻结
* Object.freeze(..) 会创建一个冻结对象，这个方法实际上会在一个现有对象上调用 Object.seal(..) 并把所有“数据访问”属性标记为 writable:false，这样就无法修改它们 的值。
* 以上方法都会不会对对象内部引用的其他引用起作用

#### 7. [[Get]]
```js
var myObject = { 
a: 2
};
myObject.a; // 2
```
