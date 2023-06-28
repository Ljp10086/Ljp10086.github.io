---
title: 解析LeetCode开源的一道TypeScript类型编程问题
---

## 前言

前一阵子偶然间看到了有LeetCode开源的一道TS题，好奇之下我点进入看了一下，没想到仔细一看，居然一行代码都看不懂，让我备受打击，于是我挑灯夜战，连夜学习Ts类型编程，终于在努力了半把个月之后于今日将此题解出，写此文章记录的同时也分享给大家。

顺手贴入原题地址：[编写复杂TS类型](https://github.com/LeetCode-OpenSource/hire/blob/master/typescript_zh.md)


## 问题定义

假设有一个叫 EffectModule 的类

```typescript
class EffectModule {}
```

这个对象上的方法只可能有两种类型签名:

```typescript
interface Action<T> {
  payload?: T
  type: string
}

asyncMethod<T, U>(input: Promise<T>): Promise<Action<U>>

syncMethod<T, U>(action: Action<T>): Action<U>
```

这个对象上还可能有一些任意的非函数属性：

```typescript
interface Action<T> {
  payload?: T;
  type: string;
}

class EffectModule {
  count = 1;
  message = "hello!";

  delay(input: Promise<number>) {
    return input.then(i => ({
      payload: `hello ${i}!`,
      type: 'delay'
    }));
  }

  setMessage(action: Action<Date>) {
    return {
      payload: action.payload!.getMilliseconds(),
      type: "set-message"
    };
  }
}
```

现在有一个叫 `connect` 的函数，它接受 `EffectModule` 实例，将它变成另一个对象，这个对象上只有`EffectModule` 的同名方法，但是方法的类型签名被改变了:

```typescript
asyncMethod<T, U>(input: Promise<T>): Promise<Action<U>>  变成了
asyncMethod<T, U>(input: T): Action<U> 
syncMethod<T, U>(action: Action<T>): Action<U>  变成了
syncMethod<T, U>(action: T): Action<U>
```

例子:

`EffectModule` 定义如下:

```typescript
interface Action<T> {
  payload?: T;
  type: string;
}

class EffectModule {
  count = 1;
  message = "hello!";

  delay(input: Promise<number>) {
    return input.then(i => ({
      payload: `hello ${i}!`,
      type: 'delay'
    }));
  }

  setMessage(action: Action<Date>) {
    return {
      payload: action.payload!.getMilliseconds(),
      type: "set-message"
    };
  }
}
connect 之后:

type Connected = {
  delay(input: number): Action<string>
  setMessage(action: Date): Action<number>
}
const effectModule = new EffectModule()
const connected: Connected = connect(effectModule)
```

要求
在 题目链接 里面的 `index.ts` 文件中，有一个 `type Connect = (module: EffectModule) => any`，将 `any` 替换成题目的解答，让编译能够顺利通过，并且 `index.ts` 中 `connected` 的类型与:

```typescript
type Connected = {
  delay(input: number): Action<string>;
  setMessage(action: Date): Action<number>;
}
```

完全匹配。

题目链接里的`index.ts`文件。

```
import { expect } from "chai";

interface Action<T> {
  payload?: T;
  type: string;
}

class EffectModule {
  count = 1;
  message = "hello!";

  delay(input: Promise<number>) {
    return input.then(i => ({
      payload: `hello ${i}!`,
      type: 'delay'
    }));
  }

  setMessage(action: Action<Date>) {
    return {
      payload: action.payload!.getMilliseconds(),
      type: "set-message"
    };
  }
}

// 修改 Connect 的类型，让 connected 的类型变成预期的类型
type Connect = (module: EffectModule) => any;

const connect: Connect = m => ({
  delay: (input: number) => ({
    type: 'delay',
    payload: `hello 2`
  }),
  setMessage: (input: Date) => ({
    type: "set-message",
    payload: input.getMilliseconds()
  })
});

type Connected = {
  delay(input: number): Action<string>;
  setMessage(action: Date): Action<number>;
};

export const connected: Connected = connect(new EffectModule());
```

### 题目解析

分析题目可知，我们需要做的就是将**28行**的`any`替换为我们自己解析出来的符合`Connected`的类型。

```
any => {
  delay(input: number): Action<string>
  setMessage(action: Date): Action<number>
}
```

> 当然我们不能把类型直接粘贴过去，还是要我们自己根据类型推断得出。

### 思路与解题过程

1. 由于我们只想要获取类中的两个方法类型，所以我们需要去除`EffectModule`类中的`count`与`message`（或者更多的非`Function`）字段。

```typescript
type PickMethodNames<T> = {[P in keyof T]: T[P] extends Function ? P : never}[keyof T];
```

这里就是对传递的泛型T进行遍历，如果值是继承自Function，则将该值设置为P，否则就是never，然后根据T的键获取值。

传入EffectModule之后类型为`delay | setMessage`
![image](https://pan.udolphin.com/files/image/2022/1/ba00390963369d165c18ecb226f81bd3.png)

2. 我们需要将`EffectModule`里方法的”旧“类型变为“新”类型。

```typescript
type asyncMethod<T, U> = (input: Promise<T>) => Promise<Action<U>>;
type syncMethod<T, U> = (action: Action<T>) => Action<U>;
type connectAsyncMethod<T, U> = (input: T) => Action<U>;
type connectSyncMethod<T, U> = (action: T) => Action<U>;
```

这里为了代码雅观提前声明好了四个类型，前两个`asyncMethod`与`syncMethod`我们称之为旧类型，`connectAsyncMethod`与`connectSyncMethod`两个类型为我们最终需要的类型，我们称之为新类型。

最终题解：

```typescript
type Solution<T, U extends keyof T = PickMethodNames<T>> = 
{ [P in U]: T[P] extends 
  asyncMethod<infer O, infer B> ? connectAsyncMethod<O, B> 
  : T[P] extends syncMethod<infer I, infer W> ? connectSyncMethod<I, W> 
  : never};

type Connect = (module: EffectModule) => Solution<EffectModule>;
```

在这里我们使用了`infer`关键字来推断类型。

首先声明了两个泛型T和U并给U赋初始值PickMethodNames<T>，然后我们去遍历U（其实就是遍历`"delay" | "setMessage"`），接着根据类型判断是否继承自”旧“类型，这里用到了工具方法`infer`来推断并定义“旧“类型两个泛型的类型，并将其赋值给我们的”新“方法，这样就完成了从旧到新的转换，之后的逻辑同理可得，这里就不多赘述了。

得到的类型：

![image](https://pan.udolphin.com/files/image/2022/1/9a908474070663baf6b94a603ed56563.png)

> 注：这里为了得到最终类型将本来放置泛型的地方改为```EffectModule```（Vsode背锅）


### 全部代码如下：

```typescript
interface Action<T> {
  payload?: T;
  type: string;
}

class EffectModule {
  count = 1;
  message = "hello!";

  delay(input: Promise<number>) {
    return input.then(i => ({
      payload: `hello ${i}!`,
      type: 'delay'
    }));
  }

  setMessage(action: Action<Date>) {
    return {
      payload: action.payload!.getMilliseconds(),
      type: "set-message"
    };
  }
}


type asyncMethod<T, U> = (input: Promise<T>) => Promise<Action<U>>;
type syncMethod<T, U> = (action: Action<T>) => Action<U>;
type connectAsyncMethod<T, U> = (input: T) => Action<U>;
type connectSyncMethod<T, U> = (action: T) => Action<U>;

// -----------------------题解-------------------------------

type PickMethodNames<T> = {[P in keyof T]: T[P] extends Function ? P : never}[keyof T];

type Solution<T, U extends keyof T = PickMethodNames<T>> = 
{ [P in U]: T[P] extends asyncMethod<infer O, infer B> ? connectAsyncMethod<O, B> : T[P] extends syncMethod<infer I, infer W> ? connectSyncMethod<I, W> : never};

type Connect = (module: EffectModule) => Solution<EffectModule>;

// -----------------------题解-------------------------------

const connect: Connect = m => ({
  delay: (input: number) => ({
    type: 'delay',
    payload: `hello 2`
  }),
  setMessage: (input: Date) => ({
    type: "set-message",
    payload: input.getMilliseconds()
  })
});

type Connected = {
  delay(input: number): Action<string>;
  setMessage(action: Date): Action<number>;
};

export const connected: Connected = connect(new EffectModule());
```




