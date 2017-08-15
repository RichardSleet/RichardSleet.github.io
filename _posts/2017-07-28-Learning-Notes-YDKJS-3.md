---
layout: post
title: 你不懂的JS学习笔记-第三部分
category: LearingNote-YDKJS
---
# You don't KnowJS
> 引语：十分感谢React中文社区开源群的张野大大给我推荐的[你不懂的JS](https://github.com/getify/You-Dont-Know-JS)这本书，网上有许多开源人士都参与了翻译这本书，再次十分感谢。这里记录一下读书的学习笔记，***个人理解仅供参考，真正的学习还需要读原著，如有改正和探讨谢谢指出***

# 第三部分:类型和语法

## 第一章 类型

### 值和类型
* js 当中的基本的类型就是前面提到最基础的六种,在es6当中新加入了symbol类型.
* 一定要注意的是js当中没有变量是没有类型的不行java当中 int a = 2.这个只能是一个int形的变量.在js当中变量是可以存放任意类型的值,而值才会有类型.除了object外其他都被成为基本类型
> • 空值(null)  
• 未定义(undefined)  
• 布尔值( boolean)  
• 数字(number)  
• 字符串(string)  
• 对象(object)  
• 符号(symbol，ES6 中新增)
* 一个变量在没有值的时候会被声明undefined,如果你引用了一个没有声明的变量就会出现报错,恰巧的是如果你是用typeof检测这样一个类型依然会显示undefined但是其实应该为undeclar
```js
var a;
a; // undefined
b; // ReferenceError: b is not defined
typeof a; // "undefined"
typeof b; // "undefined"
```
* 所以为了避免上面的情况用下面的方法来判断DEBUG变量是不是被定义了,更加保险
```js
// 这样会抛出错误 
if (DEBUG) {
         console.log( "Debugging is starting" );
     }
// 这样是安全的
if (typeof DEBUG !== "undefined") {
         console.log( "Debugging is starting" );
     }

```