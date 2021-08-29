# Vue 3.0 响应式系统

## 一、前言

我记得去年 9 月份的时候，我还在艰难地入门 Vue 2.6 的源码，结果没多久，Vue 3.0 发布了，，，泪目，学不完了。

后来把 2.6 源码梳理一遍后，又找时间去看 Vue 3.0 的源码，发现 3.0 虽然做了许多优化，但是整体的设计思想还是与 2.6 一致的，所以 Vue 3.0 源码入门起来就没有当时入门 Vue 2.6 源码那么吃力。

最近又找了点时间去回顾了一下，对比了两个版本的 Vue 在「响应式系统」实现上的差异，梳理并汇总成本文。

## 二、什么是响应式系统？

可能有一些同学第一次听到「响应式系统」这个名字有点陌生，但是「响应式」这三个字如果你用过 Vue 的话肯定很熟悉吧。

在 Vue 里，如果你在 template 里使用到了一个 data 选项数据，然后在 js 里修改这个 data 数据，就可以触发页面的重新渲染。这是 Vue 里响应式系统最常见的一个应用。

所以，响应式说白了就是：**一段代码引用到了一个变量，然后修改那个变量，可以「自动」触发这段代码的执行**。

而在 Vue 里，也对应地存在两种角色：**「响应式数据」和「观察者」**
+ 响应式数据 → 你可以修改的数据
+ 观察者 → 你修改数据后，会触发执行的代码

OK，角色介绍完了，下面我们进入正文。

## 三、对数据进行响应式观测

### 3.1 基本思路

上面我们遗留了两个问题：

1. 响应式数据与观察者之间，如何产生联系
2. 响应式数据修改后，如何自动触发观察者

答案是：**代理**。

我们可以先给响应式数据 data 进行 get 拦截，然后再去执行观察者函数，通过触发 data 的 get 代理就可以知道是哪个观察者引用到了 data，进而就可以为它们建立关联，这个步骤叫**「收集依赖」**。

同理，如果我们给响应式数据 data 进行 set 代理，那么修改 data 的时候，可以利用 set 代理来执行其依赖的观察者，这个步骤叫**「触发依赖」**。

现在，我们把这些工作统称为对数据进行**「响应式观测」**，而在 vue 3.0 里，是通过 proxy 来实现数据的响应式观测的：

```ts
/* proxy handler.get 拦截 */
function get(target, key, receiver) {
  ...
  const res = Reflect.get(target, key, receiver) 
  ...
  track(target, TrackOpTypes, key) 
  // 执行 track 方法，收集观察者作为依赖
  ...
  return res 
}

/* proxy handler.set 拦截 */
function set(target, key, value, receiver) {
  ...
  const result = Reflect.set(target, key, value, receiver) 
  ...
  trigger(target, TriggerOpType, key, value) 
  // 执行 trigger 函数，触发已收集到的观察者 
  ...
  return result 
}
```

可以看到，get、set 拦截主要是通过执行 track、trigger 两个函数来进行依赖的「收集」与「触发」的。

> 这里我隐藏了部分代码，比如对数组原型方法的拦截等，另外 Vue 3.0 proxy 除了对 get、set 操作进行拦截以外，还有对 has、deleteProperty 等操作也进行了拦截，不过最后收集和触发依赖的入口都是 track 和 trigger 函数，有兴趣的同学可以自行去研究。

### 3.2 为什么使用 Proxy

在 Vue 2.x 里，对数据的响应式观测是使用 Object.defineproperty API，而 Vue 3.0 则舍弃了这方法，转而采取 proxy 代理来实现响应式观测，这么做的理由是什么呢？

假设有下面这样一个对象：

```js
let obj = {
  a: 999,
  b: 888,
  c: 777
}
```

如果你使用 Object.defineproperty 进行拦截，首先你需要对 obj 进行键名遍历，然后对 a、b、c 三个键分别进行一次代理。而如果你使用 proxy 代理，只需要给 obj 生成一个 proxy 代理对象即可。

也就是说，**在对同一个对象进行浅度观测的时候，Vue 2.x 与 Vue 3.0 的时间复杂度比是：n ：1**，所以 Vue 3.0 对 vue 实例的初始化速度也会更快。

另外，proxy 的时候也带来了一些其他的好处，比如：

+ Vue 2.x 没有实现对象键的添加/移除的监听，Vue 3.0 借助 proxy 实现了
+ Vue 2.x 没有实现对 Set、Map 结构的响应式观测，Vue 3.0 实现了
+ Vue 2.x 没有对数组下标和长度进行监听，Vue 3.0 借助 proxy 实现了

> 我曾经疑惑过，明明 Object.defineproperty 可以对数组的下标和长度进行 get、set 拦截，但是为什么 Vue 2.x 里没有实现对数组下标和长度的监听？
>
> 后来翻阅了一下资料，才发现这个问题尤雨溪回答过：**性能问题，对数组下标进行观测的性能代价和获得的用户收益不成正比**
>
> 前面说过，你要对一个对象进行观测，首先就需要遍历它的键名，键越多，遍历越久，vue 实例初始化也就越慢。
>
> 而我们要知道，在日常开发里，一个长数组是很常见的，遍历起来很要命，但键很多的对象倒是不常见，遍历起来问题不大。所以 Vue 2.x 只观测对象的键，而不去观测数组的下标键。

## 四、观察者

### 4.1 观察者的收集与触发
前有提过几次观察者依赖的「收集」与「触发」，你可能会疑惑：收集？收集到哪里？触发？从哪里触发？

在 Vue 3.0 里，其实是给每一个你要观测的数据 target 分配了一个 depsMap，这个 depMaps 是一个 Map 结构，depMaps 的键是 target 的属性，键值是一个 Set 结构。

也就是每一个 `target[key]` 都会自己的一个 dep，这个 dep 是一个 Set 结构，当有观察者访问 `target[key]`，这个观察者就会被收集到 `target[key]` 对应的 dep Set 里，至于为什么要用 Set 结构自然是为了去重，避免依赖的重复收集。

```ts
// 收集依赖 
export function track(target, type, key) {
  ...
  // 给 target[key] 的 dep 创建一个 set 结构，用来存储它收集的 effect 依赖
  let depsMap = targetMap.get(target)
  if (!depsMap) targetMap.set(target, (depsMap = new Map()))
  let dep = depsMap.get(key)
  if (!dep) depsMap.set(key, (dep = new Set()))
  ...
  // 将 effect 依赖收集到 dep 里
}
```

而对于观察者来说，它什么时候会被收集呢？那就是该观察者执行的时候，它会被标记为 activeEffect，并放入 effectStack 栈顶，响应式数据每次收集依赖的时候只会收集  effectStack 栈顶的 activeEffect。

```ts
const effect = function reactiveEffect() {
  ...
  try {
    enableTracking() // 开启新一轮的依赖收集
    effectStack.push(effect) // 将该 effect 放到 effectStack 栈顶
    activeEffect = effect // 标记该 effect 为「当前该收集依赖」
    return fn() // 执行 effect 对应 fn 函数（比如渲染函数），开始依赖收集
  } finally {
    effectStack.pop() // 依赖收集完毕，effect 作为栈顶被弹出
    resetTracking() // 本轮依赖收集结束
    activeEffect = effectStack[effectStack.length - 1] // 重置 activeEffect，继续上一轮的依赖收集
  }
} 
```

为什么要用 effectStack 栈呢？其实是为了调度多个观察者之间的依赖收集关系。比如观察者 A 正在被收集的时候，触发观察者 B 的依赖收集，那么观察者 B 就会放到 effectStack 栈顶作为 activeEffect 进行依赖收集，中断了观察者 A  的依赖收集过程，等到观察者 B 被收集完，被推出栈顶，观察者 A 才可以继续依赖收集过程。

[![2yFAxK.jpg](https://z3.ax1x.com/2021/06/09/2yFAxK.jpg)](https://imgtu.com/i/2yFAxK)


### 4.2 观察者的分类

在 Vue 里，观察者目前被定义为三类：

1. 计算属性观察者
2. watch 选项观察者
3. 渲染函数观察者

其中，比较特殊的是计算属性观察者，它有两重定位：**既是一个响应式数据，也是一个观察者**。另外，它还有一个**惰性求值**的特性。

举个栗子说明一下：一个响应式数据 data，被计算属性使用了，而计算属性又被模板引用到了。那么，就会得到以下依赖收集关系：
+ data 的 dep 收集了计算属性观察者
+ 计算属性的 dep 收集了渲染函数观察者

这个时候，我们去修改 data，就会触发 data 的 dep 里的计算属性观察者。

而计算属性被触发后不会立刻重新求值，只是标记一下内部变量`this._dirty = true`，然后触发自己的 dep 里的渲染函数观察者。

渲染函数观察者被触发后，执行渲染函数，然后对模板里使用到的计算属性重新进行 get 取值，这个时候就会触发计算属性的 get 代理，getter 函数通过判断`_dirty === true` 然后执行 effect 来重新求值。 

[![2yFoQK.jpg](https://z3.ax1x.com/2021/06/09/2yFoQK.jpg)](https://imgtu.com/i/2yFoQK)

### 4.3 watcher 观察者与 effect 观察者

Vue 2.x 里，观察者被定义为 watcher 类，而在 Vue3.0 里，观察者被定义为 effect 函数。两者在整体逻辑上没有太大区别，不过 effect 函数是对于观察者公用逻辑的提取，更方便维护与拓展。

举个栗子，我们看一下 Vue 2.6 里，wacther 类的 run 方法 ( watcher 观察者被触发后执行 ) :
```js
function run () {
  if (this.active) {
    const value = this.get() // 重新求值（如果是渲染函数，则会重新渲染）
    if ( value !== this.value || isObject(value) || this.deep) {
      // 渲染观察者不会执行到这一步，因为渲染函数返回值 value 为空 
      const oldValue = this.value
      this.value = value
      if (this.user) {
        // 存在 this.user 标记，说明是开发者通过 wacth 选项或 $watch 定义的观察者
        const info = `callback for watcher "${this.expression}"`
        // invokeWithErrorHandling 内部会通过 try catch 来执行 this.cb，
        // 因为这个 cb 是开发者定义的，可能会出错
        invokeWithErrorHandling(this.cb, this.vm, [value, oldValue], this.vm, info)
      } else {
        this.cb.call(this.vm, value, oldValue)
      }
    }
  }
}
```

发现没有，在 watcher 类观察者里，同一个方法，因为观察者的不同而会导致这个函数有不同的执行分支。一堆 if else 耦合在一起难以梳理，我在阅读 Vue 2.x 源码的时候也经常被难受到。

而在 Vue 3.0，watcher 里公用的部分被提取了出来封装成 effect 函数，然后通过暴露一些 option 参数来实现观察者的定制化，大大提高了代码的可维护性和可拓展性。

比如，在 Vue 3.0 里，计算属性观察者是这样定义的：

```ts
// 省去部分代码
import { effect } from './effect'
...
class ComputedRefImpl<T> {
  constructor(getter) {
    this.effect = effect(getter, {...})
    ...
  }
  ...
}
```
将 effect 模块引用过来，再进行定制化，实现计算属性观察者的定义，而不是像 Vue 2.x 那样在 wacther 类的同一个方法里加一大堆 if else 来区分不同的观察者，这算是 Vue 3.0 在代码层面上的一个优化。

> 有一说一，Vue 2.6 的 wacther 类阅读起来是真的难受，同一个方法里耦合了一堆不同观察者的逻辑，我在理解响应式系统逻辑的同时，又得去梳理观察者自己本身的逻辑，比如计算属性为什么会是惰性求值，渲染函数观察者的渲染函数怎么来的……导致啃这部分源码的时候很吃力。
>
> 而在阅读 Vue 3.0 的 effect 源码的时候，思路很顺畅，因为 effect 只是作为观察者角色来参与到响应式系统里，你不用去管 effect 最后会被封装成什么类别的观察者。

## 五、异步更新队列

### 5.1 异步更新队列的意义

我们已经知道，修改一个响应式数据，可以触发它收集的观察者，然后执行特定的代码逻辑。比如渲染函数观察者被触发后，就行执行渲染函数，重新渲染页面。

我们思考两个问题：

1. *假如有一个响应式数据 data，它的 dep 里收集了多个观察者，我们修改 data 后，是不是会把这些观察者全部触发？这些观察者的执行有没有先后顺序？*
2. *如果一个响应式数据 data 收集了渲染函数观察者作为依赖，然后我在一段代码里反复修改 data 多次，那么渲染函数观察者会被触发多次吗？又会执行多次渲染吗？多次渲染岂不是很浪费渲染性能？*

答案是：**观察者的执行是异步的，在一次事件循环里同一个观察者只会被执行一次，并且不同的观察者之间是有先有顺序的。**

而这个特性，全部依靠于 Vue 内部维护的**异步更新队列**。

### 5.2 异步更新队列的实现

在 Vue 2.x 里，会维护一个 queue 异步更新队列，在同步代码里触发的 watcher 会放到 queue 里进行**去重存储**，然后调用 nextTick api 注册一个异步任务 flushSchedulerQueue，这个异步任务是用来清空 queue 里 wacther 的。

flushSchedulerQueue 执行时，首先会把 queue 里的 wacther **根据 id 从小到大进行排序**，然后再按序执行 wacther 的 run 方法。

你可能会好奇，watcher 什么时候定义的 id ？其实 wacther 在被创建的时候就会被指定一个 id，这个 id 值是全局递增的，也就是说该 wacther 越早被创建，id 就越小，在 queue 里的执行优先级也就越高。

**而 Vue 2.x 在初始化实例的时候，是先初始化 computed 选项，再初始化 wacth 选项，最后再进行模板解析的。**

所以，wacther 的创建顺序、被触发后执行顺序都是：计算属性观察者 → wacth 选项观察者 → 渲染函数观察者。

[![2sQz8g.jpg](https://z3.ax1x.com/2021/06/08/2sQz8g.jpg)](https://imgtu.com/i/2sQz8g)



> Vue 3.0 里也是类似的机制，effect 被触发后放入异步队列，然后注册一个微任务 flushJobs 去清空异步队列。
>
> 不同的是，Vue 3.0 对异步队列进行了拓展分成了三类，不同的 effect 被触发后会放到不同的异步队列，有兴趣的同学可以自行去了解。
>
> 



## 六、总结

从响应式系统的角度入手，其实就可以发现，Vue 2.6 → Vue 3.0 的升级没有牵动到它的底层设计，整个响应式系统依旧是分成这几大模块：数据观测 + 观察者 + 异步更新队列。

Vue 3.0 的真正升级，体现在某一些方面的优化。比如性能层面，利用 Proxy 优化响应式观测速度；又比如代码层面，将 watcher 进行了公有逻辑提取封装为 effect……

当然，这样的优化是很有意义的，毕竟对于前端开发者来说，优化页面性能是我们不断追求的目标 (也是我们不断秃头的原因)。



