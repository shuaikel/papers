使用SDWebImage框架加载高分辨率图片时，会导致内存暴增，用instruments运行程序，可以发现出问题的代码时![](http://cc.cocimg.com/api/uploads/20160919/1474284143125961.png)
代码定位：
![](http://cc.cocimg.com/api/uploads/20160919/1474284161779974.png)

decodeImageWithImage 这个方法用于对图片进行解压缩并且缓存起来，以保证TableView/collectionViews交互更加流畅，但是如果是加载高分辨率图片的话，会适得其反，有可能造成上G的内存消耗，因此，对于这种高分辨率的图片，可以禁止解压缩操作，

```
[[SDImageCache sharedImageCache] setShouldDecompressImages:NO];

[[SDWebImageDownloader sharedDownloader] setShouldDecompressImages:NO];
```
根据Apple官方文档对CGBitmapContextCreate函数的注解：
![](http://upload-images.jianshu.io/upload_images/1278915-15b635405fb10ca5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

图中红框部分的参数：

```
bitsPerComponent 表示存入内存中的每个像素中的每一个组件所占的位数；

bytesPerRow 表示存入内存中的位图的每一行所占的字节数；

```

禁用解压缩相关代码：

```
static BOOL SDImageCacheOldShouldDecompressImages = YES;

static BOOL SDImagedownloderOldShouldDecompressImages = YES;

SDImageCache *canche = [SDImageCache sharedImageCache];
SDImageCacheOldShouldDecompressImages = canche.shouldDecompressImages;
canche.shouldDecompressImages = NO;

SDWebImageDownloader *downloder = [SDWebImageDownloader sharedDownloader];
SDImagedownloderOldShouldDecompressImages = downloder.shouldDecompressImages;
downloder.shouldDecompressImages = NO;
```
在需要恢复解压缩的地方恢复设置代码
```
    SDImageCache *canche = [SDImageCache sharedImageCache];
    canche.shouldDecompressImages = SDImageCacheOldShouldDecompressImages;

    SDWebImageDownloader *downloder = [SDWebImageDownloader sharedDownloader];
    downloder.shouldDecompressImages = SDImagedownloderOldShouldDecompressImages;
```

当然也可以设置SDWebImage的其他参数，比如是否缓存到内存以及内存缓存最高限制等，来保证内存安全：

shouldCacheImagesInMemory 是否缓存到内存
MaxMemoryCost  : 内存缓存最高限制

```

###### 苹果官方下载高清大图的Demo，内存消耗很低
[苹果官方下载高清大图的Demo](https://developer.apple.com/library/ios/samplecode/LargeImageDownsizing/Introduction/Intro.html)


