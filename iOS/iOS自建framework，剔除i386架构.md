

<label style="color:#333;font-size:1rem;font-weight:600;">查看framework包含的平台</label>

<p style="line-height:20px;letter-spacing:0.5px;font-size:14px;font-weight:500;">有些第三方提供商为了方便开发者使用，经常将i386 x86_64 armv7 arm64等几个平台合并到一起，但是上传App Store的时候需要将i386 x86_64 两个平台删除后，才能正常审核</p>

或者

cd XXXX.framework

lipo -info XXXX


然后终端就会提示该framework集成了哪几个：

rchitectures in the fat file: XXXX.framework/Realm are: ******


##### 执行以下命令进行剔除(在xxx.framework一层执行)：


###1. mkdir ./bak

###2. cp -r xxxx.framework ./bak


###3. lipo xxxx.framework/xxxx -thin armv7 -output xxxx\_armv7

###4. lipo xxxx.framework/xxxx -thin arm64 -output xxxx\_arm64

###5. lipo -create xxxx\_armv7 xxxx\_arm64 -output xxxx

###6. mv xxxx xxxx.framework/


# iOS查看类的c++实现命令：
```
xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc Person+eat.m
```


1、扩容，把类中的方法数组和分类中的方法数组计算出来

2、memmove把类中方法放到数组的最后一位.

3、memcpy把分类中的方法放到数组的前面。








