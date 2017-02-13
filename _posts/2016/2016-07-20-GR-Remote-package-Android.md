---
layout: post
title: 明明都失业了，还蛋疼把 GR Remote 做成了APP
category: Android
tags: [WebView, GRRemote, 绑定, 重连, wifi]
---

##### 展示：

![](http://i.imgur.com/zVhqFvjm.png)
![](http://i.imgur.com/MbqxVNzm.png)
![](http://i.imgur.com/1gn8q8fm.png)


![Imgur](http://i.imgur.com/MMhMzv2.gif)

youtube
<iframe width="420" height="315" src="https://www.youtube.com/embed/JaI_4JfLw9g" frameborder="0" allowfullscreen></iframe>

##### 简介：
简单说就是把这个 [GR Remote](http://www.ricoh-imaging.co.jp/english/products/gr_remote/app/latest-appcache/index.html) 网页做了缓存，然后提供wifi绑定、重连功能。
以节省流量和时间，方便使用。


##### 使用方法：
- 初次使用要联网下载页面，请耐心等待。
- 自动连接相机网络这里，需要把页面滑动到底部，弹出设置页面，进入绑定列表，选择相机wifi进行绑定。

注：

   1. 要求系统版本6.0及以上；
   2. 这里的绑定，只能绑定连接成功过的wifi网络，因为考虑到人与人之间的信任，就不提供输入密码这种方式啦；



##### 起因：
把K5-IIs卖了，上个月买的GRII，拿出去真的太方便了。

装了一个 Image Syn ，可以同步照片，可是里边的远程控制功能无效，上周一的时候突然想起来在哪里看到说可以远程控制的，然后去官网找到了网页版的，号称可以实现除了开机和弹起闪光之外的全部功能。（其实还实现了触摸对焦、左右反转、按键锁定和关闭镜头）。


官方提供了[两个版本](http://www.ricoh-imaging.co.jp/english/products/gr_remote/index.html)，一个是普通网页版本，一个是DOM离线缓存，
但实际使用其实只能先联网打开网页，然后切换到相机的wifi热点，加载太慢了。

于是想把这个网页用WebView缓存，然后提供wifi绑定、重连功能。
这样就节省了流量和时间，不用手动连相机wifi啦。


##### 主要流程：
1. 启动时判断当前wifi状态，已打开则连接之前存储的wifi热点（如果有），关闭则打开；
2. 广播接收网络关闭和连接改变：网络关闭，尝试打开，打开则尝试连接绑定的热点；已连接到热点，判断是否是绑定的网络，如果不是则去连接绑定的网络；
3. 网页：载入网页，监听加载进度，加载完成时关闭启动动画并显示网页；
4. 监听滚动事件，判断是否到底部，是的话则弹出设置菜单；


##### 遇到的坑：
功能其实很简单，但是坑很多

1. 页面加载不完全，没有启用：`settings.setDomStorageEnabled(true)`；
2. 本来为了展示 想删除几个已配置的网络，结果搞了半天removeNetwork()方法都无效，查了一下发现在6.0里不给用了。http://stackoverflow.com/questions/32756690/android-m-unable-to-remove-wi-fi-ap-programmatically；
3. 好奇像 `wifiManager.isWifiEnabled()`这种方法是不是才加上的，网上的例子都是自己判断；
4. 系统4.2以后获取的SSID是双引号；


##### 现存问题：

1. 网页版把 顶部、左、右 都占用了，所以只好放在底部滑出。 只是使用了 `webView.setOnScrollChangeListener()`判断网页是否到达底部，问题就是
只能网页不在底部的时候才能划出，然后左右滑动的页面或者about页面 拖动也会弹出菜单；（解决方法是 判断到达底部后 等待2秒然后滚动回顶部，但觉得这样不太好）；
2. bottom sheet 有时会弹出多个；
3. 我的连接wifi策略是只要应用运行就会强制连接 绑定的wifi，这样万一没有退出而是切换到其他app了，wifi是没法改变的；

希望可以提供一下思路，不过找到工作前大概不会修复了。


##### 下载地址
[https://github.com/atever/GR-Remote-Package/blob/master/app/GR_Remote_0.1.apk](https://github.com/atever/GR-Remote-Package/blob/master/app/GR_Remote_0.1.apk)

##### 项目地址
[https://github.com/atever/GR-Remote-Package](https://github.com/atever/GR-Remote-Package)



##### 感想：
1. 有段事件没搞Android，深切体会到了前同事的那句“面向Google编程”；
2. 命名和结构都不太规范的问题，应该从一开始就规范好，后面项目越庞大越难以规范；
3. 看了下Image Syn这个APP，是用Cordova搞的，然后又有个网页版，既然都是网页为什么不整合在一起；



说一下标题，公司解散了，已经玩了两个星期，结果发现实在无法安心踏实的玩啊，
虽然工资没有准时发，但信用卡账单还是准时躺在邮箱了呢 （手动doge）。
