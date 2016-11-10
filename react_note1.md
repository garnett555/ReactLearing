# React 中文教程
- React组件类必须以大写开头
- 传递一些*方法*到`React.createClass()`创建一个新的React组件，这些方法中最重要的是`render`方法，该方法返回一棵React组件树，这棵树最终将会渲染成HTML。
- `ReactDOM.render()`*实例化根组件*，启动框架，注入标记到原始的DOM元素中。`ReactDOM.render()`应该只在复合组件被定义之后被调用。
- HTML组件是正常的React组件，就和你定义的一样，只有一个*区别*，那就是JSX编译器会自动重写HTML标签为`React.createElement(tagName)`表达式，其他什么都不做，这样是为了避免污染全局命名空间。
- 从父级组件传来的数据在子组件中作为“属性”可供使用，这些属性可以通过`this.props`访问。
- 在JSX中，通过将JavaScript表达式放在大括号中(作为属性或者子节点)，你可以把文本或者React组件放置在*组件树*中。我们以`this.props`的键名访问传递给组件的命名属性，以`this.props.children`访问任何嵌套的元素。
- `props`是不可变的，它们从父级传来并*被父级拥有*。为了实现交互，我们给组件引进了可变的`state`，`this.state`是*组件私有的*，可以通过调用`this.setState()`改变它，每当`state`更新，组件就重新渲染自己。
- `getInitialState()`在生命周期里只执行一次，并设置组件的初始状态。
- `componentDidMount`是一个当组件被渲染时被React自动调用的方法。

# @hulufei
- getInitialState()方法，用于初始化this.state的值，*只在组件装载之前调用一次*。
- getDefaultProps()方法，只在*组件创建时调用一次*，并缓存返回的对象（即在`React.createClass`之后就会调用）。在组件装载之后，这个方法缓存的结果会用来保证访问`this.props`的属性时，当这个属性没有在父组件中传入（在这个组件的JSX属性中设置），也总是有值的。如果是ES6语法，可以直接定义`defaultProps`这个类属性来替代。
- 如果`ref`是设置在原生HTML元素上，它拿到的就是DOM元素，如果设置在自定义组件上，它拿到的就是组件实例，这时候就需要通过`findDOMNode`来拿到组件的DOM元素。因为无状态组件没有实例，所以`ref`不能设置在无状态组件上，一般来说这没什么问题，因为无状态组件没有实例方法，不需要`ref`去拿实例调用相关的方法，但是如果想要拿无状态组件的 DOM 元素的时候，就需要用一个状态组件封装一层，然后通过`ref`和`findDOMNode`去获取。
- **NOTE**:不要在`render`或`render`之前访问`refs`。
- 如果组件中包含通过循环插入的子元素，为了保证重新渲染UI的时候能够正确显示这些子元素，每个元素都需要通过一个特殊的`key`属性指定一个唯一值，为了内部diff的效率。
- 父子组件间通信，就是通过`props`属性传递，在父组件给子组件设置`props`，然后子组件就可以通过`props`访问到父组件的数据/方法，这就搭建起了父子组件间通信的桥梁。
## Flux
- React 标榜自己是 MVC 里面 V 的部分，那么 Flux 就相当于添加 M 和 C 的部分。
- 一个Flux应用主要包含四个部分：
1.`the dispatcher`: 处理动作分发，维护store之间的依赖关系
2.`the stores`: 数据和逻辑部分
3.`the views`: React组件，这一层可以看做controller-views，作为视图同时响应用户交互
4.`the actions`: 提供给dispatcher  传递数据给store
- Flux核心的“单向数据流”如何运作：
``` Actions -> Dispatcher -> Store -> View ```
整个流程如下：
1.首先要有 action，通过定义一些 action creator 方法根据需要创建 Action 提供给 dispatcher 2.View 层通过用户交互（比如 onClick）会触发 Action 3.Dispatcher 会分发触发的 Action 给所有注册的 Store 的回调函数 4.Store 回调函数根据接收的 Action 更新自身数据之后会触发一个 change 事件通知 View 数据更改了 5.View 会监听这个 change 事件，拿到对应的新数据并调用 setState 更新组件 UI
所有的状态都由 Store 来维护，通过 Action 传递数据，构成了如上所述的单向数据流循环，所以应用中的各部分分工就相当明确，高度解耦了。
## Redux
### Redux三大基本原则
- 1. 整个应用只有唯一一个可信数据源，也就是只有一个 Store 2. State 只能通过触发 Action 来更改 3. State 的更改必须写成纯函数，也就是每次更改总是返回一个新的 State，在 Redux 里这种函数称为 Reducer
### Actions
Action 很简单，就是一个单纯的包含 `{ type, payload }` 的对象，`type` 是一个常量用来标示动作类型，`payload` 是这个动作携带的数据。Action 需要通过 `store.dispatch()` 方法来发送。一般来说，会使用函数（Action Creators）来生成 action，这样会有更大的灵活性，Action Creators 是一个 pure function，它最后会返回一个 action 对象。
### Reducers
Reducer 用来处理 Action 触发的对状态树的更改。所以一个 reducer 函数会接受 `oldState` 和 `action` 两个参数，返回一个新的 state：```(oldState, action) => newState```
