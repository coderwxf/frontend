redux是公共状态管理，是一个全局的用于存储和共享状态信息的容器

redux是独立状态管理库，在react中使用需要借助插件`react-redux`

`react-toolkit`是`redux + react-redux`的语法糖写法，简称`RTK`



一般仅推荐将需要在多组件复用的状态存入到redux中，并不推荐将所有的状态信息都存入redux中

并且推荐优先使用props来实现状态传递，只有当组件之间关系较远时，才使用redux



![image-20241005161230903](/Users/klaus/Documents/哈哈哈哈哈可以/0425/react18/assets/image-20241005161230903.png)    

![image-20241005161548702](/Users/klaus/Documents/哈哈哈哈哈可以/0425/react18/assets/image-20241005161548702.png) 

