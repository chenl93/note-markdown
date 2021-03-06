## web-view组件

- 如果要在小程序中嵌入H5页面，需要使用web-view组件。简单来说它是一个可以用来承载网页的容器，会自动铺满整个小程序页面。在使用时有很多需要注意的方面：

  - 账号权限：首先需要开发者账号不仅是该小程序的开发者而且还有页面开发权限，这需要在该小程序关联的公众号里面绑定开发者账号为开发者。

  - 业务域名：如果web-view组件的src属性指向的不是关联的公众号文章，而是其他网页，则需要登录小程序管理后台（设置->开发设置）中配置业务域名；
    - 配置业务域名的时候会提示需要上传**验证文件**到该域名下进行验证。如果该域名下没有验证文件或者验证文件错误，则web-view页面直接提示报错，无法正常访问; 
    - 另外，如果页面中有其他链接跳转到非业务域名，进行跳转的时候也会报错导致无法正常访问，如果富文本中的链接，iframe，数据上报或支付跳转等其他非业务域名
    - 避免在链接中带有中文字符，在IOS中会有打开白屏的问题，建议加一下encodeURIComponent

  - 登录状态：小程序登录与web-view页面登录态属于两套隔离的系统。所以得将小程序的登录态传入到web-view页面中，目前最简单的方案是把cookie作为url参数传入，然后再在H5中获取并设置cookie；更高明的方式是搭建一个中间服务，传入要跳转的url和code，中间服务通过code得到session，再返回302重定向地址

  - 小程序环境判断：官网记载在网页可通过`window.__wxjs_environment`变量判断是否在小程序环境，并且建议在WeixinJSBridgeReady回调中使用，也可以使用JSSDK(1.3.2以上版本)提供的getEev接口
    
    - 实际情况一般会直接用`window.__wxjs_environment`。但是如果页面没有加载完，它是不准的，而且如果是web-view中进入第二个页面，安卓也拿不到该值，**很不靠谱**

    - 既然“不靠谱”，就有了通过URL里面加参数判断，这是铁定的稳。如添加一个mp参数。但是如果页面有几个跳转，总不能每个都去判断下加上 mp 参数吧。所以建议在进入的第一个页面直接种下storage，往后根据storage来判断就好；通过这三层保证（变量 || mp 参数 || storage），只要一个为真，则为小程序环境。这样锁定小程序环境很稳。

    - 另外，从微信7.0.0开始，可以通过判断userAgent中包含miniProgram字样来判断小程序web-view环境
    

  - web-view页面向小程序通信：目前web-view网页可通过postMessage向小程序发送信息，但是该信息只会在特定时机（小程序后退，组件销毁，分享）触发并收到消息

    - 小程序中通过在web-view中设置bindMessage属性收到信息
  
  - web-view页面中包括iframe：首先iframe的域名得为业务域名，不然页面也会提示报名，无法正常显示。其次iframe的页面里面不能使用官网上所记载的相关接口；如果要在iframe中跳到其他小程序页面的话，安卓可以使用window.top.window.wx.miniProgram.xxxAPI，而在IOS中window.top是行不通的。

  - 页面刷新：要刷新页面，得使用web-view的src属性，即更新页面的URL，最简单的方法就是加个时间戳参数。如详细页报名，报名成功回来就需要更新详情细页报名信息
    
    - 由于页面返回触发的是生命周期中的onShow，所以需要在该函数中更新URL值
    - 除此之外，如果H5页面中有一些播放任务（音乐，视频），在页面进入后台的时候即onHide时候，应该需要把URL设置为空，不然音视频会在后台一直播放直到该小程序销毁或者到音视频结束。
