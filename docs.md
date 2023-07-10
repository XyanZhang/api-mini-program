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
