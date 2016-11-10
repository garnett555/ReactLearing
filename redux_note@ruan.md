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
上面代码中，`window.STATE_FROM_SERVER`就是整个应用的状态初始值。注意，如果提供了这个参数，它会覆盖 Reducer 函数的默认初始值。
- Redux 提供了一个`combineReducers`方法，用于 Reducer 的拆分。你只要定义各个子 Reducer 函数，然后用这个方法，将它们合成一个大的 Reducer。