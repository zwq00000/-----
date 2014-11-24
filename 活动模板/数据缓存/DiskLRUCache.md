磁盘缓存
=============

磁盘缓存是指在文件系统上实现的一个 key/Value 列表。key 用来指示文件名称，value 就是一个具体的文件。

##缓存基本操作

缓存管理器提供一些基本功能
 - 使用空间，根据资源热度（读取命中数量）清理过期和不常用的缓存内容。
 - 命中次数


DiskLRUCache 使用 journal 日志文件记录缓存事件，并据此计算命中次数，算法比较简单实用，但是缺乏扩展性。

##应用场景 —— Web访问缓存
通常可以计算 Url Hash 值作为Key,可以使用 MD5 进行Hash计算。
每个缓存条目使用两个文件，0-保存Web数据。1-用于缓存Http响应内容，或者更直接的保存 LAST-MODIFY 值。
每个应用或者站点使用一个缓存空间（站点/应用名称当做缓存目录名称）。
Web数据缓存通常不会进行修改，所以只需要一个只读缓存就可以了，一次性数据缓存比较简单，避免多线程访问导致的冲突。
在 AndroidAsync 项目中使用了一个更简单的缓存类 FileCache,基本结构是一样的，但是没有使用日志功能，也不提供命中次数计数，适用于更简单的应用场景。

##用法示例##

```java

    DiskLruCache cache = DiskLruCache.open(cacheDir, appVersion, valueCount, CACHE_SIZE);
    //写入缓存数据
    DiskLruCache.Editor creator = cache.edit(cache_key);
    creator.set(0, http_header);
    OutputStream output = creator.newInputStream(1);
    //output 写入缓存数据
    // ...
    output.close();
    creator.commit();

    //读取
    DiskLruCache.Snapshot snapshot = cache.get(cache_key);
    snapshot.getString(0);          //读取摘要信息
    InputStream input = snapshot.newOutputstream(1);    //获取流
    //处理流数据
    // ...
    snapshot.close();   //关闭基础流

```

##项目获取
>[DiskLruCache][https://github.com/JakeWharton/DiskLruCache]
>git clone git@github.com:JakeWharton/DiskLruCache.git

##运行库依赖
无


#Disk LRU Cache Readme
LRU磁盘缓存，使用有限的空间在一个文件系统。每个高速缓存条目
有一个字符串键和一个固定的数量的值。每个键必须满足正则表达式 `[a-z0-9_-]{1,64}`。
值的字节序列，可作为流或文件进行访问。
每个值必须在 '0' 和`Integer.MAX_VALUE` 字节长度的整数。
缓存中数据存储在文件系统的目录上。这个目录必须要缓存独占的，缓存可以删除或覆盖其中的文件。
多个进程同时使用相同的缓存目录会发生错误。

这个缓存限制的字节数，它将在文件系统上的文件存储。
当存储字节数超过限制，缓存将在后台删除条目直到满足空间限制。
空间限制不严格：在等待文件删除时缓存空间可以暂时超过该限制。
限制不包括文件系统开销或缓存日志，空间敏感的应用应该设定一个保守的限制。
客户端调用`edit`创建或更新记录的值。一个条目可能在一个时间只有一个编辑；如果值不可编辑`edit`将返回null。
*当一个条目被**Created**是必要的创建提供全部数据；如果必要空值应该被用来作为一个占位符。
*当一个条目被*edited*，这是没有必要的供应数据每一个值；值默认为他们以前的价值。
每一`编辑`电话必须通过调用`编辑提交`或匹配。
`中止`编辑。承诺是原子的：读观察值的全套
他们之前或之后的承诺，但没有一个混合值。
客户端调用`得到`读入口的快照。读将观察时的价值，`得到`被称为。在电话和移动更新做不影响正在进行的读取。这个类是宽容一些I/O错误。
如果文件丢失的文件系统，相应的条目将被从缓存下降。如果一个
在写高速缓存的值时发生错误，编辑会静静的失败。调用者应捕获`IOException`并进行适当的处理。
*注：此实现具体目标的兼容性。*

##如何获得##

你可以在你的项目库，[下载] [ ]。罐罐。
如果你是一个Maven用户还可以添加这个图书馆作为一个依赖。添加
在你的` XML ` POM：

```xml
<dependency>
  <groupId>com.jakewharton</groupId>
  <artifactId>disklrucache</artifactId>
  <version>(insert latest version)</version>
</dependency>
```

如果你想自己编译的版本，可以通过内置的库
运行` mvn clean verify`。输出罐将在`target/ `目录。
*（注：这需要安装Maven）*
许可证
=======
版权所有2012杰克沃顿商学院
版权所有2011 Android开源项目
在Apache许可证授权下，版本2（“许可证”）；
你可以不使用此文件的许可遵守。
你可能会获得一份许可证的拷贝在
HTTP://www.apache.org /许可证/ license-2.0
除非适用的法律要求或书面同意，软件
分发的许可证发放的一个“是”的基础上，
没有任何形式的保证或条件，无论是明示或暗示。
查看许可证管理权限的具体语言
在许可的限制。