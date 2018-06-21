---
layout: post
title: 小程序实现原理解析
date: 2016-06-02 11:15:06 
tag: 小程序实现原理解析
---

##概述
作为一名前端开发，如果你还停留在应用开发层面，那你就OUT了，快来跟我一起探讨下小程序框架本身底层实现的一些技术细节吧，让我们从小程序的运行机制来深度了解小程序。 
小程序是基于WEB规范，采用HTML,CSS和JS等搭建的一套框架，微信官方给它们取了一个很牛逼的名字：WXML,WXSS，但本质上还是在整个WEB体系之下构建的。 
WXML，个人猜测在取这个名字的是微信的Xml，说到底就是xml的一个子集。WXML采用微信自定义的少量标签WXSS，大家可以理解为就是自定义的CSS。实现逻辑部分的JS还是通用的ES规范，并且runtime还是Webview（IOS WKWEBVIEW, ANDROID X5）。
一个完整的小程序主要由以下几部分组成： 
一个入口文件：app.js 
一个全局样式：app.wxss 
一个全局配置：app.json 
页面：pages下，每个页面再按文件夹划分，每个页面4个文件 
视图：wxml，wxss 
逻辑：js，json（页面配置，不是必须）

注：pages里面还可以再根据模块划分子目录，孙子目录，只需要在app.json里注册时填写路径就行。

小程序打包
开发完成后，我们就可以通过这里可视化的按钮，点击直接打包上传发布，审核通过后用户就可以搜索到了。 
那么打包怎么实现的呢？ 
这就涉及到这个编辑器的实现原理和方式了，它本身也是基于WEB技术体系实现的，nwjs+react，nwjs是什么：简单是说就是node+webkit，node提供给我们本地api能力，而webkit提供给我们web能力，两者结合就能让我们使用JS+HTML实现本地应用程序。 
既然有nodejs，那上面的打包选项里的功能就好实现了。 
ES6转ES5：引入babel-core的node包 
CSS补全：引入postcss和autoprefixer的node包（postcss和autoprefixer的原理看这里） 
代码压缩：引入uglifyjs的node包

注：在android上使用的x5内核，对ES6的支持不好，要兼容的话，要么使用ES5的语法或者引入babel-polyfill兼容库。
所有的小程序基本都最后都被打成上面的结构 
1、WAService.js 框架JS库，提供逻辑层基础的API能力 
2、WAWebview.js 框架JS库，提供视图层基础的API能力 
3、WAConsole.js 框架JS库，控制台 
4、app-config.js 小程序完整的配置，包含我们通过app.json里的所有配置，综合了默认配置型 
5、app-service.js 我们自己的JS代码，全部打包到这个文件 
6、page-frame.html 小程序视图的模板文件，所有的页面都使用此加载渲染，且所有的WXML都拆解为JS实现打包到这里 
7、pages 所有的页面，这个不是我们之前的wxml文件了，主要是处理WXSS转换，使用js插入到header区域。

小程序架构
微信小程序的框架包含两部分View视图层、App Service逻辑层，View层用来渲染页面结构，AppService层用来逻辑处理、数据请求、接口调用，它们在两个进程（两个Webview）里运行。 
视图层和逻辑层通过系统层的JSBridage进行通信，逻辑层把数据变化通知到视图层，触发视图层页面更新，视图层把触发的事件通知到逻辑层进行业务处理。
小程序启动时会从CDN下载小程序的完整包，一般是数字命名的,如：_-2082693788_4.wxapkg

小程序技术实现
小程序的UI视图和逻辑处理是用多个webview实现的，逻辑处理的JS代码全部加载到一个Webview里面，称之为AppService，整个小程序只有一个，并且整个生命周期常驻内存，而所有的视图（wxml和wxss）都是单独的Webview来承载，称之为AppView。所以一个小程序打开至少就会有2个webview进程，正式因为每个视图都是一个独立的webview进程，考虑到性能消耗，小程序不允许打开超过5个层级的页面，当然同是也是为了体验更好。

AppService
可以理解AppService即一个简单的页面，主要功能是负责逻辑处理部分的执行，底层提供一个WAService.js的文件来提供各种api接口，主要是以下几个部分： 
消息通信封装为WeixinJSBridge（开发环境为window.postMessage, IOS下为WKWebview的window.webkit.messageHandlers.invokeHandler.postMessage，android下用WeixinJSCore.invokeHandler）

1、日志组件Reporter封装 
2、wx对象下面的api方法 
3、全局的App,Page,getApp,getCurrentPages等全局方法 
4、还有就是对AMD模块规范的实现

然后整个页面就是加载一堆JS文件，包括小程序配置config，上面的WAService.js（调试模式下有asdebug.js），剩下就是我们自己写的全部的js文件，一次性都加载。

在开发环境下
1、页面模板：app.nw/app/dist/weapp/tpl/appserviceTpl.js 
2、配置信息，是直接写入一个js变量，__wxConfig。 
3，其他配置 

线上环境
而在上线后是应用部分会打包为2个文件，名称app-config.json和app-service.js，然后微信会打开webview去加载。线上部分应该是微信自身提供了相应的模板文件，在压缩包里没有找到。 
1、WAService.js（底层支持） 
2、app-config.json（应用配置） 
3、app-service.js（应用逻辑）

然后运行在JavaScriptCore引擎里面。

AppView
这里可以理解为h5的页面，提供UI渲染，底层提供一个WAWebview.js来提供底层的功能,具体如下： 
1、消息通信封装为WeixinJSBridge（开发环境为window.postMessage, IOS下为WKWebview的window.webkit.messageHandlers.invokeHandler.postMessage，android下用WeixinJSCore.invokeHandler） 
2、日志组件Reporter封装 
3、wx对象下的api，这里的api跟WAService里的还不太一样，有几个跟那边功能差不多，但是大部分都是处理UI显示相关的方法 
4、小程序组件实现和注册 
5、VirtualDOM，Diff和Render UI实现 
6、页面事件触发

在此基础上，AppView有一个html模板文件，通过这个模板文件加载具体的页面，这个模板主要就一个方法，$gwx，主要是返回指定page的VirtualDOM，而在打包的时候，会事先把所有页面的WXML转换为ViirtualDOM放到模板文件里，而微信自己写了2个工具wcc（把WXML转换为VirtualDOM）和wcsc（把WXSS转换为一个JS字符串的形式通过style标签append到header里）。

Service和View通信
使用消息publish和subscribe机制实现两个Webview之间的通信，实现方式就是统一封装一个WeixinJSBridge对象，而不同的环境封装的接口不一样，具体实现的技术如下：

windows环境
通过window.postMessage实现（使用chrome扩展的接口注入一个contentScript.js，它封装了postMessage方法，实现webview之间的通信，并且也它通过chrome.runtime.connect方式，也提供了直接操作chrome native原生方法的接口） 
发送消息：window.postMessage(data, ‘*’);，// data里指定 webviewID 
接收消息：window.addEventListener(‘message’, messageHandler); // 消息处理并分发，同样支持调用nwjs的原生能力。 
在contentScript里面看到一句话，证实了appservice也是通过一个webview实现的，实现原理上跟view一样，只是处理的业务逻辑不一样。

'webframe' === b ? postMessageToWebPage(a) : 'appservice' === b && postMessageToWebPage(a)
1
IOS
通过 WKWebview的window.webkit.messageHandlers.NAME.postMessage实现微信navite代码里实现了两个handler消息处理器： 
invokeHandler: 调用原生能力 
publishHandler: 消息分发 
Android
通过WeixinJSCore.invokeHanlder实现，这个WeixinJSCore是微信提供给JS调用的接口（native实现） 
invokeHandler: 调用原生能力 
publishHandler: 消息分发

微信组件
在WAWebview.js里有个对象叫exparser，它完整的实现小程序里的组件，看具体的实现方式，思路上跟w3c的web components规范神似，但是具体实现上是不一样的，我们使用的所有组件，都会被提前注册好，在Webview里渲染的时候进行替换组装。 
exparser有个核心方法： 
regiisterBehavior: 注册组件的一些基础行为，供组件继承 
registerElement：注册组件，跟我们交互接口主要是属性和事件
组件触发事件（带上webviewID），调用WeixinJSBridge的接口，publish到native，然后native再分发到AppService层指定webviewID的Page注册事件处理方法。

总结
小程序底层还是基于Webview来实现的，并没有发明创造新技术，整个框架体系，比较清晰和简单，基于Web规范，保证现有技能价值的最大化，只需了解框架规范即可使用已有Web技术进行开发。易于理解和开发。

MSSM：对逻辑和UI进行了完全隔离，这个跟当前流行的react，agular，vue有本质的区别，小程序逻辑和UI完全运行在2个独立的Webview里面，而后面这几个框架还是运行在一个webview里面的，如果你想，还是可以直接操作dom对象，进行ui渲染的。

组件机制：引入组件化机制，但是不完全基于组件开发，跟vue一样大部分UI还是模板化渲染，引入组件机制能更好的规范开发模式，也更方便升级和维护。

多种节制：不能同时打开超过5个窗口，打包文件不能大于1M，dom对象不能大于16000个等，这些都是为了保证更好的体验。
