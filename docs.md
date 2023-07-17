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

## 应用

### flex 布局

### 交互反馈

**触摸反馈**

```jsx
// /*page.wxss */
.hover{
  background-color: gray;
}

// <!--page.wxml -->
<button hover-class="hover"> 点击button </button>
<view hover-class="hover"> 点击view</view>
```

**button loading**

```jsx
// <!--page.wxml -->
<button loading="{{loading}}" bindtap="tap">操作</button>
//page.js
Page({
  data: { loading: false },
  tap: function() {
    // 把按钮的loading状态显示出来
    this.setData({
      loading: true
    })
    // 接着做耗时的操作
  }
})
```

**Toast和模态对话框**

```jsx
Page({
  onLoad: function() {
    wx.showToast({ // 显示Toast
      title: '已发送',
      icon: 'success',
      duration: 1500
    })
    // wx.hideToast() // 隐藏Toast
  }
})
```

```jsx
Page({
  onLoad: function() {
    wx.showModal({
      title: '标题',
      content: '告知当前状态，信息和解决方法',
      confirmText: '主操作',
      cancelText: '次要操作',
      success: function(res) {
        if (res.confirm) {
          console.log('用户点击主操作')
        } else if (res.cancel) {
          console.log('用户点击次要操作')
        }
      }
    })
  }
})
```

**下拉刷新**

```jsx
//page.json
{"enablePullDownRefresh": true }

//page.js
Page({
  onPullDownRefresh: function() {
    // 用户触发了下拉刷新操作
    // 拉取新数据重新渲染界面
    // wx.stopPullDownRefresh() // 可以停止当前页面的下拉刷新。
  }
})
```

**上拉触底**

```jsx
//page.json
// 界面的下方距离页面底部距离小于onReachBottomDistance像素时触发onReachBottom回调
{"onReachBottomDistance": 100 }

//page.js
Page({
  onReachBottom: function() {
    // 当界面的下方距离页面底部距离小于100像素时触发回调
  }
})
```

### HTTPS网络通信

**wx.request接口**

```js
wx.request({
  url: 'https://xxx',
  success: function(res) {
    console.log(res)// 服务器回包信息
  }
})
```

wx.request详细参数

| 参数名 | 类型 | 必填 | 默认值 | 描述 |
| :--- | :--- | :--- | :--- | :--- |
| url | String | 是 |  | 开发者服务器接口地址 |
| data | Object/String | 否 |  | 请求的参数 |
| header | Object | 否 |  | 设置请求的 header，header 中不能设置 Referer，默认header\['content-type'\] = 'application/json' |
| method | String | 否 | GET | （需大写）有效值：OPTIONS, GET, HEAD, POST, PUT, DELETE, TRACE, CONNECT |
| dataType | String | 否 | json | 回包的内容格式，如果设为json，会尝试对返回的数据做一次 JSON解析 |
| success | Function | 否 |  | 收到开发者服务成功返回的回调函数，其参数是一个Object |
| fail | Function | 否 |  | 接口调用失败的回调函数 |
| complete | Function | 否 |  | 接口调用结束的回调函数（调用成功、失败都会执行） |

> 小程序宿主环境要求request发起的网络请求必须是**https协议请求**
>
> wx.request**请求的域名需要**在小程序管理平台进行**配置**
>
> 为了方便开发者进行开发调试，开发者工具、小程序的开发版和小程序的体验版在某些情况下允许wx.request请求任意域名。

wx.request的success返回参数
| 参数名 | 类型 | 描述 |
| :--- | :--- | :--- |
| data | Object/String | 开发者服务器返回的数据 |
| statusCode | Number | 开发者服务器返回的 HTTP 状态码 |
| header | Object | 开发者服务器返回的 HTTP Response Header |

**使用技巧**

1. 设置超时时间 // 小程序request默认超时时间是60秒

app.json指定wx.requset超时时间为3000毫秒

```js
{
  "networkTimeout": {
    "request": 3000
  }
}
```

2. 请求前后的状态处理

```js
var hasClick = false;
Page({
  tap: function() {
    if (hasClick) {
      return
    }
    hasClick = true
    wx.showLoading()
    wx.request({
      url: 'https://test.com/getinfo',
      method: 'POST',
      header: { 'content-type':'application/json' },
      data: { },
      success: function (res) {
        if (res.statusCode === 200) {
          console.log(res.data)// 服务器回包内容
        }
      },
      fail: function (res) {
        wx.showToast({ title: '系统错误' })
      },
      complete: function (res) {
        wx.hideLoading()
        hasClick = false
      }
    })
  }
})
```

**异常排查**

- 检查手机网络状态以及wifi连接点是否工作正常。
- 检查小程序是否为开发版或者体验版，因为开发版和体验版的小程序不会校验域名。
- 检查对应请求的HTTPS证书是否有效，同时TLS的版本必须支持1.2及以上版本，可以在开发者工具的console面板输入showRequestInfo()查看相关信息。
- 域名不要使用IP地址或者localhost，并且不能带端口号，同时域名需要经过ICP备案。
- 检查app.json配置的超时时间配置是否太短，超时时间太短会导致还没收到回报就触发fail回调。
- 检查发出去的请求是否302到其他域名的接口，这种302的情况会被视为请求别的域名接口导致无法发起请求。

## 微信登录

### 获取微信登录凭证code

> 注意：如果现在我们有个接口，通过wx.request请求 <https://test.com/getUserInfo?id=1> 拉取到微信用户id为1在我们业务侧的个人信息，那么黑客就可以通过遍历所有的id，把整个业务侧的个人信息数据全部拉走，如果我们还有其他接口也是依赖这样的方式去实现的话，那黑客就可以伪装成任意身份来操作任意账户下的数据，想想这给业务带来多大的安全风险。
>
> wx.login是生成一个带有时效性的凭证，就像是一个会过期的临时身份证一样，在wx.login调用时，会先在微信后台生成一张临时的身份证，其有效时间仅为5分钟, 然后把这个临时身份证返回给小程序方, 这个临时的身份证我们把它称为微信登录凭证code。如果5分钟内小程序的后台不拿着这个临时身份证来微信后台服务器换取微信用户id的话，那么这个身份证就会被作废，需要再调用wx.login重新生成登录凭证。
>
> 由于这个临时身份证5分钟后会过期，如果黑客要冒充一个用户的话，那他就必须在5分钟内穷举所有的身份证id，然后去开发者服务器换取真实的用户身份。显然，黑客要付出非常大的成本才能获取到一个用户信息，同时，开发者服务器也可以通过一些技术手段检测到5分钟内频繁从某个ip发送过来的登录请求，从而拒绝掉这些请求。

### 发送code到开发者服务器

在wx.login的success回调中拿到微信登录凭证，紧接着会通过wx.request把code传到开发者服务器，为了后续可以换取微信用户身份id。如果当前微信用户还没有绑定当前小程序业务的用户身份，那在这次请求应该顺便把用户输入的帐号密码一起传到后台，然后开发者服务器就可以校验账号密码之后再和微信用户id进行绑定

```js
Page({
  tapLogin: function() {
    wx.login({
      success: function(res) {
        if (res.code) {
          wx.request({
            url: 'https://test.com/login',
            data: {
              username: 'zhangsan', // 用户输入的账号
              password: 'pwd123456', // 用户输入的密码
              code: res.code
            },
            success: function(res) {
              // 登录成功
              if (res.statusCode === 200) {
               console.log(res.data.sessionId)// 服务器回包内容
              }
            }
          })
        } else {
          console.log('获取用户登录态失败！' + res.errMsg)
        }
      }
    });
  }
})
```

### 到微信服务器换取微信用户身份id

- 拿这个code到微信服务器换取微信用户身份
- 到微信服务器的请求要同时带上AppId和AppSecret
- 如果发现泄露需要到小程序管理平台进行重置AppSecret，而code在成功换取一次信息之后也会立即失效，即便凭证code生成时间还没过期

开发者服务器和微信服务器通信也是通过HTTPS协议，微信服务器提供的接口地址是：

`https://api.weixin.qq.com/sns/jscode2session?appid=<AppId>&secret=<AppSecret>&js_code=<code>&grant_type=authorization_code`

URL的query部分的参数中 `<AppId>`, `<AppSecret>`, `<code>` 就是前文所提到的三个信息，请求参数合法的话，接口会返回以下字段

| 字段 | 描述 |
| :--- | :--- |
| openid | 微信用户的唯一标识 |
| session_key | 会话密钥 |
| unionid | 用户在微信开放平台的唯一标识符。本字段在满足一定条件的情况下才返回。 |

> openid就是前文一直提到的微信用户id，可以用这个id来区分不同的微信用户。
> session_key则是微信服务器给开发者服务器颁发的身份凭证，开发者可以用session_key请求微信服务器其他接口来获取一些其他信息，session_key不应该泄露或者下发到小程序前端。

### 绑定微信用户身份id和业务用户身份

业务侧用户还没绑定微信侧身份时，会让用户填写业务侧的用户名密码，这两个值会和微信登录凭证一起请求开发者服务器的登录接口，此时开发者后台通过校验用户名密码就拿到了业务侧的用户身份id，通过code到微信服务器获取微信侧的用户身份openid。微信会建议开发者把这两个信息的对应关系存起来，我们把这个对应关系称之为“绑定”

有了这个绑定信息，小程序在下次需要用户登录的时候就可以不需要输入账号密码，因为通过wx.login()获取到code之后，可以拿到用户的微信身份openid，通过绑定信息就可以查出业务侧的用户身份id，这样静默授权的登录方式显得非常便捷。

### 业务登录凭证SessionId

## 本地数据缓存

### 读写本地数据缓存

小程序提供了读写本地数据缓存的接口，通过wx.getStorage/wx.getStorageSync读取本地缓存，通过wx.setStorage/wx.setStorageSync写数据到缓存，其中Sync后缀的接口表示是同步接口，执行完毕之后会立马返回

```js
wx.getStorage({
  key: 'key1',
  success: function(res) {
    // 异步接口在success回调才能拿到返回值
    var value1 = res.data
  },
  fail: function() {
    console.log('读取key1发生错误')
  },
  // complete: function() {}  //Function 否 异步接口调用结束的回调函数（调用成功、失败都会执行）
})


try{
  // 同步接口立即返回值
  var value2 = wx.getStorageSync('key2')
}catch (e) {
  console.log('读取key2发生错误')
}
```

```js
// 异步接口在success/fail回调才知道写入成功与否
wx.setStorage({
  key:"key",
  data:"value1"
  success: function() {
    console.log('写入value1成功')
  },
  fail: function() {
    console.log('写入value1发生错误')
  }
})


try{
  // 同步接口立即写入
  wx.setStorageSync('key', 'value2')
  console.log('写入value2成功')
}catch (e) {
  console.log('写入value2发生错误')
}
```

> 注：每个小程序的缓存空间上限为**10MB**，如果当前缓存已经达到10MB，再通过wx.setStorage写入缓存会触发fail回调

### 利用本地缓存提前渲染界面

场景：对数据实时性/一致性要求不高的页面

```js
Page({
  onLoad: function() {
    var that = this
    var list =wx.getStorageSync("list")
    if (list) { // 本地如果有缓存列表，提前渲染
      that.setData({
        list: list
      })
    }
    wx.request({
      url: 'https://test.com/getproductlist',
      success: function (res) {
        if (res.statusCode === 200) {
          list = res.data.list
          that.setData({ // 再次渲染列表
            list: list
          })
          wx.setStorageSync("list",list) // 覆盖缓存数据
        }
      }
    })
  }
})
```

### 缓存用户登录态SessionId

```js
//page.js

var app = getApp()

Page({
  onLoad: function() {
    // 调用wx.login获取微信登录凭证
    wx.login({
      success: function(res) {
        // 拿到微信登录凭证之后去自己服务器换取自己的登录凭证
        wx.request({
          url: 'https://test.com/login',
          data: { code: res.code },
          success: function(res) {
            var data = res.data
            // 把 SessionId 和过期时间放在内存中的全局对象和本地缓存里边
            app.globalData.sessionId =data.sessionId
            wx.setStorageSync('SESSIONID',data.sessionId)
            // 假设登录态保持1天
            var expiredTime = +new Date() +1*24*60*60*1000
            app.globalData.expiredTime =expiredTime
            wx.setStorageSync('EXPIREDTIME',expiredTime)
          }
        })
      }
    })
  }
})
```

在重新打开小程序的时候，我们把上一次存储的SessionId内容取出来，恢复到内存

```js
//app.js

App({
  onLaunch: function(options) {
    var sessionId =wx.getStorageSync('SESSIONID')
    var expiredTime =wx.getStorageSync('EXPIREDTIME')
    var now = +new Date()
    if (now - expiredTime <=1*24*60*60*1000) {
      this.globalData.sessionId = sessionId
      this.globalData.expiredTime = expiredTime
    }
  },
  globalData: {
    sessionId: null,
    expiredTime: 0
  }
})
```

### tabbar

页面切换但是样式不切换，有可能是 页面是 `Page` 包裹，而非 `Component` 包裹，

```js
// Page: 在index `onLoad` 生命周期 中调用 `this.getTabBar && this.getTabBar().setData({selected: 1})` 进行切换
tabbarSelect() {
  if (typeof this.getTabBar === 'function' ) {
    this.getTabBar().setData({
      selected: 1
    })
  }
},
```

```js
// Component
Component({
  pageLifetimes: {
    show() {
      if (typeof this.getTabBar === 'function' &&
        this.getTabBar()) {
        this.getTabBar().setData({
          selected: 0
        })
      }
    }
  }
})
```

> 猜想：Component作为组件，pageLifetimes中的 show 触发时进行样式选中切换

### 事件交互

在微信小程序中，可以使用以下几种方法在绑定函数时传递值：

```jsx
// html
<button data-value="example" bindtap="handleClick">按钮</button>

// js
Page({
  handleClick(e) {
    const value = e.currentTarget.dataset.value; // 获取data-value 的值
    console.log(value); // 输出 "example"
  }
});
```

### css

可以使用的伪类：

```css
.tab:nth-child(1) {
  padding-left: 0;
}
```

设置顶部菜单背景色

```jsx
// js 代码方式
// 在页面的代码中调用以下方法
wx.setNavigationBarColor({
  frontColor: '#ffffff', // 前景颜色，即按钮上的图标和文字颜色
  backgroundColor: '#ff0000', // 背景颜色，即按钮的背景色
  success: function() {
    console.log('顶部菜单按钮背景色设置成功');
  },
  fail: function(err) {
    console.log('顶部菜单按钮背景色设置失败', err);
  }
});

// 全局配置 app.json
{
  "window": {
    "navigationBarBackgroundColor": "#ff0000", // 背景颜色
    "navigationBarTextStyle": "white" // 文字样式
  }
}

// 单页面配置 xx.json
{
  "navigationBarBackgroundColor": "#ff0000", // 背景颜色
  "navigationBarTextStyle": "white" // 文字样式
}

// 注：不支持 渐变
// 如要使用渐变，可以使用以下方式：
// 1. 使用图片
// 配置图片路径： "navigationBarBackgroundImage": "xxx.png"
// 2. 使用自定义navigationBar

<view class="custom-navigation">
  <linear-gradient direction="vertical" colors="#D5E1FF, #F5F6F9"></linear-gradient>
  {/* <!-- 其他自定义内容，例如标题和按钮 --> */}
</view>
/* custom-navigation.wxss */
.custom-navigation {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 44px;
}

linear-gradient {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
// <!-- example.wxml -->
<custom-navigation></custom-navigation>
// <!-- 页面其他内容 -->
```

### 获取url上参数

```js
  const currentPage = getCurrentPages().pop();
  const { xxx } = currentPage.options;
```

### 注意

在微信小程序的wxml中，{{}}主要用来显示变量值或表达式结果，而不能直接调用函数。但你可以在Page或Component中调用函数，并将结果赋给数据变量，然后在wxml中使用数据变量来显示函数的结果。
