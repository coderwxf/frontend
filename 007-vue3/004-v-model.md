表单提交是开发中非常常⻅的功能，也是和用户交互的重要手段

如果我们想在代码逻辑中获取到用户提交的数据，我们通常会使用v-model指令来完成

v-model指令可以在表单 input、textarea以及select元素上创建双向数据绑定

v-model指令会根据控件类型自动选取正确的方法来更新元素

v-model 本质上不过是语法糖，它负责监听用户的输入事件来更新数据，并在某种极端场景下进行一些特殊 处理

```vue
<div id="app">
  <input type="text" v-model="msg">
  <div>{{ msg }}</div>
</div>

<script>
  Vue.createApp({
    data() {
      return {
        msg: ''
      }
    }
  }).mount('#app')
</script>
```

等价于

```vue
<div id="app">
  <input type="text" :value="msg" @input="msg = $event.target.value">
  <div>{{ msg }}</div>
</div>

<script>
  Vue.createApp({
    data() {
      return {
        msg: ''
      }
    }
  }).mount('#app')
</script>
```



## 绑定其它元素

### textarea

```vue
<div id="app">
  <textarea v-model="content"></textarea>
  <div>{{ content }}</div>
</div>

<script>
  Vue.createApp({
    data() {
      return {
        content: 'hello world'
      }
    }
  }).mount('#app')
</script>
```



### checkbox

`单选框`

1. v-model的值为布尔值
2. 此时input的value属性并不影响v-model的值

```vue
<div id="app">
  <input type="checkbox" v-model="isAgree">同意协议
  <div>{{ isAgree }}</div>
</div>

<script>
  Vue.createApp({
    data() {
      return {
        isAgree: false
      }
    }
  }).mount('#app')
</script>
```



`多选框`

1. 当是多个复选框时，因为可以选中多个，所以对应的data中属性是一个数组
2. 当选中某一个时，就会将input的value添加到数组中
3. 所以如果是多选框，每一个复选框必须存在value属性

```vue
<div id="app">
  <input type="checkbox" v-model="hobbies" value="basketball">basketball
  <input type="checkbox" v-model="hobbies" value="football">football
  <div>{{ hobbies }}</div>
</div>

<script>
  Vue.createApp({
    data() {
      return {
        hobbies: []
      }
    }
  }).mount('#app')
</script>
```



### radio

```html
<div id="app">
  <input type="radio" v-model="gender" value="male">male
  <input type="radio" v-model="gender" value="female">female
  <div>{{ gender }}</div>
</div>

<script>
  Vue.createApp({
    data() {
      return {
        gender: 'male'
      }
    }
  }).mount('#app')
</script>
```



### select

`单选`

```vue
<div id="app">
  <select v-model="fruit">
    <option value="apple">apple</option>
    <option value="orange">orange</option>
    <option value="banana">banana</option>
  </select>
  <div>{{ fruit }}</div>
</div>

<script>
  Vue.createApp({
    data() {
      return {
        fruit: 'apple'
      }
    }
  }).mount('#app')
</script>
```



`多选`

```html
<div id="app">
  <select multiple v-model="fruits">
    <option value="apple">apple</option>
    <option value="orange">orange</option>
    <option value="banana">banana</option>
  </select>
  <div>{{ fruits }}</div>
</div>

<script>
  Vue.createApp({
    data() {
      return {
        fruits: []
      }
    }
  }).mount('#app')
</script>
```



## 修饰符

| 修饰符 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| lazy   | 默认情况下，v-model在进行双向绑定时，绑定的是input事件<br />那么会在每次内容输入后就将最新的值和绑定的属性进行同步<br />如果我们在v-model后跟上lazy修饰符，那么会将绑定的事件切换为 change 事件，只有在提交时(比如回⻋，失焦时)才会触发 |
| number | input获取到的值默认都是string类型的<br />如果我们希望自动将输入的值转换为数字类型，那么可以使用 .number 修饰符 |
| trim   | 自动过滤输入内容的首尾空格                                   |

```vue
<div id="app">
  <!-- 默认输入框中输入的内容的类型为string类型 -->
  <input type="text" v-model="count1">
  <!-- 如果输入框的type为number的时候，输入框对应的值为number类型 -->
  <input type="number" v-model="count2">
  <!-- 如果输入框的v-model使用了number修饰符 那么对应的输入值的类型也为number类型 -->
  <input type="text" v-model.number="count3">

  <div>
    {{ typeof count1 }} --- {{ typeof count2 }} --- {{ typeof count3 }}
  </div>
</div>

<script>
  Vue.createApp({
    data() {
      return {
        count1: 0,
        count2: 0,
        count3: 0
      }
    }
  }).mount('#app')
</script>
```



