## 对象写法

1. 推荐camelCase，尽管支持kebab-case

2. 直接绑定一个样式对象通常是一个好主意，这样可以使模板更加简洁

```vue
<template>
	<div :style="styleObject"></div>
</template>

<script setup>
  const styleObject = reactive({
    color: 'red',
    fontSize: '13px',
    'background-color': 'skyblue'
  })
</script>
```



## 数组写法

可以给 `:style` 绑定一个包含多个样式对象的数组。这些对象会被合并后渲染到同一元素上

```html
<div :style="[baseStyles, overridingStyles]"></div>
```

