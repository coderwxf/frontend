## 数据驱动

前端三大框架:

+ React
+ Vue
+ Angular「NG」

都有一个主流思想: 数据驱动视图渲染



传统前端开发 => 获取DOM元素，并进行操作实现功能

1. 既要处理业务逻辑，又要关心界面渲染 => 编写麻烦
2. 频繁操作DOM比较消耗性能 => 操作不当，容易导致DOM的重排（回流）/重绘



现代前端开发 => 数据驱动视图渲染

修改状态，界面由框架进行渲染和更新

1. 不需要手动操作DOM
2. 框架底层依旧是操作DOM
   + 构建了一套 `VDOM -> DOM`的渲染机制
   + 比手动操作DOM性能更高

从而在提升项目性能的同时，提高了开发效率



## MVC VS MVVM

React框架采用的是MVC体系：

+ MVC: model数据层 + view视图层 + controller控制层

  ![image.png](https://s2.loli.net/2024/09/22/YnQFgpPb6jqxUE8.png) 

Vue框架采用的是MVVM体系:

+ MVVM: model数据层 + view视图层 + viewModel 视图模型层

  ![image.png](https://s2.loli.net/2024/09/22/fQJOpZCTMEib19y.png) 



MVC 和 MVVM 最大的区别是:

1. MVC 只实现了 **单向驱动**

   + 状态驱动界面渲染

2. MVVM 实现了 **双 向驱动** 

   + 状态驱动界面渲染

   + 界面状态 「 主要是表单元素拥有自己的状态 」和 状态可以同步 

     => 表单元素改变，对应状态也会自动更新 



使用MVC 或 MVVM编写框架，就需要将动态数据绑定到界面上，并在界面触发事件的时候去更新数据

**数据需要使用特定语法绑定到界面上**

1. React => JSX  「 大胡子语法，大括号语法 」

   Vue

   + template 「 mustache + 指令 」 => template又叫小胡子语法
   + JSX

   

2. 参与界面渲染，并在数据改变时需要更新界面的数据叫状态

   + vue会自动监听状态的改变自动更新视图，所以vue中的状态有叫 响应式数据
   + react在状态改变时不会自动触发界面渲染，需要手动调用`setState`方法

   
