create store的方法解释

## js中的注释翻译

### 什么是store？
- 原文：***Creates a Redux store that holds the state tree.***
- 译文：创建一个ReduxStore用来维护state的tree
- 理解：store就是用来存储state的地方

### 改变store的方式
- 原文：***The only way to change the data in the store is to call `dispatch()` on it.***
- 译文：只有通过调用store上的`dispatch`方法，才能改变store中的数据；
- 和react一样，只有set state才可以改变状态，redux中只有store的dispatch方法才可以把store中的数据改变

### 一个程序一个store
- 原文：***There should only be a single store in your app***
- 译文：在你的程序中，应当仅存在在一个store
- 理解：没啥好说的

### 合并处理
- 原文：***To specify how different
 * parts of the state tree respond to actions, you may combine several reducers
 * into a single reducer function by using `combineReducers`***
- 译文： 如果需要指定state树每一个部分的如何响应`action`的话，你可以通过combineReducers的方法将几个`reducer`组合成一个reducer。
- 理解：这里面出现出store以外的另外两个概念`action`和`reducer`,以及具有合并reducer功能的combineReducer，这些我们后面在看，接着往下看，总体理解，redux是只支持一颗状态树，且每一个节点的任何操作可以由一个reducer来管理；

接下来我们来看一下createStore方法,createStore共有三个参数
**入参**
- [Function] reducer
> 把当前的state tree和action给他处理，然后返回一个新的state tree
- [Any] preloadedState
> 可以作为初始的state，它可以来自于server也可以是一个之前session序列化的数据
- [Function] enhancer
> 就像他的参数名称一样，增强器，就是通过将store传递给第三方的中间件来处理，以此来增强store的能力，比如timetravel，持久化之类的，Redux有一个自带的增强器，叫`applyMiddleware`
**出参**
Redux Store，可以获得state，触发action，和订阅改变

### 重载function(reducer, preloadedState, enhancer) / function(reducer, enhancer)
createStore中有一个逻辑如果preloadedState是function的话，那么这个入参将会被认为是enhancer，也就是本来的第三个参数

### 增强器
如果enhancer存在的话，就会把createStore这个方法就给enhancer处理，并且把reducer，和preloadedState交给enhancer的返回值处理，也就是说，enhancer必定在此返回一个函数来处理“reducer”和“preloadedState”

### ensureCanMutateNextListeners
- 原文： “***This makes a shallow copy of currentListeners so we can use nextListeners as a temporary list while dispatching.
This prevents any bugs around consumers calling subscribe/unsubscribe in the middle of a dispatch.***”
- 理解：通过将当前的监听器进行浅拷贝，在dispatch的过程中，就使用这份浅拷贝，可以防止在dispatch过程因为订阅或者取消订阅导致的监听器无法找到的bug

### getState
> 获得当前应用的state tree，在dispatch的过程中如果调用getState会报错


### subscribe
> 用来添加一个监听改变的监听器，被订阅的监听器每次在一个action被dispatch的时候都会被调用，这个期间有可能statetree的一些状态已经被改变了，要想获得最新的statetree，回调中使用getstate
- 就像之前所说的，订阅的Listener都是一些快照（复制品），参照ensureCanMutateNextListeners，在displatch的发生过程中调用subscribe和unsubscrib是没用的，dispatch完之后，下一次在dispatch又会获得最新一次的快照。
- 需要确保在dispatch发生之前，所有的subscription都已经注册完毕了。

- **入参**
- [function] listener
> 每次dispatch都会调用的一个回调

一个store的isDispatching状态是共享的，
