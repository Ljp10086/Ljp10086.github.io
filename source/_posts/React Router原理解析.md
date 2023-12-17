# **手写React Router**

## 前言

本文的实现方式均基于React Router v6.8.0构建，具体Api可参阅[官方文档](https://reactrouter.com/en/6.8.0)。

## 单页应用

众所周知，用React或者Vue等框架构建的应用都是单页面应用，单页面应用是使用一个html前提下，一次性加载 js，css 等资源，所有页面都在一个容器页面下，页面切换实质是组件的切换。

## h5新增的history Api

改变路由，指的是通过调用 api 实现的路由跳转，比如开发者在 React 应用中调用 history.push 改变路由，本质上是调用 window.history.pushState 方法。

`window.history.pushState`

``` JavaScript
history.pushState(state,title,path)
```

参数和 pushState 一样，这个方法会修改当前的 history 对象记录， 但是 history.length 的长度不会改变。

2. 监听路由 popstate

```
window.addEventListener('popstate',function(e){
    /* 监听改变 */
})
```

同一个文档的 history 对象出现变化时，就会触发 popstate 事件 `history.pushState` 可以使浏览器地址改变，但是无需刷新页面。注意的是：用 `history.pushState()` 或者 `history.replaceState()` 不会触发 popstate 事件。 popstate 事件只会在浏览器某些行为下触发, 比如点击后退、前进按钮或者调用 `history.back()`、`history.forward()`、`history.go()`方法。

## 如何在React组件中监听外部数据

比如说我有这样一个组件

``` JavaScript
const store = {
  words: ''
}

const subWindowClick = (e) => {
  store.words = e.pageX.toString();
};

window.addEventListener('click', subWindowClick);

export default function App() {
  return <div>words: {store.words}</div>
}
```

需求：每一次点击window的时候同步更新App组件里的文字。

熟悉React的同学都知道我们在函数组件中每一次调用`setState`的时候才会引起组件的更新，如果可以在`window.onClick`事件中先修改`store.words`的值，再调用`setState`就可以完成我们的需求了。

所以我们修改代码如下：

``` JavaScript
const store = {
  words: '',
};

const onWindowClick = (callback) => {
  const subWindowClick = (e) => {
    store.words = e.pageX.toString();
    console.log(store.words);
    callback({ ins: store.words });
  };

  window.addEventListener('click', subWindowClick);

  return window.removeEventListener.bind(window, 'click', subWindowClick);
};

export default function App() {
  const [_, forceUpdate] = React.useState({ ins: store.words });

  React.useEffect(() => {
    const unSub = onWindowClick(forceUpdate);
    return unSub;
  });

  return <div>words: {store.words}</div>;
}
```

这段代码将`forceUpdate`当做参数传递给`onWindowClick`中，然后再`window.onClick`里先改变`store.words`的值再调用`forceUpdate`并且每次传入不同的值，这样就可以完成组件的刷新。

接下来我们将这段代码封装成一个比较通用的`Hook`：

``` JavaScript
export default function useSyncExternalStore<T>(
  sub: (fn: () => void) => () => void,
  getSnapshot: () => T
) {
  const value = getSnapshot();
  const [_, forceUpdate] = React.useState({ value, getSnapshot });

  React.useEffect(() => {
    const handlerChanged = () => {
      forceUpdate({
        value,
        getSnapshot,
      });
    };

    sub(handlerChanged);
  }, [sub]);

  return value;
}
```

用法：

``` JavaScript
const store = {
  words: '',
};

const getSnapshot = () => {
  return store.words;
};

const subscribe = (callback: any) => {
  const subWindowClick = (e) => {
    store.words = e.pageX.toString();
    callback();
  };

  window.addEventListener('click', subWindowClick);

  return () => {
    window.removeEventListener('click', subWindowClick);
  };
};

const App = () => {
  const ref = React.useRef<HTMLInputElement>(null);

  const words = useSyncExternalStore(subscribe, getSnapshot);

  return (
    <div>
      <p>words: {words}</p>
      <input
        type="text"
        ref={ref}
        onChange={(e) => {
          store.words = e.target.value;
        }}
      />
    </div>
  );
};

export default App;
```

我们将获取`store.words`与监听事件变化的方法传入`useSyncExternalStore`，这样就可以在每次需要更新的时候调用钩子内部的`forceUpdate`来达到组件更新的效果。
完整代码 [stackblitz](https://stackblitz.com/edit/react-ts-peko2g?file=App.tsx)。

这个钩子在React 18之后是默认支持的，用法也和上方示例大致相同，详情请见[useSyncExternalStore](https://beta.reactjs.org/reference/react/useSyncExternalStore)。

## 实现mini-router

本章节，我们会从 0 到 1 实现一个 React 路由功能，这里称之为`mini-router`。

### 设计思路

1. 使用React 18自带的`useSyncExternalStore`钩子作为监听外部数据源改变的方式。
2. 使用`useContext`传递内部变量值。
3. 编写与BrowserHistory模式下。

### 代码实现

#### 创建router对象 - createBrowserRouter

``` JavaScript
function matchRoute(routes, pathname) {
  return routes.find((x) => x.path === pathname);
}

export function createBrowserRouter(routes) {
  const subscribers = new Set<Function>();

  const initMatch = matchRoute(routes, location.pathname);
  let state = {
    match: initMatch,
    location: window.location,
  };

  function subscribe(fn) {
    subscribers.add(fn);
    return () => subscribers.delete(fn);
  }

  function updateState() {
    state = {
      match: matchRoute(routes, location.pathname),
      location: window.location,
    };
    subscribers.forEach((subscriber) => subscriber());
  }

  function navigate(to) {
    history.pushState({}, null, to);
    updateState();
  }

  window.addEventListener('popstate', () => {
    updateState();
  });

  return {
    get state() {
      return state;
    },
    navigate,
    subscribe,
    routes,
  };
}
```

设计思路：

* `state`属性包括了当前match到的路由与`location`对象。
* `navigate`方法提供了路由跳转的能力。
* 通过当前的`location`的`pathname`进行匹配，判断渲染组件。
* `subscribe`方法可以存储`useSyncExternalStore`钩子中的回调方法。
* `updateState`方法可以触发`useSyncExternalStore`钩子中的回调方法，从而完成组件刷新。
* 使用`popstate`事件监听浏览器的前进与后退，在浏览器前进与后退时调用`updateState`方法更新组件。

#### 控制更新 - Provider

``` JavaScript
const Provider = ({
  router,
}: {
  router: ReturnType<typeof createBrowserRouter>;
}) => {
  const state = React.useSyncExternalStore(
    router.subscribe,
    () => router.state
  );
  console.log(state);
  return (
    <DataRouterContext.Provider value={{ router, state }}>
      <Routes></Routes>
    </DataRouterContext.Provider>
  );
};

export default Provider;
```

* 该组件接收使用`createBrowserRouter`方法创建的`router`对象，使用`useSyncExternalStore`钩子控制组件的重新渲染，并且每次可以拿到最新的`state`值。
* 使用`useContext`提取出路由上下文，传递给子组件进行消费。

#### 控制渲染 - Routes

``` JavaScript
export function Routes() {
  const context = React.useContext(DataRouterContext);

  return context.state.match.element;
}
```

* 使用`useContext`获取`Provider`组件中传递的参数。
* 获取匹配的路由对象并将之渲染到DOM中。
  
#### 使用方式

```JavaScript

const Root = () => (
  <div>
    root
    <br />
    <Link to={'/root1'} />
    <br />
    <Link to={'/root2'} />
  </div>
);

const Root1 = () => (
  <div>
    root11111
    <br />
    <Link to={'/root2'} />
  </div>
);
const Root2 = () => <div>root2222</div>;

const routes = [
  {
    path: '/',
    element: <Root />,
  },

  {
    path: '/root1',
    element: <Root1 />,
  },
  {
    path: '/root2',
    element: <Root2 />,
  },
];

const router = createBrowserRouter(routes);

export default function App() {
  return (
    <div>
      <Provider router={router}></Provider>
    </div>
  );
}
```

使用方式也保持与React Router一致。
[完整代码](https://stackblitz.com/edit/react-ts-wu54qk?file=App.tsx)

## 源码解析

> 注意：源码解析会过滤掉一些代码，将重点列出。

### createBrowserRouter

该方法主要创建了对于`Window.history` API的自定义封装，同时向实例上注册监听，当路由发生变化会回调执行`setState`更新路由信息，并且触发组件更新。

``` JavaScript
export function createBrowserRouter(
  routes: RouteObject[],
  opts?: {
    basename?: string;
    hydrationData?: HydrationState;
    window?: Window;
  }
): RemixRouter {
  return createRouter({
    history: createBrowserHistory({ window: opts?.window }) // 创建history实例,
    routes: enhanceManualRouteObjects(routes) // 处理传入的routes对象数组,

    basename: opts?.basename,
    hydrationData: opts?.hydrationData || parseHydrationData(),
  }).initialize();
}
```

### createBrowserHistory

创建history对象

``` JavaScript
function createBrowserHistory() {
  // .........
  let { window = document.defaultView!, v5Compat = false } = options;
  let globalHistory = window.history;
  let listener: Listener | null = null;

  function push(to: To, state?: any) {
    console.log(to);
    action = Action.Push;
    let location = createLocation(history.location, to, state);
    if (validateLocation) validateLocation(location, to);

    index = getIndex() + 1;
    let historyState = getHistoryState(location, index);
    let url = history.createHref(location);

    try {
      // 调用history.pushState方法
      globalHistory.pushState(historyState, "", url);
    } catch (error) {
      // ios极限push 100个 超过之后会throw error 
      // catch 之后刷新页面
      window.location.assign(url);
    }

    if (v5Compat && listener) {
      // 每一次调用使路由变化的Api之后回调listener方法
      listener({ action, location: history.location, delta: 1 });
    }
  }

  // .........

  return {
    // .........
    listen(fn: Listener) {
      if (listener) {
        throw new Error("A history only accepts one active listener");
      }
      // 注册popstate事件 监听路由变化
      window.addEventListener(PopStateEventType, handlePop);
      // 赋值listener方法
      listener = fn;
      console.log("listener function");
      return () => {
        window.removeEventListener(PopStateEventType, handlePop);
        listener = null;
      };
    },
    push
  }
  // .........
}
```

### createRouter

创建Router对象。

``` JavaScript
function createRouter(init: RouterInit): Router {
  // 创建监听者对象
  let subscribers = new Set<RouterSubscriber>();

  // 初始化时获取当前match到的路由
  let initialMatches = matchRoutes(
    dataRoutes,
    init.history.location,
    init.basename
  );

  let state: RouterState = {
    location: init.history.location,
    matches: initialMatches,
    // ...
  }
  // 重点：改变state并且通知组件更新
  function updateState(newState: Partial<RouterState>): void {
    state = {
      ...state,
      ...newState,
    };
    subscribers.forEach((subscriber) => subscriber(state));
  }

  function initialize() {
    // 监听history变化，改变state状态。
    init.history.listen(
      ({ action: historyAction, location, delta }) => {
        updateState({ blockers: new Map(router.state.blockers) });
      }
    )
  }

  // ...

  return {
    initialize,
    subscribe,
    get state() {
      return state;
    },
    // ...
  }
}
```

### RouterProvider

创建Router实例，渲染UI

``` JavaScript
export function RouterProvider({
  fallbackElement,
  router,
}: RouterProviderProps): React.ReactElement {
  // 使用useSyncExternalStore订阅外部数据源
  // 将callback传入 router.subscribe
  // 之后可以通过上一章的updateState更新该组件并获取新state
  let state: RouterState = useSyncExternalStore(
    router.subscribe,
    () => router.state,
  );

  // ...

  return (
    // ...
    // 渲染UI
    <Routes></Routes>
    // ...
  );
}
```
