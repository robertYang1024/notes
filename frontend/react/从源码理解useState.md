# 从源码理解useState

## 学前思考题:
1. React Hooks 为什么必须在函数组件内部执行？React 如何能够监听 React Hooks 在外部执行并抛出异常？
2. React Hooks 如何把状态保存起来？保存的信息存在了哪里？
3. React Hooks 为什么不能写在条件语句中？


## 注意
本文中源码对应的版本是React 16.13.1

## React Hooks与Fiber的关系

### 双缓存Fiber树
我们知道React16启用了全新的架构，叫做Fiber。在React中最多会同时存在两棵Fiber树，当前HTML页面对应有fiber树叫做current fiber 树，workInProgress fiber树是正在构建的fiber树。当workInProgress fiber树构建完成并渲染后，current指针会指向 workInProgress fiber树，此时workInProgress Fiber树就变为current Fiber树，这个就是双缓存Fiber树。

<img src="./image/setState/fiberTree.png" width = "400" height = "480" align=center />  <br><br>

### fiber与Hooks
fiber节点用 `fiber.memoizedState` 保存 hooks 信息。

### Hook对象
```typescript
type Hook = {
  memoizedState: any, // 保存state
  baseState: any,      
  baseQueue: Update<any, any> | null,
  queue: UpdateQueue<any, any> | null, 
  next: Hook | null, // 指向下一个hook，形成链表
};

type UpdateQueue<S, A> = {
  pending: Update<S, A> | null, //指向待更新的update
  dispatch: (A => mixed) | null,
  lastRenderedReducer: ((S, A) => S) | null,
  lastRenderedState: S | null,
};

type Update<S, A> = {
  expirationTime: ExpirationTime,
  suspenseConfig: null | SuspenseConfig,
  action: A,
  eagerReducer: ((S, A) => S) | null,
  eagerState: S | null,
  next: Update<S, A>,   
  priority?: ReactPriorityLevel,
};
```
`Hook.memoizedState`记录着当前当前节点的hooks信息，Hook通过`Hook.next`指针形成链表。

## `HooksDispatcherOnMount`、`HooksDispatcherOnUpdate`
同样是执行`React.useState()`，函数组件初始化和更新的过程中，hooks的代码逻辑是不一样的，主要原因是，执行`React.useState()`时，会从`ReactCurrentDispatcher.current`取值，而该值会变化。

执行`React.useState()`，实际上是执行`ReactHooks.js`中的 `useState(initialState)`函数，
```js
  function useState(initialState) {
    var dispatcher = resolveDispatcher();
    return dispatcher.useState(initialState);
  }

/** 获取当前的dispatcher，也即是 ReactCurrentDispatcher.current 的值 */
  function resolveDispatcher() {
    var dispatcher = ReactCurrentDispatcher.current;

    ...

    return dispatcher;
  }
```
**React就是通过改变`ReactCurrentDispatcher.current`的值，用来在不同状态下执行不同的Hooks逻辑**。

`renderWithHooks()`是React.useState()执行过程中的一段重要逻辑，不管是初始化还是更新，都会执行到它。函数 `renderWithHooks`里的逻辑 ：(ReactFiberHooks.js)
```javaScript
 ...
 // 当是第一次时，current === null || current.memoizedState === null 会为true，赋值HooksDispatcherOnMount
 // 当时后面更新时，则为HooksDispatcherOnUpdate
    ReactCurrentDispatcher.current =
      current === null || current.memoizedState === null
        ? HooksDispatcherOnMount
        : HooksDispatcherOnUpdate;

    let children = Component(props, secondArg); //执行函数组件
 
  // 函数组件执行完后，赋值ContextOnlyDispatcher，如果后面还有调用useState()，则会报错
    ReactCurrentDispatcher.current = ContextOnlyDispatcher;
 ...

```

`ReactCurrentDispatcher.current` 的值有三种情况：
1. `ContextOnlyDispatcher`： 报错形态，只要开发者调用了这个形态下的 hooks ，就会抛出异常。
2. `HooksDispatcherOnMount`： 函数组件初始化 mount ，初次建立其 hooks 与 fiber 之间的关系。
3. `HooksDispatcherOnUpdate`：函数组件的更新，既然与 fiber 之间的桥已经建好了，那么组件再更新，就需要 hooks 去获取或者更新维护状态。

当初始化时，是 `HooksDispatcherOnMount`；当更新时，是 `HooksDispatcherOnUpdate`；当函数组件执行完后，会赋值为`ContextOnlyDispatcher`，此时如果再执行`React.useState()`，那么会是`throwInvalidHookError`，会报错。


`HooksDispatcherOnMount`、`HooksDispatcherOnUpdate`、`ContextOnlyDispatcher`对应的具体值如下：
```javascript
const HooksDispatcherOnMount = { /* 函数组件初始化用的 hooks */
    useState: mountState,   //初始化时执行
    useEffect: mountEffect,
    ...
}
const  HooksDispatcherOnUpdate ={ /* 函数组件更新用的 hooks */
   useState: updateState, // 更新时执行
   useEffect: updateEffect,
   ...
}
const ContextOnlyDispatcher = {  /* 当hooks不是函数内部调用的时候，调用这个hooks对象下的hooks，所以报错。 */
   useEffect: throwInvalidHookError,
   useState: throwInvalidHookError,  // 抛出一个错误
   ...
}
```

## useState 第一次执行
当组件初次加载时，从`beginWork`开始构建fiber节点，期间会执行函数组件以获得children节点，然后会执行函数组件里的React.useState，一直到`mountState`，其中useState的主要逻辑在mountState中，创建了hook对象和queue对象，初始化state值，到最后返回state 和 dispatchAction（绑定了当前fiber节点和queue 到 dispatchAction）。

```js
  function mountState(initialState) {

    var hook = mountWorkInProgressHook(); // 生成Hook对象，如果是第一个Hook，会存到fiber.memoizedState属性上

    if (typeof initialState === 'function') {
      initialState = initialState();
    }

    hook.memoizedState = hook.baseState = initialState; //  初始化state

    var queue = hook.queue = {  // 初始化queue
      pending: null,
      dispatch: null,
      lastRenderedReducer: basicStateReducer,
      lastRenderedState: initialState
    };
    
    // 把当前的fiber节点和queue 绑定到 dispatchAction （更新的时候用）
    var dispatch = queue.dispatch = dispatchAction.bind(null, currentlyRenderingFiber$1, queue);
    
    return [hook.memoizedState, dispatch];
  }
```
至此，第一次执行React.useState就完成了，初始化完成后会形成如下数据结构：

## useState更新

### dispatchAction

通过上面的useState的初始化过程得知，useState最后返回的是 [hook.memoizedState, dispatchAction]，那么我们通过去更新state的时候，其实执行就是dispatchAction（注意：返回的dispatchAction是绑定了fiber节点和queue的参数的）。

看一下dispatchAction的逻辑
```typescript
function dispatchAction<S, A>(fiber: Fiber,queue: UpdateQueue<S, A>,action: A,
) {

  // 创建update对象
  const update: Update<S, A> = {
    expirationTime,
    suspenseConfig,
    action,
    eagerReducer: null,
    eagerState: null,
    next: (null: any),
  };

  // 将update对象放进update链表中，queue.pending指向update链表
  const pending = queue.pending;
  if (pending === null) {
    // This is the first update. Create a circular list.
    update.next = update;
  } else {
    update.next = pending.next;
    pending.next = update;
  }
  queue.pending = update;

  const alternate = fiber.alternate;

  // 正在更新的fiber node是否是当前fiber
  if (fiber === currentlyRenderingFiber || (alternate !== null && alternate === currentlyRenderingFiber)) {
     
    // ... 
    // 说明当前fiber正在发生调和渲染更新，那么不需要更新,不发起新调度

  } else {
    if (fiber.expirationTime === NoWork && (alternate === null || alternate.expirationTime === NoWork)) {
        var currentState = queue.lastRenderedState; // 更新前的state
        var eagerState = lastRenderedReducer(currentState, action);  // 计算一下更新后的state

        if (objectIs(eagerState, currentState)) { // 如果相同，不发起更新
          return;
        }
    }
    scheduleWork(fiber, expirationTime); // scheduleWork是scheduleUpdateOnFiber，会发起更新调度
}
```
我们可以知道，当比较更新前后的state是一样的时候，是不会发起更新的。（如果setNum()的入参是个函数，那么不管state是否相同，都是会被执行的）
整个逻辑如下图：

执行完dispatchAction后，内存中的数据结构如下：

**`dispatchAction`的功能就是把每个更新创建一个update对象，然后添加到update链表中去，queue.pending指向update链表。等到更新的时候，会把这些update都取出来挨个执行，得出最终的state。**
### 执行调度更新 
```js
const scheduleWork = scheduleUpdateOnFiber
```
scheduleWork就是scheduleUpdateOnFiber，而scheduleUpdateOnFiber会发起更新调度，然后会构建workInProgress Fiber树，后面又会到`beginWork`函数，不过这一次会进入到`case FunctionComponent`分支，但是里面也是会执行`renderWithHooks()`，跟初始化时的逻辑一样。
```js
    ReactCurrentDispatcher.current =
      current === null || current.memoizedState === null
        ? HooksDispatcherOnMount
        : HooksDispatcherOnUpdate;
```
不同的是此时判断条件会为`false`，因为现在是更新，所以 `ReactCurrentDispatcher.current = HooksDispatcherOnUpdate`，此时执行`useState`取的是`HooksDispatcherOnUpdate.useState`，逻辑走的是`updateState`的逻辑。
 ```js
   function updateState(initialState) {
    return updateReducer(basicStateReducer);
  }
 ```
 可以看到，最后实际执行的是updateReducer()，这个是useState更新阶段最重要的一段逻辑了，在这里会把hook上的update链表依次执行，得出最终的state返回去，这样函数组件中就能获取到更新后的state。
 ```js
 function updateReducer(reducer, initialArg, init) {
 
  const hook = updateWorkInProgressHook(); // 以旧hook为基础，获取到当前hook的基础信息
    ...
   // 把待更新的pending队列取出来
   
   ...

    // 依次从update链表中取出update.action执行，得出最终的的state
    do {
      const action = update.action;

      newState = reducer(newState, action);

      update = update.next;
    } while (update !== null && update !== first);

    // 保存当前的state状态
    hook.memoizedState = newState;
    hook.baseState = newBaseState;
 
    ...

  const dispatch = queue.dispatch;
  return [hook.memoizedState, dispatch];
}
 ```
可以看到，在`dispatchAction`中生成的update链表，会在`updateReducer`中取出来，在`do while`循环中挨个执行，从而得出最终的state，即是函数组件中useState得到的值。

为什么hooks不能写在if条件语句中呢？
在updateReducer中，updateWorkInProgressHook是以旧hook为基础，获取到当前hook的基础信息。
```js
function updateWorkInProgressHook(): Hook {
  let nextCurrentHook: null | Hook;
  if (currentHook === null) {
    let current = currentlyRenderingFiber.alternate; // current fiber树
    if (current !== null) {
      nextCurrentHook = current.memoizedState; // current fiber树的hooks链表
    } 
  } else {
    nextCurrentHook = currentHook.next;
  }

  let nextWorkInProgressHook: null | Hook;
  if (workInProgressHook === null) {
    nextWorkInProgressHook = currentlyRenderingFiber.memoizedState;
  } else {
    nextWorkInProgressHook = workInProgressHook.next;
  }

  if (nextWorkInProgressHook !== null) {
    // There's already a work-in-progress. Reuse it.
    workInProgressHook = nextWorkInProgressHook;
    nextWorkInProgressHook = workInProgressHook.next;

    currentHook = nextCurrentHook;
  } else {
    // Clone from the current hook.
    currentHook = nextCurrentHook;

    const newHook: Hook = {
      memoizedState: currentHook.memoizedState,

      baseState: currentHook.baseState,
      baseQueue: currentHook.baseQueue,
      queue: currentHook.queue,

      next: null,
    };

    if (workInProgressHook === null) {
      // This is the first hook in the list.
      currentlyRenderingFiber.memoizedState = workInProgressHook = newHook;
    } else {
      // Append to the end of the list.
      workInProgressHook = workInProgressHook.next = newHook;
    }
  }
  return workInProgressHook;
}
```

总结：
1. Hook会形成链表，fiber.memoizedState会指向该链表（对于函数组件的fiber是这样，class组件的fiber.memoizedState存的是别的东西）
2. setNum()实际执行的是dispatchAction，会产生update链表，hook.queue.pending 指向update链表
3. setNum()会发起一个新的更新调度
4. useState更新的时候，会把update链表取出来依次执行，得到最终的state。

当在函数组件里使用自定义hooks时，自定义hooks里面的hook也会挂在fiber.memoizedState指向的Hook链表上。