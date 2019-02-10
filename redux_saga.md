# redux-saga

> redux-saga是一个管理应用程序SideEffects（副作用，例如异步获取数据，访问浏览器缓存等）的Library，在redux中，你可以看到所有的action都是同步的，而异步的action就称为SideEffect，副作用

saga提供了很多effect方法来进行管理这些副作用大家可以在redux-saga/effects中获得，常用的有take， takeEvery，put，call，select，fork 另外还有race、throttle，等高级api，这个需要我们来一点一点来挖掘了。

> ⏰ 本教程中没涉及任何关于react部分的知识

> dva是redux + redux-saga，是redux的一个最佳实践，所以dva中我们可以看到已经将redux + redux-saga + react-redux 整个给整合了非常好了，我简单来说一个起步的套路，其实也非常简单，想看官方文档的同学可以参照官方[新手教程](https://redux-saga-in-chinese.js.org/docs/introduction/BeginnerTutorial.html)

## 准备工作
```
npm install redux-saga redux --save
```

## tutorial
```
# sagas.js

import { put, call, take, takeEvery } from 'redux-saga/effects';

function setNameAsync(name) {
  console.log(`name is ${name}`);
  return new Promise((resolve)=> {
    setTimeout(()=>{
      resolve()
    }, 1000);
  });
}

function setAgeAsync(age) {
  console.log(`age is ${age}`);
  return new Promise((resolve)=> {
    setTimeout(()=>{
      resolve()
    }, 1000);
  });
}

function * changeNameAsync(action) {
  const { name } = action;
  yield call(setNameAsync, name);
  yield put({
      type: 'setName',
      name
  });
}

function * changeAgeAsync(action) {
  const { age } = action;
  yield call(setAgeAsync, age);
  yield put({
      type: 'setAge',
      age
  });
}

export default function * sagas() {
  yield takeEvery('changeName', changeNameAsync)
  yield takeEvery('changeAge', changeAgeAsync);
}

# reducers.js
import { combineReducers } from 'redux';

const reducers = {
  name: (state = '', action) => {
    switch(action.type) {
      case 'setName': {
        return  action.name
      }
      default:
        return state;
    }
  },
  age: (state = 0, action) => {
    switch (action.type) {
      case 'setAge': {
        return action.age
      }
      default:
        return state;
    }
  }
};

export default combineReducers(reducers);


# app.js
import { createStore, applyMiddleware} from 'redux';
import createSagaMiddleware from 'redux-saga';
import reducer from './reducers';
import sagas from './sagas';

const preloadedState =  {
  name: 'Seed Huang',
  age: 34
};

const sagaMiddleware = createSagaMiddleware();

const store = createStore(reducer, preloadedState, applyMiddleware(sagaMiddleware));

sagaMiddleware.run(sagas);

store.subscribe(()=> {
  console.log(store.getState());
});

store.dispatch({ type:'changeName', name: 'SkyHuang'});
store.dispatch({ type:'changeAge', age: 5});


```

## API

首先我们看以下对应的api

### take
监听dispatch的action，只会触发一次，阻塞式

take可以用来监听某一个action，但是take只能使用一次，如果想重复使用可以使用`while(true)`来循环执行，因为是generator类型的函数所以并不用怕会造成死循环，

> ⚠️ 在dispatch一个action之前，一定要确保自己的take方法已经被运行过，否则take之后的代码将无法运行

```
# sagas.js

import { take } from 'redux-saga/effects';

export default function runSaga () {
  yield take('omAction');
  console.log('omAction is dispatched');
}

# app.js

import { createStore, applyMiddleware} from 'redux';
import createSagaMiddleware from 'redux-saga';
import sagas from './sagas';

const sagaMiddleware = createSagaMiddleware();

const store = createStore(reducer, applyMiddleware(sagaMiddleware));

sagaMiddleware.run(sagas);

store.dispatch({ type:'omAction'});

// omAction is dispatched


```

>

### takeEvery
监听dispatch的action，每次触发对应action对会被触发，阻塞式, 完整例子查看[tutorial](#tutorial),这里有一点需要记住，takeEvery与take不一样的一点是，takeEvery主要是通过第二个参数的generator函数来处理副作用的，如果没有捕获对应的action则generator将不会被触发，并且takeEvery会马上执行下一个步骤，可以写一段代码试一下：

```
function * runSaga () {
  yield takeEvery('someAction', someGenerator);
  console.log('I am here');
}
```

### put
类似redux的dispatch，put可以说是嘴贱单的一个，没有什么好说的

### call
call可以接受Promise和Generator，通常用来调用远程服务，阻塞式

### select
select可以对state进行查找，阻塞式

### fork
fork用以执行一项任务，非阻塞式， fork需要配合cancel使用，被cancel的任务将立即马上停止执行
