---
title: 「ElasticSearch」ES内存分布及index加载方式
date: 2023-07-27 20:42:32
tags: [大数据, ElasticSearch]
---
#### 1. index内部存储
ES中一个index由多个shard组成，而一个shard又由多个segment组成，一个segment是一部分document数据，segment中会包含文档、域和词，即一个index按层次结构来看为：索引(index) -> 分片(shard) -> 段(segment) -> 文档(document) -> 域(Field) -> 词(Term)
![](YEYjeBD632Kfku1uxoP6EqxOmtFnP1t0DaY6hgyAsRI.jpg)
段(segment):
数据加载的单元，里面包含倒排索引等数据。当ES 收到数据时，会先将数据放入内存中(index buffer), 当到达指定的时间(index.refresh_interval配置的时间)后写入到segment, 该时间段一次生成一个segment，此时segment存在filesystem cache中，当到达指定的时间(index.translog.sync_interval配置)或指定大小(index.translog.flush_threshold_size配置)后刷新到磁盘。segment在后台自动merge成更大的segment
![](pFgm9c6JWO_6U9HAO7rdfSI1lkycJjt2H_77zCxtuHg.png)
#### 2. ES内存分布
ES的内存分为堆内内存和堆外内存，其中堆内内存主要是写入缓冲、cache等相关内存，受JVM的GC管理，该部分内存会出现OOM。堆外内存主要是lucene在使用，使用了操作系统的page cache，其大小影响查询性能，堆外内存受操作系统管理，如果操作系统内存不足会释放page cache内存，不会出现OOM问题。
细分来看，堆内内存主要有五部分组成：公共内存、fielddata缓存、query缓存、request缓存和segment相关的一些内存。堆外内存主要有两部分：加载segment相关的内存和查询相关的内存。
![](79GlWWj0abyzBfUfD7HrvBKvWels5841DTRRYMjXjMw.jpg)
##### 
common 内存
(1) index buffer
索引写入缓冲区，用于存储新写入的文档，当其被填满时，缓冲区中的文档被写入磁盘中的 segments 中。节点上所有 shard 共享。这部分空间是可以通过GC被反复利用的。默认大小为堆内存的10%。
```
indices.memory.index_buffer_size: 10%
//写入segment的缓冲区大小，到达这个大小会刷盘
indexing_pressure.memory.limit: 10%
//在处理中的document的大小，大于这个大小会限流，客户端返回429错误
```
(2) 线程池缓存及队列内存
ES使用各种内部有很多线程池，这部分线程池会占用部分内存，分析大约有几百兆
![](J2N-ig41aHAJj9_M59O1KJZNxvQqQUeqjR3-Bb7WTv0.png)
(3) PageCacheRecycler内存
page cache内存回收管理器，通过PageCacheRecycler这个类来管理page cache，内部将内存以页为单位管理和回收，缓存一页大小是16KB，这部分内存记录一些页缓存的信息
![](xrKYVZ8Wj4X_FfrgsDVMZv-vrBCbrMzKpVnDh_W1SuQ.png)
(4) metadata内存
ES的metadata主要包括indexc的mapping、shard信息，这部分内存不大，但是当shard非常多的时候也会变得比较大，metadata在每个节点上都存在一份数据，当有index创建、关闭或删除的时候，metadata都会由master节点更新然后推送到各个节点，所以当shard过多的时候，这个更新推送的过程就会变慢，从而影响es的性能。另外metadata会间隔一段时间持久化一次数据，数据写到磁盘上去，当节点重启后会从磁盘恢复metadata.
```
总结：common内存=index buffer + indexing_pressure.memory + 线程池缓存及队列内存 + PageCacheRecycler内存 + metadata内存 + 其它的一些运行内存
```
```
indices.breaker.total.limit: 70%
// 父熔断，inflight、request(agg)和fielddata不会使用超过堆内存的 70%  超过这个值报400 Data too large
network.breaker.inflight_requests.limit: 100%
// 限制请求内容的长度
```
##### fielddata cache
这部分内存分为两部分，一部分为fielddata字段的内存，另一部分为全局序数的内存。
fielddata字段：在ES写人数据时，当遇到keyword类型的字段时，ES会把字段值插入倒排索引和doc_values中，当做聚合的时候会使用doc_values对值做聚合，但是当遇到text字段的到时候，由于text字段一遍较大，所以ES不会将数据写入doc_values, 这个时候text类型的字段就不知道排序聚合了，为了解决这个问题，ES提供fielddata功能，单独维护一份text类型的数据，这部分数据时加载到内存中的使用fielddata cache，当第一次查询时生成。
全局序数：全局序数是跨多个segment的全局字段映射表。ES会在写入数据时对字段生成一个自增的数字，将这个数字个字段存入全局序数中，聚合时用全局序数聚合即可。全局序数存储在fielddata cache中，默认当一次查询时生成，也可在构建数据时候生产。
```
indices.fielddata.cache.size: 默认无上限
//默认内存没有上限，故内存不会被会回收
indices.breaker.fielddata.limit: 60% 
//当fielddata cache到达堆内存的60%，熔断请求
```
查询和清理该部分使用量
```
GET /_cat/nodes?v&h=name,fm
POST /<index>/_cache/clear?fielddata
```
![](jBjgh-RECfIDsFj4AdYuXm0lUoo1UP6_J0dJGxAtWyQ.png)
##### query cache
node级别的filter过滤器结果缓存，每个节点有一个，被所有 shard 共享，filter query查询结果要么是 yes 要么是no，不涉及 scores 的计算。使用LRU淘汰策略，内存无法被GC。默认情况下查询会开启缓存
```
对于TermQuery、MatchAllDocsQuery等这种查询都不被缓存。当BooleanQuey的字节点为空时不会被缓存，当Dis Max Query的Disjuncts为空时不会被缓存。
对于历史查询次数有要求，对于消耗高昂的Query只需要2次就加入缓存，其他的默认是5次，对于BooleanQuery和DisjunctionMaxQuery次数为4次。
```
```
indices.queries.cache.size: 10% 
// 默认为堆内存的10%，当cache满了，又新查询进入后，会冲刷掉老的查询缓存
index.queries.cache.enable: true
// 默认开启，可关闭
```
查询和清理该部分使用量
```
GET /_cat/nodes?v&h=name,qcm
POST /<index>/_cache/clear?query
```
![](B5PpJcVxaxm_yoS0aqxhEZks7WS9f2EXbBmGr_ME3c0.png)
##### request cache
shard级别的query result缓存, 每个shard一个，默认情况下该缓存只缓存request的结果size为0的查询。所以该缓存不会缓存hits，但却会缓存 hits.total, aggregations 和 suggestions。该缓存默认是关闭的，需要手动开启.
```
indices.requests.cache.size: 1%
index.requests.cache.enable: false
//通过url传参方式request_cache=true
indices.breaker.request.limit: 40%
```
查询和清理该部分使用量
```
GET /_cat/nodes?v&h=name,rcm
POST /<index>/_cache/clear?request
```
![](iqs0yGsCfYw-_cmJuy09pamWUwuXhFUZkxB_n3QBTx4.png)
##### Segment Memory
缓存段信息，包括FST,Dimensional points for numeric range filters，Deleted documents bitset ，Doc values and stored fields codec formats等数据。这部分缓存是必须的，不能进行大小设置，通常跟index息息相关，close index、force merge均会释放segmentsMemory空间。其中FST是倒排索引的前缀树，这部分内存占用非常大，故在ES 7.3中默认将其移动到堆外内存，[显著降低 Elasticsearch 堆内存使用量 | Elastic Blog](https://www.elastic.co/cn/blog/significantly-decrease-your-elasticsearch-heap-memory-usage). 该部分内存没有参数限制，不可手动释放
查询该部分使用量
```
GET /_cat/nodes?v&h=name,segmentsMemory
```
![](R3kysiIydrkEmjAdafmdwkFBwBYbf-NPyGsVS0cpiuc.png)
##### 堆外内存
ES的堆外内存均为lucene使用，机器内存除了ES的堆内存外的内存空间都可被lucene，这部分内存不会OOM, 当其它程序需要使用内存时，操作系统会直接释放这部分内存，ES中使用PageCacheRecycler类来管理，这部分内存主要为lucene的FST、doc_values, 和查询时候加载的segment等数据，查询时ES会把segment加载到page cache中类进行查询，当page cache能完全加载查询的segment时，查询会快很多
#### 3. index加载方式
index的加载方式通过index.store.type参数控制，可选的方式有下面几种
```
fs: 默认文件系统实现
simplefs: 使用随机访问文件的文件系统存储, 面向WINDOWS平台的文件系统
niofs: 使用NIO在文件系统上存储索引, 面向LINUX平台的NIO库的文件系统
mmapfs: 文件映射到内存（MMap）来将索引存储在文件系统上, 面向LINUX平台的支持MMap方式的文件系统
hybridfs: 根据文件类型选择niofs 或者 mmapfs,  默认的加载模式
```
当indexService创建shard的会同时创建一个shard对应的FSDirectory, 其中有两个参数，一个是store相关的setting，另一个就是index的存储路径
```
public synchronized IndexShard createShard(
    final ShardRouting routing,
    final Consumer<ShardId> globalCheckpointSyncer,
    final RetentionLeaseSyncer retentionLeaseSyncer
) throws IOException {
    Objects.requireNonNull(retentionLeaseSyncer);
        //省略代码
        Directory directory = directoryFactory.newDirectory(this.indexSettings, path);
        store = new Store(
            shardId,
            this.indexSettings,
            directory,
            lock,
            new StoreCloseListener(shardId, () -> eventListener.onStoreClosed(shardId))
        );
        eventListener.onStoreCreated(shardId);
        indexShard = new IndexShard(
            routing,
            this.indexSettings,
            path,
            store,
            indexSortSupplier,
            indexCache,
            mapperService,
            similarityService,
            engineFactory,
            eventListener,
            readerWrapper,
            threadPool,
            bigArrays,
            engineWarmer,
            searchOperationListeners,
            indexingOperationListeners,
            () -> globalCheckpointSyncer.accept(shardId),
            retentionLeaseSyncer,
            circuitBreakerService,
            snapshotCommitSupplier
        );
        //省略代码
}
```
其中setting相关的参数有
```
index.store.fs.fs_lock: \[native, simple\] 默认 native
//native: 文件存在时忽略异常
//simple: 文件存在时抛异常
index.store.type: 存储类型 默认 hybridfs
node.store.allow_mmap: bool值 默认true
//如果存储类型是hybridfs 或 mmapfs时，是否允许使用内存映射
index.store.preload: []
//预加载segment文件，值为文件类型后缀名
```
![](9QLexvqiUMDHJcwRT_0HiGrKJpLGE6tftQDIWSHFb_g.png)
当创建FSDirectory时会根据index.store.type和node.store.allow_mmap来选择不同的加载类型，当index.store.type为fs时，会使用IndexModule.defaultStoreType返回获取加载类型，否则就使用index.store.type配置的类型，主要就三种类型HYBRIDFS、MMAPFS和NIOFS
```
protected Directory newFSDirectory(Path location, LockFactory lockFactory, IndexSettings indexSettings) throws IOException {
    final String storeType = indexSettings.getSettings()
        .get(IndexModule.INDEX_STORE_TYPE_SETTING.getKey(), IndexModule.Type.FS.getSettingsKey());
    IndexModule.Type type;
    if (IndexModule.Type.FS.match(storeType)) {
        type = IndexModule.defaultStoreType(IndexModule.NODE_STORE_ALLOW_MMAP.get(indexSettings.getNodeSettings()));
    } else {
        type = IndexModule.Type.fromSettingsKey(storeType);
    }
    Set<String> preLoadExtensions = new HashSet<>(indexSettings.getValue(IndexModule.INDEX_STORE_PRE_LOAD_SETTING));
    switch (type) {
        case HYBRIDFS:
            // Use Lucene defaults
            final FSDirectory primaryDirectory = FSDirectory.open(location, lockFactory);
            if (primaryDirectory instanceof MMapDirectory) {
                MMapDirectory mMapDirectory = (MMapDirectory) primaryDirectory;
                return new HybridDirectory(lockFactory, setPreload(mMapDirectory, lockFactory, preLoadExtensions));
            } else {
                return primaryDirectory;
            }
        case MMAPFS:
            return setPreload(new MMapDirectory(location, lockFactory), lockFactory, preLoadExtensions);
        case SIMPLEFS:
        case NIOFS:
            return new NIOFSDirectory(location, lockFactory);
        default:
            throw new AssertionError("unexpected built-in store type \[" + type + "\]");
    }
}
```
来看看当配置为fs时，会选择那种加载方式，逻辑比较简单就是根据node.store.allow_mmap和java是否是64位的来决定是HYBRIDFS还是nio
```
public static Type defaultStoreType(final boolean allowMmap) {
  if (allowMmap && Constants.JRE_IS_64BIT && MMapDirectory.UNMAP_SUPPORTED) {
        return Type.HYBRIDFS;
  } else {
        return Type.NIOFS;
  }
}
```
对于常用的hybridfs类型，会调用lucene的FSDirectory.open来创建一个FSDirectory，代码如下，逻辑基本为，当java为64位并且操作系统为linux系统，并且操作系统是64位的则选择MMapDirectory，如果是window系统则选择SimpleFSDirectory，否则选择NIOFSDirectory，当open方法返回后，会去判断是否为MMapDirectory，如果是HybridDirectory，否则返回open方法返回的FSDirectory.
```
public static FSDirectory open(Path path, LockFactory lockFactory) throws IOException {
  if (Constants.JRE_IS_64BIT && MMapDirectory.UNMAP_SUPPORTED) {
    return new MMapDirectory(path, lockFactory);
  } else if (Constants.WINDOWS) {
    return new SimpleFSDirectory(path, lockFactory);
  } else {
    return new NIOFSDirectory(path, lockFactory);
  }
}
```
HybridDirectory的实现如下，其继承NIOFSDirectory类，NIOFSDirectory类在lucene中实现，构造函数有两个参数，lockFactory为fs_lock，delegate为setPreload返回的MMapDirectory
```
static final class HybridDirectory extends NIOFSDirectory {
        private final MMapDirectory delegate;
  
        HybridDirectory(LockFactory lockFactory, MMapDirectory delegate) throws IOException {
            super(delegate.getDirectory(), lockFactory);
            this.delegate = delegate;
        }
  
        @Override
        public IndexInput openInput(String name, IOContext context) throws IOException {
            if (useDelegate(name, context)) {
                // we need to do these checks on the outer directory since the inner doesn't know about pending deletes
                ensureOpen();
                ensureCanRead(name);
                // we only use the mmap to open inputs. Everything else is managed by the NIOFSDirectory otherwise
                // we might run into trouble with files that are pendingDelete in one directory but still
                // listed in listAll() from the other. We on the other hand don't want to list files from both dirs
                // and intersect for perf reasons.
                return delegate.openInput(name, context);
            } else {
                return super.openInput(name, context);
            }
        }
  
        @Override
        public void close() throws IOException {
            IOUtils.close(super::close, delegate);
        }
  
        boolean useDelegate(String name, IOContext ioContext) {
            if (ioContext == Store.READONCE_CHECKSUM) {
                // If we're just reading the footer for the checksum then mmap() isn't really necessary, and it's desperately inefficient
                // if pre-loading is enabled on this file.
                return false;
            }
  
            final LuceneFilesExtensions extension = LuceneFilesExtensions.fromExtension(FileSwitchDirectory.getExtension(name));
            if (extension == null || extension.shouldMmap() == false) {
                // Other files are either less performance-sensitive (e.g. stored field index, norms metadata)
                // or are large and have a random access pattern and mmap leads to page cache trashing
                // (e.g. stored fields and term vectors).
                return false;
            }
            return true;
        }
  
        MMapDirectory getDelegate() {
            return delegate;
        }
    }
```
setPreload实现如下，如果设置了预加载则返回包装后的MMapDirectory，否则直接返回mMapDirectory
```
public static MMapDirectory setPreload(MMapDirectory mMapDirectory, LockFactory lockFactory, Set<String> preLoadExtensions)
    throws IOException {
    assert mMapDirectory.getPreload() == false;
    if (preLoadExtensions.isEmpty() == false) {
        if (preLoadExtensions.contains("\*")) {
            mMapDirectory.setPreload(true);
        } else {
            return new PreLoadMMapDirectory(mMapDirectory, lockFactory, preLoadExtensions);
        }
    }
    return mMapDirectory;
}
```
先看看没有设置预加载的情况，在HybridDirectory中，有一个核心的方法openInput， 该方法用于加载index的文件，通过useDelegate方法判断文件是用MMapDirectory打开，还是通过父类NIOFSDirectory打开，ensureOpen判断是否能打开，ensureCanRead判断是否能读，具体open方式由lucene实现，其中是使用offheap还是onheap由lucene， es这端不做干预
```
public IndexInput openInput(String name, IOContext context) throws IOException {
            if (useDelegate(name, context)) {
                // we need to do these checks on the outer directory since the inner doesn't know about pending deletes
                ensureOpen();
                ensureCanRead(name);
                // we only use the mmap to open inputs. Everything else is managed by the NIOFSDirectory otherwise
                // we might run into trouble with files that are pendingDelete in one directory but still
                // listed in listAll() from the other. We on the other hand don't want to list files from both dirs
                // and intersect for perf reasons.
                return delegate.openInput(name, context);
            } else {
                return super.openInput(name, context);
            }
        }
```
其中useDelegate方法实现如下，逻辑为通过文件后缀名，来判断是否使用MMapDirectory读取
```
boolean useDelegate(String name, IOContext ioContext) {
    if (ioContext == Store.READONCE_CHECKSUM) {
        // If we're just reading the footer for the checksum then mmap() isn't really necessary, and it's desperately inefficient
        // if pre-loading is enabled on this file.
        return false;
    }
  
    final LuceneFilesExtensions extension = LuceneFilesExtensions.fromExtension(FileSwitchDirectory.getExtension(name));
    if (extension == null || extension.shouldMmap() == false) {
        // Other files are either less performance-sensitive (e.g. stored field index, norms metadata)
        // or are large and have a random access pattern and mmap leads to page cache trashing
        // (e.g. stored fields and term vectors).
        return false;
    }
    return true;
}
```
文件是否通过mmap读取，在代码中已经写死， 如下，以此为文件后缀名，描述， 是否是元数据，是否使用mmap加载
```
CFE("cfe", "Compound Files Entries", true, false),
// Compound files are tricky because they store all the information for the segment. Benchmarks
// suggested that not mapping them hurts performance.
CFS("cfs", "Compound Files", false, true),
CMP("cmp", "Completion Index", true, false),
DII("dii", "Points Index", false, false),
// dim files only apply up to lucene 8.x indices. It can be removed once we are in lucene 10
DIM("dim", "Points", false, true),
// MMapDirectory has special logic to read long\[\] arrays in little-endian order that helps speed
// up the decoding of postings. The same logic applies to positions (.pos) of offsets (.pay) but we
// are not mmaping them as queries that leverage positions are more costly and the decoding of postings
// tends to be less a bottleneck.
DOC("doc", "Frequencies", false, true),
// Doc values are typically performance-sensitive and hot in the page
// cache, so we use mmap, which provides better performance.
DVD("dvd", "DocValues", false, true),
DVM("dvm", "DocValues Metadata", true, false),
FDM("fdm", "Field Metadata", true, false),
FDT("fdt", "Field Data", false, false),
FDX("fdx", "Field Index", false, false),
FNM("fnm", "Fields", true, false),
// old extension
KDD("kdd", "Points", false, true),
// old extension
KDI("kdi", "Points Index", false, true),
// Lucene 8.6 point format metadata file
KDM("kdm", "Points Metadata", true, false),
LIV("liv", "Live Documents", false, false),
LKP("lkp", "Completion Dictionary", false, false),
// Norms are typically performance-sensitive and hot in the page
// cache, so we use mmap, which provides better performance.
NVD("nvd", "Norms", false, true),
NVM("nvm", "Norms Metadata", true, false),
PAY("pay", "Payloads", false, false),
POS("pos", "Positions", false, false),
SI("si", "Segment Info", true, false),
// Term dictionaries are typically performance-sensitive and hot in the page
// cache, so we use mmap, which provides better performance.
TIM("tim", "Term Dictionary", false, true),
// We want to open the terms index and KD-tree index off-heap to save memory, but this only performs
// well if using mmap.
TIP("tip", "Term Index", false, true),
// Lucene 8.6 terms metadata file
TMD("tmd", "Term Dictionary Metadata", true, false),
// Temporary Lucene file
TMP("tmp", "Temporary File", false, false),
TVD("tvd", "Term Vector Documents", false, false),
TVF("tvf", "Term Vector Fields", false, false),
TVM("tvm", "Term Vector Metadata", true, false),
TVX("tvx", "Term Vector Index", false, false),
VEC("vec", "Vector Data", false, false),
// Lucene 9.0 indexed vectors metadata
VEM("vem", "Vector Metadata", true, false);
```
现在看看预加载的实现，其实现方式与HybridDirectory类型，区别点为this.delegate.setPreload(true)，其真正预加载由lucene实现，useDelegate内部判断方式改成了是否在配置文件中配置，即可以更改部分文件由mmap加载
```
static final class PreLoadMMapDirectory extends MMapDirectory {
    private final MMapDirectory delegate;
    private final Set<String> preloadExtensions;
  
    PreLoadMMapDirectory(MMapDirectory delegate, LockFactory lockFactory, Set<String> preload) throws IOException {
        super(delegate.getDirectory(), lockFactory);
        super.setPreload(false);
        this.delegate = delegate;
        this.delegate.setPreload(true);
        this.preloadExtensions = preload;
        assert getPreload() == false;
    }
  
    @Override
    public void setPreload(boolean preload) {
        throw new IllegalArgumentException("can't set preload on a preload-wrapper");
    }
  
    @Override
    public IndexInput openInput(String name, IOContext context) throws IOException {
        if (useDelegate(name)) {
            // we need to do these checks on the outer directory since the inner doesn't know about pending deletes
            ensureOpen();
            ensureCanRead(name);
            return delegate.openInput(name, context);
        }
        return super.openInput(name, context);
    }
  
    @Override
    public synchronized void close() throws IOException {
        IOUtils.close(super::close, delegate);
    }
  
    boolean useDelegate(String name) {
        final String extension = FileSwitchDirectory.getExtension(name);
        return preloadExtensions.contains(extension);
    }
  
    MMapDirectory getDelegate() {
        return delegate;
    }
}
```