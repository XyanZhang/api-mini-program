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
