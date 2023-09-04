## 上拉刷新

```js
Page({
  // 下拉刷新后 触发的回调
  onPullDownRefresh() {
    console.log('pull down refresh');

    setTimeout(() => {
      // 默认情况下，下拉刷新并不会自动关闭
      // 需要调用如下方法后 手动进行关闭
      wx.stopPullDownRefresh()
    }, 3000)
  },

  handleTap() {
    // 通过JS模拟用户的下拉刷新操作
    wx.startPullDownRefresh()
  }
})
```



## 下拉加载更多

P45
