# 小程序开发指南

<https://developers.weixin.qq.com/miniprogram/dev/framework/quickstart/code.html>

## JSON配置

### 小程序配置 app.json

app.json 是当前小程序的全局配置, 包括了小程序的所有页面路径、界面表现、网络超时时间、底部 tab 等

```json
{
  "pages":[
    "pages/index/index",
    "pages/logs/logs"
  ],
  "window":{
    "backgroundTextStyle":"light",
    "navigationBarBackgroundColor": "#fff",
    "navigationBarTitleText": "Weixin",
    "navigationBarTextStyle":"black"
  }
}
```

> pages字段 —— 用于描述当前小程序所有页面路径，这是为了让微信客户端知道当前你的小程序页面定义在哪个目录。
>
> window字段 —— 定义小程序所有页面的顶部背景颜色，文字颜色定义等。

### 工具配置 project.config.json

小程序开发者工具在每个项目的根目录都会生成一个 project.config.json，你在工具上做的任何配置都会写入到这个文件，当你重新安装工具或者换电脑工作时，你只要载入同一个项目的代码包，开发者工具就自动会帮你恢复到当时你开发项目时的个性化配置，其中会包括编辑器的颜色、代码上传时自动压缩等等一系列选项

### 页面配置 page.json

小程序里边的每个页面都有不一样的色调来区分不同功能模块，因此我们提供了 page.json，让开发者可以独立定义每个页面的一些属性

## WXML 模板

```xml
<view class="container">
  <view class="userinfo">
    <button wx:if="{{!hasUserInfo && canIUse}}"> 获取头像昵称 </button>
    <block wx:else>
      <image src="{{userInfo.avatarUrl}}" background-size="cover"></image>
      <text class="userinfo-nickname">{{userInfo.nickName}}</text>
    </block>
  </view>
  <view class="usermotto">
    <text class="user-motto">{{motto}}</text>
  </view>
</view>
```

```xml
<text>{{msg}}</text>
```

```js
this.setData({ msg: "Hello World" })
```

## WXSS 样式

<https://developers.weixin.qq.com/miniprogram/dev/framework/view/wxss.html>

> WXSS 在底层支持新的尺寸单位 rpx ,开发者可以免去换算的烦恼，只要交给小程序底层来换算即可，由于换算采用的浮点数运算
>
> WXSS 仅支持部分 CSS 选择器

## JS 逻辑交互

<https://developers.weixin.qq.com/miniprogram/dev/framework/view/wxml/event.html>

```jsx
<view>{{ msg }}</view>
<button bindtap="clickMe">点击我</button>
```
<!-- 在 JS 文件里边声明了 clickMe 方法来响应这次点击操作： -->
```js
Page({
  clickMe: function() {
    this.setData({ msg: "Hello World" })
  }
})
```

## 宿主环境

WXML 模板和 WXSS 样式工作在渲染层，JS 脚本工作在逻辑层

### 小程序是如何把脚本里边的数据渲染在界面上的

```jsx
<view>{{ msg }}</view>
```

```js
Page({
  onLoad: function () {
    this.setData({ msg: 'Hello World' })
  }
})
```

> 1.渲染层和数据相关。
> 2.逻辑层负责产生、处理数据。
> 3.逻辑层通过 Page 实例的 setData 方法传递数据到渲染层。

### 通信模型

小程序的渲染层和逻辑层分别由2个线程管理：渲染层的界面使用了WebView 进行渲染；
逻辑层采用JsCore线程运行JS脚本。一个小程序存在多个界面，所以渲染层存在多个WebView线程，这两个线程的通信会经由微信客户端（下文中也会采用Native来代指微信客户端）做中转，逻辑层发送网络请求也经由Native转发

### 数据驱动

通过setData将数据改变，产生的JS对象对应的节点就会发生变化，此时可以对比前后两个JS对象得到变化的部分，然后把这个差异应用到原来的Dom树上，从而达到更新UI的目的，这就是“数据驱动”的原理

### 双线程下的界面渲染

小程序的逻辑层和渲染层是分开的两个线程。在渲染层，宿主环境会**把WXML转化成对应的JS对象**，在逻辑层发生数据变更的时候，我们需要通过宿主环境提供的**setData方法把数据从逻辑层传递到渲染层**，再经过对比前后差异，把差异应用在原来的Dom树上，渲染出正确的UI界面

## 代码层面

### 程序构造器App()

宿主环境提供了 App() 构造器用来注册一个程序App，需要留意的是App() 构造器必须写在项目根目录的app.js里，App实例是单例对象，在其他JS脚本中可以使用宿主环境提供的 getApp() 来获取程序实例。

```js
// other.js
var appInstance = getApp()
```

```js
App({
  onLaunch: function(options) {},
  onShow: function(options) {},
  onHide: function() {},
  onError: function(msg) {},
  globalData: 'I am global data'
})
```

| 参数属性 | 类型     | 描述                                                         |
| -------- | -------- | ------------------------------------------------------------ |
| onLaunch | Function | 当小程序初始化完成时，会触发 onLaunch（全局只触发一次）      |
| onShow   | Function | 当小程序启动，或从后台进入前台显示，会触发 onShow            |
| onHide   | Function | 当小程序从前台进入后台，会触发 onHide                        |
| onError  | Function | 当小程序发生脚本错误，或者 API 调用失败时，会触发 onError 并带上错误信息 |
| 其他字段 | 任意     | 可以添加任意的函数或数据到 Object 参数中，在App实例回调用 this 可以访问 |

### 生命周期

- 初始化完毕后，微信客户端就会给App实例派发onLaunch事件，App构造器参数所定义的**onLaunch**方法会被调用
- 进入小程序之后，用户可以点击右上角的关闭，或者按手机设备的Home键离开小程序，此时小程序并没有被直接销毁，我们把这种情况称为“小程序进入后台状态”，App构造器参数所定义的 **onHide** 方法会被调用
- 当再次回到微信或者再次打开小程序时，微信客户端会把“后台”的小程序唤醒，我们把这种情况称为“小程序进入前台状态”，App构造器参数所定义的**onShow**方法会被调用。
- 为了避免程序上的混乱，我们不应该从其他代码里主动调用App实例的生命周期函数。

```js
App({
  onLaunch: function(options) { console.log(options) },
  onShow: function(options) { console.log(options) }
})
```

> 微信客户端打开小程序多种途径 <https://mp.weixin.qq.com/debug/wxadoc/dev/framework/app-service/app.html>

**小程序全局数据**
小程序的JS脚本是运行在JsCore的线程里，小程序的每个页面各自有一个WebView线程进行渲染，所以小程序切换页面时，小程序逻辑层的JS脚本运行上下文依旧在同一个JsCore线程中。

App实例是单例的，因此不同页面直接可以通过App实例下的属性来共享数据。App构造器可以传递其他参数作为全局属性以达到全局共享数据的目的。

```js
// app.js
App({
  globalData: 'I am global data' // 全局共享数据
})
// 其他页面脚本other.js
var appInstance = getApp()
console.log(appInstance.globalData) // 输出: I am global data
```

> 注：所有页面的脚本逻辑都跑在同一个JsCore线程，页面使用setTimeout或者setInterval的定时器，然后跳转到其他页面时，这些定时器并没有被清除，需要开发者自己在页面离开的时候进行清理。

### 小程序页面

一个页面是分三部分组成：界面、配置和逻辑。界面由WXML文件和WXSS文件来负责描述，配置由JSON文件进行描述，页面逻辑则是由JS脚本文件负责。
一个页面的文件需要放置在同一个目录下，其中WXML文件和JS文件是必须存在的，JSON和WXSS文件是可选的。

页面构造器Page()

```js
Page({
  data: { text: "This is page data." }, // 页面的初始数据
  onLoad: function(options) { }, // 生命周期函数--监听页面加载，触发时机早于onShow和onReady
  onReady: function() { }, // 生命周期函数--监听页面初次渲染完成
  onShow: function() { }, // 生命周期函数--监听页面显示，触发事件早于onReady
  onHide: function() { }, // 生命周期函数--监听页面隐藏
  onUnload: function() { }, // 生命周期函数--监听页面卸载
  onPullDownRefresh: function() { }, // 页面相关事件处理函数--监听用户下拉动作
  onReachBottom: function() { }, // 页面上拉触底事件的处理函数
  onShareAppMessage: function () { }, // 用户点击右上角转发
  onPageScroll: function() { } // 页面滚动触发事件的处理函数
  // 其他：可以添加任意的函数或数据，在Page实例的其他函数中用 this 可以访问
})
```

**三个事件触发的时机是onLoad早于 onShow，onShow早于onReady**

page demo: 参数query，在列表页打开商品详情页时把商品的id传递过来，详情页通过刚刚说的onLoad回调的参数option就可以拿到商品id，从而绘制出对应的商品

```js
// pages/list/list.js
// 列表页使用navigateTo跳转到详情页
wx.navigateTo({ url: 'pages/detail/detail?id=1&other=abc' })

// pages/detail/detail.js
Page({
  onLoad: function(option) {
        console.log(option.id)
        console.log(option.other)
  }
})
```

setData()

由于小程序的渲染层和逻辑层分别在两个线程中运行，所以setData传递数据实际是一个异步的过程，所以setData的第二个参数是一个callback回调，在这次setData对界面渲染完毕后触发。

```js
// page.js
Page({
  onLoad: function(){
    this.setData({
      text: 'change data'
    }, function(){
      // 在这次setData对界面渲染完毕后触发
    })
  }
})
```

> 注意：
>
> 直接修改 Page实例的this.data 而不调用 this.setData 是无法改变页面的状态的，还会造成数据不一致。
>
> 由于setData是需要两个线程的一些通信消耗，为了提高性能，每次设置的数据不应超过1024kB。
>
> 不要把data中的任意一项的value设为undefined，否则可能会有引起一些不可预料的bug。

### 页面跳转

- 通过wx.navigateTo推入一个新的页面， 小程序宿主环境限制了这个页面栈的最大层级为10层（写文档的时候是10）

```js
wx.navigateTo({ url: 'pageD' })  // 往当前页面栈多推入一个pageD页面
wx.navigateBack()  // 可以退出当前页面栈的最顶上页
wx.redirectTo({ url: 'pageE' }) // 是替换当前页变成pageE
```

小程序提供了原生的Tabbar支持，我们可以在app.json声明tabBar字段来定义Tabbar页
> wx.navigateTo和wx.redirectTo只能打开非TabBar页面，wx.switchTab只能打开Tabbar页面。

**页面路由触发方式及页面生命周期函数的对应关系**

| 路由方式 | 触发时机 | 路由前页面生命周期 | 路由后页面生命周期 |
| :--- | :--- | :--- | :--- |
| 初始化 | 小程序打开的第一个页面 |  | onLoad, onShow |
| 打开新页面 | 调用 API wx.navigateTo  | onHide | onLoad, onShow |
| 页面重定向 | 调用 API wx.redirectTo  | onUnload | onLoad, onShow |
| 页面返回 | 调用 API wx.navigateBack  | onUnload | onShow |
| Tab 切换 | 调用 API wx.switchTab | 请参考表3-6 | 请参考表3-6 | 请参考表3-6 |
| 重启动 | 调用 API wx.reLaunch  | onUnload | onLoad, onShow |

Tab 切换对应的生命周期（以 A、B 页面为 Tabbar 页面，C 是从 A 页面打开的页面，D 页面是从 C 页面打开的页面为例）如表3-6所示，注意**Tabbar页面初始化之后不会被销毁**。

| 当前页面 | 路由后页面 | 触发的生命周期（按顺序） |
| :--- | :--- | :--- |
| A | A | 无 |
| A | B | A.onHide(), B.onLoad(), B.onShow() |
| A | B(再次打开) | A.onHide(), B.onShow() |
| C | A | C.onUnload(), A.onShow() |
| C | B | C.onUnload(), B.onLoad(), B.onShow() |
| D | B | D.onUnload(), C.onUnload(), B.onLoad(), B.onShow() |
| D(从转发进入) | A | D.onUnload(), A.onLoad(), A.onShow() |
| D(从转发进入) | B | D.onUnload(), B.onLoad(), B.onShow() |

### 组件

<https://mp.weixin.qq.com/debug/wxadoc/dev/component/>

### API

<https://developers.weixin.qq.com/miniprogram/dev/api/>

### 事件

常见的事件类型

- touchstart 手指触摸动作开始
- touchmove 手指触摸后移动
- touchcancel 手指触摸动作被打断，如来电提醒，弹窗
- touchend 手指触摸动作结束
- tap 手指触摸后马上离开
- longpress 手指触摸后，超过350ms再离开，如果指定了事件回调函数并触发了这个事件，tap事件将不被触发
- longtap 手指触摸后，超过350ms再离开（推荐使用longpress事件代替）
- transitionend 会在 WXSS transition 或 wx.createAnimation 动画结束后触发
- animationstart 会在一个 WXSS animation 动画开始时触发
- animationiteration 会在一个 WXSS animation 一次迭代结束时触发
- animationend 会在一个 WXSS animation 动画完成时触发


事件对象属性

| 属性 | 类型 | 说明 |
| :--- | :--- | :--- |
| type | String | 事件类型 |
| timeStamp | Integer | 页面打开到触发事件所经过的毫秒数 |
| target | Object | 触发事件的组件的一些属性值集合 |
| currentTarget | Object | 当前组件的一些属性值集合 |
| detail | Object | 额外的信息 |
| touches | Array | 触摸事件，当前停留在屏幕中的触摸点信息的数组 |
| changedTouches | Array | 触摸事件，当前变化的触摸点信息的数组 |

这里需要注意的是target和currentTarget的区别，currentTarget为当前事件所绑定的组件，而target则是触发该事件的源头组件。
