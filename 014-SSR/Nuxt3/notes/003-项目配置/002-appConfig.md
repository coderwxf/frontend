appConfig用来定义应用配置，也就是用来定义在构建时确定的公共变量，也就是来定义一些全局配置的

```ts
export default defineNuxtConfig({
  appConfig: {
    title: 'nuxt page',
    theme: {
      primary: '#34fd3'
    }
  }
})
```

```ts
const cofig = useAppConfig()

// appConfig 中配置内容即可以在服务端获取也可以在客户端获取
console.log(config.title)
console.log(config.theme.primary)

// onMounted生命周期只会在客户端执行, 因为也只有客户端需要挂载
onMounted(() => {
  document.title = config.title
})
```



除了在`appConfig`中配置，也可以抽取到一个位于项目根目录下的`app.config.ts`中

如果两个位置同时存在配置，且对应内容相同，那么`app.config.ts`中的配置会覆盖`appConfig`中的配置

```ts
// app.config.ts
export default defineAppConfig({
  title: 'nuxt page',
  theme: {
    primary: '#34fd3'
  }
})
```

