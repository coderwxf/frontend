## 视图容器类组件

### [view](https://developers.weixin.qq.com/miniprogram/dev/component/view.html)

`view`的功能和原生HTML元素中`div`元素是一样的

`view`是一个块级容器元素， 常用来实现页面的布局效果

```html
<view class="container">
  <view>1</view>
  <view>2</view>
  <view>3</view>
</view>
```

```css
.container {
  display: flex;
  align-items: center;
  justify-content: space-around;
}

/* 和原生HTML一样 如果需要使用元素选择器 直接使用组件名即可 */
.container view {
  width: 100rpx;
  height: 100rpx;
  background: gray;
  display: flex;
  align-items: center;
  justify-content: center
}
```



### [scroll-view](https://developers.weixin.qq.com/miniprogram/dev/component/scroll-view.html)

可以滚动的`view`, 常用来实现滚动列表效果

```html
<!-- 
	scroll-x 开启水平滚动
	scroll-y 开启垂直滚动
	ps: 默认情况下，scroll-view的宽度是填满整个容器，高度是根据内容自适应
			所以为了可以正常出行滚动效果，一般需要手动设置scroll-view的宽度或高度
-->
<scroll-view class="container" scroll-y>
  <view>1</view>
  <view>2</view>
  <view>3</view>
</scroll-view>
```



### [swiper](https://developers.weixin.qq.com/miniprogram/dev/component/swiper.html) 和 [swiper-item](https://developers.weixin.qq.com/miniprogram/dev/component/swiper-item.html)

`swiper`表示轮播图组件 

`swiper-item`表示轮播图组件中的每个slide

```html
<!-- indicator-dots 表示轮播图需要显示指示器 -->
<swiper class="container" indicator-dots>
  <swiper-item>1</swiper-item>
  <swiper-item>2</swiper-item>
  <swiper-item>3</swiper-item>
</swiper>
```



## 基础能力组件

### text

一个行内元素，功能类似于原生html中的span元素

```html
<!-- 
	在小程序中只有加了selectable的text组件才可以被长按选中对应文本
	其余任何情况下，都无法通过长按的方式选中对应文本
-->
<text selectable>13155254478</text>
```



### rich-text

富文本组件，支持将html字符串渲染为wxml

```html
<rich-text nodes='<h1 style="color: blue">Hello Wolrd</h2>' />
```



## 其它组件

## button

按钮组件，存在`open-type`属性

通过`open-type`的不同值，可以通过点击按钮调用不同的微信开放能力

```html
<!-- 通过type指令按钮样式 -->
<button>按钮</button>
<button type="default">按钮</button>
<button type="primary">按钮</button>
<button type="warn">按钮</button>

<!-- 
    通过size指定按钮大小
    1. default -- 按钮独占一行 居中显示
    2. mini -- 行内块元素
-->
<button size="mini">按钮</button>
<button size="default">按钮</button>

<!-- 使用镂空风格按钮 -->
<button plain>按钮</button>
```



## image

用于展示图片，默认大小为 `300 x 240` 

```html
<!-- 
  不设置src，image依旧存在自己的宽高
  默认的宽高是 300 * 240
-->
<image />

<!-- 
  通过src指定图片URL路径
  ps: 默认情况下，所有的域名对于小程序而言都是不可用的
      1. 只有在小程序后台配置了的可信任域名 才可以被小程序正常访问
      2. 在开发模式下，可以通过在本地设置中 临时关闭校验 来绕过检测
-->
<image src="https://picsum.photos/400/400" />

<!-- 
  通过mode指定 图片的缩放和裁剪模式
  具体mode值 及对应解释 可以查看 https://developers.weixin.qq.com/miniprogram/dev/component/image.html
  1. scaleToFill -- 默认值， 缩放填满整个image元素 但是不保持图片的纵横比
  2. aspectFit -- background-image 的 contain模式
  3. aspectFill -- background-image 的 cover模式
  4. widthFix -- 宽度不变 高度自适应
  5. heightFix -- 高度不变 宽度自适应
-->
<image src="https://picsum.photos/400/400" mode="heightFix" />
```





### navigator

页面导航组件，功能类似于`a`元素