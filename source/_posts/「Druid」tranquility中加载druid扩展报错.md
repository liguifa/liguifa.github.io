---
title: 「Druid」tranquility中加载druid扩展报错
date: 2023-07-27 20:34:24
tags: [大数据, Driud]
---
\[错误详细\]  
在tranquility的extensions中、或命令行中加在druid扩展，启动后被告，错误如下：  
  

```
2019-04-04 15:01:55,480 - [ERROR] - from com.metamx.tranquility.kafka.KafkaConsumer in KafkaConsumer-0
Exception:
java.lang.NoClassDefFoundError: Could not initialize class com.metamx.tranquility.druid.DruidGuicer$
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
```

\[问题原因\]  
tranquility中引用的jackson,和扩展中引用的jackson版本冲突

\[解决\]  
在0.9.0的druid中build插件放入tranquility中