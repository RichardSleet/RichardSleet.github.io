---
layout: post
title: 你不懂的JS学习笔记-第四部分
category: LearingNote-YDKJS
---
# 第四部分:异步和性能

## 异步
谈及异步的话不得不说的就是一些的CS的概念术语  
1. **并发**所为的并发概念就是在**一段时间**内可以处理两个以上的事情,好比食堂里两个窗口然后只有一个大妈盛饭所以这个大妈一会在两个窗口之间来回走动,保证两个窗口的人都可拿到饭.这就是并发
2. **并行**而对于并行其意思就是在**每个时刻**都可以处理两个以上的事情,好比食堂里面有两个窗口,两个窗口有两个大妈分别来盛饭,两个窗口的每个人都可以得到饭.这个就是并行,总体来说并行的力度要比并发的执行粒度要大很多.
3. **异步**这个概念就很基本了,就相当于在需要执行耗时操作的时候并不需要等到这个耗时操作结束再去执行后面的过程.而是不管他怎么样执行都会接着执行后面的代码.等到操作完成再回到原来的地方把后面的工作做完.这就是异步.同样好比,一个学生去食堂窗口打饭,然后因为要的饭等待厨师把饭做好,所以食堂大妈让这个学生回去等待接着让后面的学生继续打饭,等到厨师的饭做好了,再把1学生叫回来.
4. **同步**如果和上面的情况相反一直等到耗时操作结束这样的操作就叫同步.

我最开始学js的时候给身边学android的同学说这个叫js的东西好像并没有线程,随即得到同学回复:那这个js能干什么(笑哭).其实现在想想还是很傻的.JS本身虽然不能够像java一样创建一个线程,但是因为所附属的native环境(比如说浏览器)会为JS提供线程的机制.  

因为我曾经写过一丢丢ReactNative的代码所以用这个举个例子,由于RN的逻辑代码大多是用JS分别运行在不同平台的JS环境上(android是v8,iOS应该是JSCore)所以如果当你需要读取sd卡中的内容或者说对服务器进行并发的请求(等异步操作),你可能就需要在你的宿主环境封装一个线程,JS把一些信息交给这个宿主环境的线程然后由这个线程发出请求,于是这个线程阻塞而JS则会继续的执行下面的逻辑,等到这个宿主环境的线程收到请求的时候,他会把相关结果返回给JS,这时候在JS这边就会再回头处理回掉函数或者其他异步的方式.所以不是说JS没有线程而是JS的这些请求都会交给宿主环境(浏览器,手机等来做) 

### 事件循环(event loop)和任务队列(job queue)
**event loop**是一种处理消息的机制,上面就说过因为一些耗时操作都交给了Web api 或者说一些宿主环境来执行.所以当这个耗时操作执行完了以后,怎么样确保每一个耗时操作的返回内容都会被执行呢,这就是事件循环干的事情.下面上一个MDN的图![event loop](https://mdn.mozillademos.org/files/4617/default.svg)
Event Loop的机制会保证每次都从queue中拿一个事件来执行,然后压入执行上下文当中,产生调用栈,并且分配内存.  

下面是相关的伪代码
```js
// eventLoop是一个用作队列的数组 
//(先进，先出)
var eventLoop = [ ];
var event;
//“永远”执行
while (true) {
    // 一次tick
    if (eventLoop.length > 0) {
        // 拿到队列中的下一个事件 
        event = eventLoop.shift();
        // 现在，执行下一个事件 
        try {
            event(); 
        } catch (err) {
            reportError(err);
        }
    }
}
```
每一个事件被执行就是一个tick,如果有事件队列等待被执行就会每次拿下来一个事件来执行.比如我们常用的`setTimeout()`方法,其实就是在对应的时间到了以后将宿主线程传进来的事件放入事件队列当中**要知道的非常重要的一点是,因为事件队列当中可能不只有一个事件,所以大多数情况下setTimeout的函数都会延迟执行**  

**先来先服务?**
我在这里就有一个疑问`event = eventLoop.shift();`这样的一个方法,也就是说采用的先来先服务的调度算法了.这样就会有一个很大的弊端比如用户触发的响应事件应该会优先级别更高才对,但是这样就完全体现不出来了.例如有一个响应请求返回先进入了事件队列,但这时候用户点击页面上的按钮.则浏览器还是会优先的响应返回的请求.

其实浏览器早已经做了这样的优化,**event loop的机制虽然是每次从事件队列里面拿一个事件出来执行,但是对应的执行顺序是早就确定好的**因为交给web api环境执行的耗时操作当被完成之后首先会被放在**job queue**,每一个**event loop**会有对应多个**job queue**.对于相同资源的job也会被放在相同的**job queue**里面,比如所有的ui操作会被放在一个**job queue**而所有的网络请求会被放在另一个**job queue**里面.这样浏览器就会制定**job queue**的优先级别,**在这里强调一下我们这里的 job queue 其实指的是macroTask**  

### microTask和macroTask
在es6当中我们通常把任务分成两类**microTask**和**macroTask**翻译过来就是微观任务和宏观任务.其实我们上面所讲的大多数都是**macroTask**,因为**macroTask**的要求并不需要很精确的执行所以我们允许他的颗粒度较大,当然**event loop**会对应多个**macroTask**.  
而对于一些需求颗粒度较小的任务我们称为**microTask**,当每次从web api返回的的**macroTask**中取一个task执行,而每次tick执行完就会把**microTask**的队列反复执行直到清空.例如一个经典面试题
```js
setTimeout(function () {
    console.log(1);
}, 0);
new Promise(function (resolve) {
        console.log(2);
        resolve();
    }).then(function () {
        console.log(3);
    });
```
比如上述的输出结果是`2,3,1`这是因为promise.then在es6当中会被压入**microTask**队列当中,这也就代表着在当前的tick完成之后就会调用**microTask**而setTimeout则会被存入**macroTask**等待下一次的调用(不一定是下一次,也可能更多的tick之后)  
下面是常用的**macroTask**
1. setImmediate
2. setTimeout
3. setInterval
4. 其他I/O

下面是常用的**microTask**
1. process.nextTick(nodeJS)
2. Promise
3. Object.observe
4. MutaionObserver

### 协作处理和原子性
通过事件队列的机制不难发现,每一个在eventloop里面的事件都会有是一个独立执行的比如
```js
eventLoop = [funcA,funcB];
/**
* 对应的执行顺序只可能存在两种情况
* 要没funcA先要么funcB先而不会存在funcA执行到一半去执行funcB
*/
```
所以JS这种模式也会有不确定性,但是这种不确定的复杂度决定要比多线程的复杂度要低很多.  

虽然整理清除了js的异步的工作方式,但是依然会有问题.比如下面这个情况
```js
var res = [];
// response(..)从Ajax调用中取得结果数组 
function response(data) {
    // 一次处理1000个
    var chunk = data.splice( 0, 1000 );
    // 添加到已有的res组 
    res = res.concat(
    // 创建一个新的数组把chunk中所有值加倍 chunk.map( function(val){
    return val * 2; })
);
// 还有剩下的需要处理吗? 
if (data.length > 0) {
// 异步调度下一次批处理 
    setTimeout( function(){
        response( data );
    }, 0 );
} }
// ajax(..)是某个库中提供的某个Ajax函数 
ajax( "http://some.url.1", response ); 
ajax( "http://some.url.2", response );
```
比如上面的情况,如果在response这个异步函数当中需要操作大量的数据,比如说1000w条数据.因为这些闭包的执行的院子性,这些大量操作的数据只有必须要执行完才可以继续事件循环队列的流程.中间不能响应任何操作.对于这中问题的处理我们做的优化就是需要将这个闭包当中的耗时操作分解成小的操作,从而能够让事件循环队列机制控制他

```js
var res = [];
// response(..)从Ajax调用中取得结果数组 
function response(data) {
    // 一次处理1000个
    var chunk = data.splice( 0, 1000 );
    // 添加到已有的res组 
    res = res.concat(
        // 创建一个新的数组把chunk中所有值加倍 
        chunk.map( function(val){
            return val * 2; 
        })
    );
    // 还有剩下的需要处理吗? 
    if (data.length > 0) {
    // 异步调度下一次批处理 
    setTimeout( function(){
        response( data );
    }, 0 );
} }
// ajax(..)是某个库中提供的某个Ajax函数 
ajax( "http://some.url.1", response ); 
ajax( "http://some.url.2", response );
```
虽然`setTimeout()`这个方法会把这样的一个回调事件放在队伍最后面,但是如果当多个回调函数进行插入的时候,是不能保证首先调用的`setTimeout()`先被执行的.
```js
//这样的代码依然会有两种情况
setTimeout(()=>{
    console.log(1);
},0);
setTimeout(()=>{
    console.log(2);
},0))
```
不过我在使用chrome尝试的时候确出现了每次都是`1,2`的结果.可能是v8对于这样的处理又进行了优化,回头我会仔细看看

## 回调