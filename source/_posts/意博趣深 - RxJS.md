---
title: 意博趣深 - RxJS
tags:
- RxJS
---

## **认识RxJS**

Reactive Extensions Library for JavaScript

![https://rxjs.dev/generated/images/marketing/home/Rx_Logo-512-512.png](https://rxjs.dev/generated/images/marketing/home/Rx_Logo-512-512.png)

> RxJS提供了一套非常完整的非同步解决方案，让我们在面对各种非同步行为时(无论是Event，AJAX亦或是Animation)，我们都可以使用相同的API来做开发。

在网页的世界存取任何资源都是非同步的，比如说我们希望拿到一个档案，先发送一个请求后，必须等到数据回来，再执行对这个数据的操作，这就是一个非同步的行为。随着网页需求的复杂化，我们所写的JavaScript 就有各种针对非同步行为的写法，例如使用callback 或是Promise 物件甚至是新的语法糖async/await —— 但随着应用需求愈来愈复杂，撰写非同步的程式码仍然非常困难。

### **非同步常见的问题**
- 竞态危害(Race Condition)
- 内存泄漏(Memory Leak)
- 复杂的状态(Complex State)
- 例外处理(Exception Handling)

##### **[Race Condition](https://en.wikipedia.org/wiki/Race_condition)**

每当我们对同一个资源同时做多次的非同步存取时，就可能发生Race Condition 的问题。比如说我们发了一个Request 更新使用者资料，然后我们又立即发送另一个Request 取得使用者资料，这时第一个Request 和第二个Request 先后顺序就会影响到最终接收到的结果不同，这就是Race Condition。

##### **[Memory Leak](https://en.wikipedia.org/wiki/Memory_leak)**

Memory Leak 是最常被大家忽略的一点。原因是在传统网站的行为，我们每次换页都是整页重刷，并重新执行JavaScript，所以不太需要理会记忆体的问题！但是当我们希望将网站做得像应用程式时，这件事就变得很重要。比如说在A 页面监听body 的scroll 事件，但页面切换时，没有把scroll 的监听事件移除。

##### **Complex State**

当有非同步行为时，应用程序的状态就会变得非常复杂！比如说我们有一支付费用户才能播放的影片，首先可能要先抓取这部影片的资讯，接着我们要在播放时去验证使用者是否有权限播放，而使用者也有可能再按下播放后又立即按了取消，而这些都是非同步执行，这时就会有各种复杂的状态需要处理。

##### **Exception Handling**

JavaScript 的try/catch 可以捕捉同步的例外，但非同步的程式就没这么容易，尤其当我们的非同步行为很复杂时，这个问题就愈加明显。

### **各种不同的API**
我们除了要面对非同步会遇到的各种问题外，还需要烦恼很多不同的API。

- DOM Events
- XMLHttpRequest
- Fetch
- WebSockets
- Server Send Events
- Service Worker
- Node Stream
- Timer

上面的API都是非同步的，他们都拥有各自的API与写法！如果我们使用RxJS，上面的API都可以通过RxJS来处理，就可以使用同样的API操作(RxJS的API)。

这里我们举一个例子，假如我们想要监听点击事件，但点击一次之后不再监听。

**原生JavaScript**
```typescript
const handler = (e) => {
  console.log(e);
  documnet.removeEventListener(handler);
}

document.addEventListener('click', handler);
```

**RxJS**
```typescript
fromEvent(document, 'click')
  .pipe(take(1))
  .subscribe(console.log);
```
[stackblitz](https://stackblitz.com/edit/rxjs-3ima52?file=index.ts)

大致上能看得出来我们在使用RxJS后，不管是针对DOM Event还是上面列的各种API我们都可以通过RxJS的API来做操作，像是范例中使用的```take(n)```

### **RxJS可以用于生产吗？**
RxJS由微软在2012年开源，目前各个语言库由ReactiveX组织维护。RxJS在GitHub上已有25.3K的start，目前最新版本为7.2，并且持续开发维护中。
![1344853561-59f30311dfbcc_fix732.png](https://pan.udolphin.com/files/image/2021/8/39b483e61e0786e3b32b5538a4919772.png)

## **RxJS基本介绍**

RxJS 是 Reactive Extensions for JavaScript 的缩写，起源于 Reactive Extensions，是一个基于可观测数据流 Stream 结合观察者模式和迭代器模式的一种异步编程的应用库。

RxJS 中解决异步事件管理的基本概念是：
- Observable：表示未来值或事件的可调用集合的想法，是多个值的惰性推送集合。
- Observer：是一个回调集合，使用它可以监听 Observable 传递的值。
- Subscription：表示一个 Observable 的执行，主要用于取消订阅。
- Operator：是纯粹的方法，可以处理和操作Observable传入的值，如集合功能的编程风格map，filter，concat，reduce等。
- Subject：相当于一个 EventEmitter，是将一个值或事件多播给多个 Observer 的方式。
- Scheduler：是集中式的调度员控制并发性，使我们能够在计算例如发生在setTimeout或requestAnimationFrame中的事件。

### **Functional Reactive Programming**

Functional Reactive Programming是一种编程范式，涵盖了Functional Programming与Reactive Programming两种编程思想。

#### **Functional Programming**
简单说Functional Programming核心思想就是做运算处理，并且使用function来思考问题，例如像以下的算术运算：
```
(5 + 6) - 1 * 3
```
我们可以写成：
```typescript
const add = (a, b) => a + b
const mul = (a, b) => a * b
const sub = (a, b) => a - b

sub(add(5, 6), mul(1, 3))
```
我们把每个运算包成一个个不同的function，并用这些function组合出我们要的结果，这就是最简单的Functional Programming。

#### **Reactive Programming**
> 很多人一谈到Reactive Programming 就会直接联想到是在讲RxJS，但实际上Reactive Programming 仍是一种编程范式，在不同的场景都有机会遇到，而非只存在于RxJS，尤雨溪(Vue 的作者)就曾在twitter 对此表达不满！

![响应式编程](https://res.cloudinary.com/dohtkyi84/image/upload/v1480531809/%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7_2016-11-30_%E4%B8%8A%E5%8D%886.23.49_zdniva.png)

Reactive Programming 简单来说就是当变量或资源发生变动时，有变量或资源自动告诉我发生变动了。

举个例子：
当我们在使用vue 开发时，只要一有绑定的变量发生改变，相关的变量及页面也会跟着变动，而开发者不需要写这其中的任何一行代码。

Rx基本上就是上述两个观念的结合，这个部分相信读者在看完之后的章节，会有更深的感悟。

## **Observable**

> 整个RxJS 的基础就是Observable，只要弄懂Observable 就算是学会一半的RxJS 了，剩下的就只是一些方法的练习跟熟悉，但到底什么是Observable呢？

实际上，RxJS核心的Observable操作观念和FP的数组操作是极为相似的，只要学会以下几个基本的方法跟观念后，会让我们之后上手Observable简单很多！

### **观察者模式(Observer Pattern)**

定义：定义了对象之间的依赖关系.可以通知所有依赖于它的对象(前端更多的是通知事件执行)。

观察者模式其实很常用到，在许多API的设计上都用了观察者模式，最简单的例子就是DOM事件监听。

```typescript
function clickHandler(event) {
	console.log('user click!');
}

document.body.addEventListener('click', clickHandler)
```

在上面的代码中，我们先声明了一个```clickHandler```函数，再用DOM来监听点击事件，每次使用者在body上点击一下就会执行一次```clickHandler```。这就是观察者模式，我们可以对某件事注册监听，并在事件发生时，自动执行我们注册的监听者。

Observer的观念其实就是这么简单，下面我们来通过代码看看如何实作一个这样的Pattern。

首先我们需要一个类，这个类new出来的实例可以被监听。
```typescript
class Subject {
  private arrList: { [fn: string]: Function } = {};
  constructor() {}

  register(name: string, fn: Function) {
    this.arrList[name] = fn;
  }

  notify(name?: string, ...parameters: any[]) {
    if (name) {
      if (!this.arrList[name]) {
        throw new Error(`${name}方法不存在`);
      }
      this.arrList[name!]();
    } else {
      Object.values(this.arrList).forEach((fn: Function) => {
        fn.apply(fn, parameters);
      });
    }
  }

  remove(name: string) {
    delete this.arrList[name];
  }
}
```

有了上面的代码后，我们就可以来建立实例并使用了。

```typescript
const subject = new Subject();

subject.register('boil', () => {
  console.log('烧水');
});

subject.register('coding', () => {
  console.log('写代码');
});

const share = () => {
  console.log('分享');
  subject.notify('coding');
};

subject.remove('boil');

share();
```

当我们执行到这里时，会打印出：
```
分享
写代码
```
完整代码 [stackblitz](https://stackblitz.com/edit/typescript-spnmef?file=index.ts)

我们可以在程序执行时去通知注册过的方法，在执行```share```方法时，```coding```也会执行，而这些方法可以被额外添加，也可以被移除。

虽然我们的实例很简单，但它很好地说明了观察者模式如何在事件和监听者的互动中做到去耦合。

### **迭代器模式(Iterator Pattern)**

迭代器模式分为内部迭代器与外部迭代器，它像一个指针，指向一个数据结构并产生一个数列，这个数列会有结构中的所有元素。

#### **内部迭代器**

内部迭代器的内部已经定义好了迭代规则，它完全接手整个迭代过程，外部只需要一次初始调用。

```typescript
const arr = ['Angular', 'React', 'Vue'];

for (let i of arr) {
  console.log(i);
}
```
[stackblitz](https://stackblitz.com/edit/typescript-qqe5bm?file=index.ts)

优点：调用方式简单，外部仅需一次调用。

缺点：迭代规则预先设置，欠缺灵活性。无法实现复杂遍历需求（如: 同时迭代比对两个数组）。

#### **外部迭代器**

外部迭代器： 外部显示（手动）地控制迭代下一个数据项。

借助ES6新增的iterator接口来实现外部迭代器。
```typescript
var arr = [1, 2, 3];

var iterator = arr[Symbol.iterator]();

iterator.next();
// { value: 1, done: false }
iterator.next();
// { value: 2, done: false }
iterator.next();
// { value: 3, done: false }
iterator.next();
// { value: undefined, done: true }
```
[stackblitz](https://stackblitz.com/edit/rxjs-c5pnau?file=index.ts)

优点：灵活性更佳，适用面广，能应对更加复杂的迭代需求。

缺点：需显示调用迭代进行（手动控制迭代过程），外部调用方式较复杂。

#### 思考
这一段代码实现的是外部还是内部迭代器？
> 欢迎在留言中一起讨论
```typescript
const arr = ['Angular', 'React', 'Vue'];
function* generatorEach() {
  const entries = arr.entries();
  for (let [index, value] of arr.entries()) {
    yield console.log([index, value]);
  }
}

const each = generatorEach();
each.next();
each.next();
each.next();
```
[stackblitz](https://stackblitz.com/edit/rxjs-w11mw1?file=index.ts)

### **Observable**
在了解观察者和迭代器模式之后，不知道大家有没有发现其实这两种设计模式有个共同的特性，就是他们都是渐进式的取得资料，差别只在于Observer是生产者推送资料(push)而Iterator是消费者要求资料(pull)。

![push and pull](https://res.cloudinary.com/dohtkyi84/image/upload/v1482240798/push_pull.png)

Observable其实就是这两种设计模式思想的结合，Observable具有生产者推送资料的特性，同时能像数列，拥有数列推送资料的方法。

### **建立Observable(一)**

建立Observable的方法有非常多种，其中直接new出一个实例是最基本的方法。
Observable类接受一个callback function，这个callback function会接受一个observer参数。

```typescript
const observable = new Observable(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
});
```
这个callback function会决定observable将如何发送值。

我们可以订阅这个observable，来接受他送出的值。
```typescript
const subscription = observable.subscribe(
  next => {
    console.log('next', next);
  }
);
```
当我们订阅之后他就会依序送出```next 1``` ```next 2``` ```next 3```

此处有两个重点
1. Observable只有当订阅(subscribe)之后才会开始推送值。
2. 虽然RxJS主要在处理非同步行为，但也同时可以处理同步行为，像上面的代码就是同步执行的。证明如下

```typescript
import { Observable } from 'rxjs';

const observable = new Observable(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
});

console.log('just before subscribe');
const subscription = observable.subscribe(
  next => {
    console.log('next', next);
  },
);
console.log('just after subscribe');
```
[stackblitz](https://stackblitz.com/edit/rxjs-ndcz8g?file=index.ts)

当我们订阅之后他就会依序送出```just before subscribe``` ```next 1``` ```next 2``` ```next 3``` ```just after subscribe```

### **Observer(观察者)**

Observable可以被订阅，或者说可以被观察，而订阅Observable的对象又称为观察者(Observer)，观察者是一个具有三个方法的对象，每当Observable发送事件时，便会呼叫观察者相对应的方法。

观察者的三个方法：
- next: 每当Observable发送出新的值，next方法就会被执行。
- complete: 在Observable没有其他的值可以取得时，complete方法就会被执行，在complete执行之后，next方法就不会再起作用。
- error: 每当Observable内部发生错误时，error方法就会被执行，类似与Promise的catch方法。

```typescript
import { of, map, Observable } from 'rxjs';

const source = new Observable(observer => {
  observer.next('1');
  observer.next('2');
  observer.complete();
  observer.next('not work');
});

var observer = {
  next: function(value) {
    console.log(value);
  },
  error: function(error) {
    console.log(error);
  },
  complete: function() {
    console.log('complete');
  }
};

source.subscribe(observer);
```
[stackblitz](https://stackblitz.com/edit/rxjs-8r4n8a?file=index.ts)

上述代码会依次打印出```1``` ```2``` ```complete```

从上面的代码我们可以看出在complete执行后，next方法就会自动失效，所以没有印出```not work```

下面是发生错误的示例

```typescript
import { of, map, Observable } from 'rxjs';

const source = new Observable(observer => {
  observer.next('1');
  observer.next('2');
  throw 'some exception';
  observer.complete();
  observer.next('not work');
});

var observer = {
  next: function(value) {
    console.log(value);
  },
  error: function(error) {
    console.log(error);
  },
  complete: function() {
    console.log('complete');
  }
};

source.subscribe(observer);

```
[stackblitz](https://stackblitz.com/edit/rxjs-8r4n8a?file=index.ts)

这里会执行error的方法印出some exception。

另外观察者可以使不完整的，他可以只具有一个next方法。

我们也可以直接把next，error，complete三个方法依序传入```observable.subscribe```

```typescript
observable.subscribe(
    value => { console.log(value); },
    error => { console.log('Error: ', error); },
    () => { console.log('complete') }
)
```
```observable.subscribe```会在内部自动组成observer来进行操作。


### **Marble diagrams(大理石图)**
我们在传答事务时，文字其实是最糟糕的手段，虽然文字使我们平时沟通的基础，但常常千言万语都必不过一张清楚的图。如果我们能制定observable的图示，就能让我们更方便地理解observable的各种operators！

我们将observable的图示称为Marble diagrams，在网络上RxJS有非常多的Marble diagrams，规则基本上大致相同。

我们用```-```来表示一小段时间，这些```-```穿起来就表示一个observable

```
--------------------------
```

```X```(大写X)则表示有错误发生。

```
--------------------------X
```

```|```则表示observable结束。

```
--------------------------|
```

在这个时间序列中，我们可能会送出值，如果是数字我们直接用阿拉伯数字，其他类型我们用相近的英文符号代替，这里我们用```interval```举例

```
-----0-----1-----2-----3--...
```

当observable是同步推送值的时候，例如

```typescript
const source = of(1,2,3,4);
```

```source```的图形就会像这样。
```
(1234)|
```
小括号代表着同步发生。

另外Marble diagrams也可以表达operator的前后转换，例如
```typescript
const source = interval(1000);
const newest = source.pipe(map(x => x + 1));
```
这时的Marble diagrams就会像这样
```
source: -----0-----1-----2-----3--...
            map(x => x + 1)
newest: -----1-----2-----3-----4--...
```
最上面是原本的observable，中间是operator，下面则是新的observable。

> Marble diagrams相关资源：https://rxmarbles.com/

以上就是Marble diagrams如何表示operator对observable的操作，下一章让我们来看几个简单的operator。

### **建立Observable(二)**

> 通常我们并不会直接在项目中使用new Observable来建立Observable实例，而是会使用creation operator来建立Observable实例，下面我们会介绍几个常用到的operator。

#### **Creation Operator**

creation operator可以作为独立函数调用以创建新的 Observable。

下面我们列出RxJS常用的creation operator
- of 
- fromEvent
- interval
- timer

##### **of**

创建一个 observable，它将一个接一个地发出值。

> 每个参数都想成为一个next通知。
```typescript
import { of } from 'rxjs';

of(10, 20, 30)
.subscribe(
  next => console.log('next:', next),
  err => console.log('error:', err),
  () => console.log('the end'),
);

// Outputs
// next: 10
// next: 20
// next: 30
// the end
```

Marble diagrams像是这样。

```
(102030)|
```

##### **fromEvent**

创建一个 Observable，它发出来自给定事件目标的特定类型的事件，行为类似与原生JS的addEventListener方法。

```typescript
import { fromEvent } from 'rxjs';

const clicks = fromEvent(document, 'click');
clicks.subscribe(x => console.log(x));
```
这里我们给Document对象注册了点击事件，在点击页面后会打印出MouseEvent对象。

Marble diagrams像是这样。

```
----------ev-----ev
```

###### 思考

这里如果没有订阅(subscribe)，事件会被绑定到Document对象上吗？

##### **interval**

创建一个Observable，每个时间间隔发出一个序列号，行为类似与原生JS的setInterval。

```typescript
import { interval } from 'rxjs';
import { take } from 'rxjs/operators';

const numbers = interval(1000);

numbers.subscribe(x => console.log('Next: ', x));
```

这里会从0开始每隔一秒输出一次，每次递增1。

Marble diagrams像是这样。

```
-----0-----1-----2--..
```

##### **timer**

当timer有两个参数时，第一个参数代表要发出第一个值的等待时间(ms)，第二个参数代表第一次之后发送值的间隔时间。

```typescript
import { timer, interval } from 'rxjs';

timer(1000, 3000).subscribe(n => console.log('timer', n));
```
上述代码会在一秒后输出0，之后每隔三秒输出一次，每次递增1。

Marble diagrams像是这样。

```
-----0---------------1---------------2-----...
```

## **Subscription**

不知道大家有没有发现，我们在上一章介绍操作符的时候，有很多无穷的observable，例如interval与timer，但是有的时候我们可能会在某些行为后不再需要这些资源，要做到这件事最简单的方式就是```unsubscribe```。

其实在订阅observable之后，会回传一个subscription对象，这个方法具有释放资源的```unsubscribe```方法。

```typescript
import { interval } from 'rxjs';

const observable = interval(1000);
const subscription = observable.subscribe(x => console.log(x));
setTimeout(() => {
  subscription.unsubscribe();
}, 5000);
```
[stackblitz](https://stackblitz.com/edit/rxjs-qkid1f?file=index.ts)

这里我们用了```setTimeout```在5秒后执行了```subscription.unsubscribe()```来停止订阅并且释放资源。

## **Operator(操作符)**

> 尽管 Observable 是基础，但RxJS 最有用的却是它的操作符。操作符是允许以声明方式轻松组合复杂异步代码的基本部分。

### **什么是Operator？**

Operator就是**函数**。目前按照大类来分有两种运算符。
1. **Creation Operator**：是可以作为独立函数调用以创建新的Observable的一种操作符，也是我们在前些章节介绍过的。
2. **Pipeable Operator**：是另一种可以使用语法通过管道(pipe)传输到Observables的操作符。在调用时，它们不会更改现有的Observable实例，它们会返回一个新的Observable，其订阅逻辑基于第一个Observable。

> Pipeable Operator 是一个将 Observable 作为其输入并返回另一个 Observable 的函数。这是一个纯粹的操作：之前的 Observable 保持不变。

Pipeable Operator本质上是一个纯函数，它将一个Observable作为输入并生成另一个Observable作为输出。订阅输出Observable也会订阅输入Observable。

这也就是为什么说RxJS是Functional Programming的依据之一。

由于我们在前些章节介绍过creation operator，在这章我们主要介绍Pipeable operator。

让我们从两个实例开始认识管道类操作符。

### **防抖**

用到的操作符
1. debounceTime: 仅在经过特定时间跨度且没有其他源发射后才从源 Observable 发出通知。

   ```debounceTime```的Marble diagrams像是这样
   ```
   source: -----1-----2-3-45--------3--...
            debounceTime(100)
   newest: -----1----------5--------3--...
   ```   

2. map: 将给定的函数应用于源Observable发出的每个值，并将结果值作为 Observable 发出，效果用法等同于ES6的数组的map方法。

   ```map```的Marble diagrams像是这样
   ```
   source: -----0-----1-----2-----3--...
               map(x => x + 1)
   newest: -----1-----2-----3-----4--...
   ```   
   
##### 第一步，我们获取到所需的dom元素。

HTML:
```html
<input type='text' id='input'>
```

TypeScript:
```typescript
  const input = document.querySelector('#input');
```

##### 第二步，使用之前说过的fromEvent为input元素绑定input事件。

```typescript
import { of, fromEvent } from 'rxjs';
import { debounceTime, distinctUntilChanged, map } from 'rxjs/operators';

const input = document.querySelector('#input');

fromEvent(input, 'input')
  .subscribe(console.log);
```
效果图
![RxJS 防抖动态图](https://pan.udolphin.com/files/image/2021/8/e054cc33fbcd72b580078f8430c6dd2e.gif)

这时我们输入值的话可以看到每一次输入都会打印一次InputEvent，如果想要做到在每次输入之后等待100ms再打印值也很简单，只需要加入```debounceTime(100)```。

##### 第三步，这里我们使用```map```获取输入的值，然后使用```debounceTime```操作符完成基础的防抖功能。
```typescript
import { of, fromEvent } from 'rxjs';
import { debounceTime, distinctUntilChanged, map } from 'rxjs/operators';

const input = document.querySelector('#input');

fromEvent(input, 'input')
  .pipe(
    debounceTime(100),
    map((x: InputEvent) => (x.target as HTMLInputElement).value)
  )
  .subscribe(console.log);
```

效果图
![防抖debounceTime.gif](https://pan.udolphin.com/files/image/2021/8/b7b19b99f70369e0fcb1fe4d7b5c3226.gif)

此时我们可以看到在已经具有防抖的效果了。

完整代码 [stackblitz](https://stackblitz.com/edit/rxjs-qoupke?file=index.ts)

### **拖拽**

用到的操作符
1. switchMap: 高阶操作符之一，可以根据传入的Observable重新生成一个新的Observable并且返回。

    例
    ```typescript
    const source = fromEvent(document.body, 'click');
    
    const example = source.pipe(switchMap(e => interval(1000)));
    
    example.subscribe({
      next: value => {
        console.log(value);
      },
      error: err => {
        console.log('Error: ' + err);
      },
      complete: () => {
        console.log('complete');
      }
    });
    ```
    在我们点击body的时候会将点击事件的事件对象装换为interval并且使用switchMap将interval转换为一阶observable(就是被订阅过的)。

    此时的Marble Diagram像是这样。
   
    ```
      source : -----------ev-------------------...
            switchMap(c => interval(1000))
      example: ---------------0--1--2----------...
    ```
2. takeUntil: 在实际场景中takeUntil经常用到，他可以在某件事发生时让一个observable直接触发complete方法。

    例
    ```typescript
    const example = interval(1000).pipe(takeUntil(click));
    const click = fromEvent(document.body, 'click');
    
    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    ```
    这里我们为body元素注册了点击事件，随后我们创建了一个无限发送值的observable - interval并且为其绑定了takeUntil为点击事件。这样在我们点击body的时候就会调用interval的complete方法来取消interval。

    此时的Marble Diagram像是这样。

    ```
    source : -----0-----1-----2------3--
    click  : ----------------------c----
    takeUntil(click)
    example: -----0-----1-----2----|
    ```
3. withLatestFrom: 接受两个参数，第一个为另一个observable对象，第二个是一个callback function，可以将两个observable合并，并且在主要的observable送出新的值时才会执行callback。

    例
    ```typescript
      const clicks = fromEvent(document, 'click');
      const timer = interval(1000);
      const result = clicks.pipe(
        withLatestFrom(timer, (event, time) => {
          return { event, time };
        })
      );
      result.subscribe(x => console.log(x));
    ```
    这里我们为document对象注册了点击事件并且创建了interval，随后我们使用```withLatestFrom```操作符将两个observable合并，这样在主observable(click事件)发生的时候会执行```withLatestFrom```的回调函数。

    此时的Marble Diagram像是这样。

    ```
     timer : -----0------1-------2--------3--|
     clicks  : -------ev---------------ev---|
                    withLatestFrom
     example: --------ev0--------------ev2--|
    ```
##### **需求分析**

1. 首先页面上有一個元素(main)。
2. 当鼠标在元素(#main)上按下左鍵(mousedown)时，开始监听鼠标移动(mousemove)事件，获取位置。
3. 当鼠标左键放开(mouseup)时，结束监听鼠标移动事件。
4. 当鼠标移动(mousemove)被监听时，跟着修改元件的样式属性。

##### 第一步，取得会用到的dom元素。
HTML
```html
<style>
  .main {
    width: 100px;
    height: 100px;
    text-align: center;
    line-height: 100px;
    background-color: deepskyblue;
  }
</style>

<div class="main" id="main">
  main
</div>

```
TypeScript
```typescript
const main: HTMLElement = document.querySelector('.main');
```

##### 第二步，建立会用到的observable
```typescript
const mouseUp = fromEvent(document, 'mouseup');
const mouseDown = fromEvent(main, 'mousedown');
const mouseMove = fromEvent(document, 'mousemove');
```

##### 第三步，撰写代码逻辑

首先我们需要当鼠标在元素(#main)上按下左鍵(mousedown)时，开始监听鼠标移动(mousemove)事件，并且在鼠标抬起(mouseup)的时候取消对鼠标移动事件的监听。

```typescript
mouseDown
  .pipe(
    switchMap(_ => mouseMove.pipe(takeUntil(mouseUp))),
  )
  .subscribe(x => {
    console.log(x);
  });
```

此时我们已经做到了上述的需求了，接下来我们只需要获取鼠标点下时候的位置并且计算就可以得到鼠标的位置信息。
```typescript
mouseDown
  .pipe(
    switchMap(_ => mouseMove.pipe(takeUntil(mouseUp))),
    // map(_ => mouseMove.pipe(takeUntil(mouseUp))),
    // concatAll(),
    withLatestFrom(mouseDown, (move: MouseEvent, down: MouseEvent) => {
      return {
        x: move.pageX - down.offsetX,
        y: move.pageY - down.offsetY
      };
    })
  )
  .subscribe(x => {
    console.log(x);
  });
```
这里我们使用```withLatestFrom```操作符来获取鼠标按下时的事件对象，并且计算得到鼠标的坐标点(x , y)，并将之传递给subscribe方法。

最后我们改变div的样式，完成拖拽效果。
```typescript
subscribe(x => {
    main.style.transform = `translate(${x.x}px, ${x.y}px)`;
  });
```

效果图
![RxJS拖拽](https://pan.udolphin.com/files/image/2021/8/51441ad26da404e2f7f0637033a9e22f.gif)

完整代码 [stackblitz](https://stackblitz.com/edit/rxjs-xbyk73?file=index.ts)

### **skip**
```typescript
import { of, map, interval } from 'rxjs';
import { skip } from 'rxjs/operators';

const source = interval(1000).pipe(skip(2));
source.subscribe(console.log);
```
[stackblitz](https://stackblitz.com/edit/rxjs-nnzxot?file=index.ts)

原本从0开始的打印会变为从3开始，但是记得原本需要等待的时间仍然存在。

此时的Marble Diagram是这样的。

```
-----0-----1-----2--...
     skip(2)
-----------------2--...
```

### **takeLast**

```takeLast(n)```的作用是取最后的n个值。

```typescript
import { of, map } from 'rxjs';
import { takeLast } from 'rxjs/operators';

const source = of(1, 2, 3, 4, 5).pipe(takeLast(1));

source.subscribe(console.log);
```
[stackblitz](https://stackblitz.com/edit/rxjs-qft3ma?file=index.ts)

这里我们使用```of```一共发送了5个元素随后使用```takeLast(1)```获取最后一个发送的值，这里有一个重点，就是takeLast必须等到整个observable完成(complete)，才可以知道最后的元素有哪一些，并且同步送出。

此时的Marble Diagram是这样的。

```
(12345)|
takeLast(1)
-----5|
```

### **last**

last等效于```takeLast(1)```,是```takeLast(1)```的简化写法。

### **concat**

```concat```可以将多个observable实例合并为一个，范例如下
```typescript
import { interval, of } from 'rxjs';
import { concat } from 'rxjs/operators';

const source = interval(1000).pipe(take(3));
const source2 = of(3);
const source3 = of(4, 5, 6);
const example = source.pipe(concat(source2, source3));

example.subscribe({
  next: value => {
    console.log(value);
  },
  error: err => {
    console.log('Error: ' + err);
  },
  complete: () => {
    console.log('complete');
  }
});
```
[stackblitz](https://stackblitz.com/edit/rxjs-rlofht?file=index.ts)

```concat```必须要合并的操作符按顺序执行，前一个执行完毕后才会执行下一个。


此时的Marble Diagram
```
source : ----0----1----2|
source2: (3)|
source3: (456)|
            concat()
example: ----0----1----2(3456)|
```

### **merge**

```merge```和```concat```一样都是用来合并observable，但是他们的行为却完全不同！

让我们直接来看例子吧。

```typescript
import { of, map, interval } from 'rxjs';
import { merge } from 'rxjs/operators';

const source = interval(500);
const source2 = interval(300);
const example = source.pipe(merge(source2));

example.subscribe({
   next: value => {
      console.log(value);
   },
   error: err => {
      console.log('Error: ' + err);
   },
   complete: () => {
      console.log('complete');
   }
});
```
[stackblitz](https://stackblitz.com/edit/rxjs-6ayscx?file=index.ts)

从下面的大理石图可以看到merge后的example在时间的顺序上同时在跑 source 与 source2，当两件事同时发生时，会同步的送出值，当两个observable全部结束才会真的结束。

```
source : ----0----1----2|
source2: --0--1--2--3--4--5|
         merge()
example: --0-01--21-3--(24)--5|
```
## **Subject**

> 终于进入了RxJS的第二个重点Subject，不知道读者们有没有发现？我们在这篇文章之前的范例，每个observable都只订阅了一次，而实际上observable是可以多次订阅的。

### **Multiple subscriptions**

```typescript
const observable = interval(1000);
observable.subscribe(x => {
  console.log('A:', x);
});

observable.subscribe(x => {
  console.log('B:', x);
});
```
输出
```
"A next: 0"
"B next: 0"
"A next: 1"
"B next: 1"
"A next: 2"...
"B next: 2"...
```

上面的这段代码，分别用observerA和observerB订阅了source，从log我们可以看出observerA和observerB都各自收到了元素，但请记住两个observer其实是**分开执行**的，也就是说他们是完全独立的，我们将observerB延迟订阅来证明看看。

```typescript
const observable = interval(1000);
observable.subscribe(x => {
  console.log('A:', x);
});

setTimeout(() => {
  observable.subscribe(x => {
    console.log('B:', x);
  });
}, 1000);
```

这里我们延迟一秒再用observerB 订阅，可以从log 中看出1 秒后observerA 已经印到了1，这时observerB 开始印却是从0 开始，而不是接着observerA 的进度，代表这两次的订阅是完全分开来执行的，或者说是每次的订阅都建立了一个新的执行。

这样的行为在大部分的情境下适用，但有些案例下我们会希望第二次订阅source 不会从头开始接收元素，而是从第一次订阅到当前处理的元素开始发送，我们把这种处理方式称为多播(multicast)。那我们该怎样实现多播呢？

### **什么是Subject？**

RxJS Subject 是一种特殊类型的 Observable，它允许将值多播到许多观察者。虽然普通的 Observable 是单播的(每个订阅的 Observer 拥有一个独立的 Observable 执行)，但Subject是多播的。

每个 Subject 都是一个 Observable。给定一个Subject，你可以订阅(subscribe)它，提供一个观察者，它将开始正常接收值。从观察者(Observer)的角度来看，它无法判断 Observable 的执行是来自一个普通的单播 Observable 还是一个 Subject。

每个Subject都是Observer。它拥有Observer的三个方法next(v)，error(e)和complete()。要向 Subject 提供一个新值，只需调用next(theValue)，它将被多播到注册的 Observers 以侦听 Subject。

在下面的例子中，我们有两个Observer附加到一个Subject，并且在一秒后才注册observerB：
```typescript
import { Subject } from 'rxjs';

const subject = new Subject<number>();

subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`)
});

setTimeout(() => {
  subject.next(2);
  subject.subscribe({
    next: (v) => console.log(`observerB: ${v}`)
  });
}, 1000);

subject.next(1);
```
[stackblitz](https://stackblitz.com/edit/rxjs-pjznd8?file=index.ts)

输出
```typescript
observerA: 1
observerA: 2
observerB: 2
```
通过上述方法，我们通过Subject将单播Observable执行转换为多播。

还有的几个特例Subject类型：BehaviorSubject，ReplaySubject，和AsyncSubject。

### **BehaviorSubject**

BehaviorSubject 跟Subject 最大的不同就是BehaviorSubject 是用来呈现当前的值，而不是单纯的发送事件。BehaviorSubject 会记住最新一次发送的元素，并把该元素当作目前的值，在使用上BehaviorSubject 建构式需要传入一个参数来代表起始的状态，范例如下

```typescript
import { BehaviorSubject } from 'rxjs';

var subject = new BehaviorSubject(0);
var observerA = {
  next: value => console.log('A next: ' + value),
  error: error => console.log('A error: ' + error),
  complete: () => console.log('A complete!')
};

var observerB = {
  next: value => console.log('B next: ' + value),
  error: error => console.log('B error: ' + error),
  complete: () => console.log('B complete!')
};

subject.subscribe(observerA);
// "A next: 0"
subject.next(1);
// "A next: 1"
subject.next(2);
// "A next: 2"
subject.next(3);
// "A next: 3"

setTimeout(() => {
  subject.subscribe(observerB);
  // "B next: 3"
}, 3000);
```
[stackblitz](https://stackblitz.com/edit/rxjs-pjznd8?file=index.ts)

从上面这个范例可以看得出来BehaviorSubject 在建立时就需要给定一个状态，并在之后任何一次订阅，就会先送出最新的状态。其实这种行为就是一种状态的表达而非单纯的事件，就像是年龄跟生日一样，年龄是一种状态而生日就是事件；所以当我们想要用一个stream 来表达年龄时，就应该用BehaviorSubject。

### **ReplaySubject**

在某些时候我们会希望Subject 代表事件，但又能在新订阅时重新发送最后的几个元素，这时我们就可以用ReplaySubject，范例如下
```typescript
import { ReplaySubject } from 'rxjs';

var subject = new ReplaySubject(2);
var observerA = {
  next: value => console.log('A next: ' + value),
  error: error => console.log('A error: ' + error),
  complete: () => console.log('A complete!')
};

var observerB = {
  next: value => console.log('B next: ' + value),
  error: error => console.log('B error: ' + error),
  complete: () => console.log('B complete!')
};

subject.subscribe(observerA);
subject.next(1);
// "A next: 1"
subject.next(2);
// "A next: 2"
subject.next(3);
// "A next: 3"

setTimeout(() => {
  subject.subscribe(observerB);
  // "B next: 2"
  // "B next: 3"
}, 3000);
```
[stackblitz](https://stackblitz.com/edit/rxjs-fxywmh?file=index.ts)

### **AsyncSubject**
AsyncSubject 会在subject 结束后才送出最后一个值，其实这个行为跟Promise 很像，都是等到事情结束后送出一个值，但实践中我们非常非常少用到AsyncSubject，绝大部分的时候都是使用BehaviorSubject 跟ReplaySubject 或Subject。

```typescript
import { AsyncSubject } from 'rxjs';

var subject = new AsyncSubject();
var observerA = {
  next: value => console.log('A next: ' + value),
  error: error => console.log('A error: ' + error),
  complete: () => console.log('A complete!')
};

var observerB = {
  next: value => console.log('B next: ' + value),
  error: error => console.log('B error: ' + error),
  complete: () => console.log('B complete!')
};

subject.subscribe(observerA);
subject.next(1);
subject.next(2);
subject.next(3);
subject.complete();
// "A next: 3"
// "A complete!"

setTimeout(() => {
  subject.subscribe(observerB);
  // "B next: 3"
  // "B complete!"
}, 3000);
```
[stackblitz](https://stackblitz.com/edit/rxjs-b8hueg?file=index.ts)

## **冷与热的Observable**

在上一章我们介绍了各种Subject，不晓得各位读者还记不记得一开始讲到Subject时，是希望能够让Observable在有新订阅的时候，可以共用前一个订阅的执行而不是从头开始。

### **冷的Observable**

> 还记得下面的例子吗？
```typescript
import { AsyncSubject, interval } from 'rxjs';

var source = interval(1000);
var observerA = {
  next: value => console.log('A next: ' + value),
  error: error => console.log('A error: ' + error),
  complete: () => console.log('A complete!')
};

var observerB = {
  next: value => console.log('B next: ' + value),
  error: error => console.log('B error: ' + error),
  complete: () => console.log('B complete!')
};

source.subscribe(observerA);

setTimeout(() => {
  source.subscribe(observerB);
}, 1000);
```
上面这段代码我们分别用两个Observer去订阅同一个Observable，在observerA已经输出了```A next: 0```与```A next: 1```的时候observerB会输出```B next: 0```，这就是单播也叫做**冷的Observable**，这两个序列完全是按照各自的节奏走的，不同步。每个流在订阅的时候都会从0开始。

那么我们如何让使其变为多播的Observable呢？

### **热的Observable**
热的Observable：多播。所有的观察者，无论进来的早还是晚，看到的是同样内容的同样进度，订阅的时候得到的都是最新时刻发送的值。

在RxJS中想让Observable多播很简单，只需要加入```multicast```操作符(Operator)。

#### **multicast**
multicast可以用来挂载subject并回传一个可连接(connectable)的observable，如下

```typescript
import { Subject, AsyncSubject, interval } from 'rxjs';
import { multicast } from 'rxjs/operators';

var source = interval(1000).pipe(multicast(new Subject()));

var observerA = {
  next: value => console.log('A next: ' + value),
  error: error => console.log('A error: ' + error),
  complete: () => console.log('A complete!')
};

var observerB = {
  next: value => console.log('B next: ' + value),
  error: error => console.log('B error: ' + error),
  complete: () => console.log('B complete!')
};

source.connect();

source.subscribe(observerA);

setTimeout(() => {
  source.subscribe(observerB);
}, 1000);
```

上面这段代码会在observerA输出```A next: 0```与```A next: 1```的时候observerB会输出```B next: 1```，有没有发现不一样的地方，在这里两个observer才是真正同步的执行了。

上面这段代码我们通过multicast来挂载一个subject之后这个observable(上面代码中的source)的订阅其实都是订阅到subject上面。

并且必须要等到执行```connect()```之后才会整的用subject订阅source，并开始送出元素，如果没有执行```connect()```，observable是不会真正执行的。

另外值得注意的是如果这里需要取消订阅的话，需要将```connect()```回传的subscription取消订阅才会真正停止observable的执行。

```typescript
const subscription = source.connect();
subscription.unsubscribe();
```

虽然用了```multicast```感觉会让我们处理的对象少一些，但是必须搭配connect一起使用还是会使程序有一些复杂，通常我们会希望有observer订阅的时候，就立即执行并发送元素，而不要再多去执行一个方法(connect)，这个时候我们就可以用到```refCount```。

#### **refCount**

refCount 必须搭配multicast 一起使用，他可以建立一个只要有订阅就会自动connect 的observable，范例如下

```typescript
import { Subject, AsyncSubject, interval } from 'rxjs';
import { multicast, refCount } from 'rxjs/operators';

var source = interval(1000).pipe(
  multicast(new Subject()),
  refCount()
);

var observerA = {
  next: value => console.log('A next: ' + value),
  error: error => console.log('A error: ' + error),
  complete: () => console.log('A complete!')
};

var observerB = {
  next: value => console.log('B next: ' + value),
  error: error => console.log('B error: ' + error),
  complete: () => console.log('B complete!')
};

source.subscribe(observerA);

setTimeout(() => {
  source.subscribe(observerB);
}, 1000);

```

上面这段程式码，当source 一被observerA 订阅时(订阅数从0 变成1)，就会立即执行并发送元素，我们就不需要再额外执行connect。

同样的在退订时只要订阅数变成0 就会自动停止发送

#### **publish**
其实```multicast(new Subject())``` 很常用到，我们有一个简化的写法那就是publish，下面这两段代码完全是等价的。
```typescript
const source1 = interval(1000)
  .pipe(
    publish(),
    refCount(),
  )

const source2 = interval(1000)
  .pipe(
    multicast(new Subject()),
    refCount(),
  )
```

加上Subject的三种变形：

```typescript
const source1 = interval(1000)
  .pipe(
     publishReplay(1),
     refCount(),
  )
```
```typescript
const source1 = interval(1000)
  .pipe(
    publishBehavior(0) ,
    refCount(),
  )
```
```typescript
const source1 = interval(1000)
  .pipe(
     publishLast(),
     refCount(),
  )
```

#### **share**
另外publish + refCount可以继续简写为share
```typescript
const source1 = interval(1000)
  .pipe(
    share()
  )
```

## **总结**

RxJS是一个强大的Reactive编程库，提供了强大的数据流组合与分支控制能力，但是其学习门槛一直很高，本文介绍了RxJS的一些概念与实例，梳理了观察者模式和迭代器模式与RxJS之间的关系，把RxJS的一个核心两个重点(Observable + Observer + Subject)以及一些operators也有写到，期望可以让各位读者对RxJS有一个大致的了解，简化学习的过程。

感谢您的阅读，我是数字办的鲁举佩，期待与您共同成长！！！
