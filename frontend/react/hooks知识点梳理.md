# Hooks知识点梳理

## 动机
- 在组件之间复用状态逻辑很难
- 复杂组件变得难以理解
- 难以理解的 class
- 官方文档：https://zh-hans.reactjs.org/docs/hooks-intro.html#motivation

## Hook 使用规则
- 只能在函数最外层调用 Hook。不要在循环、条件判断或者子函数中调用。
- 只能在 React 的函数组件中调用 Hook。不要在其他 JavaScript 函数中调用。（自定义的 Hook 中除外）

### 在组件之间重用一些状态逻辑
1. [高阶组件](https://zh-hans.reactjs.org/docs/higher-order-components.html)
   - 具体而言，高阶组件是参数为组件，返回值为新组件的函数。
2. [render props](https://zh-hans.reactjs.org/docs/render-props.html)
   -  “render prop” 是指一种在 React 组件之间使用一个值为函数的 prop 共享代码的简单技术
   - `<DataProvider render={data => (<h1>Hello {data.target}</h1>) } />`
3. **自定义 Hook**   
   - 自定义 Hook 用来复用逻辑很好用的

## useState    
```jsx
const [count, setCount] = useState(0);
```
- `useState()`的初始 state 可以按照需要赋值数字、字符串或者对象；class的`this.setState()`是对象
- `useState()`不会合并新旧值，`this.setState()`会合并成一个对象
- `useState()`会进行浅比较，没改变不会更新；`this.setState()`只要调用就会更新
- `useState()`返回值是数组：当前 state 以及更新 state 的函数 ，需要成对的获取它们（例如：[count, setCount] ，方括号这种 JavaScript 语法叫数组解构）；class则不用
- `useState()`初次赋值后，再次去改变只能通过`setCount()`，改变`useState()`的参数是没用的

## useEffect
### 副作用操作
Effect Hook 可以让你在函数组件中执行副作用操作。**数据获取，设置订阅以及手动更改 React 组件中的 DOM 都属于副作用**。

在 React 组件中有两种常见副作用操作：需要清除的和不需要清除的

### effect的清除函数

```jsx
//无需清除的 effect
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
```

```jsx
//需要清除的 effect
  useEffect(() => {
    subscribe(); //订阅something
    return () => cleanup();  // 返回一个函数，这个函数用来清除something
  });
```

- `useEffect()`它在调用一个新的 effect 之前会对前一个 effect 进行清理（如果有清理函数的话）。
- `useEffect()`在组件销毁时也会执行清理函数。
```jsx
// Mount with { friend: { id: 100 } } props
ChatAPI.subscribeToFriendStatus(100, handleStatusChange);     // 运行第一个 effect

// Update with { friend: { id: 200 } } props
ChatAPI.unsubscribeFromFriendStatus(100, handleStatusChange); // 清除上一个 effect
ChatAPI.subscribeToFriendStatus(200, handleStatusChange);     // 运行下一个 effect

// Update with { friend: { id: 300 } } props
ChatAPI.unsubscribeFromFriendStatus(200, handleStatusChange); // 清除上一个 effect
ChatAPI.subscribeToFriendStatus(300, handleStatusChange);     // 运行下一个 effect

// Unmount
ChatAPI.unsubscribeFromFriendStatus(300, handleStatusChange); // 清除最后一个 effect
```
### 依赖项

`useEffect()`的第二个参数是依赖项，是数组, 可以不传或者空数组，所有依赖项都放里面
```jsx
//依赖项为空，每次都会执行，执行时会先执行前一次的清理函数，再执行effect，销毁时会执行清理函数
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
```
```jsx
//依赖项为空数组，只执行一次，销毁时执行清理函数
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  }, []);
```
```jsx
//依赖项为count，首次会执行一次，count有改动时会执行，没变动不执行，销毁时执行清理函数
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  }, [count]); //首次会执行一次，count有改动时会执行，没变动不执行
```

## 自定义 Hook
自定义 Hook 是一个函数，其名称以 “use” 开头，函数内部可以调用其他的 Hook。

- 自定义 Hook 必须以 “use” 开头吗？
   - 必须如此。这个约定非常重要。不遵循的话，由于无法判断某个函数是否包含对其内部 Hook 的调用，React 将无法自动检查你的 Hook 是否违反了 Hook 的规则。
- 在两个组件中使用相同的 Hook 会共享 state 吗？
   - 不会。自定义 Hook 是一种重用状态逻辑的机制(例如设置为订阅并存储当前值)，所以每次使用自定义 Hook 时，其中的所有 state 和副作用都是完全隔离的。
- 自定义 Hook 如何获取独立的 state？
   - 每次调用 Hook，它都会获取独立的 state。