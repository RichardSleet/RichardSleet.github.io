---
layout: post
title: 对ReactNative中性能优化的拙见
category: ReactNative-SHARE
---
首先在这里非常强烈的推荐十分推荐天地之灵的优化reactnative的视频[快点我](http://i.youku.com/i/UMzM5ODI5MDA4MA==/videos?order=2),然后我会总结一下天地之灵的一些优化关键点,最后我会写写自己优化相机app的一些方案.
### React的优化
如果谈到ReactNative的性能优化,是一定要知道React的性能优化的.其实天地之灵讲的优化基本上都是基于React的优化所以先简单说说React优化的关键  
#### 避免不必要的render
React的出现很大程度上优化了前端的性能,这要归功于virtual dom这样一个解决方案.React框架会维持了被创建出来的virtual dom在写法上就是React.createElement().第一次创建出来的virtual dom会自动挂载到你所指定的dom节点上.就像下面一样
```js
import react,{Component} from 'react';
import ReactDOM from 'react-dom'{
    render(){
        return <p>Hello World</p>
    }
}
class HelloWorld extends Component
ReactDOM.render(,document.querySelector('body'))
```
但是真正的精髓并不是这里,而是当你所创建的virtual dom发生改变的时候,React会十分智能的检测你所创建的virtual dom节点树.首先通过diff检查你的root节点是否发生变化,依次向下进行排查,最终锁定住变化的节点然后对变化的节点进行操作更改,让他更新到真实的dom节点上面.过程就像官网上的图片一样.[这是官网的一张图片](https://facebook.github.io/react/img/docs/should-component-update.png),以上就是React的精髓所在.  
当我看到这里的时候,我就觉得React已经非常完美了,又有什么需要优化的呢?  
其实不然就在我们上述过程中的React查找的节点采用了diff算法,但是其实主要的性能耗费就会发生在执行diff比较之前,react会生成一个virtual dom树.也就是说内存这时会有两个virtual dom 节点树.当React中通过props或者state触发render更新界面的时候React会根据你的render触发所在的虚拟节点上,将这个节点及其子节点的所有虚拟节点全部生成一遍.然后再拿新生成的这个virtual dom树和旧的virtual dom树进行我们上面说的比对过程.所以其实有很大的一部分性能消耗在创建一个不必要的virtual dom树上面.
```js
import react,{Component} from 'react';
import ReactDOM from 'react-dom'
class HelloWorld extends Component{
    onClickHandler(){
        this.setState(
            content:'Hello World'
        )
    }
    render(){
        return <div id="contianer" onClick={this.onClickHandler}> 
            <ComponentOne>
                <p>{this.state.content}</p>
            </ComponentOne>
            <ComponentTwo></ComponentTwo>
        </div>
    }
}
ReactDOM.render(,document.querySelector('body'))
```
在上面的代码当中如果ComponentOne组件发生了改变,他的结果会导致其实并不简单的会导致ComponentOne组件进行重绘.而是HelloWorld整个都会执行render方法,所以这时ComponentTwo组件即使没有发生改变也会出现重新生成一个virtual dom节点的情况.不过庆幸的是React还是提供了相关的解决方法.那就需要引入React形成的组件的生命周期了  
在React组件的生命周期当中,十分重要的一个过程就是shouldComponentUpdate()这个函数,无论是props还是state的改变最终都会导致这个函数的执行,而这个函数的返回值恰恰决定了这个组件是不是应该被重新进行比对.所以我们可以主动的实现这么一个办法来判断我的属性是不是应该被改变,而避免不必要的virtual dom的生成
```js
import react,{Component} from 'react';
import ReactDOM from 'react-dom'
class HelloWorld extends Component{
    onClickHandler(){
        this.setState(
            content:'Hello World'
        )
    }
    render(){
        return <div id="contianer" onClick={()=>{this.onClickHandler()}}> 
            <ComponentOne>
                <p>{this.state.content}</p>
            </ComponentOne>
            <ComponentTwo></ComponentTwo>
        </div>
    }
}
class ComponentTwo extends Component{
    shouldComponentUpdate(nextProps,nextState){
        //对props的值进行简单的比较相同的话就返回false,不同的话就返回true
        return compareProps(this.props,nextProps)
    }
    render(){
        return <p>{'----sleet'}</p>
    }
}
ReactDOM.render(,document.querySelector('body'))
```
当我们给ComponentTwo的组件当中加入这个函数的时候,就会避免ComponentTwo会生成新的virtual dom进行比对.也就避免了不必要的render.FB也为我们提供了这样的一个组件叫做PureComponet所以上面的代码还可以优化成这个样子
```js
import react,{Component,PureComponet} from 'react';
import ReactDOM from 'react-dom'
class HelloWorld extends Component{
    onClickHandler(){
        this.setState(
            content:'Hello World'
        )
    }
    render(){
        return <div id="contianer" onClick={()=>{this.onClickHandler()}}> 
            <ComponentOne>
                <p>{this.state.content}</p>
            </ComponentOne>
            <ComponentTwo></ComponentTwo>
        </div>
    }
}
class ComponentTwo extends PureComponet{
    render(){
        return <p>{'----sleet'}</p>
    }
}
ReactDOM.render(,document.querySelector('body'))
```
#### 处理函数绑定的优化
如果不是天地之灵的视频,这个倒还真的没有注意过.那就是在绑定事件的处理函数的时候也会出现可以优化的地方.接着那上面的例子举例
```js
import react,{Component} from 'react';
import ReactDOM from 'react-dom'
class HelloWorld extends Component{
    render(){
        return <div id="contianer"> 
            <ComponentOne>
                <p>{this.state.content}</p>
            </ComponentOne>
            <ComponentTwo onClick={()=>{this.onClickHandler()}}></ComponentTwo>
        </div>
    }
}
ReactDOM.render(,document.querySelector('body'))
```
假设我们在ComponentTwo组件当中添加了这样的一个事件函数,而且ComponentTwo组件当中同样的继承了PureComponent,这样的话即便每次render执行没有任何变化ComponentTwo也会被重新创建Dom.惊不惊喜?意不意外?这是因为每一次进行shouldComponent的判断的时候都回把onClick中的箭头函数重新生成一个,而因为shouldComponent比较的两个前后状态的参数是一个浅比较的过程,由于是内存中完全不同的两个地方,在这种情况下他会认为两个函数并不星等,所以会生成一个全新的virtual dom节点.所以我们需要改变的是这种书写格式
```js
import react,{Component} from 'react';
import ReactDOM from 'react-dom'
class HelloWorld extends Component{
    constructor(props){
        super(props);
    }
    onClickHandler = ()=>{
        //....
    }
    render(){
        return <div id="contianer"> 
            <ComponentOne>
                <p>{this.state.content}</p>
            </ComponentOne>
            <ComponentTwo onClick={this.onClickHandler}></ComponentTwo>
        </div>
    }
}
ReactDOM.render(,document.querySelector('body'))
```
这样一来因为总是指向的是变量onClickHandler的内存地址就可以避免不必要的刷新.

### 对于ReactNative的优化
对于RN的优化我觉得是分为几个步骤的
```js
1.首先通过Systrace或者其他性能分析工具,找出来卡顿所在的线程(一般来说都是js线程)
2.整理一下当前页面的代码逻辑,尤其是哪些大量的循环,timer等等非常频繁的操作,以及分析一下dom树的性能.
3,定位到问题代码,更改逻辑或者代码的位置,因为这些复杂的操作通常会导致js线程和ui线程交互的时候,因为处理js线程当中操作泰国复杂,就会导致js线程没办法把生成的改变及时交给ui去处理.
```
上面的步骤是自己总结的不一定正确,但还是希望能够分享一下解决问题的思路.  
在RN方面优化的途径是十分多的,不过终究来说就是为了减少render.

### ReactNative中下载进度更新的简单优化
当我看完天地之灵的视频的时候发现哇塞看来我的代码要大改一遍了,然而这个过程还并不是很容易的.对于上面的React优化是一个基础知识,但是真正需要优化一个ReactNative的性能的时候是和代码结构甚至是你的代码逻辑都是有巨大的关系的.就比如说在做下载队列的图片的时候.页面基本非常的卡.我简单写一下我的代码
```js
class DownloadListItem extends Component{
    componentDidMount(){
        //首次挂载上去以后进行下载,并且会随时更新下载进度
        downloadFunc((progress)=>{
            this.setState({
                progress:progress
            })
        })
    }
    donwloadFunc(progresssCallBack){

    } 
    render(){
        <View>
            <Progress progress={this.state.progress}>
        </View>
    }
}
```
以上就是我当时非常正常的下载思路,可是事实是运行出来的app十分的卡,简直是卡到爆!不过我也迅速的意识道了问题出现在一下几个方面
```js
1.progress是我封装的android的AsyncTask和iOS的NSURlSession的回调函数,这种函数会被反复的执行,执行量是非常的大的.(所以反复的导致render被刷新)
2.我自己封装原生的是多线程下载的,也就是说在下载队列当中的控件不止一个
3.当js线程大量的计算的时候就会导致没有时间和ui线程做交互,也就导致了页面发生的卡顿啊问题
```
**解决方法:**下面说说我的解决方法,第一步不要依赖于progress的频繁回调来刷新自己的视图,我的解决方式时起了一个Timer方法没过0.3s就会去拿一次数据,把这个简单的代码更改一下就是下面的这个样子
```js
class DownloadListItem extends Component{
    componentDidMount(){
        //首次挂载上去以后进行下载,并且会随时更新下载进度
        downloadFunc((progress)=>{
            this.progress = progress;
        })
        this.timer = setTimeInterval(()=>{
            this.setState({
                progress:this.progress
            })
        },300)
    }
    donwloadFunc(progresssCallBack){
        //下载进程
    } 
    render(){
        <View>
            <Progress progress={this.state.progress}>
        </View>
    }
}
```
改成这个样子以后就会发现减少了非常大的一部分的render,但是这又有了另一个问题.由于我分装的progress是一个根据props改变视图的简单view,也就是说如果传入的props是0.10,0.11,0.12,0.13,这样的进度的话用户会看到一个平滑的进度条更新.但是由于我们并没有频繁的进行render更新所以每次穿过来的进度可能会很大比如0.2,0.4,0.6.这会让用户看起来一停一停的.这个问题的解决方案是在于将progress这个控件改为用动画的方式进行刷新.这里我是采用了[react-native-progress](https://github.com/oblador/react-native-progress).这样以来界面一下子就流畅了许多.
```js
//我写的Progress感觉就是这样的
class Progress extends Component{
    render(){
        <View style={{width:'100px'}}>
            <View style={{width: this.props.progress*100+'px'}}>
            </View>
        </View>    
    }
}
```
但是!还没有完为了确保更好的用户体验,我们希望如果用户点击了屏幕的按钮,但是突然js线程需要进行更新progress进度这样一个耗时的操作(虽然可能性很小),所以我们希望我们的js线程能够优先处理用户的操作这里我们就用到了InteractionManager[InteractionManager](https://reactnative.cn/docs/0.31/timers.html)再次优化一下上面的代码
```js
class DownloadListItem extends Component{
    componentDidMount(){
        //首次挂载上去以后进行下载,并且会随时更新下载进度
        downloadFunc((progress)=>{
            this.progress = progress;
        })
        this.timer = setTimeInterval(()=>{
            InteractionManager.runAfterInteractions(()=> {
                this.setState({
                    progress:this.progress
                })
            });
        },300)
    }
    donwloadFunc(progresssCallBack){
        //下载进程
    } 
    render(){
        <View>
            <Progress progress={this.state.progress}>
        </View>
    }
}
```
当当当,这样一来页面就会更加的流畅.其实做到现在依然可以进行再次的优化不如你使用进度条的动画可以才用requestAnimationFrame来进行控制,结合之前提到的react知识你还需要对这个控件的其他组件进行判定避免你的progress发生更新的时候,带动了你所有的组件创建多余的virtualdom节点.

### ReactNative中借助ScrollView对Flatlist和Sectionlist进行深度优化
接下来首先说说我的应用场景,我是用FlatList去展现我的照片,但是随着加载的照片越来越多.也就是一直滑动到最底部将近加载过600多张的图片的时候,会有一个选择的功能,当点击选择的时候会有将近3,4s的延迟.就像下面一样  
在阐述我的引起我问题的原因之前我先说说FLatList这个控件,列表一直都是移动端使用的最最频繁的一种控件,比如你看到的微博,qq,微信,几乎都是用列表控件,所以这个控件的性能直接影响了app的性能,对于iOS和android为了提高各自列表版本的效率,他们使用的应该是同一种优化方法,就是**复用cell**可能在移动端是一个非常常用的概念,但是起初的我并不知道.所以还是来解释一下复用cell其实就是当这个控件显示到屏幕上并且从上方离开屏幕以后,就会从下方又出现一个'新'的cell.然而其实新出现的cell会用离开屏幕的cell对象.比如说一共有500条数据其实并不会对应有500个cell而是只有屏幕上显示的10个cell在反复的利用.  
所以现在flatlist性能之所以会比之前的listview就是因为做出了很大的优化.接着回到我们之前的问题就是在对图片进行选中为什么会发生卡顿.如果说Flatlist已经采用了**复用cell**,那么应该完全不会有卡顿才对.因为只对屏幕上的cell进行操作,这样的性能要求并不是很大.而且卡顿会随着加载的图片数量边的更慢.所以我突然意识到,其实可以把在android和iOS上的控件,比作前端当中真实的dom节点而React所创建的dom节点都是virtual dom节点.也就是说虽然ListView划出屏幕的部分会被卸载掉但是,因为真实的listview控件并没有卸载,所以virtual dom依然存在,而virtual dom却不会被卸载而是被保存住.当我进行点击选中的时候所有的图片(将近600张的照片)的virtual dom都回被重新生成新的virtual dom然后再进行两个dom树之间的diff算法对比.最后更新视图.也就是说我们会对大量的Virtual dom进行操作即便使用了我们之前的PureComponent进行封装,依然不会有改变.因为这些virtual dom 的改变都是一个正常过程.
所以如果我们能够吧virtual dom 也用上**复用cell**的理念只对显示在屏幕上的dom进行virtual dom的改变就会快很多.(当然我在项目当中使用的是sectionlist但是这里简单的用FlatList写一下).  
下面是我的第一个版本
```js
class Photo extends Component{
    render(){
        return(
            <Image source={require(this.props.uri)}>
                {this.props.showSelect?<Image source={require('selectIcon.png')}></Image>:null}
            </Image>
        );
    }
}
class PhotoGallery extends Component{
    constuctor(){
        this.datasource = [
            {key:1,src:'1.png'},
            ...
        ]
    }
    render(){
        return <FlatList
                data={this.datasource}
                renderItem={({item}) => <Photo uri={item.src} />}
                />
    }
}
```
接下来需要实现的就是通过SCrollview的方法在这里面当中加入一个判断函数判断是不是在屏幕显示的virtual dom,如果是就进行更新,不是就不进行更新.并且在组件的外部保留当前是不是开始选择的状态,简单的更改一下就是
```js
class Photo extends Component{
    shouldComponentUpdate(){
        //action所传入的在屏幕上显示的图片的数组
        this.props.viewableArr
        //判断如果这个元素显示在屏幕上就返回true否则返回false
        //.....
    }
    render(){
        return(
            <Image source={require(this.props.uri)}>
                {this.props.showSelect?<Image source={require('selectIcon.png')}></Image>:null}
            </Image>
        );
    }
}
class PhotoGallery extends Component{
    constuctor(){
        this.datasource = [
            {key:1,src:'1.png'},
            ...
        ]
        this.handleViewableItems = this.handleViewableItems.bind(this);
    }
    handleViewableItems(items){
        //会分别传入被加入进来的item和移除的item,在这里设定一个flag
        //需要更新的数组
        let needChangeViewavlePropsArr = []
        info.changed.forEach((vaule)=>{
            needChangeViewavlePropsArr.push({item:vaule.item,isViewable:vaule.isViewable});
        })
        //这里我使用了redux去更新这些子组件的状态让他们和是否显示的状态对应起来
        this.props.dispatch(CHANGE_VIEWABLE({viewableArr:needChangeViewavlePropsArr}));
    }
    render(){
        return <FlatList
                onViewableItemsChanged={this.handleViewableItems}
                data={this.datasource}
                renderItem={({item}) => <Photo uri={item.src} />}
                />
    }
}
```
因为项目中用到的东西太多了只能简单的写写表达意思,简单的来说就是保存一个在屏幕上显示的item的数组,ScrollView的onViewableItemsChanged方法会对这个数组做出不断的更新,并且把这个数组通过redux传递给所有的item,让每个item根据自己是不是在数组当中进行判断自己需不需要更新.
这一段说的十分潦草主要是这里面涉及的东西还是很多的所以写的仔细是需要点时间的,回来有空会把这里再补充一下.但是总归来说React的框架的优化莫过于**减少render**