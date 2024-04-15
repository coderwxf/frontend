## 对象写法

传入的样式格式应该为`Record<样式名，boolean>`

+ 布尔为true 加入样式，为false，移除样式

```html
<div :class="{ active: isActive }"></div>
```



`:class` 指令也可以和一般的 `class` attribute 共存

```html
<div
  class="static"
  :class="{ active: isActive, 'text-danger': hasError }"
></div>
```



## 数组写法

```vue
<template>
	<div :class="[activeClass, errorClass]"></div>
</template>

<script setup>
	const activeClass = ref('active')
	const errorClass = ref('text-danger')
</script>
```

在编译时，会运行数组中的每一个值，并将结果为真的值或符合`{ string: boolean }`结构的值留下并解析

```html
<div 
   :class="[
       isActive ? activeClass : '', 
       errorClass,
       { warn: isWraning }
    ]"
></div>
```

