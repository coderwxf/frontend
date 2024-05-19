### Vue.js 教学演示：实例创建与视图绑定

#### 创建Vue实例

在Vue.js中，创建实例是开始任何项目的第一步。通过`new Vue()`方法，我们可以初始化一个新的Vue对象。例如：

```javascript
var vm = new Vue({
  // 配置项
});
```

#### 视图绑定方法

在Vue中，有多种方法可以绑定视图，主要包括：

1. **使用`el`属性**：
   `el`属性用于指定一个页面已存在的DOM元素作为Vue实例的挂载目标。它可以是一个CSS选择器字符串，或直接绑定到一个DOM元素。

   ```javascript
   var vm = new Vue({
     el: '#app'
   });
   ```

2. **使用`template`属性**：
   `template`属性允许我们定义实例的HTML模板。它可以包含HTML字符串，Vue将使用这个模板替换挂载的元素。

   ```javascript
   var vm = new Vue({
     template: '<div>{{ message }}</div>'
   });
   ```

3. **使用`$mount`方法**：
   如果在实例创建时没有使用`el`属性，可以通过`$mount`方法手动挂载一个未挂载的实例。

   ```javascript
   var vm = new Vue({
     template: '<div>{{ message }}</div>'
   }).$mount('#app');
   ```

#### 实例的配置选项

Vue实例在创建时接受多种配置选项，如`data`, `methods`, `computed`, `watch`, `props`等，这些选项允许我们定义数据、计算属性、方法及更多功能。

#### 生命周期和模板编译

Vue实例有一个完整的生命周期，从创建到销毁过程中，Vue提供了多个生命周期钩子，如`created`, `mounted`, `updated`, `destroyed`等，允许我们在不同阶段添加自定义的行为。

在模板编译过程中，Vue将`template`选项或挂载元素的HTML内容编译成渲染函数。如果定义了`render`函数，Vue将使用它而不是将模板编译成渲染函数。

#### 示例

以下是一个简单的示例，展示了如何创建一个Vue实例，通过`el`属性挂载，并在页面上显示消息：

```html
<div id="app"></div>

<script>
var vm = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
});
</script>
```

这段代码创建了一个Vue实例，通过`el`属性指定了页面中的一个元素作为挂载点，并通过数据绑定显示了一条消息。