# Redux Tutorial
## @Ruan
- Redux 的设计思想很简单，就两句话。
（1）**Web 应用是一个状态机，视图与状态是一一对应的。**
（2）**所有的状态，保存在一个对象里面。**
- Store 就是保存数据的地方，你可以把它看成一个容器。整个应用只能有一个 Store。Redux 提供`createStore`这个函数，用来生成 Store。`createStore`函数接受另一个函数作为参数，返回新生成的 Store 对象。
``` Import {createStore} from 'redux';
		const store = createStore(fn); ```
- Store对象包含所有数据。如果想得到某个时点的数据，就要对 Store 生成快照。这种时点的数据集合，就叫做 State。当前时刻的 State，可以通过`store.getState()`拿到。`const state = store.getState();`
Redux 规定， 一个 State 对应一个 View。只要 State 相同，View 就相同。你知道 State，就知道 View 是什么样，反之亦然。