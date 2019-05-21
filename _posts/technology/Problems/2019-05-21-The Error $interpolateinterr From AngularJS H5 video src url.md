---
layout: post
title: The Error: $interpolate:interr From AngularJS H5 video src url
date: 2019/5/21 11:12:09 
categories: problem
tag: 踩过的坑

---


* content
{:toc}


最近写一个后台管理项目，前后端分离项目，前端架构是：AngularJS+ H5，列表需要展示图片和视频，后端获取列表数据返回给前端，前端进行展示，图片展示无误，但是到了视频展示的时候，出现了bug，一直展示不出来。仔细检查后端代码无误，能正确返回数据，浏览器控制台打印输出数据，视频链接展示无误，拷贝视频链接在浏览器地址栏里就可以访问，唯独放到项目里面就不能展示，发现浏览器控制台已经报错了。错误信息如下图：
 ![](/styles/images/problem/angularjs_error/0.png)

对于一个写后端的来说确实有点头疼。。。。

h5代码如下：

```
<td ng-if="v.leftIsVido== true">
   <video class="video" autoplay="autoplay" width="80" height="60" >
   	 <source ng-src="{{v.leftFileUri}}" type="video/mp4">
   </video>
 </td>

```

好在控制台的异常链接可以点击访问，是AngularJS的异常帮助文档之类的，点进去如下图，异常解释：
 ![](/styles/images/problem/angularjs_error/1.png)

大致意思为：在angularJs中为了避免安全漏洞，一些ng-src或src或者ng-include都会进行安全校验，因此常常会遇到ng-src无法使用的情况。
说白了就是用angular里面的 ng-src 时会进行安全检查。因为我们是访问资源服务器上面的视频所以它不给这个url通过，所以我们就播放不了视频。

对于后端的我，到此就嗝屁了，问题虽然知道了，但是不知道怎么处理。然后就找来前端大神求助。（好在还认识几个前端大神），问题给大神叙述一遍，各种异常信息截图发过去，问题原因给大神描述一遍，（反正就是一顿乱聊）大神果然是大神，立马给出了解决方案，用服务sce防止注入。
后端的小伙伴肯定会问sce是什么鬼？我也很蒙啊，查了一下sce服务，大致意思如下：
$sce服务把一些地址变成安全的、授权的链接，简单地说，就像告诉门卫，这个陌生链接其实是我信任的链接，很值得信赖，不必拦截它，放他通过！
sce常用的方法有：

$sce.trustAs(type,name);
$sce.trustAsHtml(value);
$sce.trustAsUrl(value);
$sce.trustAsResourceUrl(value);
$sce.trustAsJs(value);

很明显我们是用来处理视频链接的应该使用$sce.trustAsUrl(value)或$sce.trustAsResourceUrl(value);这两个方法。

大神给出了解决思路，但是另一个问题来了我这可是列表循环展示的。没办法试试for循环用上面两个方法处理一下链接吧。这一试惊呆了，成了！，具体代码如下：
h5的代码,对视频链接增加一个videoUrl()进行处理：

```
<td ng-if="v.leftIsVido== true">
   <video class="video" autoplay="autoplay" width="80" height="60" >
   	 <source ng-src="{{videoUrl(v.leftFileUri)}}" type="video/mp4">
   </video>
 </td>

```
js代码，从后台获取的数据，for循环取出链接，定义一个videoUrl方法对取出链接进行处理，用$sce.trustAsResourceUrl(url)来对视频链接进行免安全检查处理，具体代码如图：

 ![](/styles/images/problem/angularjs_error/2.png)


然后Ctrl+F5强刷一下页面，视频正确展示。

如图，图片和视频都能正常展示了

 ![](/styles/images/problem/angularjs_error/3.png)

项目急着上线，顺利解决了bug，还好没有造成拖延！