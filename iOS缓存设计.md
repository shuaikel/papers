###### 磁盘缓存技术设计方案

> 在大量的开源数据磁盘缓存开源库中，包括TMDiskCache，PINDiskCache、SDWebImage、FastImageCache等，也包括一些闭源的实现，包括NSURLCache、Facebook的FBDiskCache等。他们的实现技术大致分为三类：基于文件读写、基于mmap文件内存映射、基于数据库。
> 
> TMDiskCache、PINDiskCache、SDWebImage等缓存，都是基于文件系统的，即一个value对应一个文件，通过文件读写来缓存数据，他们的实现都比较简单，性能也都相近，缺点也是同样的：不方便扩展，没有元数据、难以实现较好的淘汰算法，数据统计较慢。
> 
> FastImageCache采用的是mmap将文件映射到内存，
> 	
> 	+ mmap（内存映射文件）的缺陷：
> 		
> 		+ 热数据的文件不要超过物理内存大小，不然mmap会导致内存交换严重降低性能；另外内存中的数据是定时flush到文件的，如果数据还未同步时程序挂掉，就会导致数据错误。抛开这些缺陷来说，mmap性能很高。
> 
> 
> NSURLCache、FBDiskCache都是基于SQLite数据库的。基于数据库的缓存可以很好的支持元数据、方便扩展、数据统计速度快，也很容易实现LRU或其他淘汰算法，唯一不确定的就是数据库读写的性能，在实际测试下，当单条数据小于20K时，数据越小SQLite读取性能越高；单条数据大于20K时，直接写为文件速度会更快一些，这和[SQLite 官网的描述](http://www.sqlite.org/intern-v-extern-blob.html)基本一致,另外，直接从官网下载最新的SQLite源码编译，会比iOS系统自带的sqlite3.dylib性能要高很多。基于SQLite的这种表现，磁盘缓存最好是把SQLite和文件存储结合起来：key-Value元数据保存在SQLite中，而Value数据则根据大小不同选择SQLite或文件存储。NSURLCache选定的数据大小的阈值是16K，FBDiskCache则把所有value数据都保存为了文件。
> 
> YYDiskCache也是采用的SQLite配合文件的存储方式，在实际基准测试下，在存取较小数据（NSNumber）时，YYDiskCache的性能远远高出基于文件存储的库；而较大数据的存储性能则比较接近，但是得益于SQLite存储的元数据，YYDiskCache实现了LRU淘汰算法、更快的数据统计，更多的容量控制选项。
> 
> 
> 
> 
> 



#####备注
：关于锁
：OSSPinLock自旋锁，原理很简单，一直do while忙等。它的缺点是等待时会消耗大量的CPU资源，所以它不适合较长时间的任务。对于内存缓存的存取来说，非常合适

：dispatch\_semaphore是信号量，但当信号量总量设为1时也可以当做锁来用，在没有等待情况出现时，它的性能比pthread\_mutex还要高，但一旦有等待情况出现时，性能就会下降许多。相对于OSSPinlock来说，它的优势在于等待时不会消耗CPU资源，对于磁盘缓存来说，它比较合适。

： Realm 是一个比较新的数据库，针对移动应用所设计。它的 API 对于开发者来说非常友好，比 SQLite、CoreData 要易用很多，但相对的坑也有不少。我在测试 SQLite 性能时，也尝试对它做了些简单的评测。我从 Realm 官网下载了它提供的 benchmark 项目，更新 SQLite 到官网最新的版本，并启用了 SQLite 的 sqlite3\_stmt 缓存。我的评测结果显示 Realm 在写入性能上差于 SQLite，读取小数据时也差 SQLite 不少，读取较大数据时 Realm 有很大的优势。当然这只是我个人的评测，可能并不能反映真实项目中具体的使用情况。我想看看它的实现原理，但发现 Realm 的核心 realm-core 是闭源的（评论里 Realm 员工提到目前有在 Apache 2.0 授权下的开源计划），能知道的是 Realm 应该用 了 mmap 把文件映射到内存，所以才在较大数据读取时获得很高的性能。另外我注意到添加了 Realm 的 App 会在启动时向某几个 IP 发送数据，评论中有 Realm 员工反馈这是发送匿名统计数据，并且只针对模拟器和 Debug 模式。这部分代码目前是开源的，并且可以通过环境变量 REALM\_DISABLE\_ANALYTICS 来关闭，如果有使用 Realm 的可以注意一下。

