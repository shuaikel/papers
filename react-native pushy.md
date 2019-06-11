+ npm install -g react-native-update-cli rnpm

+ npm install --save react-native-update

+ 使用：

+ pushy bundle
+ > 生成资源包 platform：ios|android 对应的平台
+ > entryFile:入口脚本文件
+ > intermediaDir：临时文件输出目录
+ > output：最终ppk文件输出路径
+ > dev: 是否打包开发版本
+ > verbose:是否展现打包过程的详细信息

+ pushy diff
+ 提供两个ppk文件，生成从origin到next版本的差异更新包。
+ output：diff文件输出路径

+ pushy diffFromApk
+ 提供一个apk文件和一个ppk文件，生成apk文件到next版本的差异更新包。如果使用热更新开放平台，你不需要自己执行此命令：
+ > output: diff文件输出路径

+ pushy diffFromIpa
+ 提供一个ipa文件和一个ppk文件，生成从ipa文件到next版本的差异性更新包，如果使用热更新开放平台，你不需要自己执行此命令：
+ output:diff文件输出路径

+ pushy login [][]
+ 登录热更新开放平台，

+ pushy logout
+ 登出并清除本地的登录信息

+ pushy me
+ 查看自己是否已经登录，以及昵称信息

+ pushy createApp
+ 创建应用并立刻绑定到当前工程，
+ platform : ios|androif对应的平台
+ name：应用名称
+ downloadUrl：应用安装包的下载地址


+ pushy deleteApp [appId]
+ 删除已有应用，所有已创建的应用包，热更新版本都会被同时删除，
+ platform：ios|android对应的平台

+ pushy apps
+ 查看当前已创建的全部应用，这项操作也可以在网页端进行
+ platform：ios|android 对应的平台

+ pushy uploadIpa 上传apk文件到开放平台

+ pushy packages
+ 查看已经上传的包，这项操作也可以在网页端进行
+ platform：ios|android 对应的平台

+ pushy publish
+ 发布新的更新版本
+ platform：ios|android对应的平台
+ name：当前版本的名称（版本号）
+ description：当前版本得描述信息，可以对用户进行展示
+ metaInfo：当前版本得元信息，可以保持一些额外信息

+ pushy versions
+ 分页列举可用的版本，
+ platform：ios|android

+ pushy update
+ 为一个包绑定一个更新版本，这项操作也可以在网页端进行
+ platform：ios|android对应的平台
+ versionId：要绑定的版本ID
+ packageId：要绑定的包ID