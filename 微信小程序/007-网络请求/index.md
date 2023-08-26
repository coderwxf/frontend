## 限制

小程序官方对数据接口情况做出了限制：

1. 只能是`HTTPS`类型接口

2. 请求的接口域名必须添加到小程序管理后台的信任列表中

3. 跨域的原因是因为浏览器的同源安全策略

   这个同源安全策略并不存在于小程序中

4. 域名不可以是IP地址 或 localhost

5. 域名必须通过ICP备案

6. 一个月最多修改五次

7. 在开发环境下，可以临时取消这些限制

   生产环境，无法取消这些限制

```js
Page({
  getFun() {
    wx.request({
      // 请求地址
      url: 'https://httpbin.org/post',
      // 请求方式 --- 如果省略不写 默认是GET
      method: 'POST',
      // 参数 get/post参数都通过data进行配置
      data: {
        name: 'Klaus',
        age: 23
      },
      // 成功回调
      success(res) {
        // 参数是请求返回结果
        // 结果是axios格式结果 所以实际数据需要通过res.data进行获取
        console.log(res.data);
      },
      // 失败回调
      fail(err) {
        console.log(err);
      },
      // 无论失败还是成功都需要执行的回调
      complete() {
        console.log('complete');
      }
    })
  },

  // Page文件中，当页面开始准备加载时，会回调onLoad回调函数
  onLoad() {
    // this就是当前页面实例
    this.getFun()
  }
})
```

