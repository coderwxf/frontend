AppID（应用ID）是在微信小程序系统中用于唯一标识一个小程序的字符串。每个小程序都有一个独特的 AppID，用于区分不同的应用

企业、政府、媒体、其他组织主体可以注册50个小程序，个体户和个人类型主体可注册5个小程序



## 小程序和网页的区别

1. 运行环境不同

   网页运行在浏览器环境下

   小程序运行为微信环境下，也就是webview中

2. 可供调用的API不同

   因为运行环境的不同，在小程序中，无法调用DOM和BOM

   只能调用微信环境所提供的API，例如 一键登录，微信支付等

3. 开发环境不同

   网页 只需要 浏览器 和 编辑器 即可

   小程序类似于app开发，需要申请自己独立的开发者账号

   并拥有自己独立的编辑器和模拟器



## 项目结构

```shell
.
├── app.js # 应用入口文件
├── app.json # 应用全局配置文件
├── app.wxss # 全局样式文件
├── pages # pages 目录用于存在小程序页面
│   ├── index # 每一个页面都是由四个文件组件的文件夹
# js，json，wxml，wxss可以直接在微信小程序环境中直接运行，不需要事先编译为一个文件
│   │   ├── index.js # 页面逻辑
│   │   ├── index.json # 页面配置
│   │   ├── index.wxml # 页面结构
│   │   └── index.wxss # 页面样式
│   └── logs 
│       ├── logs.js
│       ├── logs.json
│       ├── logs.wxml
│       └── logs.wxss
├── project.config.json # 项目配置 --- 多人协作共享的配置
├── project.private.config.json  # 项目配置文件 --- 多人协作个人独立的配置 --- 一般不被git所管理
├── sitemap.json # 站点索引文件，供微信爬虫索引(抓取)
└── utils # 存放工具方法的文件夹
    └── util.js
```



## 配置文件

小程序项目中总共有四种配置文件

1. 位于项目根目录的`app.json`
2. 位于项目根目录的`project.config.json`和`project.private.config.json`
3. 位于项目根目录的`sitemap.json`
4. 每一个页面独有的`<page>.json`



### app.json

`app.json`是当前小程序的全局配置页面，主要用于注册页面，配置界面外观和设置顶部tab栏等

```json
{
  // pages: string[]
  // 所有小程序页面都在pages数组中注册后才可以被访问
  // 初始页面是pages数组中的第一项
  "pages":[
    "pages/index/index",
    "pages/logs/logs"
  ],
  // 应用配置 -- 对整个应用整体外观的配置
  "window":{
    "backgroundTextStyle":"light",
    "navigationBarBackgroundColor": "#fff",
    "navigationBarTitleText": "Weixin",
    "navigationBarTextStyle":"black"
  },
  // 组件样式风格 -- 默认是v2
  // 默认不写是 v1 即老版本风格
  "style": "v2",
  // 指定映射文件存放位置
  "sitemapLocation": "sitemap.json"
}
```



### project.config.json

`project.config.json`是对整个应用和小程序IDE所进行的全局配置

> `project.config.json`是需要再团队成员之间共享的配置信息
>
> 如果是对个人的单独配置，需要配置在`project.private.config.json`中
>
> `project.private.config.json`会覆盖`project.config.json`中的同名配置项

```json
{
  // 应用描述
  "description": "项目配置文件",
  "packOptions": {
    "ignore": [],
    "include": []
  },
  // 项目编译时使用的配置 --- 这些选项可以在本地设置中以可视化的形式进行修改
  "setting": {
    // ...
  },
  // 指定小程序模式
  //     --- miniprogram --- 普通小程序
  //     --- plugin --- 小程序插件 --- 用于进行小程序的功能扩展
  "compileType": "miniprogram",
  // 小程序底层内核框架是MINA -- 指定内核版本
  "libVersion": "2.19.4",
  "appid": "xxxxxxxxxxxx", // appid
  // 小程序名 --- 项目名和小程序名 不需要一致
  // 小程序名在小程序管理后台中进行配置
  "projectname": "miniprogram-92",
  // 指定小程序的运行环境限制 --- 类似于package.json中的对等依赖
  "condition": {},
  // 对小程序IDE的配置
  "editorSetting": {
    "tabIndent": "insertSpaces",
    "tabSize": 2
  }
}
```



### sitmap.json

应用的索引文件，用于指定微信小程序爬虫抓取应用页面时候所采取的[收录规则](https://developers.weixin.qq.com/miniprogram/dev/framework/sitemap.html)

如果允许建立索引，爬虫会自动抓取对应页面并解析其中内容， 并根据页面的URL和解析出来的关键字建立索引

当用户在搜索的时候，输入的关键字和索引匹配的时候，就会为用户进行展示

如果存在多个匹配结果，会根据匹配程度的大小，进行降序排列

```json
{
  "desc": "关于本文件的更多信息，请参考文档 ",
  // 通过rules数组定义爬虫抓取规则
  // 默认情况下，所有页面都是可以被收录的
  "rules": [{
		// 允许爬虫执行的操作
  	"action": "allow",
    // 规则对应页面
  	"page": "*"
  }]
}
```

> 默认情况下，会在控制台显示 `索引提示`
>
> 该提示可以通过将`project.config.json`中`setting`选项的`checkSiteMap`字段的值设置为`false`来进行关闭



### `<page>.json`

每一个页面都有一个和自己页面同名的配置文件，例如 页面`detail`会存在页面配置文件`detail.json`

在这个配置文件中，可以对当前页面进行自定义配置，如注册组件，修改界面外观

如果`<page>.json`中存在和`app.json`一样的同名配置，`<page>.json`中的配置会覆盖`app.json`中的同名配置

```json
{
  // 用于注册组件
  "usingComponents": {},
  // 应用配置 --- 覆盖app.json中的同名配置
  // 不需要写在window选项中
  "navigationBarBackgroundColor": "#0000ff"
}
```



页面创建的两种方式

1. 在pages下新建文件夹，随后右击`新建page`, 就可以创建页面所需的四个文件
2. 在`app.json`中的`pages`字段中，添加新的路径，如果这条路径对应页面不存在，会自动新建对应页面



## 权限管理

在中大型公司中，人员分工相对较细。

这也就意味着 同一个小程序项目可能会有不同的岗位和不同角色的员工同时参与设计和开发

为此小程序为了方便我们进行管理，可以对不同岗位和不同角色的员工的权限进行边界的话费，以便于可以更高效的进行协同工作

![image-20230819144407112](https://s2.loli.net/2023/08/19/ZVJoFlWpYyqEiju.png) 

![image-20230819144428052](https://s2.loli.net/2023/08/19/6SUwBn2GkIdampu.png) 



### 成员管理

小程序中的成员可以分为`项目成员`和`体验成员`



**项目成员**

1. 参与小程序的开发和运营的成员
2. 可以登录小程序管理后台
3. 管理员可以添加和删除项目成员并设置项目成员对应角色

		+ 运营者
		+ 开发者
		+ 数据分析者



**体验成员**

1. 参与小程序内测体验的成员
2. 可使用体验版的小程序，但不属于项目成员
3. 管理员和项目成员可以添加、删除体验成员

![image-20230819144501211](https://s2.loli.net/2023/08/19/vRyxBNLf59jmKto.png) 

![image-20230819145010566](https://s2.loli.net/2023/08/19/hvLWIAlVMqCY3y6.png) 



## 版本

小程序主要可以分为

1. 开发版本 --- 开发者正在开发的代码版本，可以对项目版本进行自测
2. 体验版本 -- 小程序程序达到一个稳定可体验的状态时，开发者会将体验版本交给测试和产品经理进行体验测试
3. 正式版本 --- 体验版本进行功能修复和bug修复后，发布为正式版以后外部用户使用

![image-20230819145354597](https://s2.loli.net/2023/08/19/LPBoRghkJfqr9ET.png) 



### 发布上线

小程序的发布上线流程是`上传代码 -> 提交审核 -> 发布`

![image-20230819145836092](https://s2.loli.net/2023/08/19/NAmhb7I4UMGOvcE.png) 

![image-20230819145859528](https://s2.loli.net/2023/08/19/2tosTj1vAUQSNGq.png) 



## 推广

传播小程序可以使用两种形式的二维码

1. 普通二维码 --- 方型的，内部logo相对较小
2. 小程序码 --- 圆型的，内部logo相对较大



### 小程序码获取方式

1. 登录小程序后台
2. 设置 --- 基本设置 --- 完善小程序相关基本信息
3. 小程序码及线下物料下载 --- 下载小程序码



### 运营数据查看

1. 在小程序后台查看
   + 登录小程序管理后台
   + 点击 `统计` 页签，查看相关数据
2. 使用`小程序数据助手` 查看
   + 打开微信
   + 搜索`小程序数据助手`
   + 查看已发布的小程序相关数据

