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
- getDefaultProps()方法，只在*组件**创建**时调用一次*，并缓存返回的对象（即在`React.createClass`之后就会调用）。在组件装载之后，这个方法缓存的结果会用来保证访问`this.props`的属性时，当这个属性没有在父组件中传入（在这个组件的JSX属性中设置），也总是有值的。如果是ES6语法，可以直接定义`defaultProps`这个类属性来替代。