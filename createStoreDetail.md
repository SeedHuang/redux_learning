## createStore 详解

```
let currentReducer = reducer
let currentState = preloadedState
let currentListeners = []
let nextListeners = currentListeners
let isDispatching = false
```

## currentReducer

currentReducer也就是当前reducer，createStore会在内部把reducer，在createStore之后只有通过store.replaceReducer方法才能够替换reducer，通常用于combinedReducers的产物，因为有许多应用中会用到懒加载和懒执行之类的方式，所以有可能reducer一开始是不完整的，通过replaceReducer可以替换当前reducer

> ⏰ 通过使用了repalceReducer之后，redux会立即触发一个action，anctionType为PROBE_UNKNOWN_ACTION，触发该action的目的为了立即生成新reducer对应部分的state或者去除已删除的reducer那部分对应的state


## isDispatching

是一个非常重要的标志位，他与一些重要的外部调用函数有直接关系，都是在dispatching（调遣的过程中）如果调用这些函数会触发“throw exception”

来看看有哪些函数吧

- subscribe & unsubscribe
- getState
- dispatch

从这里看来，redux还是禁止异步触发dispatch的，与dispatch有关的操作都需要同步进行

## currentListeners & nextListeners
currentListeners 和 nextlisteners在大多数的情况下是一样的
在`dispatch`的时候，会将最新的`nextListeners`复制给`currenListeners`，如果之后再也没有subscribe和`unsubscribe`发生，则`currentListeners` 就等于 `nextListeners`,如果发生`subscribe`和`unsubscribe`，则两个函数内部都会通过`ensureCanMutateNextListeners`来浅复制`currentListeners`，并赋值给`nextListeners`，然后将新的listener放入`nextListeners`的数组中，在下次`dispatch`的时候再将`nextListeners`赋值给`currentListeners`;

### ensureCanMutateNextListeners
```
function ensureCanMutateNextListeners() {
  if (nextListeners === currentListeners) {
    nextListeners = currentListeners.slice()
  }
}
```
该方法只有在nextListeners == currentListeners时才会产生效果，为什么呢？注解中给出了解释
```
This prevents any bugs around consumers calling subscribe/unsubscribe in the middle of a dispatch.
```
是为了防止在dispatching中由于subscribe和unsubscribe发生listeners增多或者减少，从而使查找不到对应的运行不了对应的listener，那么什么时候才会产生相等和不想等的情况呢？首先说一下相等的情况，之前也提到过，在dispatching中会将nextListeners赋值给currentListeners，之后如果没有subscribe或者unsubscribe运行的话currentListeners就会一直等于nextListeners，反之如果一致subscribe或者unsubscribe的话则nextListeners和currrentListeners将永远不会相等。为了避免subscsribe的过程中给dispatching带来问题，所以需要将两个listener交由不同的两个对象进行保存，并且在下次dispatch的时候在进行同步。
> 🤔️❓ 其实仔细想一下，目前subscribe和unsubscribe的方法中都有标志位isdispatching，一旦在dispatching的状态下使用了这两个方法就会报错，所以这里我只能理解`ensureCanMutateNextListeners`是一个兜底的解决方案，或者是isDispatching这个标志位出现前的一个解决方案
