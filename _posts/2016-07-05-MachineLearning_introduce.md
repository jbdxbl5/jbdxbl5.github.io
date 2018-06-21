---
layout: post
title: "小程序小知识"
date: 2016-07-05 
description: "小程序小知识"
tag: 小程序小知识  
---     
1，小程序中没有的标签：a标签，div标签，需要点击元素跳转时有两种办法：
    ①在wxml页面中使用navigate标签
    例如：<navigator  url="../main/index">sdsfgf</navigator> 注意：url中的地址只能写在app.js中注册过的页面。
   不然报错：   
   ②绑定bindtap事件，然后在点击的时候使用wx.naviagateTo({})跳转到相应页面。
         
二。button的disabled属性的作用：比如将一个button作为表单的提交按钮，为了防止用户不停点击向服务器发送请求，所以在没有达到要求之前（比如输入内容不符合规则）,使其无法点击


三。var that=this; 有时看到这行，这个是因为this作用域问题。比如
bindonetap:functin(){
var that=this   
wx.getUserInfo({
 this._________________//这里边的this就是指向这个函数的
 that._________________//这里边的that就是指向外层函数的
})
}
四。onshareappmessage的path路径必须要写全。比如 path:'pages/share/share?id=3'(不要粗心，我因为把pages写成了page，一直找不到原因)


五。animation动画，
首先在wxml文件中应该有一个组件。<viewanimation="{{animationData}}"style="background:red;height:100rpx;width:100rpx"></view>
Page({})文件中的data中有一个animation属性，data: {animationData: {}}


六。可以直接使用this.__，即使这个__在上文没有定义过，也可以直接使用。


7.因为小程序代码的运行环境并不是浏览器，所以并没有window,document对象，即有些 js文件是没法用的。


8.替换字符串。只有replace()方法，没有replaceall()方法。


9.微信小程序  导航栏设置
在app.json中配置
{
  "window":{
    "navigationBarBackgroundColor": "#ffffff",
    "navigationBarTextStyle": "black",
    "navigationBarTitleText": "微信接口功能演示",
    "backgroundColor": "#eeeeee",
    "backgroundTextStyle": "light"
  }
}


10.小程序中使用的图片的名字前缀不要用中文


11.小程序中无法使用 <x< 只可以写成 <x&&x> 这种形式（不可以连着写）


12.wx.getSystemInfo({})和wx.getSystemInfoSync()获取的返回值有两种，screenheight(屏幕高度)和windowheight（屏幕可用高度）使用setinterval(functions(){},时间)的时候，时间参数可以选择40,看起来比较流畅
