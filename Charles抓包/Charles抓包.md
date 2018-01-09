##Charles 抓包总结

**官网下载Charles安装包：[下载地址](https://www.charlesproxy.com/download/)**

**下载安装完成之后先设置HTTP抓包：**
>(1)查看电脑IP地址：![](https://upload-images.jianshu.io/upload_images/2469183-ff851ce2abe6cfe8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/672)


>(2)设置手机HTTP代理:手机连上电脑，点击“设置->无线局域网->连接的WIFI”，设置HTTP代理：如 192.168.1.125 端口：8888![](https://upload-images.jianshu.io/upload_images/2469183-ad19fa10a1815cbc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/350)
>
>设置代理后，需要在电脑上打开Charles才能上网。


>(3)电脑打开Charles进行HTTP抓包
>>手机上打开某个App或者浏览器什么的，如果不能上网，检查前面步骤是否正确
>>![](https://upload-images.jianshu.io/upload_images/2469183-8630cf0087d20187.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
>>点击“Allow”允许，出现手机的HTTP请求列表


**3.HTTPS抓包**
>HTTPS的抓包需要在HTTP的基础上在进行配置
>设置前抓包HTTPS是这样的![](https://upload-images.jianshu.io/upload_images/2469183-81c9d7cd686f86eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/636)



####以下为在HTTP抓包基础上进行HTTPS抓包的进一步设置步骤：
>点击Help->SSL Proxying-> Install Charles Root Certificate on a Mobile Device![](https://upload-images.jianshu.io/upload_images/2469183-8f47a1b1c1540ef7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

>出现弹窗得到地址 [地址](chls.pro/ssl) :  chls.pro/ssl ![](https://upload-images.jianshu.io/upload_images/2469183-c7f6ad4a204b0bd4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/699)

>在手机Safari浏览器输入地址：chls.pro/ssl,出现正式安装页面，点击安装手机设置有密码的输入密码进行安装![](https://upload-images.jianshu.io/upload_images/2469183-7ed4a5c8c2a36217.png?imageMogr2/auto-orient/strip%7Cimage![]()View2/2/w/249)
>>注意：在10.0.3系统，需要在设置->通用->关于本机->证书信任启用完全信任Charles证书。
>Charles设置Proxy
>
>Proxy->SSL Proxying Setting
>![](https://upload-images.jianshu.io/upload_images/2469183-2c460b4652797ccf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/549)
>勾选Enable SSL Proxying 点击Add
>![](https://upload-images.jianshu.io/upload_images/2469183-11eb2be75eae13fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/592)
>Host设置要抓取的Https接口，比如想抓这个
>![](https://upload-images.jianshu.io/upload_images/2469183-b39831342a11daca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/285)

>Host填写：https://api.weibo.cn

>Port端口填写：443
>![](https://upload-images.jianshu.io/upload_images/2469183-ca37de9cdb920511.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
>
>进行HTTPS抓包
>让手机重新发送https请求，可看到抓包


**注意：不抓包请关闭手机HTTP代理，否则断开与电脑连接后会连不上网**



