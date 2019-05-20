#### 1.相关证书

**证书：**对iOS项目开发、发布资格进行授权的，主要会用到下面两种：一个是Development开发证书，另外一个就是Distribution发布证书。具体存在形式就是我们开发人员经常收到并双击安装的那些P12证书。


**描述文件：**包含了证书信息，App ID，具备调试、测试权限的设备，以及应用的一些功能信息等，后缀名为Provisioning Profile，双击安装后，其实拷贝到了*~/Library/MobileDevice/Provisioning Profiles*目录下：

![](https://upload-images.jianshu.io/upload_images/7079027-aeb1a75ac1333c01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/804)

**重签名的解释：**


当在XCode进行archive或者通过脚本打包ipa的时候，通常到了最后一步，要对包有一个签名过程，对签名起到关键作用的是配置好证书和描述文件。也就是一套证书，描述文件最终签名好一个对应的ipa。

而所谓的**重签名的概念就是，可以把一个已经存在的ipa重新配置一套证书和描述文件，在签名生成一个新的ipa包**。

#### 2.开发背景

> 当开发中，我们通常需要打出好几种签名包：**Development（开发证书签名包），Distribution包（用于提审苹果商店包），有时候还会用企业证书签名包给一些运营人员**，这样三个包就需要**三倍的打包时间，效率非常低。**
> 
> 特别当包体积比较大时，耗费的时间是很大的，非常影响开发的效率。


所以我们目前的实践方案是：

+ 1、正式提审前，还按照之前正常的打包流程用dis（发布）证书打出一个ipa，用于正常的提审。
+ 2、然后利用重签名工具重新签名为dev类型的包，两个包的二进制，以及资源、功能完全一样，可以交付给QA测试。
+ 3、若运营或渠道有需求，继续可以用企业证书重新签名，做分发。


#### 3、需要准备的材料

**Provisioning Profile：**用于重签名的描述文件。

**Signing Certificate: **重签名证书文件

手动重签过程：

+ 1、第一步，把我们需要重签名的ipa包和我们下载下来安装的 embedded.mobileprovison放在同级文件夹目录下。

+ 2、第二步，终端cd到这个ipa的文件夹目录下，执行sigh resing或者使用fastlane 命令：**fastlane sigh resign**命令。

+ 3、第三步，这时候，sigh会直接弹出这个指令要你输入：Signing Identity这个就是你的证书的十六进制串，输入之后回车，然后就可以了



## iOS使用企业证书为包含多个target的应用重签名

当重签名的包使用了app group功能时，App Group主要用于应用共享数据等，这种应用一般有一个主target跟着一个extension组成。尝试直接签名的话会失败，再查看app group的资料，新建app group：

![](https://upload-images.jianshu.io/upload_images/3880746-793aa89604105f3f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

随后输入App group的ID，注意要按照规范输入
![](https://upload-images.jianshu.io/upload_images/3880746-0a60205258379e6b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

新建完成后，点击app ids，新建两个app id，按照规范，两个id的前缀需要一致，比方说：
aaa.bbb.ccc和aaa.bbb.ccc.extension。

随后分别点击两个id，点编辑：

![](https://upload-images.jianshu.io/upload_images/3880746-90ac0eb4a112b841.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)


添加app group，并通过edit选择希望加入的group:

![](https://upload-images.jianshu.io/upload_images/3880746-b691cdc42ca5cf22.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

![](https://upload-images.jianshu.io/upload_images/3880746-2bb4054fec684346.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

加入完成后，也就新建成功了，随后导出两个id对应的provision文件，

使用命令行重新打包：

```
sigh resign .ipa文件路径 --signing_identity "公司名称" -p 原extension的bundle_id=给extension打的provision文件 -p 主target bundle_id=主target的provision文件
```





### 其它的重签名方案

+ <a href="https://github.com/maciekish/iReSign" style="color:#2f9898">iResign</a>

+ <a href="https://github.com/DanTheMan827/ios-app-signer" style="color:#2f9898">iOS App Signer</a>



#### 命令行相关

+ security find-identity -v -p codesigning 

> 在终端使用此命令，可以挑选出所需签名的证书名字




#### 签名失败可能的问题以及解决方案

+ 1. 目标机有多个版本xcode，命令行环境下没有select对应的当前的xcode版本：

> /Applications/Xcode.app/Contents/Developer


如果发现指定版本不是当前所用xcode，就使用以下命令指定xcode：

> sudo xcode-select -switch /Applications/XcodeXXX.app/Contents/Developer 


+ 2. 缺少Apple Worldwide Developer Relations Certification Authority证书

检查是否安装AppleWWDRCA.cer：

> security find-certificate -c "Apple Worldwide Developer Relations 
Certification Authority"


#### 重签名过程中可能用到的有用的命令

+ 1. 查看app文件信息

> codesign -vv -d /xxx/xxx.app 

+ 2. 查看描述文件信息

> security cms -D -i /xxx/xxx.mobileprovision







