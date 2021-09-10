## 学前思考题:
1. React Hooks 为什么必须在函数组件内部执行？React 如何能够监听 React Hooks 在外部执行并抛出异常？
2. React Hooks 如何把状态保存起来？保存的信息存在了哪里？
3. React Hooks 为什么不能写在条件语句中？


## React Hooks与Fiber的关系

### 双缓存Fiber树
我们知道React16启用了全新的架构，叫做Fiber。在React中最多会同时存在两棵Fiber树，当前HTML页面对应有fiber树叫做current fiber 树，workInProgress fiber树是正在构建的fiber树。当workInProgress fiber树构建完成并渲染后，current指针会指向 workInProgress fiber树，此时workInProgress Fiber树就变为current Fiber树，这个就是双缓存Fiber树。

<img src="./image/setState/fiberTree.png" width = "400" height = "480" align=center />  <br><br>

### fiber与Hooks
fiber节点保存了组件的状态，函数组件对应的fiber ，用 fiber.memoizedState 保存 hooks 信息。

### Hook对象
```typescript
export type Hook = {
  memoizedState: any, // 保存state
  baseState: any,      
  baseQueue: Update<any, any> | null,
  queue: UpdateQueue<any, any> | null, 
  next: Hook | null, // 指向下一个hook，形成链表
};

type UpdateQueue<S, A> = {|
  pending: Update<S, A> | null, //指向待更新的update
  dispatch: (A => mixed) | null,
  lastRenderedReducer: ((S, A) => S) | null,
  lastRenderedState: S | null,
|};

type Update<S, A> = {|
  expirationTime: ExpirationTime,
  suspenseConfig: null | SuspenseConfig,
  action: A,
  eagerReducer: ((S, A) => S) | null,
  eagerState: S | null,
  next: Update<S, A>,   
  priority?: ReactPriorityLevel,
|};
```
Hook.memoizedState记录着当前