# 02 | React出现的历史背景及特性介绍

## 4个必须的APi
1. ReactDOM.render 方法让 React 组件渲染到某个具体的 DOM 节点。
2. 组件的 render 方法，渲染对应的组件
3. 组件的 setState 方法，用于改变组件状态，触发 render
4. 如何通过 props 给 React 组件传递参数（数据只能从上往下流动）

<br>

- Flux 是一种设计模式，而不是具体实现，类似于 MVC 设计模式。
- Flux 有多种具体实现，比如 React 的 Redux，Vue 的 Vuex等等。

## 整体的刷新相对于局部的刷新，实际上性能会不会较差？
> 并不会，因为 virtual dom 是完全内存中的计算，速度非常快。而 React 在将 dom 变化应用到真实 dom 时会用最优的方法，性能不会比手动的差。
(这里说的整体刷新是指逻辑上纯 UI 的重绘)

# 03 | 以组件方式考虑UI的构建

## 何时创建组件：单一职责原则
1. 每个组件只做一件事
2. 如果组件变得复杂，那么应该拆分成小组件

## 数据状态管理：DRY原则（Don’t Repeat Yourself）
1. 能计算得到的状态就k要单独存储
2. 组件尽量无状态f所需数据通过 TVSTW 获取

# 04 | JSX的本质 : 不是模板引擎，而是语法糖

## 约定自定义组件以大写字母开头
1. react认为小写的tag是原生DOM节点，如 div
2. 大写字母开头为自定义组件
3. JSX 标记可以直接使用属性语法，例如<menu.Item /> （这时可以不用大写字母开头)

# 05 | React组件的生命周期及其使用场景

## constructor
1. 用于初始化内部状态，很少使用
2. 唯一可以直接修改 state 的地方

## getDerivedStateFromProps
1. 当 state 需要从 props 初始化时使用
2. 尽量不要使用：维护两者状态一致性会增加复杂度
3. 每次 render 都会调用
4. 典型场景：表单控件获取默认值

## componentDidMount
1. UI 渲染完成后调用
2. 只执行一次
3. 典型场景：获取外部资源

## componentWillUnMount
1. 组件移除时被调用
2. 典型场景:资源释放

## getSnapshotBeforeUpdate
1. 在页面 render 之前调用，state 已更新
2. 典型场景：获取 render 之前的 DOM 状态
3. 此生命周期方法的任何返回值将作为参数传递给 componentDidUpdate()。

## componentDidUpdate
1. 每次 UI 更新时被调用
2. 典型场景:页面需要根据 props 变化重新获取数据

## shouldComponentUpdate
1.决定 Virtual DOM 是否要重绘
2.一般可以由 Pure Component 自动实现
3.典型场景:性能优化

# 06 | 理解Virtual DOM及key属性的作用

什么是虚拟DOM?

>和虚拟 DOM 对应的是实体 DOM，实体 DOM 是真正展示给用户看的 UI 背后的 DOM 结构。虚拟 DOM 只在内存中维护，当所有对其的修改完成后，React 将其应用到实体 DOM。应用的过程是找到所有虚拟 DOM 和当前实体 DOM 的区别（diff），从而高效的完成应用（修改实体 DOM）过程。理解了 diff 也就理解了虚拟 DOM。


为啥普通算法两棵树的diff算法是O(N^3) 而不是O(N^2)?

>如果有兴趣可以看下相关的算法实现：https://grfia.dlsi.ua.es/ml/algorithms/references/editsurvey_bille.pdf ，这是 React 文档中提到的普通 tree 的 diff 算法。https://reactjs.org/docs/reconciliation.html

请问现在已有像 Flow 和 Typescript 这种静态类型检查工具，是否可以不用 【prop-types】库了?

>是的，如果用 typescript 或者 flow，就不需要 prop-types 做检查。