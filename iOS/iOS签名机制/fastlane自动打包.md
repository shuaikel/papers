**Fastlane安装**
>Fastlane是一套工具，帮助你简化和自动化App发布或部署的过程，将之变成一条平直的工作流
>
>Fastlane运行要求：
>>OS X 10.9 (Mavericks) 以上
>>Ruby 2.0 以上
>>Xcode
>>拥有一个付费的苹果开发者账号
>>


>因为 fastlane 其实是一个 Ruby 脚本的集合，你必须安装正确的 Ruby 版本。 OS X 10.9(Mavericks) 以后默认安装的是 Ruby 2.0 本。你可以在终端窗口中用下列命令来确认：
>>ruby -v

>然后检查 Xcode 命令行工具(CLT)是否安装。在终端窗口中输入命令：
>>xcode-select --install
>>>如果 Xcode CLT 已经安装，则会报如下错误：command line tools are already installed, use "Software Update" to install updates. 
如果未安装，终端会开始安装 CLT。 

>安装Fastlane
>>sudo gem install fastlane --verbose 
>>>如果报错：ERROR: While executing gem ... (Errno::EPERM) Operation not permitted - /usr/bin/commander
>sudo gem install -n /usr/local/bin fastlane  
>>安装结束后，查看fastlane版本：
>>fastlane --version 
>>



**在项目中使用Fastlane**
>>cd到项目文件夹： fastlane init
>>>需要按照提示输入 AppID以及密码, 这个是你项目的开发者帐号，下边要输入项目的bundleIdentifier，然后出现了提示
>>>
>>>

**Fastlane基础组件**
>1) 测试工具
>>scan：自动运行测试工具，可以生成漂亮的HTML报告

>2) 生成证书、配置文件工具
>>cert：自动创建iOS代码签名证书(.cert文件)

>>sigh：自动创建、更新、下载、修复Provisioning Profile

>>pem：自动生成、更新推送配置文件


>3)截图、描设备边框
>>deliver：上传截图、元数据、App到iTunesConnect

>>snapshot：使用UI test功能实现自动截图

>>frameit：在截图的图片外层套上物理设备边框

>4)自动化编译工具
>>gym：自动化编译工具

>5) App公测工具
>>pilot：管理TestFlight测试用户，上传二进制文件

>>firim：管理firim


>为我们的截图加上物理设备边框
>>frameit将帮助我们为App截图构建漂亮的设备边框，只需要运行命令：fastlane frameit


>


**安装Fastlane fir插件**
>fastlane add_plugin fir


