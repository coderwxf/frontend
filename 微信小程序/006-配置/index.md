## 全局配置

`app.json`是小程序的全局配置文件，常用的配置如下

| 配置   | 说明                         |
| ------ | ---------------------------- |
| pages  | 当前小程序所有页面的存放路径 |
| window | 小程序窗口的全局外观设置     |
| tabBar | 小程序底部tabBar配置         |
| style  | 是否开启新版的组件样式       |

![img](https://s2.loli.net/2023/08/24/jEIkSZcRpdLeOwu.png)

### window配置

`导航条属性`

| 属性名                       | 类型     | 默认值  | 说明                                              |
| ---------------------------- | -------- | ------- | ------------------------------------------------- |
| navigationBarBackgroundColor | HexColor | #000000 | 1. 导航栏背景色<br />2. 只能是HEX格式颜色         |
| navigationBarTextStyle       | String   | white   | 1. 导航栏文本颜色<br />2. 可选值为 black \| white |
| navigationBarTitleText       | String   | 字符串  | 小程序标题                                        |



`背景区域设置`

| 属性名              | 类型     | 默认值  | 说明                                              |
| ------------------- | -------- | ------- | ------------------------------------------------- |
| backgroundColor     | HexColor | #ffffff | 1. 窗口背景色<br />2. 只支持HEX格式颜色           |
| backgroundTextStyle | String   | dark    | 1. 下拉loading样式<br />2. 可选值为 dark \| light |



`页面设置`

| 属性名                | 类型    | 默认值 | 说明                                                         |
| --------------------- | ------- | ------ | ------------------------------------------------------------ |
| enablePullDownRefresh | Boolean | false  | 是否开启全局下拉刷新<br />在真机环境下，下拉刷新 显示的背景 并不会自动关闭<br />在模拟器环境下，下拉刷新 显示的背景 会过一段时间自动关闭 |
| onReachBottomDistance | Number  | 50     | 滚动条光标距离底部还有多少距离的时候进行下拉加载更多<br />单位为px |



### tabBar

`tabBar`是移动端应用常见效果，用于实现多页面的快速切换

1. 在小程序中，`tabBar`可以分为 `底部tabBar` 和 `顶部tabBar`

   + 当`tabBar`是`底部tabBar`时，可以显示 `tab文本和图片`

   + 当`tabBar`是 `顶部tabBar`时，只能显示 `tab文本`，并不能显示 `tab图片`

2. `tabBar`中的页签 `最少需要2个，最多不超过5个`
3. 只有在`tabBar`的`list`配置项中配置了的页面才会显示`tabBar`, 否则页面不会显示`tabBar`



![image-20230824012524875](https://s2.loli.net/2023/08/24/uDkMCrhlj1mYQAw.png)



`tabBar配置项`

| 配置项          | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| backgroundColor | tabBar的背景色                                               |
| borderStyle     | tabBar 上边框颜色<br />可选值为 black` / `white              |
| color           | 未选中tab中的文本颜色                                        |
| selectedColor   | 选中tab中的文本颜色                                          |
| list            | tabBar中的页签列表<br />长度应该在`[2, 5]`<br />唯一一个必填项 |
| position        | tabBar所处位置<br />可选值为`bottom` / `top`                 |



`tab页签配置项`

| 配置项           | 说明                                 |
| ---------------- | ------------------------------------ |
| pagePath         | 页面路径，必须在 pages 中先定义      |
| text             | tab 上按钮文字                       |
| iconPath         | 未选中时图片路径<br />不支持网络图片 |
| selectedIconPath | 选中时图片路径<br />不支持网络图片   |

```js
{
  "pages": [
    "pages/index/index",
    "pages/logs/logs",
    "pages/profile/profile",
    "pages/home/home"
  ],
  "window": {
    "backgroundTextStyle": "light",
    "navigationBarBackgroundColor": "#fff",
    "navigationBarTitleText": "Weixin",
    "navigationBarTextStyle": "black"
  },
  "tabBar": {
    "position": "top",
    "list": [
      {
        "pagePath": "pages/index/index",
        "text": "index"
      },
      {
        "pagePath": "pages/home/home",
        "text": "home"
      }
    ]
  },
  "style": "v2",
  "sitemapLocation": "sitemap.json"
}
```



### 页面配置

`app.json`中的配置 是全局配置，会作用于每一个小程序页面

在小程序的每一个页面中都有`<page>.json`，可以对单独的小程序页面进行具体的配置

如果页面配置和全局配置产生了冲突，页面配置会覆盖全局配置，也就是说存在`就近原则`

```js
{
  // 组件注册
  "usingComponents": {},
    
  // 所有app.json中window选项下的属性都可以作为<page>.json中的顶层属性
  // 且<page>.json 中的属性 可以覆盖 app.json的window属性中所有的同名属性
  // ...
}
```

