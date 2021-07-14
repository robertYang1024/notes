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

<br>

# 07 | 组件设计模式 : 高阶组件和函数作为子组件

1. 高阶组件和函数子组件都是设计模式
2. 可以实现更多场景的组件复用
### 高阶组件
```jsx
import React from "react";

export default function withTimer(WrappedComponent) {
  return class extends React.Component {
    state = { time: new Date() };
    componentDidMount() {
      this.timerID = setInterval(() => this.tick(), 1000);
    }

    componentWillUnmount() {
      clearInterval(this.timerID);
    }

    tick() {
      this.setState({
        time: new Date()
      });
    }
    render() {
      // 给入参WrappedComponent组件添加time属性
      return <WrappedComponent time={this.state.time} {...this.props} />;
    }
  };
}

// 使用 ,可以直接导出
export default withTimer(MyTestComponent);
```
### 函数作为子组件
```jsx
class MyComponent extends React.component {
  render () {
    return (
      <div>
        {this.props.children('')}
      </div>
    )
  }
}

// 使用
<MyComponent>
  { (name) => (
    <div>{name}</div>
  )}
</MyComponent>
```

没能明白函数子组件是怎么回事？没看到children在哪里被赋值了啊？

>`children` 是 `React` 组件的一个特殊内置属性，`<Comp>xxx</Comp>` 里的 `xxx` 部分会作为 `children` 传递给 `Comp` 组件，如果 `xxx` 是函数，那么 `Comp` 里主动调用它去得到结果。

请问老师在示例代码`return class  extends React.Component `中间处没有类名呢？类名不写时其类名是什么呢？

>相当于返回了一个匿名的类，class 的作用是声明一个类，并不一定需要名字。和匿名函数 `return function() {}` 是一样的原理。

<br>

# 08 | Context API及其使用场景

官方文档 https://zh-hans.reactjs.org/docs/context.html

这一讲没说个啥，我借此把Context的API整理下。现在由于有了hooks，我更加**推荐hooks的使用方式** `const value = useContext(MyContext);` 用起来也很方便，比 `Context.Consumer` 要简约多了。

### API
- `React.createContext`
   - `const MyContext = React.createContext(defaultValue);`
- `Context.Provider`
   - `<MyContext.Provider value={/* 某个值 */}>`
- `useContext` (hooks写法，推荐)   
- `Class.contextType`
   - 这个跟Context.Consumer都是用来获取Context的值，只不过写法要比Consumer简单
- `Context.Consumer`
- `Context.displayName`

```jsx
//  创建一个 context（“light”为默认值）。
const ThemeContext = React.createContext('light');

class App extends React.Component {
  render() {
    // 使用一个 Provider 来将当前的 theme 传递给以下的组件树。无论多深，任何组件都能读取这个值。
    // 在这个例子中，我们将 “dark” 作为当前的值传递下去。
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}
```
useContext:
```jsx
const value = useContext(MyContext);

// useContext(MyContext) 只是让你能够读取 context 的值以及订阅 context 的变化。你仍然需要在上层组件树中使用 <MyContext.Provider> 来为下层组件提供 context。
```


Class.contextType用法:

此属性能让你使用 this.context 来消费最近 Context 上的那个值。你可以在任何生命周期中访问到它，包括 render 函数中。
```jsx
// 如果你正在使用实验性的 public class fields 语法，你可以使用 static 这个类属性来初始化你的 contextType。
class MyClass extends React.Component {
  static contextType = MyContext;
  render() {
    // this.context就是最近Context 上的那个值，比如上面的dark
    let value = this.context;
    <div>{value}</div>
  }
}
//------------------------------------------------------
// 如果不支持static语法，就要这样用
class MyClass extends React.Component {
  componentDidMount() {
    let value = this.context;
    /* 在组件挂载完成后，使用 MyContext 组件的值来执行一些有副作用的操作 */
  }
  render() {
    let value = this.context;
    /* 基于 MyContext 组件的值进行渲染 */
  }
}

// 在这里赋值
MyClass.contextType = MyContext;

```

Context.Consumer：
```jsx
// 这种方法需要一个函数作为子元素（function as a child）

<MyContext.Consumer>
  {value => /* 基于 context 值进行渲染*/}
</MyContext.Consumer>

// 如果跟MyContext不在同一个文件，那就要把MyContext import进来 
//（自然，创建的MyContext要先export出去）
```

Context.displayName：
```jsx
const MyContext = React.createContext(/* some value */);
MyContext.displayName = 'MyDisplayName';

<MyContext.Provider> // "MyDisplayName.Provider" 在 DevTools 中
<MyContext.Consumer> // "MyDisplayName.Consumer" 在 DevTools 中
```


# 09 | 使用脚手架工具创建React项目

- create-react-app （适合实验新特性）
- Rekit (基于cra,适合大型项目)
- Codesandbox.io (online)

实际企业开发中用到的脚手架工具是rekit嘛，还有其他比较受欢迎的脚手架工具嘛
> 国内用的比较多的还有 dva ，是蚂蚁金服的一套脚手架工具。Rekit 是我个人的开源项目，算是对自己的实践方式的一个工具化，在 SAP, eBay, 阿里 也都有用到，适合中大型单页应用。

<br>

# 10 | 打包和部署

打包注意事项：
1. 设置 nodejs 环境为 production
2. 禁用开发时l用代码，比如 logger
3. 设置应用根路径