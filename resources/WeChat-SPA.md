# vue单页路由在微信上的坑

众所周知，微信公众号开发，里面的坑特多。当决定用 vue SPA 来重构我们的微信商城时，我内心是很忐忑的。但为了更好的用户体验，再多坑也值得躺一躺。以下是我整理的 vue SPA 在 微信端的坑。


由于 Android 6.2 以下的微信客户端不支持pushState的H5新特性，所以我们的 vue 路由使用默认的 hash 模式，所以链接路径是这样的：http://hostname.com/#/pagename?userId=123 具体的使用方法就不说了，直接说问题：


## 调用 JS-SDK 微信分享方法无反应。

项目中使用的是 onMenuShareTimeline、onMenuShareAppMessage 这两个方法。 

由于在其他无数个非 SPA 项目中使用过，因此这次直接引入我们结合业务后封装好的方法库。 

在微信开发者工具上测试， perfect，没有任何问题。

接着部署到测试公众号上，terrible，没有任何反应。

```
具体场景是这样的：
测试：你的代码有bug！
我：你的环境有问题吧，你会用吗？
过了一会儿…
测试：你这个程序和预期的有点不一致，你看看是不是我的使用方法有问题？
我：卧槽，是不是出bug了，我赶紧看看。
```

接下来就是debug了。 

微信返回的结果是：onMenuShareAppMessage：ok。 
这就奇怪了，方法调用成功，为啥就是没反应呢？ 
网上有说参数为空时，如 title:'' 这样是不行的，必须要这样 title: ' '。 

再debug，title、link、imgUrl 都正常。 

经过一通调试后发现是分享链接的问题，正常的链接如 http://hostname.com/pagename.html?userId=123 不经过 encodeURIComponent 转码是没有问题的，但 
http://hostname.com/#/pagename?userId=123 这样的链接就必须要 encodeURIComponent  转码。

改代码、打包部署、测试。

点击分享是有反应了，但是！不怕万一就怕但是啊！ 

分享出去的信息并不是我配置的信息，而是微信默认的分享。 

点击这个分享链接，并不是进入到我分享的那个页面。在此，引出了第二个坑。 

由于这两个坑互相关联，那就暂时不管这个但是了，是但啦。 

先看看第二个坑…

## 点击微信分享的链接并没有跳转到对应的页面

刚才分享出去的链接，传入onMenuShareAppMessage 方法的 link 参数是 http://hostname.com/#/pagename?userId=123  

然而我在进入路由前获取的 href 却是 http://hostname.com?from=singlemessage。 

WTF？这是什么鬼？ 

查了一下原来微信分享会在链接带上from=singlemessage、timeline 等参数判断分享来源； 

至此，恍然大悟。 

再看回我们的分享链接，微信在 location.origin 后面 插入参数，原来的 /#/pagename?userId=123 就被截掉了。 

这样的话我就只好把路由路径当成参数传进来再重新拼接。

画公仔不用画出肠，具体怎么做这里省略N行代码。

改代码、打包部署、测试。

点击分享，有反应，分享的信息也对。 

点击分享链接，成功进入到指定页面。

由此可见，第一个坑点击分享无反应的原因，就是因为分享链接没有符合相关规定基本法等。

后面还有很多坑，迟点再写。

