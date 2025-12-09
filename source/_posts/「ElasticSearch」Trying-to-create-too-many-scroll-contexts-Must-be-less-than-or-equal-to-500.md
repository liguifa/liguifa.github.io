---
title: "「ElasticSearch」Trying to create too many scroll contexts. Must be less than or equal to: 500"
date: 2023-07-27 22:36:46
tags: [大数据, ElasticSearch]
---
## 问题
报错Trying to create too many scroll contexts. Must be less than or equal to: [500]. This limit can be set by changing the [search.max_open_scroll_context] setting.\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121156512Z"}

## 现象

### ES端现象
bigdata-es后端报错，集群节点反复脱离集群
```
{"log":"{\"type\": \"server\", \"timestamp\": \"2023-07-27T02:42:13,117Z\", \"level\": \"WARN\", \"component\": \"r.suppressed\", \"cluster.name\": \"bigdata-es\", \"node.name\": \"node-5\", \"message\": \"path: /bigdata-fusioncdn-%2A/_search, params: {size=10000, scroll=10m, index=bigdata-fusioncdn-*}\", \"cluster.uuid\": \"fa4e1bPrRBCki8U4WPj6IQ\", \"node.id\": \"MmQZlAlIRMm8NIR04mCflA\" , \n","stream":"stdout","time":"2023-07-27T02:42:13.120708259Z"}
{"log":"\"stacktrace\": [\"org.elasticsearch.action.search.SearchPhaseExecutionException: all shards failed\",\n","stream":"stdout","time":"2023-07-27T02:42:13.12076008Z"}
{"log":"\"at org.elasticsearch.action.search.AbstractSearchAsyncAction.onPhaseFailure(AbstractSearchAsyncAction.java:713) [elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120770392Z"}
{"log":"\"at org.elasticsearch.action.search.AbstractSearchAsyncAction.executeNextPhase(AbstractSearchAsyncAction.java:400) [elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120777406Z"}
{"log":"\"at org.elasticsearch.action.search.AbstractSearchAsyncAction.onPhaseDone(AbstractSearchAsyncAction.java:745) [elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120784871Z"}
{"log":"\"at org.elasticsearch.action.search.AbstractSearchAsyncAction.onShardFailure(AbstractSearchAsyncAction.java:497) [elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120792229Z"}
{"log":"\"at org.elasticsearch.action.search.AbstractSearchAsyncAction.access$000(AbstractSearchAsyncAction.java:64) [elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120798743Z"}
{"log":"\"at org.elasticsearch.action.search.AbstractSearchAsyncAction$1.onFailure(AbstractSearchAsyncAction.java:331) [elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120805019Z"}
{"log":"\"at org.elasticsearch.action.ActionListener$Delegating.onFailure(ActionListener.java:66) [elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120811316Z"}
{"log":"\"at org.elasticsearch.action.ActionListenerResponseHandler.handleException(ActionListenerResponseHandler.java:48) [elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120819001Z"}
{"log":"\"at org.elasticsearch.action.search.SearchTransportService$ConnectionCountingHandler.handleException(SearchTransportService.java:651) [elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120825432Z"}
{"log":"\"at org.elasticsearch.transport.TransportService$4.handleException(TransportService.java:853) [elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120832109Z"}
{"log":"\"at org.elasticsearch.transport.TransportService$ContextRestoreResponseHandler.handleException(TransportService.java:1481) [elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120841065Z"}
{"log":"\"at org.elasticsearch.transport.InboundHandler.lambda$handleException$3(InboundHandler.java:368) [elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120847448Z"}
{"log":"\"at org.elasticsearch.common.util.concurrent.EsExecutors$DirectExecutorService.execute(EsExecutors.java:288) [elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120853633Z"}
{"log":"\"at org.elasticsearch.transport.InboundHandler.handleException(InboundHandler.java:366) [elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120861163Z"}
{"log":"\"at org.elasticsearch.transport.InboundHandler.handlerResponseError(InboundHandler.java:358) [elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120888266Z"}
{"log":"\"at org.elasticsearch.transport.InboundHandler.messageReceived(InboundHandler.java:132) [elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120894865Z"}
{"log":"\"at org.elasticsearch.transport.InboundHandler.inboundMessage(InboundHandler.java:88) [elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120900812Z"}
{"log":"\"at org.elasticsearch.transport.TcpTransport.inboundMessage(TcpTransport.java:743) [elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120906642Z"}
{"log":"\"at org.elasticsearch.transport.InboundPipeline.forwardFragments(InboundPipeline.java:147) [elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120912335Z"}
{"log":"\"at org.elasticsearch.transport.InboundPipeline.doHandleBytes(InboundPipeline.java:119) [elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120918135Z"}
{"log":"\"at org.elasticsearch.transport.InboundPipeline.handleBytes(InboundPipeline.java:84) [elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120924029Z"}
{"log":"\"at org.elasticsearch.transport.netty4.Netty4MessageChannelHandler.channelRead(Netty4MessageChannelHandler.java:71) [transport-netty4-client-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.12093038Z"}
{"log":"\"at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:379) [netty-transport-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120936377Z"}
{"log":"\"at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:365) [netty-transport-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120942397Z"}
{"log":"\"at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:357) [netty-transport-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120948258Z"}
{"log":"\"at io.netty.handler.logging.LoggingHandler.channelRead(LoggingHandler.java:280) [netty-handler-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120954846Z"}
{"log":"\"at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:379) [netty-transport-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120960625Z"}
{"log":"\"at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:365) [netty-transport-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120967072Z"}
{"log":"\"at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:357) [netty-transport-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120973076Z"}
{"log":"\"at io.netty.handler.codec.MessageToMessageDecoder.channelRead(MessageToMessageDecoder.java:103) [netty-codec-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120978889Z"}
{"log":"\"at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:379) [netty-transport-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120984793Z"}
{"log":"\"at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:365) [netty-transport-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120991038Z"}
{"log":"\"at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:357) [netty-transport-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.120997054Z"}
{"log":"\"at io.netty.handler.ssl.SslHandler.unwrap(SslHandler.java:1374) [netty-handler-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121008248Z"}
{"log":"\"at io.netty.handler.ssl.SslHandler.decodeJdkCompatible(SslHandler.java:1237) [netty-handler-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121014484Z"}
{"log":"\"at io.netty.handler.ssl.SslHandler.decode(SslHandler.java:1286) [netty-handler-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121020506Z"}
{"log":"\"at io.netty.handler.codec.ByteToMessageDecoder.decodeRemovalReentryProtection(ByteToMessageDecoder.java:507) [netty-codec-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121037738Z"}
{"log":"\"at io.netty.handler.codec.ByteToMessageDecoder.callDecode(ByteToMessageDecoder.java:446) [netty-codec-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121046005Z"}
{"log":"\"at io.netty.handler.codec.ByteToMessageDecoder.channelRead(ByteToMessageDecoder.java:276) [netty-codec-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121051891Z"}
{"log":"\"at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:379) [netty-transport-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121058709Z"}
{"log":"\"at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:365) [netty-transport-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121066525Z"}
{"log":"\"at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:357) [netty-transport-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121072568Z"}
{"log":"\"at io.netty.channel.DefaultChannelPipeline$HeadContext.channelRead(DefaultChannelPipeline.java:1410) [netty-transport-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121078504Z"}
{"log":"\"at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:379) [netty-transport-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121084638Z"}
{"log":"\"at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:365) [netty-transport-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121090575Z"}
{"log":"\"at io.netty.channel.DefaultChannelPipeline.fireChannelRead(DefaultChannelPipeline.java:919) [netty-transport-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121096623Z"}
{"log":"\"at io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:166) [netty-transport-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121102472Z"}
{"log":"\"at io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:719) [netty-transport-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121108681Z"}
{"log":"\"at io.netty.channel.nio.NioEventLoop.processSelectedKeysPlain(NioEventLoop.java:620) [netty-transport-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.12111462Z"}
{"log":"\"at io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:583) [netty-transport-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121120917Z"}
{"log":"\"at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:493) [netty-transport-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121126699Z"}
{"log":"\"at io.netty.util.concurrent.SingleThreadEventExecutor$4.run(SingleThreadEventExecutor.java:986) [netty-common-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121132507Z"}
{"log":"\"at io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74) [netty-common-4.1.66.Final.jar:4.1.66.Final]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121138617Z"}
{"log":"\"at java.lang.Thread.run(Thread.java:833) [?:?]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121150102Z"}
{"log":"\"Caused by: org.elasticsearch.ElasticsearchException: Trying to create too many scroll contexts. Must be less than or equal to: [500]. This limit can be set by changing the [search.max_open_scroll_context] setting.\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121156512Z"}
{"log":"\"at org.elasticsearch.search.SearchService.createAndPutReaderContext(SearchService.java:881) ~[elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121162705Z"}
{"log":"\"at org.elasticsearch.search.SearchService.createOrGetReaderContext(SearchService.java:859) ~[elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121169163Z"}
{"log":"\"at org.elasticsearch.search.SearchService.executeQueryPhase(SearchService.java:615) ~[elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121174973Z"}
{"log":"\"at org.elasticsearch.search.SearchService.lambda$executeQueryPhase$2(SearchService.java:483) ~[elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121180971Z"}
{"log":"\"at org.elasticsearch.action.ActionRunnable.lambda$supply$0(ActionRunnable.java:47) ~[elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121186902Z"}
{"log":"\"at org.elasticsearch.action.ActionRunnable$2.doRun(ActionRunnable.java:62) ~[elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121192724Z"}
{"log":"\"at org.elasticsearch.common.util.concurrent.AbstractRunnable.run(AbstractRunnable.java:26) ~[elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121198465Z"}
{"log":"\"at org.elasticsearch.common.util.concurrent.TimedRunnable.doRun(TimedRunnable.java:33) ~[elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.1212048Z"}
{"log":"\"at org.elasticsearch.common.util.concurrent.ThreadContext$ContextPreservingAbstractRunnable.doRun(ThreadContext.java:777) ~[elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121210622Z"}
{"log":"\"at org.elasticsearch.common.util.concurrent.AbstractRunnable.run(AbstractRunnable.java:26) ~[elasticsearch-7.17.5.jar:7.17.5]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121219247Z"}
{"log":"\"at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1136) ~[?:?]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121225215Z"}
{"log":"\"at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:635) ~[?:?]\",\n","stream":"stdout","time":"2023-07-27T02:42:13.121231115Z"}
{"log":"\"... 1 more\"] }\n","stream":"stdout","time":"2023-07-27T02:42:13.121236842Z"}
```

## 原因
看报告是存活的游标数据积压了，超过了配置的最大值，而且游标时间设置是10分钟，这个是否合理？询问业务端，业务端反馈查询语句为
![](ziQWvHza6y4MhyS2bcmTpEncL5xsSJlguBCOq-Qs6Vs.png)
通过这条语句，发现业务端的游标设置不合理，这表示执行这条语句时要将此游标保持开启10分钟，到是数据处理仅仅只格式化做下判断，ES中游标保持时间这个值的时间不是整个数据处理的时间，是上一次使用游标到下一次使用游标的时间。因此这个值的时间不能设置太长

## 解决
解决方案：业务端调整过期时间，调整为1秒钟