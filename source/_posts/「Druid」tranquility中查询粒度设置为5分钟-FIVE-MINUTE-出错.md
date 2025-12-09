---
title: 「Druid」tranquility中查询粒度设置为5分钟-FIVE-MINUTE-出错
date: 2023-07-27 20:31:34
tags: [大数据, Driud]
---
\[错误详细\]  
在druid中查询时间粒度可以设置为MINUTE、HOURE、FILVE\_MINUTE等，但在tranquility设置查询粒度为5分钟，启动后报错，错误信息如下  
  

```
ava.lang.IllegalArgumentException: Instantiation of [simple type, class io.druid.granularity.QueryGranularity] value failed: No enum constant io.druid.granularity.QueryGranularity.MillisIn.FIVE_MINUTE
at com.fasterxml.jackson.databind.ObjectMapper._convert(ObjectMapper.java:2774) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.ObjectMapper.convertValue(ObjectMapper.java:2700) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.metamx.tranquility.druid.DruidBeams$.makeFireDepartment(DruidBeams.scala:406) ~[io.druid.tranquility-core-0.8.0.jar:0.8.0]
at com.metamx.tranquility.druid.DruidBeams$.fromConfigInternal(DruidBeams.scala:291) ~[io.druid.tranquility-core-0.8.0.jar:0.8.0]
at com.metamx.tranquility.druid.DruidBeams$.fromConfig(DruidBeams.scala:199) ~[io.druid.tranquility-core-0.8.0.jar:0.8.0]
at com.metamx.tranquility.kafka.KafkaBeamUtils$.createTranquilizer(KafkaBeamUtils.scala:40) ~[io.druid.tranquility-kafka-0.8.0.jar:0.8.0]
at com.metamx.tranquility.kafka.KafkaBeamUtils.createTranquilizer(KafkaBeamUtils.scala) ~[io.druid.tranquility-kafka-0.8.0.jar:0.8.0]
at com.metamx.tranquility.kafka.writer.TranquilityEventWriter.<init>(TranquilityEventWriter.java:64) ~[io.druid.tranquility-kafka-0.8.0.jar:0.8.0]
at com.metamx.tranquility.kafka.writer.WriterController.createWriter(WriterController.java:171) ~[io.druid.tranquility-kafka-0.8.0.jar:0.8.0]
at com.metamx.tranquility.kafka.writer.WriterController.getWriter(WriterController.java:98) ~[io.druid.tranquility-kafka-0.8.0.jar:0.8.0]
at com.metamx.tranquility.kafka.KafkaConsumer$2.run(KafkaConsumer.java:231) ~[io.druid.tranquility-kafka-0.8.0.jar:0.8.0]
at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511) [na:1.8.0_201]
at java.util.concurrent.FutureTask.run(FutureTask.java:266) [na:1.8.0_201]
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_201]
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_201]
at java.lang.Thread.run(Thread.java:748) [na:1.8.0_201]
Caused by: com.fasterxml.jackson.databind.JsonMappingException: Instantiation of [simple type, class io.druid.granularity.QueryGranularity] value failed: No enum constant io.druid.granularity.QueryGranularity.MillisIn.FIVE_MINUTE
at com.fasterxml.jackson.databind.deser.std.StdValueInstantiator.wrapException(StdValueInstantiator.java:405) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.std.StdValueInstantiator.createFromString(StdValueInstantiator.java:284) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.BeanDeserializerBase.deserializeFromString(BeanDeserializerBase.java:1141) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.BeanDeserializer._deserializeOther(BeanDeserializer.java:135) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserialize(BeanDeserializer.java:126) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.jsontype.impl.AsPropertyTypeDeserializer._deserializeTypedUsingDefaultImpl(AsPropertyTypeDeserializer.java:129) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.jsontype.impl.AsPropertyTypeDeserializer.deserializeTypedFromObject(AsPropertyTypeDeserializer.java:75) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.BeanDeserializerBase.deserializeWithType(BeanDeserializerBase.java:956) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.SettableBeanProperty.deserialize(SettableBeanProperty.java:536) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.BeanDeserializer._deserializeUsingPropertyBased(BeanDeserializer.java:344) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.BeanDeserializerBase.deserializeFromObjectUsingNonDefault(BeanDeserializerBase.java:1064) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserializeFromObject(BeanDeserializer.java:264) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.BeanDeserializer._deserializeOther(BeanDeserializer.java:156) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserialize(BeanDeserializer.java:126) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.jsontype.impl.AsPropertyTypeDeserializer._deserializeTypedForId(AsPropertyTypeDeserializer.java:113) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.jsontype.impl.AsPropertyTypeDeserializer.deserializeTypedFromObject(AsPropertyTypeDeserializer.java:84) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.AbstractDeserializer.deserializeWithType(AbstractDeserializer.java:132) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.SettableBeanProperty.deserialize(SettableBeanProperty.java:536) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.BeanDeserializer._deserializeUsingPropertyBased(BeanDeserializer.java:344) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.BeanDeserializerBase.deserializeFromObjectUsingNonDefault(BeanDeserializerBase.java:1064) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserializeFromObject(BeanDeserializer.java:264) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserialize(BeanDeserializer.java:124) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.SettableBeanProperty.deserialize(SettableBeanProperty.java:538) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.BeanDeserializer._deserializeUsingPropertyBased(BeanDeserializer.java:344) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.BeanDeserializerBase.deserializeFromObjectUsingNonDefault(BeanDeserializerBase.java:1064) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserializeFromObject(BeanDeserializer.java:264) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserialize(BeanDeserializer.java:124) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.ObjectMapper._convert(ObjectMapper.java:2769) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
... 15 common frames omitted
Caused by: java.lang.IllegalArgumentException: No enum constant io.druid.granularity.QueryGranularity.MillisIn.FIVE_MINUTE
at java.lang.Enum.valueOf(Enum.java:238) ~[na:1.8.0_201]
at io.druid.granularity.QueryGranularity$MillisIn.valueOf(QueryGranularity.java:62) ~[io.druid.druid-processing-0.9.0.jar:0.9.0]
at io.druid.granularity.QueryGranularity.convertValue(QueryGranularity.java:79) ~[io.druid.druid-processing-0.9.0.jar:0.9.0]
at io.druid.granularity.QueryGranularity.fromString(QueryGranularity.java:59) ~[io.druid.druid-processing-0.9.0.jar:0.9.0]
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_201]
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_201]
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_201]
at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_201]
at com.fasterxml.jackson.databind.introspect.AnnotatedMethod.call1(AnnotatedMethod.java:125) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
at com.fasterxml.jackson.databind.deser.std.StdValueInstantiator.createFromString(StdValueInstantiator.java:282) ~[com.fasterxml.jackson.core.jackson-databind-2.4.6.jar:2.4.6]
... 41 common frames omitted

2019-04-03 11:06:08,485 - [ERROR] - from com.metamx.tranquility.kafka.KafkaConsumer in KafkaConsumer-1
```

\[问题原因\]  
在tranquility的最新版本(0.8.2)中 引用的druid为0.9.0，0.9.0的druid不支持查询粒度设置为5分钟

\[解决方法\]  
下载druid 0.9.0源代码，修改代码文件([https://github.com/apache/incubator-druid/blob/0.9.0/processing/src/main/java/io/druid/granularity/QueryGranularity.java)](https://github.com/apache/incubator-druid/blob/0.9.0/processing/src/main/java/io/druid/granularity/QueryGranularity.java)), 在MillisIn中加入FIVE\_MINUTE，之后build代码文件，替换io.druid.druid-processing-0.9.0.jar