# iOS正式签名机制

## 证书（Certificate）
**1、证书 (Certificate)** 
在 Xcode 8 之前，每个账号一般有两个证书，一个是开发证书，一个是发布证书。开发证书和发布证书都只能存在一份，所以如果有多台 Mac 开发设备，需要通过证书的导出导入来同步证书（和密钥）。在 Xcode 8 之后，支持多个 开发证书 (发布证书依然只能有一个)，也就是说，多台 mac 开发设备可以 自动 生成多份有效的开发证书（和密钥），就不再需要导出导入了。

**2、Provisioning profile**
用来授权给包含在 profile 里的 iOS 开发设备来安装 app。在 Xcode 8 之前，每次添加新的设备都会生成新一个新的 profile，并产生一个唯一的 id，所以在每次添加设备之后，因为 profile id 变了，需要更新并提交 project 文件，Xcode 8 以后是用文件名的方式引用，就是说添加了新的设备，只要 profile 文件名不变，就不会修改 project 文件了。这也算是一个比较方便的改进吧。

**3、Entitlement**
其实就是管理我们开启的 Capabilities，比如 IAP，Push Notifications，iCloud 等等

## 签名的流程
签名的流程 首先 Xcode 根据所择的 team 从 key chains 里选择 最新的 证书，然后根据 app identifier 选择 最新的 provisioning profile，在 build 的时候 profile 会被放在 app 包里，code sign 工具根据证书生成一个 code seal（可以理解为盖一个戳）。如果有人篡改了 app，这个戳就不 match 了，iOS 系统会阻止 app 安装。


##自动化签名 (Automatic Singing)
> 在这种模式下，Xcode 全自动的为我们管理整个签名的流程，整个过程会在后台执行，会保证所有签名需要的文件是最新的。
> 
> 
> 我们所需要做的就是勾选上自动化签名，然后选择 team。剩下的 Xcode 都会接管。比如创建证书，创建和更新 profile 等等。但是当插入了一台新的 iOS 设备，Xcode 8 还是会提示是否把这台设备添加到测试设备中，如果选择是，Xcode 8 会自动添加到设备列表里，并自动更新 profile 文件。
> 
> 
> Xcode 自动化签名只会自动化开发阶段的签名，不会修改发布的签名设置。既然这样，如何设置 release 版本的签名呢？其实我们在 Archive 的时候，Xcode 默认使用的还是开发证书做的签名，然后在 Orgnizer 里选择 export 到 App Store 发布版本的时候，会让我们重新选择 证书重新签名，这里再选择发布证书。
> 
> 
> 越狱使应用可以绕过代码签名和沙盒安全机制的全部限制
> 
> 
> 
> 
> 



###概括的讲，一个证书是一个公钥加上许多附加信息，这些附加信息都是被某个认证机构（Certificate Authority 简称 CA）进行签名认证过的，认证这个证书中的信息是准确无误的。对于 iOS 开发来说这个认证机构就是苹果的认证部门 Apple Worldwide Developer Relations CA。认证的签名有固定的有效期，这就意味着当前系统时间需要被正确设置，因为证书是基于当前时间进行核对。这也是为什么将系统时间设定到过去会对 iOS 造成多方面破坏的原因之一。

>>~/Library/MobileDevice/Provisioning Profiles。Xcode 将从开发者中心下载的全部配置文件都放在了这里
>>
>>不要惊讶，配置文件并不是一个 plist 文件，它是一个根据密码讯息语法 (Cryptographic Message Syntax) 加密的文件（下文中会简称 CMS，但不要用这个简写 Google，这不是一个很好的关键字）。如果你处理过 S/MIME 邮件或者证书你会对这种加密比较熟悉，详细信息可以查看互联网工程任务组 (IETF) 制定的 RFC 3852。

>>采用 CMS 格式进行加密使得配置文件可以被设置签名，所以在苹果给你这个文件之后文件就不能被改变了。配置文件的签名和应用的签名不是一回事，它是由苹果直接在开发者中心 (developer portal) 中设置好了的。
>>
>>某些版本的 OpenSSL 可以读取这种格式，但是 OS X 自带那个版本并不行。幸运的是命令行工具 security 也可以解码这个 CMS 格式，那么我们就用 security 来看看一个 .mobileprovision 文件内部是什么样子：
>>
>>security cms -D -i example.mobileprovision[provision文件路劲];
>>
>>>>1.首先来看 DeveloperCertificates 这项，这一项是一个列表，包含了可以为使用这个配置文件的应用签名的所有证书。如果你用了一个不在这个列表中的证书进行签名，无论这个证书是否有效，这个应用都无法运行。所有的证书都是基于 Base64 编码符合 PEM (Privacy Enhanced Mail, RFC 1848) 格式的。
>>>>
>>>>2.Entitlements 一项中包含了你的应用的所有授权信息，键值就和之前在授权那节看到的一模一样。这些授权信息是你在开发者中心下载配置文件时在 App ID 中设置的，理想的情况下，这个文件应该和 Xcode 为应用设置签名时使用的那一个同步，但这种同步并不能得到保证。这个文件的不一致是比较难发现的问题之一。举例来说，如果你在 Xcode 中添加了 iCloud 键值对存储授权 (com.apple.developer.ubiquity-kvstore-identifier)，但是没有更新，重新设置并下载新的配置文件，旧的配置文件规定你的应用并没有这一项授权。那么如果你的应用使用了这个功能，iOS 就会拒绝你的应用运行。这也是当你在开发者中心编辑了应用的授权，对应的配置文件会被标记为无效的原因。
>>>>
>>>>如果你打开的是一个用于开发测试的证书，你会看到一项 ProvisionedDevices，在这一项里包含了所有可以用于测试的设备列表。因为配置文件需要被苹果签名，所以每次你添加了新的设备进去就要重新下载新的配置文件。


