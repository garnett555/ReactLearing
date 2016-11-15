# Redux Tutorial
## @Ruan
- Redux 的设计思想很简单，就两句话。
（1）**Web 应用是一个状态机，视图与状态是一一对应的。**
（2）**所有的状态，保存在一个对象里面。**
- Store 就是保存数据的地方，你可以把它看成一个容器。整个应用只能有一个 Store。Redux 提供`createStore`这个函数，用来生成 Store。`createStore`函数接受另一个函数作为参数，返回新生成的 Store 对象。
``` Import {createStore} from 'redux';
		const store = createStore(fn); ```
- Store对象包含所有数据。如果想得到某个时点的数据，就要对 Store 生成快照。这种时点的数据集合，就叫做 State。当前时刻的 State，可以通过`store.getState()`拿到。`const state = store.getState();`Redux 规定， 一个 State 对应一个 View。只要 State 相同，View 就相同。你知道 State，就知道 View 是什么样，反之亦然。
- State 的变化，会导致 View 的变化。但是，用户接触不到 State，只能接触到 View。所以，State 的变化必须是 View 导致的。Action 就是 View 发出的通知，表示 State 应该要发生变化了。Action 是一个对象。其中的type属性是必须的，表示 Action 的名称。其他属性可以自由设置。
```const action = {
  type: 'ADD_TODO',
  payload: 'Learn Redux'
};```
可以这样理解，Action 描述当前发生的事情。改变 State 的唯一办法，就是使用 Action。它会运送数据到 Store。
- View 要发送多少种消息，就会有多少种 Action。如果都手写，会很麻烦。可以定义一个函数来生成 Action，这个函数就叫 Action Creator。
```
const ADD_TODO = '添加 TODO';

function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}

const action = addTodo('Learn Redux');
```
- store.dispatch()是 View 发出 Action 的唯一方法
```
import { createStore } from 'redux';
const store = createStore(fn);

store.dispatch({
  type: 'ADD_TODO',
  payload: 'Learn Redux'
});
```
结合 Action Creator，这段代码可以改写如下。
```store.dispatch(addTodo('Learn Redux'));```
- Store 收到 Action 以后，必须给出一个新的 State，这样 View 才会发生变化。这种 State 的计算过程就叫做 Reducer。Reducer 是一个函数，它接受 Action 和当前 State 作为参数，返回一个新的 State。
```
const reducer = function (state, action) {
  // ...
  return new_state;
};
```
*An example*
```
const defaultState = 0;
const reducer = (state = defaultState, action) => {
  switch (action.type) {
    case 'ADD':
      return state + action.payload;
    default: 
      return state;
  }
};

const state = reducer(1, {
  type: 'ADD',
  payload: 2
});
```
实际应用中，Reducer 函数不用像上面这样手动调用，`store.dispatch`方法会触发 Reducer 的自动执行。为此，Store 需要知道 Reducer 函数，做法就是在生成 Store 的时候，将 Reducer 传入`createStore`方法。
```
import { createStore } from 'redux';
const store = createStore(reducer);
```
上面代码中，createStore接受 Reducer 作为参数，生成一个新的 Store。以后每当store.dispatch发送过来一个新的 Action，就会自动调用 Reducer，得到新的 State。
- Reducer 函数最重要的特征是，它是一个纯函数。也就是说，只要是同样的输入，必定得到同样的输出。由于 Reducer 是纯函数，就可以保证同样的State，必定得到同样的 View。但也正因为这一点，Reducer 函数里面不能改变 State，必须返回一个全新的对象。纯函数是函数式编程的概念，必须遵守以下一些约束。
*
不得改写参数
不能调用系统 I/O 的API
不能调用Date.now()或者Math.random()等不纯的方法，因为每次会得到不一样的结果
*
- Store 允许使用`store.subscribe`方法设置监听函数，一旦 State 发生变化，就自动执行这个函数。
```
import { createStore } from 'redux';
const store = createStore(reducer);

store.subscribe(listener);
```  
显然，只要把 View 的更新函数（对于 React 项目，就是组件的`render`方法或`setState`方法）放入listen，就会实现 View 的自动渲染。`store.subscribe`方法返回一个函数，调用这个函数就可以解除监听。
- `createStore`方法还可以接受第二个参数，表示 State 的最初状态。这通常是服务器给出的，
```
let store = createStore(todoApp, window.STATE_FROM_SERVER)
```
上面代码中，`window.STATE_FROM_SERVER`就是整个应用的状态初始值。注意，**如果提供了这个参数，它会覆盖 Reducer 函数的默认初始值。**
- Redux 提供了一个`combineReducers`方法，用于 Reducer 的拆分。你只要定义各个子 Reducer 函数，然后用这个方法，将它们合成一个大的 Reducer。
```
import { combineReducers } from 'redux';

const chatReducer = combineReducers({
  chatLog,
  statusMessage,
  userName
})

export default todoApp;
```
*这种写法有一个前提，就是 State 的属性名必须与子 Reducer 同名。*

- Action 发出以后，Reducer 立即算出 State，这叫做同步；Action 发出以后，过一段时间再执行 Reducer，这就是异步。怎么才能 Reducer 在异步操作结束后自动执行呢？这就要用到新的工具：中间件（middleware）。常用的中间件都有现成的，只要引用别人写好的模块即可。
```
import { applyMiddleware, createStore } from 'redux';
import createLogger from 'redux-logger';
const logger = createLogger();

const store = createStore(
  reducer,
  applyMiddleware(logger)
);
```
上面代码中，`redux-logger`提供一个生成器`createLogger`，可以生成日志中间件`logger`。然后，将它放在`applyMiddleware`方法之中，传入`createStore`方法，就完成了`store.dispatch()`的功能增强。
这里有两点需要注意：
（1）`createStore`方法可以接受整个应用的初始状态作为参数，那样的话，`applyMiddleware`就是第三个参数了。
```
const store = createStore(
  reducer,
  initial_state,
  applyMiddleware(logger)
);
```
（2）中间件的次序有讲究。
```
const store = createStore(
  reducer,
  applyMiddleware(thunk, promise, logger)
);
```
上面代码中，`applyMiddleware`方法的三个参数，就是三个中间件。有的中间件有次序要求，使用前要查一下文档。*比如，logger就一定要放在最后，否则输出结果会不正确。* `applyMiddlewares`是 Redux 的原生方法，作用是将所有中间件组成一个数组，依次执行。
- 同步操作只要发出一种 Action 即可，异步操作的差别是它要发出三种 Action。(1)操作发起时的 Action (2)操作成功时的 Action (3)操作失败时的 Action
以向服务器取出数据为例，三种 Action 可以有两种不同的写法。
```
// 写法一：名称相同，参数不同
{ type: 'FETCH_POSTS' }
{ type: 'FETCH_POSTS', status: 'error', error: 'Oops' }
{ type: 'FETCH_POSTS', status: 'success', response: { ... } }

// 写法二：名称不同
{ type: 'FETCH_POSTS_REQUEST' }
{ type: 'FETCH_POSTS_FAILURE', error: 'Oops' }
{ type: 'FETCH_POSTS_SUCCESS', response: { ... } }
```
除了 Action 种类不同，异步操作的 State 也要进行改造，反映不同的操作状态。下面是 State 的一个例子。
```
let state = {
  // ... 
  isFetching: true,
  didInvalidate: true,
  lastUpdated: 'xxxxxxx'
};
```
- 异步操作的思路
操作开始时，送出一个 Action，触发 State 更新为"正在操作"状态，View 重新渲染
操作结束后，再送出一个 Action，触发 State 更新为"操作结束"状态，View 再一次重新渲染
- 异步操作的第一种解决方案就是，写出一个返回函数的 `Action Creator`，然后使用`redux-thunk`中间件改造`store.dispatch`。既然 `Action Creator` 可以返回函数，当然也可以返回其他值。另一种异步操作的解决方案，就是让 `Action Creator` 返回一个 Promise 对象。
- React-Redux 将所有组件分成两大类：UI 组件（presentational component）和容器组件（container component）
- UI 组件有以下几个特征。
只负责 UI 的呈现，不带有任何业务逻辑
没有状态（即不使用this.state这个变量）
所有数据都由参数（this.props）提供
不使用任何 Redux 的 API
因为不含有状态，UI 组件又称为"纯组件"，即它和纯函数一样，纯粹由参数决定它的值。
- 容器组件的特征恰恰相反。
负责管理数据和业务逻辑，不负责 UI 的呈现
带有内部状态
使用 Redux 的 API
*总之，只要记住一句话就可以了：UI 组件负责 UI 的呈现，容器组件负责管理数据和逻辑。*
你可能会问，如果一个组件既有UI又有业务逻辑，那怎么办？回答是，将它拆分成下面的结构：外面是一个容器组件，里面包了一个UI 组件。前者负责与外部的通信，将数据传给后者，由后者渲染出视图。
*React-Redux 规定，所有的 UI 组件都由用户提供，容器组件则是由 React-Redux 自动生成。也就是说，用户负责视觉层，状态管理则是全部交给它。*
- React-Redux 提供`connect`方法，用于从 UI 组件生成容器组件。`connect`的意思，就是将这两种组件连起来。
```
import { connect } from 'react-redux'
const VisibleTodoList = connect()(TodoList);
```
上面代码中，`TodoList`是 UI 组件，`VisibleTodoList`就是由 React-Redux 通过connect方法自动生成的容器组件。

为了定义业务逻辑，需要给出下面两方面的信息。
（1）输入逻辑：外部的数据（即`state`对象）如何转换为 UI 组件的参数
（2）输出逻辑：用户发出的动作如何变为 Action 对象，从 UI 组件传出去。
因此，connect方法的完整 API 如下。
```
import { connect } from 'react-redux'

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)
```
上面代码中，`connect`方法接受两个参数：`mapStateToProps`和`mapDispatchToProps`。它们定义了 UI 组件的业务逻辑。前者负责输入逻辑，即将state映射到 UI 组件的参数（props），后者负责输出逻辑，即将用户对 UI 组件的操作映射成 Action。
- `mapStateToProps`是一个函数。它的作用就是像它的名字那样，建立一个从（外部的）state对象到（UI 组件的）props对象的映射关系。作为函数，`mapStateToProps`执行后应该返回一个对象，里面的每一个键值对就是一个映射。
```
const mapStateToProps = (state) => {
  return {
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}
```
上面代码中，`mapStateToProps`是一个函数，它接受state作为参数，返回一个对象。这个对象有一个todos属性，代表 *UI 组件的同名参数*，后面的`getVisibleTodos`也是一个函数，可以从state算出 todos 的值。
`mapStateToProps`会订阅 Store，每当state更新的时候，就会自动执行，重新计算 UI 组件的参数，从而触发 UI 组件的重新渲染。
`mapStateToProps`的第一个参数总是state对象，还可以使用第二个参数，代表容器组件的props对象。
```
// 容器组件的代码
//    <FilterLink filter="SHOW_ALL">
//      All
//    </FilterLink>

const mapStateToProps = (state, ownProps) => {
  return {
    active: ownProps.filter === state.visibilityFilter
  }
}
```
- `mapDispatchToProps`是`connect`函数的第二个参数，用来建立 UI 组件的参数到`store.dispatch`方法的映射。也就是说，它定义了哪些用户的操作应该当作 Action，传给 Store。它可以是一个函数，也可以是一个对象。
- 如果mapDispatchToProps是一个函数，会得到dispatch和ownProps（容器组件的props对象）两个参数
```
const mapDispatchToProps = (
  dispatch,
  ownProps
) => {
  return {
    onClick: () => {
      dispatch({
        type: 'SET_VISIBILITY_FILTER',
        filter: ownProps.filter
      });
    }
  };
}
```
从上面代码可以看到，`mapDispatchToProps`作为函数，应该返回一个对象，该对象的每个键值对都是一个映射，定义了 UI 组件的参数怎样发出 Action。
如果`mapDispatchToProps`是一个对象，它的每个键名也是对应 UI 组件的同名参数，键值应该是一个函数，会被当作 Action creator ，返回的 Action 会由 Redux 自动发出。举例来说，上面的mapDispatchToProps写成对象就是下面这样。
```
const mapDispatchToProps = {
  onClick: (filter) => {
    type: 'SET_VISIBILITY_FILTER',
    filter: filter
  };
}
```
- `connect`方法生成容器组件以后，需要让容器组件拿到`state`对象，才能生成 UI 组件的参数。
一种解决方法是将`state`对象作为参数，传入容器组件。但是，这样做比较麻烦，尤其是容器组件可能在很深的层级，一级级将state传下去就很麻烦。React-Redux 提供`Provider`组件，可以让容器组件拿到`state`。
```
import { Provider } from 'react-redux'
import { createStore } from 'redux'
import todoApp from './reducers'
import App from './components/App'

let store = createStore(todoApp);

render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```
**上面代码中，`Provider`在根组件外面包了一层，这样一来，`App`的所有子组件就默认都可以拿到`state`了。**
