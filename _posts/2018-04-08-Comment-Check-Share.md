---
layout: post
title: 一个注释率检查脚本引发的血案
category: Project-SHARE
---
在搜车实习期间和杨洋大佬写了一个检查注释率脚本的工具(当然大部分是杨洋大佬写的).下面来总结一下这个小脚本的各种坑.  

### 需求场景

这个脚本注释率的检查方式并不是十分暴力的直接检查整个文件的注释行数和代码行数的比值.而是只需要检测一些共有的变量,比如 oc 当中.h文件注释, java class 当中的 public字段以及 export 的变量.前两个不是很麻烦.但是如果想要找到 js 文件当中 export 的变量的话就有些复杂了.  

如下所示
```js
import a from './a';
import b from './b';
//这是 c 函数
function c(){
}
export default{
    a,
    b,
    c,
    //这是 d 函数
    d: function d-temp(){}
}
```
比如 c 和 d 两个函数都是写了注释的,所以注释检查也是应该通过的.但是由于我加入了a, b 这些 import进来的变量.就会导致 a, b 没有加注释,从而使检查不通过  

另一种状况是
```js
function c(){
    //这是 e 函数
    const e = function(){} 
}
export default{
    c
}
```
这时我们希望看到的是由于 c 函数并没有写注释所应该显示不通过,但是 e 函数写了注释就.就可能出现识别有误的情况,导致显示了注释通过.  

以上的情况也只是各种检查情况的冰山一角,因为实际业务当中的情况要考虑的实在是太多,所以现在写出来的脚本注释率也并不是很准确,但还是值得总结一下的.

### .js 和 .vue 文件的处理大概流程
由于出现了export时的各种情况,最开始我采取的方法是使用一些解析引擎转换 ast 语法树进行一些语义的处理.但是.....实在是太难了.....于是我更改了想法对js 文件做一层预处理流程如下.
1. 记录 import 中的变量
2. 移除 function 的函数体, 留住声明
3. 记录 export 中的变量(如果 export 的是一个对象,则会对这个对象递归.如果是一个 class 则会对扫描 除了 constructor 方法的注释)
4. export 的变量减去 import的变量 然后扫描全局注释率代码

### 解决 js 正则匹配高级特性不支持的问题
在上述过程的第二步,由于要剪切 function(){}中函数体的内容,但是遇到的情况如下
```js
//去掉前
function a(){
    function b(){
    }
}
export default {
    a,
    b
}
//去掉后
function a(){}
export default {
    a,
    b
}
```
对于这种情况需要识别的是最外层的 fucntion 的函数体.如果想要使用正则进行匹配,就需要用到正则的递归匹配.
```js
let regex = /(?<=function\(\))(.*\{(?1)*.*\}))/;
```
当我写出这个递归匹配的正则的时候惊讶的发现 js的正则引擎根本不支持这样的高级特性.于是乎我又想到了通过调用正则老大 perl 脚本来做这件事情.于是也就简单的学了一下 perl(不得不说是在是太难了)
```perl
#!/usr/bin/perl
use strict;
open(DATA, "<$ARGV[0]") or die "文件无法打开或路径不存在, $!";
my $content = do { local $/; <DATA> };
$content =~s/(?<=function\(\))(.*\{(?1)*.*\}))//mg;
print $content;
```
于是通过 child-process 的模块调用 perl处理脚本.由于代码文件一般都不是很大,所以也就直接采用了读取的方式.

### 正则的注释
当我看了 perl 的正则以后发现 perl 对正则的支持十分的好.加入 x 正则后缀可以支持
```perl
$regex = qr/
  \(                 # (1) match an open paren (                    
    (                #(2)，紧接着是
      [^()]+        #(3)，1个或多个非括号字符
    |               #(4)，或
      (??{$regex})  #(5)，正则式自身
    )*              #(6)，重复0或多次
  \)                #(7)，紧接着是闭括号
/x;
```
而我转头看看 js里面的正则
```js
const propRegExp = '\\s+props:.*?';
const commentRegExp = '[\\*\\/]\\/.*?\\n+';
const exportObjectRegExp = 'export(?:\\sdefault)?\\s\\{((.|\\n|\\r)*?)\\}';
```
我的神这都是什么鬼,写成字符串形式的正则不仅要写\\而且也不能换行.这种可读性实在是太差劲了.于是我自己简单的封装了一下.
```js
const regExpComment = function(flag) {
    this.regexp = '';
}
regExpComment.prototype.getRegEx = function _getRegEx() {
    var result;
    try {
        result = new RegExp(this.regExp);
    } catch (e) {
        console.error(e);
    }
    return result;
}
regExpComment.prototype.write = function _write() {
    var args = Array.prototype.slice.call(arguments);
    switch(args.length) {
        case 1:
            //注释被写在一个字符串中
            this.regexp.concat(filter(args[0]));
            break;
        case 2:
            //注释写在两个字符串中
            this.regexp.concat(transform(args[0]));
            break;
        default:
            return;
    }
}
const transform = function(str) {
    return str.replace(/\\/g,'\\');
}
const filter = function(str) {
    var resultArr = str.replace(/\n/g, '^&*(').split('^&*(');
    resultArr = Array.prototype.map.call(resultArr, (item) => {
        return item !== '';
    });
    resultArr.map((item) => {
        //先翻转字符串,去掉#后面的内容
        var reverseResult = item.split('').reverse().join('').replace(/#\w*\s*/,'');
        return reverseResult.split('').reverse().join('');
    })
}
// 使用方式一
var regExp = new regExpComment();
regExp.write('abc','匹配 abc 字符串');
regExp.write('\d', '匹配数字');
console.log(regExp.getRegEx().test('abc123'));
//使用方式二
var regExp = new regExpComment();
regExp.write(
    `
        abc #匹配abc
        \d #匹配数字
    `
);
```
这样就方便自己在写注释的时候的提高可读性.

### 总结
这个脚本注释率其实目前还有很多的问题需要解决,有很大的改进空间.比如最开始说的进行一些语义的检查,最开始下了很多功夫在上面,但是后来发现难度不小工作量野也很大就放弃掉了.回头改进改进~