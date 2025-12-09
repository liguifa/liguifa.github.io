---
title: 「FLINK-OPERATOR」Flink-operator组件参数
date: 2025-12-09 21:44:47
tags: [Flink, 大数据]
---
# Flink Operator 组件参数

## Flink 应用状态

- **READY**: JobManager在运行中了，api也可以调用了
- **DEPLOYED_NOT_READY**: JobManager在运行中了，但是api不可以调用
- **DEPLOYING**: JobManager在启动中
- **MISSING**: JobManager找不到了
- **ERROR**: 部署异常了

---

## 核心配置参数

### `reconcile.interval`
- **默认值**: 60s
- **描述**: reconcile方法调度的周期
- **reconcile的调度时间规则**:
  - 默认: `reconcile.interval`
  - `DEPLOYED_NOT_READY`: `observer.rest-ready.delay`
  - `MISSING`和`ERROR`: `reconcile.interval`
  - `DEPLOYING`: `observer.progress-check.interval`
  - `READY`: 如果savepoint的`.triggerId`不为空，则为`observer.progress-check.interval`，否则为`reconcile.interval`

### `observer.rest-ready.delay`
- **默认值**: 10s
- **描述**: 当任务状态是`DEPLOYED_NOT_READY`(jobmanager是运行中，但是api还不能调用)时，reconcile方法的调度时间

### `reconcile.parallelism`
- **默认值**: 1
- **描述**: reconcile方法调度的默认线程数，-1表示不限制

### `observer.progress-check.interval`
- **默认值**: 10s
- **描述**: 当任务状态是`DEPLOYING`时，reconcile方法的调度时间

### `savepoint.trigger.grace-period`
- **默认值**: 1m
- **描述**: savepoint超时时间，Snapshot任务使用

### `checkpoint.trigger.grace-period`
- **默认值**: 1m
- **描述**: checkpoint超时时间，Snapshot任务使用

### `flink.client.timeout`
- **默认值**: 10s
- **描述**: flink请求超时时间

### `flink.client.cancel.timeout`
- **默认值**: 1m
- **描述**: flink任务cancel超时时间

### `resource.cleanup.timeout`
- **默认值**: 
- **描述**: flink集群关闭清理资源超时时间

### `deployment.rollback.enabled`
- **默认值**: false
- **描述**: 部署失败是否自动回滚

### `deployment.readiness.timeout`
- **默认值**: 5m
- **描述**: 启用回滚后，这个参数指定任务的稳定时间，超过这个时间才会回滚

### `user.artifacts.base.dir`
- **默认值**: /opt/flink/artifacts
- **描述**: jar文件的放置目录

### `job.upgrade.ignore-pending-savepoint`
- **默认值**: false
- **描述**: 任务升级的过程中，是否忽略pending状态的savepoint

### `job.upgrade.inplace-scaling.enabled`
- **默认值**: true
- **描述**: 任务升级后，是否就地缩放，1.18+版本支持

### `dynamic.config.enabled`
- **默认值**: true
- **描述**: 是否对任务启用热更新

### `dynamic.config.check.interval`
- **默认值**: 5m
- **描述**: 热更新的频率

### `config.cache.timeout`
- **默认值**: 10m
- **描述**: flink配置的缓存超时时间，超时后重新更新

### `config.cache.size`
- **默认值**: 1000
- **描述**: 配置缓存项的数量

### `jm-deployment-recovery.enabled`
- **默认值**: true
- **描述**: 是否恢复挂掉的jobmanager

### `savepoint.dispose-on-delete`
- **默认值**: false
- **描述**: FlinkStateSnapshot任务创建时候使用的资源，是否自动删除

### `savepoint.format.type`
- **默认值**: SavepointFormatType.DEFAULT
- **描述**: savepoint文件的默认格式

### `checkpoint.type`
- **默认值**: CheckpointType.FULL
- **描述**: checkpoint的类型

### `savepoint.cleanup.enabled`
- **默认值**: true
- **描述**: 是否开启清理savepoint数据，如果开启，那么在首次部署或者挂掉拉取的时候，会先清理savepoint数据

### `savepoint.history.max.count`
- **默认值**: 10
- **描述**: savepoint最大保留数量，FlinkStateSnapshot使用

### `savepoint.history.max.age`
- **默认值**: 24H
- **描述**: savepoint最大保留时间，FlinkStateSnapshot使用

### `savepoint.history.max.age.threshold`
- **默认值**: 无
- **描述**: 同`savepoint.history.max.age`，优先级更高

### `exception.stacktrace.enabled`
- **默认值**: false
- **描述**: 是否保存异常信息

### `exception.stacktrace.max.length`
- **默认值**: 2048
- **描述**: 异常信息最大值，超过限制会被截取掉，堆栈部分

### `exception.field.max.length`
- **默认值**: 2048
- **描述**: 异常信息最大值，超过限制会被截取掉，异常原因部分

### `exception.throwable.list.max.count`
- **默认值**: 2
- **描述**: 保存的最大异常数

### `exception.label.mapper`
- **默认值**: 无
- **描述**: HashMap key是正则表达式，对于满足正则表达式的异常，添加应该标签，标签值是value的内容

### `user.artifacts.http.header`
- **默认值**: 无
- **描述**: 对与session任务，需要上传jar文件，如果通过http上传，这里可以在请求中添加请求头

### `snapshot.resource.enabled`
- **默认值**: true

### `periodic.savepoint.interval`
- **默认值**: 无
- **描述**: 自动保存点的触发周期，10m、0 1 * * *

### `periodic.checkpoint.interval`
- **默认值**: 无
- **描述**: 自动checkpoint点的触发周期，10m、0 1 * * *

### `plugins.listeners.<listener-name>.class`
- **默认值**: 没有使用

### `watched.namespaces`
- **默认值**: all
- **描述**: 监听的名称空间,多个值用逗号分隔

### `label.selector`
- **默认值**: 无
- **描述**: 标记选择，满足这个标签选择器的才会被管理

### `dynamic.namespaces.enabled`
- **默认值**: false
- **描述**: 如果为true，可以动态调整namespace

### `retry.initial.interval`
- **默认值**: 5s
- **描述**: 控制器错误的第一次重试间隔

### `retry.max.interval`
- **默认值**: 无
- **描述**: 控制器错误的最大重试间隔

### `retry.interval.multiplier`
- **默认值**: 1.5
- **描述**: 控制器错误的重试间隔倍率，例如: 第一次 1s，第二次 1s * 1.5，第三次 1s * 1.5 * 1.5

### `retry.max.attempts`
- **默认值**: 15
- **描述**: 控制器错误的重试次数

### `rate-limiter.refresh-period`
- **默认值**: 15s
- **描述**: 任务资源刷新周期, todo

### `rate-limiter.limit`
- **默认值**: 5
- **描述**: todo

### `job.upgrade.last-state-fallback.enabled`
- **默认值**: true
- **描述**: 使用上个保存点还原

### `job.upgrade.last-state.max.allowed.checkpoint.age`
- **默认值**: 无
- **描述**: 使用上个保存点还原的时候，确保上个保存点的时间在这个参数的范围内

### `job.upgrade.last-state.job-cancel.enabled`
- **默认值**: false
- **描述**: 是否允许savepoint取消

### `health.probe.enabled`
- **默认值**: true
- **描述**: 是否对operator启用健康检查

### `health.probe.port`
- **默认值**: 8085
- **描述**: 健康检查的端口

### `startup.stop-on-informer-error`
- **默认值**: true
- **描述**: 在启动的过程中，出现错误是否停止启动

### `termination.timeout`
- **默认值**: 10s
- **描述**: 任务关闭的超时时间

### `cluster.health-check.enabled`
- **默认值**: false
- **描述**: reconcile方法中，是否需要检查集群的健康情况

### `cluster.health-check.restarts.window`
- **默认值**: 2m
- **描述**: 任务重启之后的时间窗口

### `cluster.health-check.restarts.threshold`
- **默认值**: 64
- **描述**: 任务重启的最大次数

### `cluster.health-check.checkpoint-progress.enabled`
- **默认值**: 无
- **描述**: 完成checkpoint的超时时间，如果在指定时间内没完成checkpoint，则认为任务不健康

### `job.restart.failed`
- **默认值**: false
- **描述**: 如果设置为true，则当任务状态为失败时，会重启任务

### `leader-election.enabled`
- **默认值**: false
- **描述**: 是否operator leader选举

### `leader-election.lease-name`
- **默认值**: 无
- **描述**: operator leader选举的分组名称

### `leader-election.lease-duration`
- **默认值**: LeaderElectionConfiguration.LEASE_DURATION_DEFAULT_VALUE
- **描述**: operator leader的租约时间

### `leader-election.renew-deadline`
- **默认值**: LeaderElectionConfiguration.RENEW_DEADLINE_DEFAULT_VALUE
- **描述**: 选举延长最后期限

### `leader-election.retry-period`
- **默认值**: LeaderElectionConfiguration.RETRY_PERIOD_DEFAULT_VALUE
- **描述**: 选举重试期限

### `jm-deployment.shutdown-ttl`
- **默认值**: 1d
- **描述**: jobmanager关闭后，k8s上的任务如果没有自动清理掉，过在这个参数指定的时间之后清理掉

### `health.canary.resource.timeout`
- **默认值**: 1m
- **描述**: canary健康检查超时时间

### `jm-deployment.startup.probe.enabled`
- **默认值**: true
- **描述**: 是否开启jobmanager监控检查

### `pod-template.merge-arrays-by-name`
- **默认值**: false
- **描述**: 公共的pod模板和指定的pod模板合并的时候，array按名称合并还是按下标合并，按名称合并就是按对应的数值追加到指定的名称下，按下标合并就是两个数组追加起来

### `resource.deletion.propagation`
- **默认值**: io.fabric8.kubernetes.api.model.DeletionPropagation.FOREGROUND
- **描述**: 任务升级更新的时候需要删除deployment等资源，设置删除的方式

### `job.savepoint-on-deletion`
- **默认值**: false
- **描述**: 配置在删除job的时候，是否必须先执行savepoint

### `job.drain-on-savepoint-deletion`
- **默认值**: false
- **描述**: 配置在删除job的之后，是否清理掉savepoint

### `cluster.resource-view.refresh-interval`
- **默认值**: -1
- **描述**: 多久检索一次Kubernetes集群资源使用信息, -1 禁用该功能