---
title: '「Nomad」nomad学习笔记: 1 nomad环境部署'
date: 2023-08-05 22:11:16
tags: ["Nomad"]
---
nomad 是一个灵活的对应用进行部署、调度、管理，用于跨本地和云大规模部署和管理容器和非容器化应用程序。（个人理解类似K8S，不过比较轻量，管理部署微服务，镜像等）

### nomad 必要组成
server: nomad的服务端管理进程，用于管理所有的nomad的机器，负责将job、ctask等下发给client端

client: 一个nomad集群由多个client，client用于负载执行server端下发的job，已即部署应用的节点

### nomad 安装
nomad 安装比较简单
###### 第一步，下载nomad的二进制文件, 下载地址如下
```
https://developer.hashicorp.com/nomad/downloads
```
###### 第二步，下载后解压，配置环境变量
###### 第三步， 验证安装
```
nomad -v

//显示
Nomad v1.6.1
BuildDate 2023-07-21T13:49:42Z
Revision 515895c7690cdc72278018dc5dc58aca41204ccc
```

### 启动server端
nomad 主要有两个进程server和client， 这里先启动server端
新建文件夹 nomad, 并新建文件 server.nomad, 内容如下:
```
name = "server1"

log_level = "DEBUG"

data_dir = "/Users/liguifa/nomad"

server {
    enabled = true
    bootstrap_expect = 1
}

ports {
  http = 4646
  rpc  = 4647
  serf = 4648
}
```
其中:
name 指定server的名称为server1
log_level 指定日志的输出类型为debug
data_dir 指定nomad对应的数据存储文件夹，这里需要调整成对应的目录
ports 指定server的端口

文件修改好之后，使用命令启动server
```
nomad agent -config /Users/liguifa/nomad/conf/server.name
```

启动完成后可使用下面地址访问nomad的管理页面
```
http://127.0.0.1:4646
```

### 启动client
在文件夹 nomad 中新建文件 cient.nomad, 内容如下:
```
name = "client1"

# Increase log verbosity
log_level = "DEBUG"

# Setup data dir
data_dir = "/Users/liguifa/nomad"

# Enable the client
client {
enabled = true

# For demo assume we are talking to server1. For production,
# this should be like "nomad.service.consul:4647" and a system
# like Consul used for service discovery.
servers = ["192.168.2.101:4647"]

options {
    "driver.raw_exec.enable" = "1"
  }
}

# Modify our port to avoid a collision with server1
ports {
http = 5656
}

datacenter = "release"
```
其中
name 指定client的名称是client1
servers 指定需要连接的server地址，已即是上一步中启动的server

文件修改好之后，使用下面命令启动client
```
nomad agent -config /Users/liguifa/nomad/conf/client.nomad
```

### 启动job
server和client都启动完成后，我们就可以启动job了
新建文件job.name，内容如下
```
job "cron" {
    datacenters = ["release"]
    type = "batch"
    group "test" {
        count = 1
        task "one" {
            driver = "raw_exec"
            config {
                command = "/bin/sleep"
            }
        }
    }
}
```
文件修改好之后，使用下面命令启动
```
nomad job run /Users/liguifa/nomad/conf/job.nomad
```
任务启动后我们就能在管理页面上看到我们的任务

### 总结
nomad的环境部署比较简单，总结来说就是先启动server, 然后启动client，之后我们便可提交job，三步执行之后便实现了如下效果
![](20230805.png)