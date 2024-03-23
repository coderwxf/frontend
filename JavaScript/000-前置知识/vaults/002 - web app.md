## App 分类

1. 原生APP
2. Hybird APP
3. Web App



### 原生APP

原生APP (native app) 直接运行在操作系统上

1. 性能好
2. 功能强大 --- 可以调用手机软硬件 如 通讯录，手机相册，摄像头等

缺点:

1. 不能跨平台
   + 成本大
   + 安卓和ios 功能可能出现不同步



### web app

web app 本质就是H5项目，运行在浏览器上

1. 可以跨平台
2. 及时更新 --- 不需要审核

缺点:

1. 性能差
2. 功能没有强大 --- 支不支持某项功能，得看浏览器支不支持



### Hybird App (混合APP开发)

1. 需要调用原生功能的 和 软件基础架构壳子 --- 使用原生语言开发
2. 通过webview来在app中加载我们的网页
   + webview 可以认为是浏览器中的浏览器 --- 内核是webkit
   + webview 通过js bridge 去调用 原生app中的一些功能

壳子也开始使用web技术编写，并可以调用原生功能

+ React-native --- react
+ flutter  --- dart
+ uni-app --- vue

