## 计算属性 

我们知道，在模板中可以直接通过插值语法显示一些data中的数据，但是在某些情况，我们可能需要对数据进行一些转化后再显示，或者需要将多个数据结合起来进行显示

在模板中使用表达式，可以非常方便的实现，但是设计它们的初衷是用于简单的运算，在模板中放入太多的逻辑会让模板过重和难以维护，并且如果多个地方都使用到，那么会有大量重复的代码

所以我们希望将业务逻辑和UI界面进行分离，其中一种方式就是将逻辑抽取到一个method中，但这种做法有以下弊端

1. 所有的data使用过程都会变成了一个方法的调用

2. 多次获取数据，需要多次调用方法，执行对应的逻辑，没有缓存

事实上，`对于任何包含响应式数据的复杂逻辑，你都应该使用计算属性`

```html
<div id="app">
  <!-- 计算属性的使用和普通状态的使用方式是一致的 -->
  <h2>{{ fullname }}</h2>
</div>

<script>
  Vue.createApp({
    data() {
      return {
        firstname: 'Klaus',
        lastname: 'Wang'
      }
    },
    computed: {
      fullname() {
        return this.firstname + ' ' + this.lastname
      }
    }
  }).mount('#app')
```



### 缓存

计算属性会基于它们的依赖关系进行缓存，在数据不发生变化时，计算属性是不需要重新计算的

但是如果依赖的数据发生变化，在使用时，计算属性依然会重新进行计算

并且界面会使用最新的计算属性的值进行重新渲染



### getter 和 setter

计算属性在大多数情况下，只需要一个getter方法即可，所以我们会将计算属性直接写成一个函数

```html
<div id="app">
  <!-- 计算属性的使用和普通状态的使用方式是一致的 -->
  <h2>{{ fullname }}</h2>
  <button @click="change">change</button>
</div>

<script>
  Vue.createApp({
    data() {
      return {
        firstname: 'Klaus',
        lastname: 'Wang'
      }
    },
    methods: {
      change() {
        this.fullname = 'Alex Li'
      }
    },
    computed: {
      // 计算属性的完整写法
      fullname: {
        get() {
          return this.firstname + ' ' + this.lastname
        },
        set(v) {
          this.firstname = v.split(' ')[0]
          this.lastname = v.split(' ')[1]
        }
      }
    }
  }).mount('#app')
</script>
```



## 侦听器

在data返回的对象中定义了数据，这个数据通过插值语法等方式绑定到template中，当数据变化时，template会自动进行更新来显示最新的数据

但是在某些情况下，我们希望在代码逻辑中监听某个数据的变化，这个时候就需要用侦听器watch来完成了

```js
Vue.createApp({
  data() {
    return {
      info: {
        name: 'Klaus'
      }
    }
  },
  watch: {
    // 可以使用watch监听响应式数据的改变
    // 对应有两个参数
    // 参数一 --- 新值
    // 参数二 --- 旧值
    info(newV, oldV) {
      // 如果监听的值是对象，获取到的新值和旧值是对应对象的代理对象
      console.log(newV, oldV)

      // 代理对象 转 原生对象
      // 1. 使用浅拷贝获取一个新的对象，获取的新的对象为原生对象
      console.log({...newV})

      // 2. 使用Vue.toRaw方法获取原生对象
      console.log(Vue.toRaw(newV))
    }
  },
  methods: {
    change() {
      this.info = {
        name: 'Steven'
      }
    }
  }
}).mount('#app')
```



### 配置选项

| 属性      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| deep      | 是否开启深度监听<br />值为boolean<br />未开启的时候，如果监听的是对象，那么只有对象的引用发生改变的时候，才会触发watch回调<br />开始后，如果监听的是对象，那么只要对象中的任意一个属性发生了改变，就会触发watch回调 |
| immediate | 是否立即开始监听<br />默认情况下，初次渲染是不会触发watch监听，只有当值发生改变后，才会触发watch监听<br />将immediate设置为true后，初次渲染也会触发watch监听，此时oldValue的值为undefined |

```js
Vue.createApp({
  data() {
    return {
      info: {
        name: 'Klaus'
      }
    }
  },
  watch: {
    info: {
      // 开启了深度监听后，当info的属性发生改变的时候，就会触发对应的watch回调
      // 注意: 和直接修改info引用不同的是，如果直接修改的是对象的属性
      // 那么此时newV和oldV是同一个对象的引用, 此时也就获取不到对应的旧值
      handler(newV, oldV) {
        console.log(newV, oldV)
        console.log(newV === oldV)  // => true
      },
      deep: true,
      immediate: true
    }
  },
  methods: {
    change() {
      this.info.name = 'Steven'
    }
  }
}).mount('#app')
```



### 其它写法

`直接监听对象属性`

```js
watch: {
  'info.name'(newV, oldV){
    console.log(newV, oldV)
  }
}
```



`字符串写法`

```js
Vue.createApp({
  data() {
    return {
      info: {
        name: 'Klaus'
      }
    }
  },
  watch: {
    // watch的值如果是一个字符串的时候
    // 会自动以该字符串作为函数名去methods中查找对应的方法
    'info.name': 'watchHandler'
  },
  methods: {
    change() {
      this.info.name = 'Steven'
    },
    watchHandler(newV, oldV){
      console.log(newV, oldV)
    }
  }
}).mount('#app')
```



`数组写法`

```js
Vue.createApp({
  data() {
    return {
      info: {
        name: 'Klaus'
      }
    }
  },
  watch: {
    'info.name': [
      'watchHandler',

      function handle() {
        console.log('handler2')
      },

      {
        handler() {
          console.log('handler3')
        }
      }
    ]
  },
  methods: {
    change() {
      this.info.name = 'Steven'
    },
    watchHandler(){
      console.log('handler1')
    }
  }
}).mount('#app')
```



`$watch`

```js
Vue.createApp({
  data() {
    return {
      info: {
        name: 'Klaus'
      }
    }
  },
  created() {
    /*
          $watch 参数列表
            参数一 --- 侦听源
            参数二 --- 侦听回调
            参数三 --- 配置对象
        */
    this.$watch('info.name', (newV, oldV) => console.log(newV, oldV), {
      immediate: true
    })
  },
  methods: {
    change() {
      this.info.name = 'Steven'
    }
  }
}).mount('#app')
```



