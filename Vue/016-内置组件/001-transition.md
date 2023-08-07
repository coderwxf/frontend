Vue 提供了 [`<transition>`](https://cn.vuejs.org/guide/built-ins/transition.html) 和 [`<transition-group>`](https://cn.vuejs.org/guide/built-ins/transition-group.html) 组件来处理元素进入、离开和列表顺序变化时的过渡效果

+ `<transition>` 会在一个元素或组件进入和离开 DOM 时应用动画
+ `<transition-group>` 会在一个 `v-for` 列表中的元素或组件被插入，移动，或移除时应用动画



`<transition>` 是一个内置组件，这意味着 `<transition>`可以在任何组件内直接使用，无需注册

可以将需要添加动画的元素以插槽的形式传递给`<transition>`,`<transition>`组件会检测插槽内容的进入和离开，并根据您提供的动画效果来应用动画

`<transition>`是给单个元素添加动画，所以传递给`<transition>`的元素必须是`单根组件` 或 `单一元素`

`<transition>`默认没有任何的动效，可以通过给`<transition>`添加 `CSS过渡样式` 或 `JavaScript生命周期钩子`来指定`<transition>`的过渡效果，也就是说`<transition>`需要结合原生CSS过渡一起使用



除了通过 `v-if` / `v-show` 切换一个元素，我们也可以通过 `v-if` / `v-else` / `v-else-if` 在几个组件间进行切换，只要确保任一时刻只会有一个元素被渲染即可：

```html
<Transition>
  <button v-if="docState === 'saved'">Edit</button>
  <button v-else-if="docState === 'edited'">Save</button>
  <button v-else-if="docState === 'editing'">Cancel</button>
</Transition>
```





## CSS过渡动画

![过渡图示](https://cn.vuejs.org/assets/transition-classes.f0f7b3c9.png) 
| 样式             | 说明             | 自定义样式名         |
| ---------------- | ---------------- | -------------------- |
| `v-enter-from`   | 执行入场动画前   | `enter-from-class`   |
| `v-enter-to`     | 入场动画执行完毕 | `enter-to-class`     |
| `v-enter-active` | 入场动画执行中   | `enter-active-class` |
| `v-leave-from`   | 开始执行离场动画 | `leave-from-class`   |
| `v-leave-to`     | 离场动画执行完毕 | `leave-to-class`     |
| `v-leave-active` | 离场动画执行中   | `leave-active-class` |

```html
<template>
	<button @click="show = !show">Toggle</button>
  <Transition>
    <p v-if="show">hello</p>
  </Transition>
</template>

<style scoped>
.v-enter-active,
.v-leave-active {
  transition: opacity 0.5s ease;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}
</style>
```



### 具名样式

可以给 `<Transition>` 组件传一个 `name` prop 来声明一个过渡效果名

```html
<template>
	<Transition name="slide-fade">
    <p v-if="show">hello</p>
  </Transition>
</template>

<style scoped>
.slide-fade-enter-active {
  transition: all 0.3s ease-out;
}

.slide-fade-leave-active {
  transition: all 0.8s cubic-bezier(1, 0.5, 0.8, 1);
}

.slide-fade-enter-from,
.slide-fade-leave-to {
  transform: translateX(20px);
  opacity: 0;
}
</style>
```

`<Transition>` 过渡样式的默认名是`v`, 例如`v-enter-active`

如果手动指定了过渡样式名，那么就会使用name的值取代`v`， 例如在这里 `name-enter-active` 取代了 `v-enter-active`



### 自定义样式名

可以向 `<Transition>` 传递以下的 props 来指定自定义的过渡 class：

- `enter-from-class`
- `enter-active-class`
- `enter-to-class`
- `leave-from-class`
- `leave-active-class`
- `leave-to-class`

传入的这些 class 会覆盖相应阶段的默认 class 名。这个功能在你想要在 Vue 的动画机制下集成其他的第三方 CSS 动画库时非常有用，比如 [Animate.css](https://daneden.github.io/animate.css/)

```html
<Transition
  enter-active-class="animate__animated animate__tada"
  leave-active-class="animate__animated animate__bounceOutRight"
>
  <p v-if="show">hello</p>
</Transition>
```



## animation动画

animation动画 和 transition动画的使用方式 基本一模一样

唯一的区别是；

transition动画 是在元素进入或离开页面的一瞬间 结束的动画 所以在这个过程中并不会触发`transitionend`事件

`animation动画`是监听`animationend`事件执行完毕，并不是在进入或离开页面的那一瞬间结束动画

所以在动画执行时间相同的前提条件下，animation动画的执行时间 实际比 transition动画的执行时候 略晚一些

```html
<Transition name="bounce">
  <p v-if="show" style="text-align: center;">
    Hello here is some bouncy text!
  </p>
</Transition>

<style scoped>
.bounce-enter-active {
  animation: bounce-in 0.5s;
}
  
.bounce-leave-active {
  animation: bounce-in 0.5s reverse;
}
  
@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.25);
  }
  100% {
    transform: scale(1);
  }
}
</style>
```



### 同时使用 transition 和 animation

默认情况下，Vue会通过 `transitionend` 或 `animationend`来检测动画是否执行完毕

然而在某些场景中，你或许想要在同一个元素上同时使用它们两个

此时你需要显式地通过`type`属性告知Vue，以那个动画时间为准

`type`属性的可选值为 `animation` 或 `transition`

```html
<Transition type="animation">...</Transition>
```



## 深层过渡

默认情况下，`<transition> `通过监听通过插槽传入的DOM模板根元素的 `transitionend` 或者` `animationend`

但是有的时候 执行过渡功效的并不是模板根元素，而是其子元素

```html
<Transition name="nested">
  <div v-if="show" class="outer">
    <div class="inner">
      Hello
    </div>
  </div>
</Transition>
```

```css
.nested-enter-active .inner,
.nested-leave-active .inner {
  transition: all 0.3s ease-in-out;
}

.nested-enter-from .inner,
.nested-leave-to .inner {
  transform: translateX(30px);
  opacity: 0;
}
```



在这种情况下，Vue无法自动检测到动画何时结束，所以需要通过`duration`属性手动传入动画的执行时间

```html
<Transition :duration="550">...</Transition>
```

`duration`属性对应的值 为 动画的延迟执行时间 + 动画的实际执行时间，单位是`毫秒`

如果有必要的话，你也可以用对象的形式传入，分开指定进入和离开所需的时间

```html
<Transition :duration="{ enter: 500, leave: 800 }">...</Transition>
```



> 推荐通过`transform`、`opacity`、`perspective` 和 `filter`  这类会在渲染时单独开启渲染层的属性 来执行动画效果
>
> 1. 这类属性并不会修改DOM结构，不会触发昂贵的 CSS 布局重新计算
> 2. 大多数的现代浏览器在执行transform动画的过程中，都会自动开启GPU的硬件加速



## 生命周期钩子

和组件的生命周期钩子一样，我们可以通过`<transition>`提供的生命周期钩子 来在特定的时刻执行自定义逻辑

```html
<Transition
  @before-enter="onBeforeEnter"
  @enter="onEnter"
  @after-enter="onAfterEnter"
  @enter-cancelled="onEnterCancelled"
  @before-leave="onBeforeLeave"
  @leave="onLeave"
  @after-leave="onAfterLeave"
  @leave-cancelled="onLeaveCancelled"
>
  <!-- ... -->
</Transition>
```

```js
// 执行入场动画前 --- el 表示 执行动画的当前元素
function onBeforeEnter(el) {}

// 开始执行入场动画
function onEnter(el, done) {
  // 调用done函数 表示入场动画 执行完毕
  done()
}

// 入场动画执行完毕
function onAfterEnter(el) {}

// 入场动画被取消
function onEnterCancelled(el) {}

// 离场动画执行前
function onBeforeLeave(el) {}

// 开始执行离场动画
function onLeave(el, done) {
  // 离场动画执行完毕
  done()
}

// 离场动画执行完毕
function onAfterLeave(el) {}

// 离场动画被取消
function onLeaveCancelled(el) {}
```

这些钩子可以与 CSS 过渡或动画结合使用，也可以单独使用

如果单独使用，推荐显示将css prop的值设置为false，因为它的默认值是true

这样既可以显示告诉Vue可以跳过对 CSS 过渡的自动探测，也可以避免CSS 规则意外地干扰过渡效果

```html
<Transition
  ...
  :css="false"
>
  ...
</Transition>
```

设置 `:css="false"` 后，我们就自己全权负责控制什么时候过渡结束了。这种情况下对于 `@enter` 和 `@leave` 钩子来说，回调函数 `done` 就是必须的。否则，钩子将被同步调用，过渡将立即完成。



## 出现时过渡

如果你想在某个节点初次渲染时应用一个过渡效果，你可以添加 `appear` prop：

```html
<Transition appear>
  ...
</Transition>
```



## 过渡模式

在之前的例子中，进入和离开的元素都是在同时开始动画的，因此我们不得不将它们设为 `position: absolute` 以避免二者同时存在时出现的布局问题。

然而，很多情况下这可能并不符合需求。我们可能想要先执行离开动画，然后在其完成**之后**再执行元素的进入动画。手动编排这样的动画是非常复杂的，好在我们可以通过向 `<Transition>` 传入一个 `mode` prop 来实现这个行为：

```html
<Transition mode="out-in">
  ...
</Transition>
```

`<Transition>` 也支持 `mode="in-out"`，虽然这并不常用。









---

TransitionStart 来检测动画开始吗 ？

```js
// 在元素被插入到 DOM 之前被调用
// 用这个来设置元素的 "enter-from" 状态
function onBeforeEnter(el) {}

// 在元素被插入到 DOM 之后的下一帧被调用
// 用这个来开始进入动画
function onEnter(el, done) {
  // 调用回调函数 done 表示过渡结束
  // 如果与 CSS 结合使用，则这个回调是可选参数
  done()
}

// 当进入过渡完成时调用。
function onAfterEnter(el) {}
function onEnterCancelled(el) {}

// 在 leave 钩子之前调用
// 大多数时候，你应该只会用到 leave 钩子
function onBeforeLeave(el) {}

// 在离开过渡开始时调用
// 用这个来开始离开动画
function onLeave(el, done) {
  // 调用回调函数 done 表示过渡结束
  // 如果与 CSS 结合使用，则这个回调是可选参数
  done()
}

// 在离开过渡完成、
// 且元素已从 DOM 中移除时调用
function onAfterLeave(el) {}

// 仅在 v-show 过渡中可用
function onLeaveCancelled(el) {}
```



这样既可以显示告诉Vue可以跳过对 CSS 过渡的自动探测，也可以避免CSS 规则意外地干扰过渡效果



script setup 组件名是自动生成的 规则是什么

## 动态过渡

`<Transition>` 的 props (比如 `name`) 也可以是动态的！这让我们可以根据状态变化动态地应用不同类型的过渡：

template

```
<Transition :name="transitionName">
  <!-- ... -->
</Transition>
```

这个特性的用处是可以提前定义好多组 CSS 过渡或动画的 class，然后在它们之间动态切换。

你也可以根据你的组件的当前状态在 JavaScript 过渡钩子中应用不同的行为。最后，创建动态过渡的终极方式还是创建[可复用的过渡组件](https://cn.vuejs.org/guide/built-ins/transition.html#reusable-transitions)，并让这些组件根据动态的 props 来改变过渡的效果。掌握了这些技巧后，就真的只有你想不到，没有做不到的了。





----

样式穿透

toRef

toRefs

css中编写变量