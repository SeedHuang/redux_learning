在createStore的返回的对象上我们看到了一个Symbol.observable的属性，今天简单的来解析一下这个新的Symbol，是一个用来处理事件流事件的方法，

该方法返回一个对象，上面包含了subscribe方法和[Symbol.observable]方法，
subscrible方法：会返回一个unsubscrible方法，subscrible的入参包括以下三个方法
```
{
  next(data)=>{}, //在触发每一次的事件后触发
  error(error)=>{}, // 在事件中发生了错误后触发
  complete=>{} // 在事件完成后触发
}
```
在Symbol.observable的文档中中写到
> 原文：sure you never `next`, `error` or `complete` on your observer after `error` or `complete` was called.

> 译文：在error和complete之后不要触发 **next**, **error**, **complete** 方法

> 原文： make sure you don't `next`, `error` or `complete` after `unsubscribe` is called on the returned object.

> 译文：取消订阅之后也不要触发**next**, **error**, **complete** 方法

[Symbol.observable]:
当然这还是一个一个方法返回一个相同结构的函数；


```
someObject[Symbol_observable] = () => {
  return {
    subscribe(observer) {
      const handler = e => observer.next(e);
      someObject.addEventListener('data', handler);
      return {
        unsubscribe() {
          someObject.removeEventListener('data', handler);
        }
      }
    },
    [Symbol_observable]() { return this }
  }
}
```
