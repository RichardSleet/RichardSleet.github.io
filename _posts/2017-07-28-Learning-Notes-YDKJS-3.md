---
layout: post
title: 你不懂的JS学习笔记-第三部分
category: LearingNote-YDKJS
---
# You don't KnowJS

# 第三部分:类型和语法  

## 第一章 类型  
### 值和类型
* js 当中的基本的类型就是前面提到最基础的六种,在es6当中新加入了symbol类型.
* 一定要注意的是js当中没有变量是没有类型的不像java当中 int a = 2.这个只能是一个int形的变量.在js当中变量是可以存放任意类型的值,而值才会有类型.除了object外其他都被成为基本类型
> • 空值(null)  
• 未定义(undefined)  
• 布尔值( boolean)  
• 数字(number)  
• 字符串(string)  
• 对象(object)  
• 符号(symbol，ES6 中新增)
* function会作为object的一个子类型被识别,所有的函数类型都会有一个内置的[[Call]]属性,可以理解可以被调用的对象,所以函数同样也可以拥有属性性.我个人认为函数当中的可以分为两部分一部分是函数部分,一部分是对象部分.顾名思义函数部分就是被调用[[call]]属性执行的一个域,对象部分就是一个对象.当我们访问foo.length的时候返回的是这个函数参数的长度  
```js
function foo(a,b,c){
    foo.a = 100;
    a = 200;
    console.log(a);
}
console.log(foo.a)//100
foo()//200
console.log(foo.length)//3
```
* 一个变量在没有值的时候会被声明undefined,如果你引用了一个没有声明的变量就会出现报错,恰巧的是如果你是用typeof检测这样一个类型依然会显示undefined但是其实应该为undeclar.这么说不如下面的代码来的直接
```js
var a;
a; // undefined
b; // ReferenceError: b is not defined
typeof a; // "undefined"
typeof b; // "undefined"
//我们无法通过typeof来判断 a和b是否被执行
```
* 虽然我们无法通过typeof判断是被申明还是不存在,但是这确实是js提供的安全机制
```js
// 这样会抛出错误,因为可能并没有声明这个变量
if (DEBUG) {
         console.log( "Debugging is starting" );
}
// 这样是安全的知道是不是有值,但是并没发判断是不是被声明了
if (typeof DEBUG !== "undefined") {
         console.log( "Debugging is starting" );
}
```
* 书上提到了一个很有疑惑的地方为了检测全局属性是不是都被声明了可以使用下面代码  
```js
//这是因为访问window的属性是不存在不会报错的
if (window.DEBUG) {
         // ..
}
if (!window.atob) {
         // ..
}
```
* `1发现的巧妙的问题
```js
let a = 1;
var b = 2;
console.log(window.a)//undefine
console.log(window.b)//2
//不明白为什么var属性会被绑定到window属性当中
```
## 第二章 值  
### 数组
* js当中的数组并不是和java,c#一样固定了数组的长度,对超过数组长度内容操作就会报错.js并不是这样的,js中数组可以容纳任何类型的值，可以是字符串、 数字、对象(object)，甚至是其他数组(多维数组就是通过这种方式来实现的).正因为如此在使用数组的时候更要小心
* 例如稀疏数组的使用和delete运算(即便是使用delete运算符依然没有办法改变length)
```js
var a = [];
a[0] = 0;
a[2] = 2;
a[1];//undefine
a.length; // 3
```
* 同样恶心的是数组也是对象就像对于funtion有了[[call]]可调用的区域,数组特别的地方在于在一个对象的基础之上加入了数字索引length属性等功能  
```js
var a = [ ];
a[0] = 1;
a["foobar"] = 2;
a.length;
a["foobar"];
a.foobar;
// 1 // 2 // 2
//如果使用的是可以被将至转换为number的值就会当做数字
var a = [ ];
a["13"] = 42;
a.length; // 14
```
* `1以后会添加一些数组常用的操作方法  
#### Array.prototype.slice方法(String类型也有slice方法)
* slice()方法顾明思意就是切割的意思
```js
const array = ['zero','one','two'];
let oneArray = array.slice(0)
//从0到最后一个
let anotherArray = array.slice(0,1)
//只剪切0
//下文提到会被用来转换类数组
```
#### Array.prototype.from方法
### 类数组
* 什么是类数组,类数组其实就是一个对象,只不过对象的属性名恰巧用了number.下面就是类数组
```js
let a ={
    1:123,
    2:1231
}
a[1]//123
```
* 在js当中类数组经常出现 1. 查询返回的dom元素列表 2.arguments参数类数组
```js
//采用slice方法转化
function foo() {
    var arr = Array.prototype.slice.call( arguments );
    arr.push( "bam" );
    console.log( arr );
    //或者使用es6
    var arr = Array.from( arguments );
}
foo( "bar", "baz" ); // ["bar","baz","bam"]
```
### 字符串
```js
var a = "foo";//字符串
var b = ["f","o","o"];//字符数组
a.length;// 3
b.length;// 3
a.indexOf( "o" );//1
b.indexOf( "o" );//1
var c = a.concat( "bar" ); //"foobar"
var d = b.concat( ["b","a","r"] );  // ["f","o","o","b","a","r"]
a === c; // false
b === d; // false
a; 
b;
// "foo"
// ["f","o","o"]
a[1] = "O";
b[1] = "O";
a; // "foo"
b; // ["f","O","o"]
```
* 我认为字符串和字符数组并没有什么关系只是可以相互转化而已,其实a[1]并不是很兼容所有的浏览器正确的是a.charAt(1)
* 数组是可边的向数组里面添加数据是不过是向数组当中的值附近的存储空间再次放入值而已
* 字符串是不可变,我们理解的可变只不过把字符串赋值给一个新的字符串
```js
let a = '123'
a[0] = 4
console.log(a)//123
```
* 所以依靠数组可变的特性可以解决很多的一些问题(例如反转)
* 在字符串转化字符数组的时候可能会因为一些unicode带来的问题  
* `1这里会收集string常用的放的方法
### 数字
* JavaScript 中的数字类型是基 于IEEE 754标准来实现的(没错就是和java一样这种标准很是恶心,java也会有.精度会有误差)
* 接下来就说说恶心的地方
```js
0.1 + 0.2 === 0.3; // false
```
* 如何解决这么一个问题呢就是给定一个误差范围
```js
//es6可以使用Number.EPSILON
function numbersCloseEnoughToEqual(n1,n2) {
    //指定误差小数
    Number.EPSILON = Math.pow(2,-52);
    return Math.abs( n1 - n2 ) < Number.EPSILON;
}
var a = 0.1 + 0.2;
var b = 0.3;
numbersCloseEnoughToEqual( a, b );
numbersCloseEnoughToEqual( 0.0000001, 0.0000002 );  // false
```
* 最大最小的浮点数可以使用Number.MAX_VAULE 和Number.MIN_VALUE 
* 最大整数和最小整数 Number.MAX_SAFE_INTEGER 和Number.MIN_SAFE_INTEGER
#### 
* `1这里会收集number常用的方法
### 特殊值
#### undefine和null
* undefined 和 null 常被用来表示“空的”值或“不是值”的值。二者之间有一些细微的差 别。例如
* null指空值(empty value)
* undefined指没有值(missing value)
* undefined指从未赋值
* null指曾赋过值，但是目前没有值
* void运算符可以规定表达式没有返回的值
```js
var a = 42;
console.log( void a, a ); // undefined 42
```
#### NaN
* 如果数学运算的操作数不是数字类型(或者无法解析为常规的十进制或十六进制数字)， 就无法返回一个有效的数字，这种情况下返回值为 NaN。(但是认识Number类型)但是每一个NaN却不相等
```js
var a = 2 * 'foo' //NaN
typeof a === "number";  // true
a == NaN;   // false
a === NaN;  // false
a !== NaN;  // true 
//尴尬不
```
* 可以通过isNAN()方法来盘对岸是不是NaN但是
```js
var a = 2 / "foo";
var b = "foo";
a; // NaN
b; "foo"
window.isNaN( a ); // true 
window.isNaN( b ); // true——晕!
```
* 不过可以使用es6的Number.isNaN(..)
```js
if (!Number.isNaN) {
    Number.isNaN = function(n) {
        return (
            typeof n === "number" &&
            window.isNaN( n )
); };
}
var a = 2 / "foo";
var b = "foo";
Number.isNaN( a ); // true 
Number.isNaN( b ); // false——好!
```
#### 无穷数
```js
var a = 1/0;//Infinity
var b = -1/0; // -Infinity
var b = 1/-0; // -Infinity
var a = Number.MAX_VALUE; // 1.7976931348623157e+308
a + a; // Infinity
a + Math.pow( 2, 970 ); // Infinity
a + Math.pow( 2, 969 ); // 1.7976931348623157e+308
a/a //NaN
```
* 计算结果一旦溢出为无穷数(infinity)就无法再得到有穷数。换句话说，就是你可以从有 穷走向无穷，但无法从无穷回到有穷。 
#### 零值
* `1这里是在很不常用用的时候我会补上的
### 值和引用
>例如，在 C++ 中如果要向函数传递一个数字并在函数中更改它的值，就可以这样来声明参 数int& myNum，即如果传递的变量是x，myNum就是指向x的引用。引用就像一种特殊的指 针，是来指向变量的指针(别名)。如果参数不声明为引用的话，参数值总是通过值复制 的方式传递，即便对复杂的对象值也是如此
* 请看下面代码
```js
var a = 2;
var b = a; // b是a的值的一个副本 b++;
a; // 2
b; // 3
var c = [1,2,3];
var d = c; // d是[1,2,3]的一个引用 d.push( 4 );
c; // [1,2,3,4]
d; // [1,2,3,4]
```
* 简单值(即标量基本类型值，scalar primitive)总是通过值复制的方式来赋值/传递，包括 null、undefined、字符串、数字、布尔和 ES6 中的 symbol。
* 复合值(compound value)——对象(包括数组和封装对象，参见第3章)和函数，则总 是通过引用复制的方式来赋值 / 传递。
* 下面的代码把b=[4,5,6]不要理解成给一个变量赋值而是理解成将这个引用指向另一个变量
```js
var a = [1,2,3];
var b = a;
a; // [1,2,3]
b; // [1,2,3]
// 然后
b = [4,5,6]; 
a; // [1,2,3] 
b; // [4,5,6]
```
* 而对于函数的参数来说,参数也就是
```js
function foo(x) {
x.push( 4 );
x; // [1,2,3,4]
// 然后
x = [4,5,6]; 
x.push( 7 );
x; // [4,5,6,7]
}
var a = [1,2,3];
foo( a );
a; // 是[1,2,3,4]，不是[4,5,6,7]
```
* 如果并不想在函数中修改外部参数的值可以使用数组的slice()方法,因为slice会返回一个全新的数组赋本(我认为这里是深复制)所以操作的是这个值的副本.对于对象可以使用json巧妙转化
```js
foo( a.slice() );
foo( JSON.prase(JSON.stringfy(a)))
```
* 如果将简单转为复杂就要注意了
```js
function foo(x) {
x = x + 1;
x; // 3 
}
var a = 2;
var b = new Number( a ); // Object(a)也一样
foo( b );
console.log( b ); // 是2，不是3
```
* 这是因为基本类型的值是无法更改的,和string无法更改因为他是简单值,array可以更改因为他是复杂值
* 上面代码中x=x+1,其实可以理解为把x原来的数字对象拆成数字然后把x再次指向新的数字
## 第三章 原生函数
* js当中的内建函数
```js
• String() 
• Number() 
• Boolean() 
• Array()
• Object()
• Function()
• RegExp()
• Date()
• Error()
• Symbol()——ES6中新加入的!
```
* 我理解的js内置函数就是为了给各种类型的值提供一些基础操作的方法所以可以通过内置还是将其封装(或默认)为对象方便操作.这样的操作会在你调用这个基本类型的工具方法时默认转化
```js
var a = new String( "abc" );
typeof a; // 是"object"，不是"String" 
a instanceof String; // true 
Object.prototype.toString.call( a ); // "[object String]"
```
### 内部属性[[class]]
* 所有的object基本类型function,object,array都回有这么一种类型(undefine和null例外),和[[prototype]]一样无法通过属性直接访问
但是可以使用Object.prototype.toString(..)进行查看
```js
Object.prototype.toString.call( [1,2,3] );
// "[object Array]"
Object.prototype.toString.call( /regex-literal/i );
// "[object RegExp]"
Object.prototype.toString.call( null );
// "[object Null]"
Object.prototype.toString.call( undefined );
// "[object Undefined]"
```
### 封装对象
* 封装对象需要注意的是
```js
var a = new Boolean( false );
if (!a) {
console.log( "Oops" ); // 执行不到这里
}
```
* 因为a已经是一个对象了,肯定会存在
* 如果你封装了一个对象然后有希望对他进行操作的时候比如你把数字封装成为对象但是有需要进行操作运算等可以使用
valueOf方法
```js
var a = new String( "abc" );
var b = new Number( 42 );
var c = new Boolean( true );
a.valueOf(); // "abc"
b.valueOf(); // 42
c.valueOf(); // true
```
* 这些操作尽量都使用引擎默认的转化最好
### 原生函数作为构造函数
#### array(..)
* 使用array作为构造函数的时候如果只有一个参数会被当做length属性的赋值
```js
var a = new Array( 3 );
a.length; // 3
a;
```
*`1这里笔者说了array只有length情况的时候的不同浏览器的处理
#### Object(..)、Function(..) 和 RegExp(..)
```js
var c = new Object();
c.foo = "bar";
c; // { foo: "bar" }
var d = { foo: "bar" };
d; // { foo: "bar" }
var e = new Function( "a", "return a * 2;" );
var f = function(a) { return a * 2; }
function g(a) { return a * 2; }
var h = new RegExp( "^a*b+", "g" );
var i = /^a*b+/g;
```
* 像上面的Function函数尽量应该避免使用
* 而RegExp在使用的动态的正则是很厉害的
```js
var name = "Kyle";
var namePattern = new RegExp( "\\b(?:" + name + ")+\\b", "ig" );
var matches = someText.match( namePattern );
```
#### Date(..) 和 Error(..)
* Dare 和 Error 对象的创建只能靠这两个了
```js
function foo(x) {
    if (!x) {
        throw new Error( "x wasn’t provided" );
    } 
}
if (!Date.now) {
    Date.now = function(){
        return (new Date()).getTime();
    };
}
```
>除 Error(..) 之外，还有一些针对特定错误类型的原生构造函数，如 EvalError(..)、 RangeError(..)、ReferenceError(..)、SyntaxError(..)、 TypeError(..) 和 URIError(..)。这些构造函数很少被直接使用，它们在程序 发生异常(比如试图使用未声明的变量产生 ReferenceError 错误)时会被自 动调用。 
* 我是用来排版的 
#### Symbol(..)
* 为了保证前端在使用其他模块时出现的命名冲突问题es6长生了Symbol()类型
* 切记符号和符号之间是不等的
```js
var mysym = Symbol( "my own symbol" );
 mysym;
mysym.toString();
typeof mysym;
// Symbol(my own symbol)
// "Symbol(my own symbol)"
// "symbol"
var a = { };
a[mysym] = "foobar";
Object.getOwnPropertySymbols( a );
// [ Symbol(my own symbol) ]
~~~分割线~~~
var a = Symbol("a");
var b = Symbol("a");
a === b //false
```
#### 原生原型
```js
typeof Function.prototype; // "function"
Function.prototype();  // 空函数!
RegExp.prototype.toString();// "/(?:)/"
"abc".match( RegExp.prototype );
~~~~~~我是分割线~~~~~~
function foo(){
    let a = 0;
}
foo.__proto__ == Function.prototype //true
```
* 函数的prototype是指向Object.prototype而函数的[[prototype]]是指向Function.prototype
* 你甚至还可以修改Function上面原生原型

## 第四章 强制类型转换
### 值类型转换
* 切记对象类型一类的是不存在强制类型转换这么一说的,在其他语言中可能会出现对象的强制类型比如动态字典和静态字典
* 类型转换发生在静态类型语言的编译阶段，而强制类型转换则发生在动态类型语言的运行时
* 而强制类型转换又可以分成隐式和显式
* 注意可没有使用new操作符
```js
var a = 42;
var b = a + ""; // 隐式强制类型转换 
var c = String( a ); // 显式强制类型转换
```
### 抽象值操作
#### ToString
* 如果对象自己有ToString方法就不会调用Object.prototype.toString()
* JSOn.stringfy()和ToString的效果差不错(前提是json字符串化的但愿需要解析)
```js
JSON.stringify( 42 ); // "42"
JSON.stringify( "42" ); // ""42""(含有双引号的字符串) JSON.stringify( null ); // "null"
JSON.stringify( true ); // "true"
JSON.stringify( undefined );
JSON.stringify( function(){} );
JSON.stringify(
[1,undefined,function(){},4]
);
JSON.stringify(
    {a:2,b:function(){})}
};  //"{a:2}"
```
* JSON.strigngfy()执行的时候会首先调用toJson()(如果有的话)对其得到的结果进行序列化.所以一般使用toJson返回一个安全的json值进行序列化
```js
var o = { };
var a = { 
b: 42,
c: o,
d: function(){}
};
// 在a中创建一个循环引用 
o.e = a;
// 循环引用在这里会产生错误 
// JSON.stringify( a );
// 自定义的JSON序列化 
a.toJSON = function() {
// 序列化仅包含b
         return { b: this.b };
};
JSON.stringify( a ); // "{"b":42}"
```
* 循环引用简单的来说就是对象a里面有指向对象b的指针,对象b也有指向对象a的指针.所以当释放内存的时候两个对象都无法释放.二者里面json在序列化的时候首先序列化a再序列化b再序列化a这样就会产生错误
* JSON.stringify()的第二个参数replacer可以帮助我们制定安全的json对象
```js
var a = { b: 42,
c: "42",
d: [1,2,3] 
};
JSON.stringify( a, ["b","c"] ); // "{"b":42,"c":"42"}"
JSON.stringify( a, function(key,value){
    if (k !== "c") return v;
} );
// "{"b":42,"d":[1,2,3]}"
```
* 参数是数组的时候会只返回数组当中的属性,如果是一个函数会得到key和value的参数确定返回的结果,如果是undefine就会自动忽视.
* 第三个参数还会调整缩进格式
```js
var a = { b: 42,
c: "42",
d: [1,2,3] };
JSON.stringify( a, null, 3 );
/*
{
    缩进三个空格
   'b':42    
}
*/
```
* JSON.stringify(..) 并不是强制类型转换,因为他只是调用了toString()而已
### ToNumber
*  true 转换为 1，false 转换为 0。undefined 转换为 NaN，null 转换为 0。
* `1笔者这里讲了许多ToNUmber的例子以后补上
### TOBoolean
>
首先也是最重要的一点是，JavaScript 中有两个关键词 true 和 false，分别代表布尔类型 中的真和假。我们常误以为数值 1 和 0 分别等同于 true 和 false。在有些语言中可能是这 样，但在 JavaScript 中布尔值和数字是不一样的。虽然我们可以将 1 强制类型转换为 true， 将 0 强制类型转换为 false，反之亦然，但它们并不是一回事。
* 我是用来排版的
#### 假值
* js当中只有可以被转换成false的值和其他(剩余的就是true),就是下面所列的是假值表
```js
• undefined
• null
• false
• +0、-0 和 NaN
• "" //我认为''也是假值
```
* 如果把0封装成为一个那么就不再属于假值,例如以下
```js
var a = new Boolean( false );
var b = new Number( 0 );
var c = new String( "" );
var d = Boolean( a && b && c );
d; // true
//如果不用Boolean进行强制转换就会变成string
```
#### 假值对象
* 对象一定都会是真的值,但是如果对象其实不是来自js范围呢,而是外部为他注入的则就会是个假值,最明显的就是document.all()(已经废除)
* `1这个看的不是很多回来补上
#### 真值
* 除了假值表以外的都是真值
```js
var a = "false";
var b = "0";
var c = "''";
var d = Boolean( a && b && c );
d;
~~~~分割线~~~~
var a = [];
var b = {};
var c = function(){};
var d = Boolean( a && b && c );
d;
```
* 上面的都不是假值所以是真值
### 显示强类型转化
#### 字符串和数字之间
```js
var a = 42;
var b = String( a );
var c = "3.14";
var d = Number( c );
b; // "42"
d; // 3.14
~~~不仅仅有上述的方法实现,前面提到的函数也是转换~~~
var a = 42;
var b = a.toString();
var c = "3.14";
var d = +c;
b; // "42"
d; // 3.14
```
* 其实使用toString()是显示和隐式都会用到的因为调用tostring的方法是需要将其封装成为一个类才可以进行的
* 而+是不是取决于个人理解,一般认为单元运算是显示的.看看下面的奇葩代码
```js
var c = "3.14";
var d = 5+ +c;
d; // 8.14
```
* 因为++会被当做递增来处理所以中间加上空格
* 使用+还可以让日期显示的转换成数字
```js
var d = new Date( "Mon, 18 Aug 2014 08:53:06 CDT")
+d //1408369986000
~~~另一种方法~~~~~~
var timestamp = new Date().getTime();
// var timestamp = (new Date()).getTime();
// var timestamp = (new Date).getTime();
```
* 这简直不要太方便了
* `1对于书中用到的~运算符我就不做解释了(感觉会很少用到)(60)
#### 解析数字字符串
```js
var a = "42";
var b = "42px";
Number( a );    // 42
parseInt( a );  // 42
Number( b );    // NaN
parseInt( b );  // 42
```
* 看了上面的代码应该明白解析被转换是有区别的解转换是全部转换.而解析代码的时候会从左往右一次解析,遇到没有办法解析的才会返回解析的值
* 当然parseInt是只支持字符串的
* `1接下来这一页我就抑郁了的不懂了是在是有点怪癖回来补上(64)
#### 布尔的转换
* `1我直接跳过了(65)
### 隐式前传类型转换
#### 字符串和数字之间的隐式
```js
var a = "42";
var b = "0";
var c = 42;
var d = 0;
a + b; // "420"
c + d; // 42
```
* 前面认为的+单元算符为显示可是这里面的,+是双运算符之所以这么认为是因为,你的本意其实是想让两个东西加起来.转换是次要的,所以这样就被认为是显示的(然而这是有诟病的)
* 很明显如果使用了加号两边有数字和字符串应该怎么转换
```js
var a = [1,2];
var b = [3,4];
a + b; // "1,23,4"
```
* `1我认为这就是上面的弊端,难以理解.对于为什么这样子以后补上
* 在这里我简化一下列举常用的隐式操作,对于一些奇葩的就不做笔记了
```js
var a = 42;
var b = a + "";
b; // "42"
~~~分割线~~~~
//默认调用不一样
//因为toString屏蔽了原型上的tostring
var a = {
valueOf: function() { return 42; },
toString: function() { return 4; }
};
a + "";         // "42"
String( a );    // "4"
~~~分割线~~~~
var a = "3.14";
var b = a - 0;
b; // 3.14
```
* `1剩下的转换我就跳过了
### || 和 &&
* 这个就值得一提了,因为其实他们返回的并不是true和false,这是因为我们用的时候会进行显示的转换(我也不确定是不是显示)
```js
var a = 42;
var b = "abc"; 
var c = null;
a || b; //42
a && b; //"abc"
c || b; // "abc"
 c && b; // null
```
* 且 和 或 首先会对第一个操作数(a 和 c)执行条件判断，如果其不是布尔值(如上例)就 先进行 ToBoolean 强制类型转换，然后再执行条件判断。
* 对于 或 来说，如果条件判断结果为 true 就返回第一个操作数(a 和 c)的值，如果为 false 就返回第二个操作数(b)的值。
* 且 则相反，如果条件判断结果为 true 就返回第二个操作数(b)的值，如果为 false 就返 回第一个操作数(a 和 c)的值。
* 其实就是&&有假返回假 否则返回第一个 ,||有真返回真.否则返回后一个.所以就有下面的代码
```js
a || b;
// 大致相当于(roughly equivalent to): 
a ? a : b;
a && b;
// 大致相当于(roughly equivalent to): 
a ? b : a;
```
* 所以会经常见到这样的一个代码
```js
 function foo(a,b) {
    a = a || "hello";
    b = b || "world";
    console.log( a + " " + b );
}
foo();                  // "hello world"
foo( "yeah", "yeah!" ); // "yeah yeah!"
```
* `1 剩下的再次跳过(76)
### 宽松相等和严格相等
* 一句话 = 允许在相等比较中进行强制类型转换，而 === 不允许
* `1 剩下的比较实例十分详细,有空补上
## 第五章 语法
### 语句和表达式
* 在第一部分看到自执行的函数的时候,有一段对于语法的分析`1当时就没有弄懂什么是表达式
* 简单的来说,语句相对于句子,表达式相当于短语,运算符相当于标点和句子
```js
var a = 3 * 6;
var b = a;
b;
```
* var a = 3 * 6 是一个声明语句(赋值只是加上的)当然我认为a = 3 * 6 也是一个语句应该叫做赋值语句 b也是一个语句只不过没有意义
### 语句的结果值
* 当我在chrome中写练习代码的时候还是每次执行完成以后都会有一些undefined或者其他的值出来的,这并不是chrome自带的功能.而是每一个语句都有一个结果值(和函数的返回值不是一个概念)
* 比如你在chrome输入var a = 42 会得到一个undefined.一般来说声明语句(尽管有赋值表达式)结果值都回为undefined而赋值语句会返回你进行赋值的结果,无意义的语句也是返回undeined
* 对于代码块中的返回值,会返回最后一个语句的结果值
```js
var b;
if(true){
    b = 4 +2
}
```
* 则上面会返回 6.如果使用了eval(...)方法也会返回传入语句字符串的结果值,es7中也加入了do表达式的提案
### 表达式的副作用
* 说几个特例
* ++和--是一元运算符,++a中++在前面的时候会产生副作用就是让a的值递增
* a++的时候就是就没有产生副作用存放值,因为后面什么都没有.
* 而 b = a ++ 的实质就是首先把a的结果值输出再对a进行递增, 而 b =++a则是先将a进行递增再返回结果值
```js
var a = 42;
var b = a++;
a;  // 43
b;  // 42
~~~分割线~~~~
var a = 42;
a++;    // 42
a;      // 43
++a;    // 44
a;      // 44
```
* 其实完全可以把a 和 ++ 分开看待 简单来说谁在前先干什么事情
* `1这一节讲的很细致回头补上
### 上下文规则
* {...}大括号的不同解释
* 不用说的就是用来作为对象字面量,可是如果没有let person = 的话是个什么情况
```js
let persone = {
    name : 'Jobs'
}
~~~分割线~~~
{
    name : 'Jobs'
}
```
* 对于不进行赋值的其实这并不是一个没有赋值的对象.只是单纯的想其他语言一样的语法块,就像if(){}中一样
* 如果{...}代表的是一个语法快那name : 'Jobs'就像当于是一个标签就想其他语言当中在使用goto,break,continue的时候的标签一样,当你不进行跳转的时候也就没有什么意义了
* 标签也能用于非循环代码块，但只有 break 才可以。
```js
// 标签为foo的循环
foo: for (var i=0; i<4; i++) {
    for (var j=0; j<4; j++) {
        // 如果j和i相等，继续外层循环 
        if (j == i) {
        // 跳转到foo的下一个循环
                  continue foo;
        }
        // 跳过奇数结果
        if ((j * i) % 2 == 1) {
            // 继续内层循环(没有标签的)
                continue; 
        }
                console.log( i, j );
        }
}
```
* JSON 的确是 JavaScript 语法的一个子集，但是 JSON 本身并不是合法的 JavaScript 语法。
* `1 102
### 对象的解构
* 下面就是对象的解构,当然使用require经常用到
```js
function getData() {
          // ..
          return {
              a: 42,
              b: "foo" 
        };
}
var { a, b } = getData();
console.log( a, b ); // 42 "foo"
~~~分割线~~~~
//文件a
module.export ={
    a : b,
    c : d
}
//文件b
var {a ,b} = require('./a')
```
* 对于if和else 来说其实返回的就是一个语句的表达式,而{}只是吧多个语句返回的结果变成一个
```js
if (a) doSomething( a );
if (a) { doSomething( a ); }
```
## 运算符优先级
* [我觉得直接看我就行了](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Operator_Precedence);
* `1 125
## 自动分号
* 刚刚学js的时候好奇的就是竟然不加分号也不报错,这因为ASI会帮助我们自动的进行补齐分号
* 但是特别注意的是只会在换行符的情况下
* 有时不带分号可能会给ASI带来一些误解的问题,下面代码就会声明c变量
```js
var a = 42, b 
c;
```
* ASI判断的标准应该就是是不是可以当做一个语句处理可以不加分号,不可以加上分号
## 错误机制
* `1 134
* TDZ 暂时性死去就是在代码中的变量还没有初始化是不能北银用的
```js
{
          a = 2;      // ReferenceError!
          let a;
}
~~~分割线~~~~
{
    //神奇的是如果你不声明不报错,你申明反而有错
          typeof a;   // undefined
          typeof b;   // ReferenceError! (TDZ)
          let b;
}
```
## 函数的参数
* `1 136
## try catch finally 
* 这里先说说特例吧
```js
function foo() {
try {
    return 42;
}
finally {
    console.log( "Hello" );
    console.log( "never runs" );
}}
console.log( foo() );
// Hello
// 42
```
* 这里 return 42 先执行，并将 foo() 函数的返回值设置为 42。然后 try 执行完毕，接着执 行 finally。最后 foo() 函数执行完毕，console.log(..) 显示返回值。
* 如果连finally都有异常则函数就会终止,即使try中有了return也会丢弃
* `1 139










