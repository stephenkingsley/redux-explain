# Redux 原理解析

Redux 是现在我看到的最合理的一个单项数据流，也是最满足`flux`的一个库。充分的使用到了`react`的机制。并且做出了`react-redux`这个连接库，`redux-thunk`异步请求专用的中间件。
<br />
本文档适合对react有一定了解的人看，因为有些react的东西没有做任何解释

## Redux使用教程
首先如果你是用redux，我建议你使用`Stateless functional components`，函数式组件，不提供任何的生命周期，两个参数，`props`，`context`，足够管理状态。（如果组件真的需要有自己的state，还是要用class去写一个组件，函数式组件只是一个工具）

### 第一步
安装redux需要的依赖

	npm i --save redux react-redux redux-thunk react react-dom

### 第二步

#### 创建store
`store`是redux里面的一个核心，几乎大部分的逻辑都在store里面。下面的是一个store的例子

```javascript
// configureStore.js
import { createStore, applyMiddleware } from 'redux'
import thunkMiddleware from 'redux-thunk'
import rootReducer from '../reducers'

export default function configureStore(initialState) {
  const store = createStore(
    rootReducer, // 你的reducer
    initialState,
    applyMiddleware(thunkMiddleware) // 使用到了异步请求的一个必须的中间件，可以不用管它
  )

  return store
}**strong text**
```

##### 现在来扯一下原理
首先，`createStore()`里面有几个变量需要知道的。redux源码里面将你自己定义的`reducer`赋给了currentReducer，其实用过redux的都知道，reducers跟actions其实最后都是分别被连接起来了，分别变成一个大的对象去管理。（这个扯远了）
<br />
其中里面有一个变量`currentListeners`,这个可以简单理解为store变化之后的回调方法。store变化唯一的方法就是调用`dispatch`，而dispatch里面有一段代码

```javascript
var listeners = currentListeners = nextListeners
for (var i = 0; i < listeners.length; i++) {
	listeners[i]()
}
```

其实很容易懂，`listeners`其实是你自己定义的方法，只要你定义了一些回调方法，他就会依次执行。不过在这个执行之前， 它先对reducer做了操作，跟新了reducer之后才会执行回调。
<br />
在reudx里面提供了监听变化的方法`subscribe`，这个方法是每次state改变之后都会回调。做的事情非常简单，就是将你定义的回调赋值到一个数组变量`listeners`。这也解释了为什么要用`dispatch`来执行action。

```javascript
import { addTodo, toggleTodo, setVisibilityFilter, VisibilityFilters } from './actions';

// Log the initial state
console.log(store.getState())

// Every time the state changes, log it
// Note that subscribe() returns a function for unregistering the listener
let unsubscribe = store.subscribe(() =>
	console.log(store.getState())
)

// Dispatch some actions
store.dispatch(addTodo('Learn about actions'))
store.dispatch(addTodo('Learn about reducers'))
store.dispatch(addTodo('Learn about store'))
store.dispatch(toggleTodo(0))
store.dispatch(toggleTodo(1))
store.dispatch(setVisibilityFilter(VisibilityFilters.SHOW_COMPLETED))

// Stop listening to state updates
unsubscribe()
```

所以react-redux里面`connect`通过这个方法进行了`state`的改变，所以组件会重新渲染。

#### 创建reducers

创建reducers其实非常简单，就用一个API就可以了。

```javascript
import { combineReducers } from 'redux';
const rootReducer = combineReducers({
  counter,
})
export default rootReducer
```

`combineReducers`其实就是将你定义的reducer连接起来，没什么好深入的~

#### 创建actions
actions其实更加简单，就是一个一个的方法而已。不过这里有一个建议，按照下面这样写会比较好。

```javascript
const add = (count) => {
	return { type: 'ADD', count, }
}
```

#### react组件
其实在react组件上面，redux只提供了两个API。一个是`Provider`，一个是`connect`。
<br />
`Provider`将store用`context`声明了一下。`context`这个API的作用很简单，也很好用。举个例子，如果你有10个组件，A到J，A套装B，B套着C，C套着D...A获取了一个数据obj，现在要传给J组件。如果没有`context`组件，你需要从第一层一直传到最后一层。现在用这个API，只要声明一下，它的所有子孙组件都可以使用，不需要传递。所以一般会在root component用`Provider`包裹，并且传`store`给它。

```javascript
const store = configureStore();
render(
  <Provider store={store}>
    <App />  
  </Provider>,
  document.getElementById('root'))
```
<br />

`connect`里面注册了一个`subscribe`，只要是dispatch之后，store改变了，connect会改变state。
<br />
这里面提供两个参数，其实我只觉得第一个参数比较必要——`mapStateToProps`。
这个是一个方法，里面有一个参数`state`，其实就是store里面的数据。这个参数可以定义你需要监听什么数据。举个例子，store里面有A，B，C三个数据。如果你监听了A。当B，C变化的时候，你的组件没有任何变化，只有A有变化的时候，你的组件才会再次渲染。

```javascript
function mapStateToProps(state) {
  return state
}

module.exports = connect(mapStateToProps)(App);
```

## 类似的库
redux不是唯一的单向数据流的库。在我看来，理念比较好的redux暂时还是比较好的，最近有一个叫做mobX的库。它来的比redux更加直接，有兴趣可以去看一下。
