## 小程序echarts+canvasdrawer实现页面转化图片并保存到相册
> 场景：小程序测试活动，实现echarts雷达图展示不同的结果、微信头像、二维码、测试结果文字，最终绘制出一张图片用户保存相册。考虑到开发时间及各种坑使用了canvasdrawer组件，其中开发过程中遇到的一些问题总结。
#### 1.安卓部分机型图片输出不执行
> - [x]  安卓输出图片错位问题同样异步解决
> - [x]  measureText注意线上版本库1.9.1+

```
//业务层代码
let that = this
    // 保存图片到临时的本地文件
    //注意chart初始化实例不能输出图片
    const ecComponent = this.selectComponent('#mychart-dom-graph');//获取echarts组件
    ecComponent.canvasToTempFilePath({
    //安卓机型此处不会成功回调
      success: res => {
        that.eventDraw(res.tempFilePath)
      },
      fail: res => console.log('失败', res)
});

//ec-canvas.js源码
canvasToTempFilePath(opt) {
      if (!opt.canvasId) {
        opt.canvasId = this.data.canvasId;
      }
      const system = wx.getSystemInfoSync().system
      ctx.draw(true, () => {//此处微信api在安卓部分机型不会回调 ，相反ctx.draw(false)清空画布会执行，导致echarts已经绘制画布清空，输出为空图片         
          wx.canvasToTempFilePath(opt, this);
      });  
    }
//修改后
canvasToTempFilePath(opt) {
      if (!opt.canvasId) {
        opt.canvasId = this.data.canvasId;
      }
      const system = wx.getSystemInfoSync().system
      if (/ios/i.test(system)) {
        ctx.draw(true, () => {          
          wx.canvasToTempFilePath(opt, this);          
        });  
      } else {//针对安卓机型异步获取已绘制图层
      ctx.draw(true,()=>{
        //断点打印依旧不会执行setTimeout(() => {wx.canvasToTempFilePath(opt, this);}, 1000);}});
        ctx.draw(true);
        setTimeout(() => {//延迟获取
          wx.canvasToTempFilePath(opt, this);
        }, 1000);
      }
    },
```
#### 2.onReady动态修改echarts[options]失败

```
 onReady: function() {
    let that = this;
    setTimeout(() => {//异步解决echarts实例没有加载成功的无法设置options
      option.series[0].data[0].value = that.data.canvasListData
      chart.setOption(option);
    }, 500);
  }
<!--备注-->
//提前声明变量
let chart = null;
var option = {}
```
#### 3.网络图片需下载到本地，注意配置downloadFile合法域名，尤其是微信头像链接

#### 4.相册授权拒绝后，button不会再次弹出授权弹窗，openSetting强制打开设置开启授权。

#### 5.圆形头像建议切镂空图覆盖，rect，clip有bug很难实现UI效果
建议查看：[微信小程序社区的帖子。](https://developers.weixin.qq.com/community/develop/doc/00086255ef09d0df4b0751f6651000)

#### 6.cancvas要画2倍图，否则输出图片模糊。

### 参考
- [ECharts 的微信小程序版本。](https://github.com/ecomfe/echarts-for-weixin)
- [小程序在安卓手机上绘制canvas，文字错乱。](https://developers.weixin.qq.com/community/develop/doc/000ee497d8cd78e50257df8d156400?highLine=%25E5%25AE%2589%25E5%258D%2593%25E4%25BF%259D%25E5%25AD%2598%25E5%259B%25BE%25E7%2589%2587)
-  [drawImage画图在Android真机上显示空白](https://developers.weixin.qq.com/community/develop/doc/0008e4f983cef006b3c7631be51c00?highLine=%25E5%25AE%2589%25E5%258D%2593%25E4%25BF%259D%25E5%25AD%2598%25E5%259B%25BE%25E7%2589%2587)
- [安卓 canvas组件draw函数的回调不执行](https://developers.weixin.qq.com/community/develop/doc/000204bc9441f854b2b768d1456c00?highLine=%25E5%25AE%2589%25E5%258D%2593%25E4%25BF%259D%25E5%25AD%2598%25E5%259B%25BE%25E7%2589%2587)
-  [wx.canvasToTempFilePath安卓手机无法生成图片](https://developers.weixin.qq.com/community/develop/doc/00088a6abb40b8f1c746e47aa51000?highLine=%25E5%25AE%2589%25E5%258D%2593%25E4%25BF%259D%25E5%25AD%2598%25E5%259B%25BE%25E7%2589%2587)
 -  [安卓手机获取不到canvasToTempFilePath路径的图片](https://developers.weixin.qq.com/community/develop/doc/d4a4564cc0c3e412420914598dc24b49?highLine=%25E5%25AE%2589%25E5%258D%2593%25E4%25BF%259D%25E5%25AD%2598%25E5%259B%25BE%25E7%2589%2587)
 -  [canvasdrawer](https://github.com/kuckboy1994/mp_canvas_drawer#api)

### 最后
> ###### HTML页面可以使用 html2canvas转换节点生成图片保存
- [基于html2canvas实现网页保存为图片及图片清晰度优化](https://segmentfault.com/a/1190000011478657?utm_source=tag-newest)

#### TIPS
> 由于时间限制，很多地方理解不够深刻，瑕疵很多，欢迎提出
