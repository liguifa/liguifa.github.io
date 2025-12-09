---
title: 「ElasticSearch」ES 任务管理机制和常见任务介绍
date: 2023-08-28 21:20:24
tags: [大数据, ElasticSearch]
---
### 背景
ES 里面会有一些长时间运行的任务(ES中所有请求都被看做是任务，无论是内部还是外部的请求)，为了管理这些任务，ES在5.0中引入了一个通用的任务管理机制。通过这个管理机制，我们可以方便的查看任务的运行状态、进度，甚至能进行一定的管理操作，比如取消。 该机制会提供一个 task management 的 API 来查看和获取当前在执行的所有任务，实现上会基于现有的 TransportAction 框架，这样就可以将所有的 transport action 请求都变成 task 进行管理。 还会提供父子关系的 Task 类型，主要是由协调节点触发的父任务和下发到其他节点的子任务组成。

### 命令
1、获取Tasks
```
GET /_tasks/<task_id>

//GET /_cat/pending_tasks 用于获取在排队等待更新master节点state的任务，与本次分享task不一样
```
命令参数
| 参数         | 说明         | 默认值 |
| ----------- | ----------- |  ----------- |
| timeout     | 请求超时时间，如果超时时间内没有响应，则请求失败并返回错误信息 | 30s |
| wait_for_completion     | 如果为true，则等待匹配的任务完成 | false |
| nodes     | 逗号分隔的节点id或节点名等。检索指定节点的任务信息 |  |
| parent_task_id     | 指定父任务id，检索其所有子任务信息 | |
| actions     | 指定任务功能，返回指定功能的任务信息。接受通配符表达式 |  |
| detailed     | 如果为true，返回任务信息中会额外包括description任务描述信息 | false |
| group_by     | 对返回结果进行分组，可选值为nodes(按节点分组) none(禁用分组) parents(按父任务分组)
nodes | |
 
命令返回
```
{
  "nodes" : {
    "_0tUFfY7TnmTtgnuuu-1vQ" : {
      "name" : "node1",
      "transport_address" : "xxx:28207",
      "host" : "xxx",
      "ip" : "xxx",
      "roles" : [
        "data",
        "data_cold",
        "data_content",
        "data_frozen",
        "data_hot",
        "data_warm",
        "ingest",
        "ml",
        "remote_cluster_client",
        "transform"
      ],
      "attributes" : {
        "ml.machine_memory" : "25165824000",
        "ml.max_open_jobs" : "512",
        "box_type" : "cold",
        "xpack.installed" : "true",
        "ml.max_jvm_size" : "19327352832",
        "transform.node" : "true"
      },
      "tasks" : {
        "_0tUFfY7TnmTtgnuuu-1vQ:403526893" : {
          "node" : "_0tUFfY7TnmTtgnuuu-1vQ",
          "id" : 403526893,
          "type" : "transport",
          "action" : "cluster:monitor/tasks/lists[n]",
          "start_time_in_millis" : 1692017271538,
          "running_time_in_nanos" : 435359,
          "cancellable" : false,
          "parent_task_id" : "Qwlc397-TaWQZOma2vwU3g:296638302",
          "headers" : { }
        },
        "_0tUFfY7TnmTtgnuuu-1vQ:403526847" : {
          "node" : "_0tUFfY7TnmTtgnuuu-1vQ",
          "id" : 403526847,
          "type" : "transport",
          "action" : "indices:data/write/bulk[s][r]",
          "start_time_in_millis" : 1692017271460,
          "running_time_in_nanos" : 79076675,
          "cancellable" : false,
          "parent_task_id" : "jCFCnGK-SmWp7BSRGo6qpw:301696757",
          "headers" : { }
        }
      }
    }
}
```
| 字段         | 说明         |
| ----------- | ----------- |
|node | 节点id |
| id | 任务编号 | 
| type | 任务类型, transport、direct、persistent |
| action | 任务功能 |
| description | 任务描述信息，detailed为true时返回 |
| start_time_in_millis | 任务开始时间 |
| running_time_in_nanos | 任务执行纳秒数 |
| cancellable | 任务是否支持取消操作 |
| parent_task_id | 父任务id |
| headers | 头信息 |
取消任务
```
POST /_tasks/<task_id>/_cancel
```
<table class="drag-preview-element" style="min-width: 997px;">
    <colgroup>
        <col style="width: 53px;">
        <col style="width: 413px;">
        <col style="width: 284px;">
        <col style="width: 247px;">
    </colgroup>
    <tbody>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="mqTXQTSb_">
                    <div class="container-blocks">
                        <div class="text-block" id="mZJ3lPsRL" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="r3L6QXc2S">
                    <div class="container-blocks">
                        <div class="text-block" id="rHSTkAkzf" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">action</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="rASUHJLiJ">
                    <div class="container-blocks">
                        <div class="text-block" id="ruj2IQMDc" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">说明</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="sJVhmLDFN">
                    <div class="container-blocks">
                        <div class="text-block" id="szBKGERR1" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">备注</span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="s2nfimxvr">
                    <div class="container-blocks">
                        <div class="text-block" id="sgbgF51SK" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">1</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="sEsisoCR2">
                    <div class="container-blocks">
                        <div class="text-block" id="sequSAVHa" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:monitor/allocation/explain</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="sM3wPOrwV">
                    <div class="container-blocks">
                        <div class="text-block" id="s5SkXybAc" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">用于获取集群中shard分配相关的信息</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="sNkxoGl4R">
                    <div class="container-blocks">
                        <div class="text-block" id="sci25MTUi" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="tmWCSojWZ">
                    <div class="container-blocks">
                        <div class="text-block" id="t9RSDTTPu" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">2</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="taTeHsq5B">
                    <div class="container-blocks">
                        <div class="text-block" id="tUMs4k8NH" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span
                                    class="text">cluster:admin/voting_config/add_exclusions</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="te1G07kp_">
                    <div class="container-blocks">
                        <div class="text-block" id="t4cREoPoP" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">用于对集群投票配置添加排除点</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="uCSH2s77I">
                    <div class="container-blocks">
                        <div class="text-block" id="uiWiQry7P" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ghXAisJjh">
                    <div class="container-blocks">
                        <div class="text-block" id="gJF5Ie9P0" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">3</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="gZRhcgVu9">
                    <div class="container-blocks">
                        <div class="text-block" id="g1bvt37nG" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span
                                    class="text">cluster:admin/voting_config/clear_exclusions</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="gl8PjCgfk">
                    <div class="container-blocks">
                        <div class="text-block" id="g991v_XOs" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">用于清空集群投票排除点</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="hwvIsA0n6">
                    <div class="container-blocks">
                        <div class="text-block" id="h9GE1NZWU" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Q0hO7ehuz">
                    <div class="container-blocks">
                        <div class="text-block" id="Q71zvABEV" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">4</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Ry2tBqeck">
                    <div class="container-blocks">
                        <div class="text-block" id="RrdT8T6Ac" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:monitor/health</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="RWGHxHnpB">
                    <div class="container-blocks">
                        <div class="text-block" id="Rd7fGzTLy" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取集群状态</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ReKDJHpZa">
                    <div class="container-blocks">
                        <div class="text-block" id="RdUf9Qfpq" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="yd3SWq1Ya">
                    <div class="container-blocks">
                        <div class="text-block" id="yIRo7rULF" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">5</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ARuDjVpYz">
                    <div class="container-blocks">
                        <div class="text-block" id="AzjTebb31" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span
                                    class="text">cluster:admin/migration/get_system_feature</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="APy5bDxAb">
                    <div class="container-blocks">
                        <div class="text-block" id="AlaPi7V2F" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取集群迁移相关的信息</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="BXTsTOOY3">
                    <div class="container-blocks">
                        <div class="text-block" id="BEb8chVsr" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="kIbNkeLtG">
                    <div class="container-blocks">
                        <div class="text-block" id="kM5NPz6Sg" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">6</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="pwyv8OaXJ">
                    <div class="container-blocks">
                        <div class="text-block" id="prwu5bNfd" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:monitor/nodes/hot_threads</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="rpGjvPXKr">
                    <div class="container-blocks">
                        <div class="text-block" id="rQvR_gusn" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取集群hot线程</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="s6K6qTpCr">
                    <div class="container-blocks">
                        <div class="text-block" id="sqgW_8skD" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Xp3n1UIa7">
                    <div class="container-blocks">
                        <div class="text-block" id="XaGl4VY_A" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">7</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="cJpyzuhLe">
                    <div class="container-blocks">
                        <div class="text-block" id="c9SiBgyVf" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:monitor/nodes/info</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="cyHreCrZI">
                    <div class="container-blocks">
                        <div class="text-block" id="cw6o3sxfO" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取集群节点信息</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="czBCYh4nf">
                    <div class="container-blocks">
                        <div class="text-block" id="csNzbP1zv" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="HFtcyxjZ3">
                    <div class="container-blocks">
                        <div class="text-block" id="HpOgtvzKm" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">8</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="L7VhRYQ7u">
                    <div class="container-blocks">
                        <div class="text-block" id="LnUetraYT" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span
                                    class="text">cluster:admin/nodes/reload_secure_settings</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="LWTUNfHr0">
                    <div class="container-blocks">
                        <div class="text-block" id="LoXGR9Cc9" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">reload权限设置</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="LwY1GG2pQ">
                    <div class="container-blocks">
                        <div class="text-block" id="LcszYdAO3" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Elg2Of69B">
                    <div class="container-blocks">
                        <div class="text-block" id="Eqnb5s4j2" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">9</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="KlVR0blVc">
                    <div class="container-blocks">
                        <div class="text-block" id="KzOZr2Wm0" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:monitor/nodes/stats</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="LkOapqWXs">
                    <div class="container-blocks">
                        <div class="text-block" id="LCqYz1fLh" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取集群状态信息</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="LPhkaLwVB">
                    <div class="container-blocks">
                        <div class="text-block" id="LV74wahou" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="NQmImAudC">
                    <div class="container-blocks">
                        <div class="text-block" id="NCJaekaS4" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">10</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="NGGouwOaS">
                    <div class="container-blocks">
                        <div class="text-block" id="NZmKbgu8a" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/tasks/cancel</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="N54JuWE4y">
                    <div class="container-blocks">
                        <div class="text-block" id="NtfJ5Onvd" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">取消task</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="NjRUotN7L">
                    <div class="container-blocks">
                        <div class="text-block" id="N_iZ_AjcW" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="viSSxpnth">
                    <div class="container-blocks">
                        <div class="text-block" id="v12xf4Xld" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">11</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="vsa0W4NPz">
                    <div class="container-blocks">
                        <div class="text-block" id="v3BMb4UYg" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:monitor/task/get</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="v61Z7MQor">
                    <div class="container-blocks">
                        <div class="text-block" id="v1KRnxrCZ" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取指定task</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="vNbe02P1I">
                    <div class="container-blocks">
                        <div class="text-block" id="vEzwoikhh" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="fOioJ67fn">
                    <div class="container-blocks">
                        <div class="text-block" id="fMUMIpzM9" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">12</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="fCyPOFl2M">
                    <div class="container-blocks">
                        <div class="text-block" id="fQDchLIXc" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:monitor/tasks/lists</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="fW9p0UHrk">
                    <div class="container-blocks">
                        <div class="text-block" id="fQg78xeKK" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取全部task</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="gfxa3AXOl">
                    <div class="container-blocks">
                        <div class="text-block" id="gtLhL8DOK" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="MTawzl0OZ">
                    <div class="container-blocks">
                        <div class="text-block" id="MpEgzxWfk" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">13</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="MRyhNy5wg">
                    <div class="container-blocks">
                        <div class="text-block" id="MqXi8Mr0O" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:monitor/nodes/usage</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="NrDYtO6v6">
                    <div class="container-blocks">
                        <div class="text-block" id="NPIv2LZsO" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取节点使用情况</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="NyCXS9P_i">
                    <div class="container-blocks">
                        <div class="text-block" id="N82SJO4ry" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="vJhTEKrTv">
                    <div class="container-blocks">
                        <div class="text-block" id="vLPqdOFI3" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">14</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="vbCekjILz">
                    <div class="container-blocks">
                        <div class="text-block" id="v32Z98klN" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:monitor/remote/info</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="vvH5_05l3">
                    <div class="container-blocks">
                        <div class="text-block" id="vmWquUzTK" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取远程节点信息</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="wBEZ8PMr2">
                    <div class="container-blocks">
                        <div class="text-block" id="wkS41I0_D" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="PhG3J_tDi">
                    <div class="container-blocks">
                        <div class="text-block" id="PrXKDQKR7" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">15</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Pi0nhw_lu">
                    <div class="container-blocks">
                        <div class="text-block hover" id="P8iI4rrPQ" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/repository/_cleanup</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="P5zEDhl3b">
                    <div class="container-blocks">
                        <div class="text-block" id="P5TcIoYCh" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">清空reposity(快照相关)</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="QwMaigC3T">
                    <div class="container-blocks">
                        <div class="text-block" id="QuewibfU2" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="hADNrR3Nw">
                    <div class="container-blocks">
                        <div class="text-block" id="haRYgjthg" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">16</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ilpb1KdxB">
                    <div class="container-blocks">
                        <div class="text-block" id="irPNIaOmC" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/repository/delete</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="iuy3O9qJr">
                    <div class="container-blocks">
                        <div class="text-block" id="isQVd6wGl" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">删除reposity</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="miZB5uvM8">
                    <div class="container-blocks">
                        <div class="text-block" id="m_CCNovXW" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="TkrqaRIPV">
                    <div class="container-blocks">
                        <div class="text-block" id="TW2q7fUMw" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">17</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="TQIYZhJUS">
                    <div class="container-blocks">
                        <div class="text-block" id="TwxSgtFL_" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/repository/get</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="TsVIye1NQ">
                    <div class="container-blocks">
                        <div class="text-block" id="TV4ObjESF" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取reposity</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ZZuFLGMOm">
                    <div class="container-blocks">
                        <div class="text-block" id="ZHfTtuvme" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="vf1DdWCPH">
                    <div class="container-blocks">
                        <div class="text-block" id="vo22h9j_u" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">18</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ASFXVgNwM">
                    <div class="container-blocks">
                        <div class="text-block" id="AiLIVvR3q" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/repository/put</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="AWgBBSEas">
                    <div class="container-blocks">
                        <div class="text-block" id="AteWgxl5Z" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">添加reposity</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="BtQpTJihg">
                    <div class="container-blocks">
                        <div class="text-block" id="Bus6RxU86" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="OYUUaJRMS">
                    <div class="container-blocks">
                        <div class="text-block" id="OxdqoWtPb" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">19</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="TQQFJpoQP">
                    <div class="container-blocks">
                        <div class="text-block" id="TXknKA3Iv" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/repository/verify</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="TMLVeXxVj">
                    <div class="container-blocks">
                        <div class="text-block" id="TIrA_ZQ3D" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">验证reposity</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="UrZysYTx5">
                    <div class="container-blocks">
                        <div class="text-block" id="UOSKw2UlB" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ZUAlfjoPZ">
                    <div class="container-blocks">
                        <div class="text-block" id="ZRyHSwUSq" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">20</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="atlucpQqM">
                    <div class="container-blocks">
                        <div class="text-block" id="ahELuKBaf" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/reroute</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="aaxVEgiOl">
                    <div class="container-blocks">
                        <div class="text-block" id="a2DO_tkYa" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">分配分片</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="b47KejpdC">
                    <div class="container-blocks">
                        <div class="text-block" id="bywoHmKqx" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="iZAvp7tnw">
                    <div class="container-blocks">
                        <div class="text-block" id="idOOyXh6G" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">21</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="oTG0f3wcj">
                    <div class="container-blocks">
                        <div class="text-block" id="oUAdsQHA0" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/settings/update</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ooHdvjgaR">
                    <div class="container-blocks">
                        <div class="text-block" id="oCad5BYF9" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">更新设置</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="p9FRj6SnX">
                    <div class="container-blocks">
                        <div class="text-block" id="pdid78bKi" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="nXKK1XDu8">
                    <div class="container-blocks">
                        <div class="text-block" id="naJmYVGYQ" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">22</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="UDFi3ICq2">
                    <div class="container-blocks">
                        <div class="text-block" id="UpbKfcBj5" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/shards/search_shards</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Y3DPIwiEB">
                    <div class="container-blocks">
                        <div class="text-block" id="YlAD31Y5X" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">查询分片</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ZFph99moR">
                    <div class="container-blocks">
                        <div class="text-block" id="ZjhZmjvsU" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="UL5Eg5GD1">
                    <div class="container-blocks">
                        <div class="text-block" id="TbH6xfCJK" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">23</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="UkpOPz3Mw">
                    <div class="container-blocks">
                        <div class="text-block" id="UF7NwiXNa" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/snapshot/clone</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="VM74z11VK">
                    <div class="container-blocks">
                        <div class="text-block" id="V2t4u6QL8" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">克隆快照</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="WgM9nLiSO">
                    <div class="container-blocks">
                        <div class="text-block" id="WUDIy5LaS" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="JYHyc4nGQ">
                    <div class="container-blocks">
                        <div class="text-block" id="Jqx32l0lO" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">24</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Kf3KrF0eS">
                    <div class="container-blocks">
                        <div class="text-block" id="KbUh6nMVg" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/snapshot/create</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="K7IOyoTZA">
                    <div class="container-blocks">
                        <div class="text-block" id="KYeN76LKA" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">创建快照</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="L8L_ZM6Jq">
                    <div class="container-blocks">
                        <div class="text-block" id="L8Wda5xUF" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="uA_ltH0f_">
                    <div class="container-blocks">
                        <div class="text-block" id="uStev8cD7" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">25</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="utLf_8sb8">
                    <div class="container-blocks">
                        <div class="text-block" id="uUUKVUHoY" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/snapshot/delete</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="wN1SxDE5T">
                    <div class="container-blocks">
                        <div class="text-block" id="wzMNq4umj" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">删除快照</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="x1JVN3Sni">
                    <div class="container-blocks">
                        <div class="text-block" id="x0_7v5EZx" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="AOAdTRrtn">
                    <div class="container-blocks">
                        <div class="text-block" id="A0ESDTu6W" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">26</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Aaw5CmSP3">
                    <div class="container-blocks">
                        <div class="text-block" id="Aektlwc2Z" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/features/reset</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="BaU4cnNTu">
                    <div class="container-blocks">
                        <div class="text-block" id="BbAQygPgH" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">清除存储在系统索引中的所有状态信息</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="CCPs5rDaP">
                    <div class="container-blocks">
                        <div class="text-block" id="CQWxDtgVT" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="V2njB01vM">
                    <div class="container-blocks">
                        <div class="text-block" id="V8neL37Ti" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">27</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ar8mwjrGx">
                    <div class="container-blocks">
                        <div class="text-block" id="a6i0uEb9J" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">internal:admin/snapshot/get_shard</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="e30gO9Uow">
                    <div class="container-blocks">
                        <div class="text-block" id="eN4qvXTT9" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取分片快照</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="eqfdxuA9T">
                    <div class="container-blocks">
                        <div class="text-block" id="ejrWfSSba" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="wt2cxw3rG">
                    <div class="container-blocks">
                        <div class="text-block" id="wlPrnRhNT" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">28</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="xv5qMcZ4R">
                    <div class="container-blocks">
                        <div class="text-block" id="xHaEWsPP_" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/snapshot/get</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="y7IMFmqvS">
                    <div class="container-blocks">
                        <div class="text-block" id="ysnQBktAj" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取快照</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="z7DcjMdzO">
                    <div class="container-blocks">
                        <div class="text-block" id="z3gKJcaYZ" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="TPTT4SU3F">
                    <div class="container-blocks">
                        <div class="text-block" id="TU4Dh_iEz" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">29</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ZBtJdTix5">
                    <div class="container-blocks">
                        <div class="text-block" id="ZzNver1cE" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/snapshot/restore</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="au2fghORQ">
                    <div class="container-blocks">
                        <div class="text-block" id="al11qSFn1" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">还原快照数据</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="abeuomUZZ">
                    <div class="container-blocks">
                        <div class="text-block" id="aJzezGOHF" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="deX449tZH">
                    <div class="container-blocks">
                        <div class="text-block" id="dGnKzL5Xg" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">30</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="dp4PHiDIo">
                    <div class="container-blocks">
                        <div class="text-block" id="dNPDVet96" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/snapshot/status</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ei4yVRZA0">
                    <div class="container-blocks">
                        <div class="text-block" id="ePeS0olXb" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取快照状态</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="eoH11h3sS">
                    <div class="container-blocks">
                        <div class="text-block" id="efVJTdBK5" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="yJWnJlmqw">
                    <div class="container-blocks">
                        <div class="text-block" id="yrnRrBMZa" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">31</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="yd0VRJALx">
                    <div class="container-blocks">
                        <div class="text-block" id="yw6JoITYb" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:monitor/state</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="zpT9qQSBL">
                    <div class="container-blocks">
                        <div class="text-block" id="z39keGuJG" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取集群state信息</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="zmp0o17xr">
                    <div class="container-blocks">
                        <div class="text-block" id="zdwhtmJ5n" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="U_8DrVtyR">
                    <div class="container-blocks">
                        <div class="text-block" id="UEw2gnIJx" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">32</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="V4xJJJEBL">
                    <div class="container-blocks">
                        <div class="text-block" id="VvV9LHHu9" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:monitor/stats</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="WAh_sfj0T">
                    <div class="container-blocks">
                        <div class="text-block" id="W6ijGFaj4" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取集群状态信息</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="XdJX9woxz">
                    <div class="container-blocks">
                        <div class="text-block" id="XuD43UpsN" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ikmWJp4kY">
                    <div class="container-blocks">
                        <div class="text-block" id="iMlLRo00E" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">33</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="jPcs35g_T">
                    <div class="container-blocks">
                        <div class="text-block" id="jT1X2YKVH" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/script/delete</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="jO2uxU0lr">
                    <div class="container-blocks">
                        <div class="text-block" id="jl7sa_XuV" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">删除脚本</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="jDlBa3Uz0">
                    <div class="container-blocks">
                        <div class="text-block" id="j0hFt4th2" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="bFjdagQ8H">
                    <div class="container-blocks">
                        <div class="text-block" id="bN4Nj0Ulq" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">34</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="cNw_27jog">
                    <div class="container-blocks">
                        <div class="text-block" id="cibSeOgLE" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/script_context/get</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="clhGvCG4x">
                    <div class="container-blocks">
                        <div class="text-block" id="c5lAfp6OB" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取脚本内容</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="cXqil9mOn">
                    <div class="container-blocks">
                        <div class="text-block" id="c4OHiRG6s" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="E4LSKT6mB">
                    <div class="container-blocks">
                        <div class="text-block" id="EzfwD9AFg" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">35</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Jfd4XuRHl">
                    <div class="container-blocks">
                        <div class="text-block" id="JTTIh_p5y" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/script_language/get</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="KCGhC3Iwz">
                    <div class="container-blocks">
                        <div class="text-block" id="Ko1zUsyRw" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取脚本语言</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="LaDi6RYA4">
                    <div class="container-blocks">
                        <div class="text-block" id="LLEhW1vpk" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="RYxRICHZm">
                    <div class="container-blocks">
                        <div class="text-block" id="RAeJuIOXy" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">36</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="SXqXJk9xZ">
                    <div class="container-blocks">
                        <div class="text-block" id="SPUtnq5YH" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/script/get</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="S_2V7P9h5">
                    <div class="container-blocks">
                        <div class="text-block" id="SK9EcJ6AX" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取脚本</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="S2yGK41A0">
                    <div class="container-blocks">
                        <div class="text-block" id="Ssqghj1wY" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="q8QFoBKpK">
                    <div class="container-blocks">
                        <div class="text-block" id="q1JLGYEst" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">37</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="rs8IC4BBm">
                    <div class="container-blocks">
                        <div class="text-block" id="rraZkn7Aq" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/script/put</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="sEeosK48N">
                    <div class="container-blocks">
                        <div class="text-block" id="sDPFuDHzs" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">提交脚本</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="sARYWFeJY">
                    <div class="container-blocks">
                        <div class="text-block" id="sdAMmPy4L" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="eQgdMEqPZ">
                    <div class="container-blocks">
                        <div class="text-block" id="eV6pewdyQ" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">38</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ftNj2HLo0">
                    <div class="container-blocks">
                        <div class="text-block" id="fYHgh9fLI" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:monitor/task</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="fhqIdtJ68">
                    <div class="container-blocks">
                        <div class="text-block" id="fPbReGiMx" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取task信息</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="f_SRz4zWy">
                    <div class="container-blocks">
                        <div class="text-block" id="ft2INwFTd" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="MPvElfZ0S">
                    <div class="container-blocks">
                        <div class="text-block" id="MI0LIMSZz" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">39</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="MzKCp5hoa">
                    <div class="container-blocks">
                        <div class="text-block" id="MjJl0WmO5" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/aliases/exists</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Nq5hRb3Zr">
                    <div class="container-blocks">
                        <div class="text-block" id="NRtlnWdP9" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">别名是否存在</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="NcAZHzgmr">
                    <div class="container-blocks">
                        <div class="text-block" id="N5PlRtoRw" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="DxX9bsaXM">
                    <div class="container-blocks">
                        <div class="text-block" id="DM_B_ju2K" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">40</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="E_WX5B7zW">
                    <div class="container-blocks">
                        <div class="text-block" id="E2mjrDRYw" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/aliases/get</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="F8oUCxxV_">
                    <div class="container-blocks">
                        <div class="text-block" id="FFq64g4EM" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">过去别名</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="FoXtFbiMT">
                    <div class="container-blocks">
                        <div class="text-block" id="FCxw_awZD" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="PzHxUWiU2">
                    <div class="container-blocks">
                        <div class="text-block" id="P7_bPPBpY" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">41</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="PSADgks_p">
                    <div class="container-blocks">
                        <div class="text-block" id="PZXA8VSRF" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/aliases</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="POzBRuZ5m">
                    <div class="container-blocks">
                        <div class="text-block" id="PwHDnMLRj" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取全部别名</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="RfEV1EOOS">
                    <div class="container-blocks">
                        <div class="text-block" id="RDAEHLu5y" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Uu0AEA76S">
                    <div class="container-blocks">
                        <div class="text-block" id="UjmeCB1N3" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">42</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="U7eG3Bspb">
                    <div class="container-blocks">
                        <div class="text-block" id="UPx3HAlLO" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/analyze</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Ui7Rv1kCX">
                    <div class="container-blocks">
                        <div class="text-block" id="UoCwgco15" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">预处理数据</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Vx7Z6zoHE">
                    <div class="container-blocks">
                        <div class="text-block" id="VbvBpBNFD" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="WjMqQj7zu">
                    <div class="container-blocks">
                        <div class="text-block" id="WUNQxl03Q" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">43</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Xg1QpQcBC">
                    <div class="container-blocks">
                        <div class="text-block" id="X8qpL0oxv" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/cache/clear</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="YcMX3oaj6">
                    <div class="container-blocks">
                        <div class="text-block" id="YNrtzZL2r" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">清楚缓存</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="YGQwGAGCM">
                    <div class="container-blocks">
                        <div class="text-block" id="YRUnZHNMY" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="GZS24ZaMV">
                    <div class="container-blocks">
                        <div class="text-block" id="GJfBlNJSL" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">44</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="I6lc_V1up">
                    <div class="container-blocks">
                        <div class="text-block" id="IPq_a63oX" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/close</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="If1W3f1PT">
                    <div class="container-blocks">
                        <div class="text-block" id="IFpSjgFdA" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">关闭index</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="IxVvN7OC7">
                    <div class="container-blocks">
                        <div class="text-block" id="IC6EEy4Rj" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="LCegJTQqF">
                    <div class="container-blocks">
                        <div class="text-block" id="LGJmtHPGl" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">45</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="RGoVrnfyY">
                    <div class="container-blocks">
                        <div class="text-block" id="RI6wpjtU1" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/auto_create</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="RMr7B1v6B">
                    <div class="container-blocks">
                        <div class="text-block" id="RrWyIaGv9" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">创建index</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="SgmUd0TCk">
                    <div class="container-blocks">
                        <div class="text-block" id="SW2kDahNS" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="wEk74LRdA">
                    <div class="container-blocks">
                        <div class="text-block" id="w2hn95jWy" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">46</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="wKBk4i4ii">
                    <div class="container-blocks">
                        <div class="text-block" id="wg88ogzm2" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/create</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="w4JV3mM2R">
                    <div class="container-blocks">
                        <div class="text-block" id="wMH1CXN7I" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">创建index</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="xY5gfvmJl">
                    <div class="container-blocks">
                        <div class="text-block" id="xD_8CEvGJ" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="w9FNUDRmG">
                    <div class="container-blocks">
                        <div class="text-block" id="w3rKlwZmf" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">47</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="yzo92Td8m">
                    <div class="container-blocks">
                        <div class="text-block" id="xlXoklYJO" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span
                                    class="text">cluster:admin/indices/dangling/delete</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="zOdv2WdD5">
                    <div class="container-blocks">
                        <div class="text-block" id="zr3UkEQGh" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">删除daangline索引</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="zt2aMuSK9">
                    <div class="container-blocks">
                        <div class="text-block" id="z7cN77sjc" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">在本地存在，但是在集群中尚未存在的index， 称为dangline</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="wJAyey1PY">
                    <div class="container-blocks">
                        <div class="text-block" id="wtRenjK1b" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">48</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="B_xxWoMvE">
                    <div class="container-blocks">
                        <div class="text-block" id="AQV3ScKdo" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/indices/dangling/find</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="BL5sDt_Xf">
                    <div class="container-blocks">
                        <div class="text-block" id="BmSvoL9Dy" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">查找dangline索引</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="BRTP1AQPz">
                    <div class="container-blocks">
                        <div class="text-block" id="B3CrZiC3_" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="hrSTsqFqw">
                    <div class="container-blocks">
                        <div class="text-block" id="h29S4RXb4" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">49</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="lXE9aSueC">
                    <div class="container-blocks">
                        <div class="text-block" id="lTByoUE5F" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span
                                    class="text">cluster:admin/indices/dangling/import</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="l680K4QDz">
                    <div class="container-blocks">
                        <div class="text-block" id="lhSMwyT3E" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">导入dangline索引</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="l9NVIAPAt">
                    <div class="container-blocks">
                        <div class="text-block" id="lVAhJMQmI" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="BxuReX_yP">
                    <div class="container-blocks">
                        <div class="text-block" id="BzfZbEEY4" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">50</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Bo6F_2k78">
                    <div class="container-blocks">
                        <div class="text-block" id="BoB4qiKKA" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/indices/dangling/list</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="CNMXHdy8_">
                    <div class="container-blocks">
                        <div class="text-block" id="CCGt8TRIs" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">列出dangline索引</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Cv1eKaUG0">
                    <div class="container-blocks">
                        <div class="text-block" id="CV2B31NIF" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="tVVBDk1Wq">
                    <div class="container-blocks">
                        <div class="text-block" id="t8HoHkQzj" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">51</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="xdB1sOy5k">
                    <div class="container-blocks">
                        <div class="text-block" id="xlLfXqZ_b" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/delete</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="yHzTnAzve">
                    <div class="container-blocks">
                        <div class="text-block" id="y9ILIN7Tx" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">删除index</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="zNWnJUos_">
                    <div class="container-blocks">
                        <div class="text-block" id="zmT9yGV3J" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="eKcY_4sC6">
                    <div class="container-blocks">
                        <div class="text-block" id="dYgyMq3wc" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">52</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="igLN1P_cr">
                    <div class="container-blocks">
                        <div class="text-block" id="id8H2UQPj" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/analyze_disk_usage</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="k5Y7_lG0G">
                    <div class="container-blocks">
                        <div class="text-block" id="kaq6Kv8Gh" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">分析磁盘使用率</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="lwEw3kBRl">
                    <div class="container-blocks">
                        <div class="text-block" id="lmWOsyPKI" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="eYOLrCSnl">
                    <div class="container-blocks">
                        <div class="text-block" id="em04EgBo6" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">53</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="eXhGiBfD4">
                    <div class="container-blocks">
                        <div class="text-block" id="eVsuSVMJz" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/exists</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="fWmHgxkfw">
                    <div class="container-blocks">
                        <div class="text-block" id="frXVyBzk6" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">index是否存在</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="f4DgR8AtW">
                    <div class="container-blocks">
                        <div class="text-block" id="fWg5MemAy" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="lXsxmlOgw">
                    <div class="container-blocks">
                        <div class="text-block" id="lkwtIXxWr" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">54</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ln9C4rn6f">
                    <div class="container-blocks">
                        <div class="text-block" id="luaSMg1vn" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/types/exists</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="nGCjM9bEN">
                    <div class="container-blocks">
                        <div class="text-block" id="nghbS4eyB" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">type是否存在</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="oRBxDAYnf">
                    <div class="container-blocks">
                        <div class="text-block" id="oPlzq2jBl" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="P5Oz6eQ2N">
                    <div class="container-blocks">
                        <div class="text-block" id="Px5q20dCJ" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">55</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Q4T8LKpPM">
                    <div class="container-blocks">
                        <div class="text-block" id="KhV96XciT" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/flush</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="QdwfOGsZw">
                    <div class="container-blocks">
                        <div class="text-block" id="QxjQFZVLl" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">将page cache中的数据刷新到磁盘上</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Q2U1YfP8i">
                    <div class="container-blocks">
                        <div class="text-block" id="Qh53MTjRP" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ua6DqKtAu">
                    <div class="container-blocks">
                        <div class="text-block" id="u9UeR3ApC" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">56</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="vI5N9oXsO">
                    <div class="container-blocks">
                        <div class="text-block" id="vINrHA1g9" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/synced_flush</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="vLeSht5v2">
                    <div class="container-blocks">
                        <div class="text-block" id="v7pi3VThd" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">异步刷盘</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="vOQ3W2Sf0">
                    <div class="container-blocks">
                        <div class="text-block" id="vF61w2lm8" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="MtBizCzPG">
                    <div class="container-blocks">
                        <div class="text-block" id="MuQvD0DXa" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">57</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="McS4RQZip">
                    <div class="container-blocks">
                        <div class="text-block" id="MMlrCbJAm" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/forcemerge</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ODyztHuFC">
                    <div class="container-blocks">
                        <div class="text-block" id="O_h_kghqM" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">全部segment合并</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="OEeh2E3pG">
                    <div class="container-blocks">
                        <div class="text-block" id="Okd7RgAwh" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="mirJIT9s2">
                    <div class="container-blocks">
                        <div class="text-block" id="mLlomxCGI" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">58</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="qqhDnITaQ">
                    <div class="container-blocks">
                        <div class="text-block" id="qH9w_r7vr" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/get</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="rL1deVoEn">
                    <div class="container-blocks">
                        <div class="text-block" id="rvRTBH01Y" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">过去index信息</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="rs3nd5J4V">
                    <div class="container-blocks">
                        <div class="text-block" id="rQoIQQGpy" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="wcnVzQnmC">
                    <div class="container-blocks">
                        <div class="text-block" id="wlHiEzx67" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">59</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="w9TYXUIqN">
                    <div class="container-blocks">
                        <div class="text-block" id="waW5Lpzbl" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/mappings/fields/get</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="xyJsutnCY">
                    <div class="container-blocks">
                        <div class="text-block" id="xV2Yq2RBD" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取字段信息</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="xKs0jOAGO">
                    <div class="container-blocks">
                        <div class="text-block" id="xvf238OdA" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="gAeLuhyVZ">
                    <div class="container-blocks">
                        <div class="text-block" id="gQhF8fkmd" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">60</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="hMjIzgORt">
                    <div class="container-blocks">
                        <div class="text-block" id="hsyov40wF" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/mappings/get</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="hgJXPvLa2">
                    <div class="container-blocks">
                        <div class="text-block" id="hRL2GI_uB" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取mapping信息</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ijZRJTAWW">
                    <div class="container-blocks">
                        <div class="text-block" id="igerggC9c" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="CEciPw3HU">
                    <div class="container-blocks">
                        <div class="text-block" id="CxLa51EtP" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">61</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="C3PYJ_jFg">
                    <div class="container-blocks">
                        <div class="text-block" id="Cah_xYg4D" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/mapping/auto_put</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Dw3nTjl5M">
                    <div class="container-blocks">
                        <div class="text-block" id="D6JUF7ay3" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">自动生成mapping</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="DgPQJ2A9O">
                    <div class="container-blocks">
                        <div class="text-block" id="DenGDzT1C" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="m7JLqZnKn">
                    <div class="container-blocks">
                        <div class="text-block" id="mms30Fdad" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">62</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="pyWdWiCib">
                    <div class="container-blocks">
                        <div class="text-block" id="pCOXdnfUj" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/open</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="qQDXkJttX">
                    <div class="container-blocks">
                        <div class="text-block" id="qUFmfW01S" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">打开index</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="qjpTRTIh0">
                    <div class="container-blocks">
                        <div class="text-block" id="qYGFIFKaF" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ecFVeoeWs">
                    <div class="container-blocks">
                        <div class="text-block" id="ebluQe51w" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">63</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="hJVpwDihr">
                    <div class="container-blocks">
                        <div class="text-block" id="heSHQvO7x" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/block/add</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="iDbTo8DN9">
                    <div class="container-blocks">
                        <div class="text-block" id="iaH4LBrMe" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">state中添加块</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="jVFig9mb5">
                    <div class="container-blocks">
                        <div class="text-block" id="jnQruYuqd" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="MfC7tHMuO">
                    <div class="container-blocks">
                        <div class="text-block" id="MDwuv1qS7" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">64</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Pbs1TQWCy">
                    <div class="container-blocks">
                        <div class="text-block" id="PQcgSChNQ" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:monitor/recovery</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="QZKMMnOkb">
                    <div class="container-blocks">
                        <div class="text-block" id="QTHN42nie" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">恢复数据</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Q9V3hLJUO">
                    <div class="container-blocks">
                        <div class="text-block" id="QZUmIW22Q" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="prjMylyPU">
                    <div class="container-blocks">
                        <div class="text-block" id="pOJUMj8KZ" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">65</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="pI7VLtaF7">
                    <div class="container-blocks">
                        <div class="text-block" id="pfxJglibm" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/refresh</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="qV5lP2U6I">
                    <div class="container-blocks">
                        <div class="text-block" id="qaUBoJ6OA" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">将内存中的数据刷新到page cache</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="qh9yWIuqo">
                    <div class="container-blocks">
                        <div class="text-block" id="q64ov9Aiw" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="J3k20hNg8">
                    <div class="container-blocks">
                        <div class="text-block" id="JCiYrii1_" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">66</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="KxCVid5kb">
                    <div class="container-blocks">
                        <div class="text-block" id="Kmd0dfZ1K" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/resolve/index</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="KDU9smsLp">
                    <div class="container-blocks">
                        <div class="text-block" id="KY_LABRT7" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">解析索引、别名和数据流的指定名称</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="KwlyQzJFX">
                    <div class="container-blocks">
                        <div class="text-block" id="Kgx9xLpkU" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="fYhCVE2Lm">
                    <div class="container-blocks">
                        <div class="text-block" id="fhL_CNFVQ" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">67</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="gw6FFiMQT">
                    <div class="container-blocks">
                        <div class="text-block" id="gj6tWUd1r" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/rollover</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="g8yJmzj82">
                    <div class="container-blocks">
                        <div class="text-block" id="gYPm9991x" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">滚动index</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="gdLxUlOWA">
                    <div class="container-blocks">
                        <div class="text-block" id="g01jx4MI5" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="zg7Wls1cF">
                    <div class="container-blocks">
                        <div class="text-block" id="zYcMC1Ilj" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">68</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="zdPZftfT0">
                    <div class="container-blocks">
                        <div class="text-block" id="zbjpUTZ6K" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:monitor/segments</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="zvcCQuekm">
                    <div class="container-blocks">
                        <div class="text-block" id="zzmVORMGM" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取segments</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="A2nUwHCxn">
                    <div class="container-blocks">
                        <div class="text-block" id="AEzdHabry" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="lt5GDr7r8">
                    <div class="container-blocks">
                        <div class="text-block" id="lhed6oI7w" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">69</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="pXdtZ2f__">
                    <div class="container-blocks">
                        <div class="text-block" id="p8iO66xTF" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:monitor/settings/get</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="qjYn5UXAt">
                    <div class="container-blocks">
                        <div class="text-block" id="q80LdRz7O" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取指定的segment</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="qRYIanxFs">
                    <div class="container-blocks">
                        <div class="text-block" id="ql6xfohIq" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="sLa4iST9r">
                    <div class="container-blocks">
                        <div class="text-block" id="spycttaEU" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">70</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="sT0Nnu2t5">
                    <div class="container-blocks">
                        <div class="text-block" id="sjM1ORowK" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/settings/update</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="t9l0ht5I3">
                    <div class="container-blocks">
                        <div class="text-block" id="tUq3qahk_" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">更新index设置</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="tu5xOSFNq">
                    <div class="container-blocks">
                        <div class="text-block" id="tpNQgvO_Q" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="zkPT13xWO">
                    <div class="container-blocks">
                        <div class="text-block" id="zka_UNaxb" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">71</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="AOUqGHfhq">
                    <div class="container-blocks">
                        <div class="text-block" id="AMBQhqpmG" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:monitor/shard_stores</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Ai5z4yiTG">
                    <div class="container-blocks">
                        <div class="text-block" id="AxFdTKpsc" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取shard存储信息</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="BoeSEU9aF">
                    <div class="container-blocks">
                        <div class="text-block" id="Bv6pQAHfV" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="BD0LY9iPW">
                    <div class="container-blocks">
                        <div class="text-block" id="Bg5bf_Ag3" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">72</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="B8eWo_P3o">
                    <div class="container-blocks">
                        <div class="text-block" id="BQ1UkSrwY" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/resize</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="CTPuJTnop">
                    <div class="container-blocks">
                        <div class="text-block" id="CjnYRuOpN" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">resize index</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="CXURH4ODG">
                    <div class="container-blocks">
                        <div class="text-block" id="C5lp72pC5" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="FqnQM_anq">
                    <div class="container-blocks">
                        <div class="text-block" id="F2vQGl858" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">73</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="GgQzAEPhj">
                    <div class="container-blocks">
                        <div class="text-block" id="GVatuyEPu" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/shrink</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="GfLow8w5A">
                    <div class="container-blocks">
                        <div class="text-block" id="GOWoLkw0B" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">将现有索引缩减为具有较少主碎片的新索引</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="GLf81U9lY">
                    <div class="container-blocks">
                        <div class="text-block" id="GsQgkN2PQ" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="MdZ5GbBTD">
                    <div class="container-blocks">
                        <div class="text-block" id="MOzuMLqnU" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">74</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Pe80uHWX1">
                    <div class="container-blocks">
                        <div class="text-block" id="PNUDGvckj" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:monitor/field_usage_stats</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Qya0pmPWk">
                    <div class="container-blocks">
                        <div class="text-block" id="Q2aaKvQbm" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取字段xin'x信息</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Q6bn1UEh9">
                    <div class="container-blocks">
                        <div class="text-block" id="Q2Q2QzCaY" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="agcNB2_dw">
                    <div class="container-blocks">
                        <div class="text-block" id="aTzxaImyO" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">75</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="b4se0U9b_">
                    <div class="container-blocks">
                        <div class="text-block" id="bf2gQ_hYw" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:monitor/stats</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="bhkemVt6G">
                    <div class="container-blocks">
                        <div class="text-block" id="b1CAFYRPL" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取index的监控信息</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="dOy77mLzv">
                    <div class="container-blocks">
                        <div class="text-block" id="dlvTk4l2O" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Yeov0nVvC">
                    <div class="container-blocks">
                        <div class="text-block" id="YX5YU7DLW" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">76</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Yqiucsp9d">
                    <div class="container-blocks">
                        <div class="text-block" id="Y0CoSmWdM" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span
                                    class="text">cluster:admin/component_template/delete</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="a2lre4MOk">
                    <div class="container-blocks">
                        <div class="text-block" id="a67zLbLbJ" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">删除模板组件</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="a37NWg3IO">
                    <div class="container-blocks">
                        <div class="text-block" id="aavWpl95_" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="juPstfTwU">
                    <div class="container-blocks">
                        <div class="text-block" id="jIKAyEJOp" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">77</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="lvtzsAG7O">
                    <div class="container-blocks">
                        <div class="text-block" id="lsv2mnhAu" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/template/delete</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="lzGyZLC1G">
                    <div class="container-blocks">
                        <div class="text-block" id="lXb6ZQbSh" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">删除模板</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="mdpkBwP_k">
                    <div class="container-blocks">
                        <div class="text-block" id="mr8k92Rpt" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="wravNgEWr">
                    <div class="container-blocks">
                        <div class="text-block" id="wMiKRXsLu" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">78</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="xljQqXoSm">
                    <div class="container-blocks">
                        <div class="text-block" id="xU0VxZoNS" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span
                                    class="text">cluster:admin/component_template/get</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="y7tRQ7glW">
                    <div class="container-blocks">
                        <div class="text-block" id="yRg4uhJS1" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取模板组件</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="z82jl2fG6">
                    <div class="container-blocks">
                        <div class="text-block" id="zLaQEpMgG" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="t1pTybHpp">
                    <div class="container-blocks">
                        <div class="text-block" id="tOL3AL8x1" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">79</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="tykHv9Mof">
                    <div class="container-blocks">
                        <div class="text-block" id="tZi2xOryj" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/index_template/get</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="uYsevio8B">
                    <div class="container-blocks">
                        <div class="text-block" id="uhIRDXEod" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取指定模板</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="utMGRFsIr">
                    <div class="container-blocks">
                        <div class="text-block" id="uT4erwyJQ" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="tXXRonTg9">
                    <div class="container-blocks">
                        <div class="text-block" id="td0CBwKEY" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">80</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="zI6SoHWDE">
                    <div class="container-blocks">
                        <div class="text-block" id="z94G0rL0i" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/template/get</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="AlD6NP7HK">
                    <div class="container-blocks">
                        <div class="text-block" id="ANQW6BSPE" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取指定模板</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Bi7EV8lFZ">
                    <div class="container-blocks">
                        <div class="text-block" id="BsBnt3Ou6" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="SvJ13yaiQ">
                    <div class="container-blocks">
                        <div class="text-block" id="SrvDFHDke" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">81</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="VlBm4wQy6">
                    <div class="container-blocks">
                        <div class="text-block" id="VWKJXexqQ" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span
                                    class="text">indices:admin/index_template/simulate_index</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="WYxoKVorX">
                    <div class="container-blocks">
                        <div class="text-block" id="WnN7vOfiq" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">返回将由特定索引模板应用的索引配置</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="WygfvbCbR">
                    <div class="container-blocks">
                        <div class="text-block" id="Wk5o648GQ" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="EjMRKZMYq">
                    <div class="container-blocks">
                        <div class="text-block" id="EIOlplVgT" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">82</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="JQC4mwJgn">
                    <div class="container-blocks">
                        <div class="text-block" id="Jl6VJJkdl" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span
                                    class="text">indices:admin/index_template/simulate</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="JuVAhkhpJ">
                    <div class="container-blocks">
                        <div class="text-block" id="JBOIYs_gp" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">返回将由特定索引模板应用的索引配置</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="JzoBKKdYF">
                    <div class="container-blocks">
                        <div class="text-block" id="JqIrTjpnC" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="chg0u1hjR">
                    <div class="container-blocks">
                        <div class="text-block" id="cLUFK3kys" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">83</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="c5CPT58dQ">
                    <div class="container-blocks">
                        <div class="text-block" id="cIGo8KCbG" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span
                                    class="text">cluster:admin/component_template/put</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="c9DPZMkBh">
                    <div class="container-blocks">
                        <div class="text-block" id="cShoY5sh_" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">添加模板组件</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="d3EiiQurM">
                    <div class="container-blocks">
                        <div class="text-block" id="dcJ3JHNnb" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="F8QlfgwSs">
                    <div class="container-blocks">
                        <div class="text-block" id="FiPrKbMe3" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">84</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="F0rJql4mP">
                    <div class="container-blocks">
                        <div class="text-block" id="FUMDx_Wt1" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/template/put</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Gx1B5QMwx">
                    <div class="container-blocks">
                        <div class="text-block" id="Gxj12ruB6" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">添加模板</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="GtMZI0CpF">
                    <div class="container-blocks">
                        <div class="text-block" id="G3_IkGZ3N" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="buQPRNJoK">
                    <div class="container-blocks">
                        <div class="text-block" id="bIq6hnaI4" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">85</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ctbVqkvhH">
                    <div class="container-blocks">
                        <div class="text-block" id="c7uNN0pOf" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:monitor/upgrade</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="dxvZrxDBL">
                    <div class="container-blocks">
                        <div class="text-block" id="dNZi4kbY2" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取迁移信息</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="dSuHcicfa">
                    <div class="container-blocks">
                        <div class="text-block" id="dA8UryrPc" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="HuUIywy_r">
                    <div class="container-blocks">
                        <div class="text-block" id="HscaENH1W" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">86</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="NqTZiNZeb">
                    <div class="container-blocks">
                        <div class="text-block" id="LQcFKvQml" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/upgrade</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Na__jtAYc">
                    <div class="container-blocks">
                        <div class="text-block" id="NS6M_sm83" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">迁移index</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="NsiUQHJ4V">
                    <div class="container-blocks">
                        <div class="text-block" id="Nrzl5YnlA" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="p4Vdeq2_F">
                    <div class="container-blocks">
                        <div class="text-block" id="pPi_0dJtt" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">87</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="puBbTO8ih">
                    <div class="container-blocks">
                        <div class="text-block" id="pq89gtz1J" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">internal:indices/admin/upgrade</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="p8_9cMdak">
                    <div class="container-blocks">
                        <div class="text-block" id="ptPnoearo" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">内部迁移index</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="qdIGubtQM">
                    <div class="container-blocks">
                        <div class="text-block" id="qGJ5JFEJw" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="c81n2_kgo">
                    <div class="container-blocks">
                        <div class="text-block" id="cdKkWtlPh" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">88</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="cobIvcUQW">
                    <div class="container-blocks">
                        <div class="text-block" id="clA5Z6hLB" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/validate/query</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="cXUUpAue8">
                    <div class="container-blocks">
                        <div class="text-block" id="ceE2NHZ_Y" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">验证查询</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="cNUsDzlaM">
                    <div class="container-blocks">
                        <div class="text-block" id="c1yoI_Mr2" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Zx2SbDmBd">
                    <div class="container-blocks">
                        <div class="text-block" id="Z_zag_9be" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">89</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Zu8HVKQG8">
                    <div class="container-blocks">
                        <div class="text-block" id="Z72BlJF12" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:data/write/bulk</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="brvFDwkeI">
                    <div class="container-blocks">
                        <div class="text-block" id="bHwjHJKgW" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">批量写</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="bjdSjtwBU">
                    <div class="container-blocks">
                        <div class="text-block" id="bn3qXvioR" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="q05ny4k7Z">
                    <div class="container-blocks">
                        <div class="text-block" id="qcK5L3qHr" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">90</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="qrsYmS93U">
                    <div class="container-blocks">
                        <div class="text-block" id="qE7cA0uSJ" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:admin/data_stream/modify</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="sajuX2hTd">
                    <div class="container-blocks">
                        <div class="text-block" id="safhqp2mI" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">修改数据流</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="sh0M8PgJz">
                    <div class="container-blocks">
                        <div class="text-block" id="sYUZL56p7" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="IdJbw3ymA">
                    <div class="container-blocks">
                        <div class="text-block" id="Icd1JguQG" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">91</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="LqeKD7a6A">
                    <div class="container-blocks">
                        <div class="text-block" id="LAjpcTN0U" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:data/write/delete</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="M7M83p6SW">
                    <div class="container-blocks">
                        <div class="text-block" id="M2gl_qwEe" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">删除文档</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="MiPLekRBC">
                    <div class="container-blocks">
                        <div class="text-block" id="MoXBrPIGf" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="M4b8YW0mz">
                    <div class="container-blocks">
                        <div class="text-block" id="MHU3vekbK" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">92</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Nh4qIwo8W">
                    <div class="container-blocks">
                        <div class="text-block" id="NUthtMJXN" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:data/read/explain</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="NxxTxnUbC">
                    <div class="container-blocks">
                        <div class="text-block" id="N0wEtMhiu" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取读取的异常信息</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="NhTkvxPiz">
                    <div class="container-blocks">
                        <div class="text-block" id="N98ykzJhQ" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="BVgYb4ivW">
                    <div class="container-blocks">
                        <div class="text-block" id="BWwz28svl" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">93</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="CmDvRyCFB">
                    <div class="container-blocks">
                        <div class="text-block" id="CINSXZAZs" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:data/read/field_caps</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="C8ygl2D1M">
                    <div class="container-blocks">
                        <div class="text-block" id="CxAefr48C" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取字段caps</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="CKmob8qV5">
                    <div class="container-blocks">
                        <div class="text-block" id="ChACtBy8T" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="cfflzQItG">
                    <div class="container-blocks">
                        <div class="text-block" id="ckfZIMWmu" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">94</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="eneQGU8Hz">
                    <div class="container-blocks">
                        <div class="text-block" id="eZMriU5N_" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:data/read/get</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="fJl_AKDqG">
                    <div class="container-blocks">
                        <div class="text-block" id="fmKTvoQF9" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">通过id获取数据</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="f4kbIjHwZ">
                    <div class="container-blocks">
                        <div class="text-block" id="fWRkzgCDG" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="PprAGG_Jz">
                    <div class="container-blocks">
                        <div class="text-block" id="P0FOy_dbk" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">95</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="QOypbQOBU">
                    <div class="container-blocks">
                        <div class="text-block" id="Q7BpEhzAe" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:data/read/mget</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="RriOtPiNk">
                    <div class="container-blocks">
                        <div class="text-block" id="RV6KhzjoX" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">批量获取数据</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="R7gJZif2S">
                    <div class="container-blocks">
                        <div class="text-block" id="RnRasmaHk" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="EaKACgwPw">
                    <div class="container-blocks">
                        <div class="text-block" id="DtUqhKVCg" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">96</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="GMWFCxNqs">
                    <div class="container-blocks">
                        <div class="text-block" id="G5cA2OEph" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:data/write/index</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="HTeHpk1fM">
                    <div class="container-blocks">
                        <div class="text-block" id="HoOqW2ar4" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">写数据</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="HjSGEcueO">
                    <div class="container-blocks">
                        <div class="text-block" id="HCmj7FUMR" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="AgjZfdnHG">
                    <div class="container-blocks">
                        <div class="text-block" id="ARWOfAreA" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">97</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="BkKJWRLdq">
                    <div class="container-blocks">
                        <div class="text-block" id="BLYA4BP4e" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span
                                    class="text">cluster:admin/ingest/pipeline/delete</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="BQjd0facK">
                    <div class="container-blocks">
                        <div class="text-block" id="BwukWmQNT" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">删除预处理脚本</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Cpib2W7Zs">
                    <div class="container-blocks">
                        <div class="text-block" id="CLQfMAKM1" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="g9ttXsj5Z">
                    <div class="container-blocks">
                        <div class="text-block" id="gL6X1INbN" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">98</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="gG93t3tzm">
                    <div class="container-blocks">
                        <div class="text-block" id="gytjCWuUu" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/ingest/pipeline/get</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="gIMta3nnd">
                    <div class="container-blocks">
                        <div class="text-block" id="gp1fGTEbI" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取预处理脚本</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="gLjI04Alh">
                    <div class="container-blocks">
                        <div class="text-block" id="gJnVss7OI" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Sb02VGwnA">
                    <div class="container-blocks">
                        <div class="text-block" id="Sb1DlWEZd" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">99</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="TTY4VYcgg">
                    <div class="container-blocks">
                        <div class="text-block" id="SCkOMcKQs" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:admin/ingest/pipeline/put</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="T8CzX9nNs">
                    <div class="container-blocks">
                        <div class="text-block" id="TKmJq9Kpj" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">添加预处理脚本</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="TEg_407Sx">
                    <div class="container-blocks">
                        <div class="text-block" id="T7HcvEGwQ" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ORfCMJz41">
                    <div class="container-blocks">
                        <div class="text-block" id="OcwhBzjl0" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">100</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="PaunbO1cb">
                    <div class="container-blocks">
                        <div class="text-block" id="PD5wq_qoy" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span
                                    class="text">cluster:admin/ingest/pipeline/simulate</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Puf6MKoJ9">
                    <div class="container-blocks">
                        <div class="text-block" id="PRUk1L6aF" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取预处理脚本的配置</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="PblZpeD58">
                    <div class="container-blocks">
                        <div class="text-block" id="PRs8k_CRv" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="NqSe2meN0">
                    <div class="container-blocks">
                        <div class="text-block" id="NogqxYm0r" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">101</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="NGE1T3vEu">
                    <div class="container-blocks">
                        <div class="text-block" id="NXIghrw47" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">cluster:monitor/main</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Or54Knjtp">
                    <div class="container-blocks">
                        <div class="text-block" id="OdjwmgSe6" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">获取集群信息</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Ot8DMV8lh">
                    <div class="container-blocks">
                        <div class="text-block" id="Od5CsG7pI" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="ab2E36raf">
                    <div class="container-blocks">
                        <div class="text-block" id="a3Yw0BQDu" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">102</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="aY1GyJx1V">
                    <div class="container-blocks">
                        <div class="text-block" id="afuuty3JF" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:data/read/scroll/clear</span>
                            </div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="akx5uuYRO">
                    <div class="container-blocks">
                        <div class="text-block" id="adKyFw6br" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">清理游标</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="bR7Krlhlx">
                    <div class="container-blocks">
                        <div class="text-block" id="bbKRZg566" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="BlJn1QefQ">
                    <div class="container-blocks">
                        <div class="text-block" id="BVMtYiuVc" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">103</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="DaswaNThF">
                    <div class="container-blocks">
                        <div class="text-block" id="DDoT5waRn" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span
                                    class="text">indices:data/read/close_point_in_time</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="EyBguCxT5">
                    <div class="container-blocks">
                        <div class="text-block" id="Es7_daL72" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="EIIu5bf6Q">
                    <div class="container-blocks">
                        <div class="text-block" id="EwVej8v5v" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="IXOnZh2_r">
                    <div class="container-blocks">
                        <div class="text-block" id="IAUjg7Wi8" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">104</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="IQ4dqNOW0">
                    <div class="container-blocks">
                        <div class="text-block" id="IQadVROBQ" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:data/read/msearch</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="IsDlYx1Rr">
                    <div class="container-blocks">
                        <div class="text-block" id="IxQezypRV" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">批量搜索</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="JT_6Ldasq">
                    <div class="container-blocks">
                        <div class="text-block" id="JURFjUD1C" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="YqFjVmicP">
                    <div class="container-blocks">
                        <div class="text-block" id="YkwBMWE6t" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">105</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="bTp56qviS">
                    <div class="container-blocks">
                        <div class="text-block" id="b9ROzmg9P" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span
                                    class="text">indices:data/read/open_point_in_time</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="cNM8cU35J">
                    <div class="container-blocks">
                        <div class="text-block" id="crWtD_UvY" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="dv5lFgQwY">
                    <div class="container-blocks">
                        <div class="text-block" id="duPHj46Mw" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="KgcnDfffk">
                    <div class="container-blocks">
                        <div class="text-block" id="KzOtCqLq9" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">106</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="KXztDfOXk">
                    <div class="container-blocks">
                        <div class="text-block" id="KHryb9V3G" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:data/read/search</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="KANTC_8ZU">
                    <div class="container-blocks">
                        <div class="text-block" id="KnKl9YrFj" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">搜索</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="LtLfCNW56">
                    <div class="container-blocks">
                        <div class="text-block" id="LLk8E39C0" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="XAf9H45m9">
                    <div class="container-blocks">
                        <div class="text-block" id="Xruym6_XD" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">107</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Yytcui1kf">
                    <div class="container-blocks">
                        <div class="text-block" id="Y_Y779z7u" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:data/read/scroll</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Yvp5gi3fp">
                    <div class="container-blocks">
                        <div class="text-block" id="YO5vwY2vH" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">游标查询</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Z3Rj3Xs4q">
                    <div class="container-blocks">
                        <div class="text-block" id="ZnjxZqCmv" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="HfjKkYrJX">
                    <div class="container-blocks">
                        <div class="text-block" id="HBG9UdS6T" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">108</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="KBghUSUPg">
                    <div class="container-blocks">
                        <div class="text-block" id="KE9PxKHid" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:data/read/mtv</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="LIAaUBYTM">
                    <div class="container-blocks">
                        <div class="text-block" id="LoNEoH31t" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="LYy1PoiYM">
                    <div class="container-blocks">
                        <div class="text-block" id="LOhFuhSsk" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="B33a_ffS4">
                    <div class="container-blocks">
                        <div class="text-block" id="BHpKF1qXK" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">109</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="EMwXZslQO">
                    <div class="container-blocks">
                        <div class="text-block" id="EncwjRNit" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:data/read/tv</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="FMnioNcWR">
                    <div class="container-blocks">
                        <div class="text-block" id="FlJpaJCF0" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="FuUER21ea">
                    <div class="container-blocks">
                        <div class="text-block" id="F3ZJj8aGg" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Cfwl_bWLK">
                    <div class="container-blocks">
                        <div class="text-block" id="C3CSM2zV6" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">110</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Dxc4QeohQ">
                    <div class="container-blocks">
                        <div class="text-block" id="D8fC18zkO" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">indices:data/write/update</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="DfWh8IacZ">
                    <div class="container-blocks">
                        <div class="text-block" id="DWFsyv8Gz" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span class="text">更新数据</span></div>
                        </div>
                    </div>
                </div>
            </td>
            <td>
                <div class="child" data-type="editor-container" data-container-id="Dhl4Ldh9h">
                    <div class="container-blocks">
                        <div class="text-block" id="D0m0axOFP" data-type="editor-block" data-block-type="text">
                            <div data-type="block-content"><span><br></span></div>
                        </div>
                    </div>
                </div>
            </td>
        </tr>
    </tbody>
</table>

### 实现
task创建的时机：
1、client通过reset api发起请求后，请求会装换成一个transport action，transport action在执行之前会创建一个task
2、node与node之间通信，这个通信也是通过 transport action 请求完成的，在目标 Node 收到请求后，也会生成一个 task
整体流程:
1、client发送请求 -> rest controller接收到请求 -> 注册task -> 执行transport action -> 持久化task结果 -> 注销task -> 响应客户端
2、node1发送请求 -> node2 接收到请求 -> 注册task -> 执行transport action -> 持久化task结果 -> 注销task -> 响应node1
#### 注册
Task的注册在TaskManager类中实现，方法共有3个参数，type: 任务类型，action: 任务名称, request: 请求. 其中TaskAwareRequest是一个接口，不同的请求会实现不同的接口，
第一步: 获取到请求header, 这里会对请求头做一个大小判断， 通过http.max_header_size参数设置， 默认8K 过大会报错
第二步: 创建task，这个会调用请求类的createTask方法来创建对应的task，这里的话，task的类型会比较多，一般分类就是可取消和不可取消
第三步: 将task分为可取消和其它，其它类型的task就直接放在tasks这个列表里面，而可取消的task通过其它方法来注册
可以取消的任务主要是一些需要长时间运行的的任务，比如 reindex、update by query、delete by query、search 等
task注册完之后就将注册的task放回给注册端使用
```
public Task register(String type, String action, TaskAwareRequest request) {
    Map<String, String> headers = new HashMap<>();
    long headerSize = 0;
    long maxSize = maxHeaderSize.getBytes();
    ThreadContext threadContext = threadPool.getThreadContext();
    for (String key : taskHeaders) {
        String httpHeader = threadContext.getHeader(key);
        if (httpHeader != null) {
            headerSize += key.length() * 2 + httpHeader.length() * 2;
            if (headerSize > maxSize) {
                throw new IllegalArgumentException("Request exceeded the maximum size of task headers " + maxHeaderSize);
            }
            headers.put(key, httpHeader);
        }
    }
    Task task = request.createTask(taskIdGenerator.incrementAndGet(), type, action, request.getParentTask(), headers);
    Objects.requireNonNull(task);
    assert task.getParentTaskId().equals(request.getParentTask()) : "Request [ " + request + "] didn't preserve it parentTaskId";
    if (logger.isTraceEnabled()) {
        logger.trace("register {} [{}] [{}] [{}]", task.getId(), type, action, task.getDescription());
    }

    if (task instanceof CancellableTask) {
        registerCancellableTask(task);
    } else {
        Task previousTask = tasks.put(task.getId(), task);
        assert previousTask == null;
    }
    return task;
}
```
创建task这里类型会比较多，下面是其中两个，其中id是AtomicLong生成的一个自增id, 这个id在node内唯一
```
public Task createTask(long id, String type, String action, TaskId parentTaskId, Map<String, String> headers) {
    return new CancellableTask(id, type, action, getDescription(), parentTaskId, headers) {
        @Override
        public boolean shouldCancelChildrenOnCancellation() {
            return true;
        }
    };
}

public Task createTask(long id, String type, String action, TaskId parentTaskId, Map<String, String> headers) {
    return new SearchShardTask(id, type, action, getDescription(), parentTaskId, headers);
}
```
注册可取消的task, 这里的话就是将task包装成为一下，然后将包装后的task存大cancellableTasks这个列表中，也就是说，不可取消的在tasks列表，可取消的在cancellableTasks列表
```
private void registerCancellableTask(Task task) {
    CancellableTask cancellableTask = (CancellableTask) task;
    CancellableTaskHolder holder = new CancellableTaskHolder(cancellableTask);
    cancellableTasks.put(task, holder);
    // Check if this task was banned before we start it.
    if (task.getParentTaskId().isSet()) {
        final Ban ban = bannedParents.get(task.getParentTaskId());
        if (ban != null) {
            try {
                holder.cancel(ban.reason);
                throw new TaskCancelledException("task cancelled before starting [" + ban.reason + ']');
            } finally {
                // let's clean up the registration
                unregister(task);
            }
        }
    }
}
```
注册流程总结: 获取请求header -> 判断header大小 -> 创建task -> 如果是可取消的task就做一下包装 -> 将其添加到对应的列表在中

#### 注销
注销这里比较比较简单
1、对于可取消的任务，这里就会先将任务设置为完成，然后从cancellableTasks列表中移除
2、对于不可取消的任务，这里就直接从不可取消的tasks列表中移除
```
public Task unregister(Task task) {
    logger.trace("unregister task for id: {}", task.getId());
    if (task instanceof CancellableTask) {
        CancellableTaskHolder holder = cancellableTasks.remove(task);
        if (holder != null) {
            holder.finish();
            assert holder.task == task;
            return holder.getTask();
        } else {
            return null;
        }
    } else {
        final Task removedTask = tasks.remove(task.getId());
        assert removedTask == null || removedTask == task;
        return removedTask;
    }
}
```
完成方法就是简单的将任务标记为完成， 标记之后广播任务完成事件
```
public void finish() {
    final List<Runnable> listeners;
    synchronized (this) {
        this.finished = true;
        if (cancellationListeners != null) {
            listeners = cancellationListeners;
            cancellationListeners = null;
        } else {
            listeners = Collections.emptyList();
        }
    }
    // We need to call the listener outside of the synchronised section to avoid potential bottle necks
    // in the listener synchronization
    notifyListeners(listeners);
}
```
注销流程总结: 判断任务类型 -> 可取消的任务调用完成方法 -> 标记任务完成 -> 广播完成事件 -> 从可取消列表中移除

#### 取消
取消操作，来自客户端调用取消api, 内部会调用取消方法来取消task或内部执行异常了调用取消，这里的逻辑是，先从可取消列表中获取对应的task，如果能获取到就执行对应任务的取消操作，如果获取不到就执行task取消的广播事件
```
public void cancel(CancellableTask task, String reason, Runnable listener) {
    CancellableTaskHolder holder = cancellableTasks.get(task.getId());
    if (holder != null) {
        logger.trace("cancelling task with id {}", task.getId());
        holder.cancel(reason, listener);
    } else {
        listener.run();
    }
}
```
这里就是先关闭一下事件，然后调用对应task的取消方法，这里每个task的取消方法实现不一
```
void cancel(String reason, Runnable listener) {
    final Runnable toRun;
    synchronized (this) {
        if (finished) {
            assert cancellationListeners == null;
            toRun = listener;
        } else {
            toRun = () -> {};
            if (listener != null) {
                if (cancellationListeners == null) {
                    cancellationListeners = new ArrayList<>();
                }
                cancellationListeners.add(listener);
            }
        }
    }
    try {
        task.cancel(reason);
    } finally {
        if (toRun != null) {
            toRun.run();
        }
    }
}

void cancel(String reason) {
    task.cancel(reason);
}
```
这里看看其中一种取消的实现, 这个是reindex操作的其中一个功能的取消实现，内部实现可以概括为，重置了一些内部的数据，可以重新开始下一次操作
```
final void cancel(String reason) {
    assert reason != null;
    synchronized (this) {
        if (this.isCancelled) {
            return;
        }
        this.isCancelled = true;
        this.reason = reason;
    }
    listeners.forEach(CancellationListener::onCancelled);
    onCancelled();
}

public void onCancelled() {
    if (isWorker()) {
        workerState.handleCancel();
    }
}

public boolean isWorker() {
    return workerState != null;
}

public void handleCancel() {
    rethrottle(Float.POSITIVE_INFINITY);
}

public void rethrottle(float newRequestsPerSecond) {
    synchronized (delayedPrepareBulkRequestReference) {
        logger.debug("[{}]: rethrottling to [{}] requests per second", task.getId(), newRequestsPerSecond);
        setRequestsPerSecond(newRequestsPerSecond);

        DelayedPrepareBulkRequest delayedPrepareBulkRequest = this.delayedPrepareBulkRequestReference.get();
        if (delayedPrepareBulkRequest == null) {
            // No request has been queued so nothing to reschedule.
            logger.debug("[{}]: skipping rescheduling because there is no scheduled task", task.getId());
            return;
        }
        this.delayedPrepareBulkRequestReference.set(delayedPrepareBulkRequest.rethrottle(newRequestsPerSecond));
    }
}
```
取消流程总结: client 发送取消请求(内部调用取消) -> 获取对应的task -> 广播取消事件 -> 调用具体的任务的取消操作

#### 持久化
1、什么样的任务会被持久化？
在ES中有一些长时间运行的task，比如allocation, cluster_routing, reindex, replicas, restore, snapshot等，这些请求中会有一个参数wait_for_completion，当这个参数设置为false，任务的结果会被持久化存储
2、持久化都那里去？
当有需要持久化的任务产生时，ES会创建索引名称为.task的索引，这个索引内部就存储任务的信息和结果
3、持久化什么内容?
会持久化任务的执行的结果，会失败的结果，还有任务的信息
```
{
        "_index" : ".tasks",
        "_type" : "task",
        "_id" : "0IBSz1ouRIegZRWbXYpJng:65687962",
        "_score" : 1.0,
        "_source" : {
          "completed" : true,
          "task" : {
            "node" : "0IBSz1ouRIegZRWbXYpJng",
            "id" : 65687962,
            "type" : "transport",
            "action" : "indices:data/write/reindex",
            "status" : {
              "total" : 2226451,
              "updated" : 2212038,
              "created" : 14413,
              "deleted" : 0,
              "batches" : 2227,
              "version_conflicts" : 0,
              "noops" : 0,
              "retries" : {
                "bulk" : 0,
                "search" : 0
              },
              "throttled_millis" : 0,
              "requests_per_second" : -1.0,
              "throttled_until_millis" : 0
            },
            "description" : "reindex from [xxx-proclog-2023.01.19] to [sdwn-test-2023.01.19][_doc]",
            "start_time_in_millis" : 1692167705775,
            "running_time_in_nanos" : 650132524134,
            "cancellable" : true,
            "headers" : { }
          },
          "response" : {
            "took" : 650100,
            "timed_out" : false,
            "total" : 2226451,
            "updated" : 2212038,
            "created" : 14413,
            "deleted" : 0,
            "batches" : 2227,
            "version_conflicts" : 0,
            "noops" : 0,
            "retries" : {
              "bulk" : 0,
              "search" : 0
            },
            "throttled" : "0s",
            "throttled_millis" : 0,
            "requests_per_second" : -1.0,
            "throttled_until" : "0s",
            "throttled_until_millis" : 0,
            "failures" : [ ]
          }
        }
      }
```
下面来看看具体实现，持久化操作分为两种类型，第一种是持久化失败结果，第二种的持久化成功结果
1、持久化失败结果
第一步: 拿到任务的结果
第二步: 调用taskservice来持久化数据
```
public <Response extends ActionResponse> void storeResult(Task task, Exception error, ActionListener<Response> listener) {
    DiscoveryNode localNode = lastDiscoveryNodes.getLocalNode();
    if (localNode == null) {
        // too early to store anything, shouldn't really be here - just pass the error along
        listener.onFailure(error);
        return;
    }
    final TaskResult taskResult;
    try {
        taskResult = task.result(localNode, error);
    } catch (IOException ex) {
        logger.warn(() -> new ParameterizedMessage("couldn't store error {}", ExceptionsHelper.detailedMessage(error)), ex);
        listener.onFailure(ex);
        return;
    }
    taskResultsService.storeResult(taskResult, new ActionListener<Void>() {
        @Override
        public void onResponse(Void aVoid) {
            listener.onFailure(error);
        }

        @Override
        public void onFailure(Exception e) {
            logger.warn(() -> new ParameterizedMessage("couldn't store error {}", ExceptionsHelper.detailedMessage(error)), e);
            listener.onFailure(e);
        }
    });
}
```
2、持久化失败结果
第一步: 拿到任务的结果
第二步: 调用taskservice来持久化数据
持久化成功和失败，核心方式都是一样的，不一样的是，广播事件的方法不一样，成功的会广播成功事件，失败的会广告失败的事件
```
public <Response extends ActionResponse> void storeResult(Task task, Response response, ActionListener<Response> listener) {
    DiscoveryNode localNode = lastDiscoveryNodes.getLocalNode();
    if (localNode == null) {
        // too early to store anything, shouldn't really be here - just pass the response along
        logger.warn("couldn't store response {}, the node didn't join the cluster yet", response);
        listener.onResponse(response);
        return;
    }
    final TaskResult taskResult;
    try {
        taskResult = task.result(localNode, response);
    } catch (IOException ex) {
        logger.warn(() -> new ParameterizedMessage("couldn't store response {}", response), ex);
        listener.onFailure(ex);
        return;
    }

    taskResultsService.storeResult(taskResult, new ActionListener<Void>() {
        @Override
        public void onResponse(Void aVoid) {
            listener.onResponse(response);
        }

        @Override
        public void onFailure(Exception e) {
            logger.warn(() -> new ParameterizedMessage("couldn't store response {}", response), e);
            listener.onFailure(e);
        }
    });
}
```
核心的持久化操作在taskResultService里面
第一步: 构建持久化的index信息，index名称是.tasks，设置写入的文档id等，文档id就是task的id,  上面说过一个id, 这个id是taskId， 组成为 nodeid + 上面创建的那个id
第二步: 构建文档数据，主要的字段就是response、error、start_time_in_millis、start_time_in_millis、node和action等
第三步: 调用方法将数据写入index中， 这里如果数据写入失败，会执行重试重试14次
```
 public static final String TASK_INDEX = ".tasks";
 public static final String TASK_TYPE = "task";
 public static final String TASK_RESULT_MAPPING_VERSION_META_FIELD = "version";

public void storeResult(TaskResult taskResult, ActionListener<Void> listener) {
    IndexRequestBuilder index = client.prepareIndex(TASK_INDEX, TASK_TYPE).setId(taskResult.getTask().getTaskId().toString());
    try (XContentBuilder builder = XContentFactory.contentBuilder(Requests.INDEX_CONTENT_TYPE)) {
        taskResult.toXContent(builder, new ToXContent.MapParams(Collections.singletonMap(INCLUDE_CANCELLED_PARAM, "false")));
        index.setSource(builder);
    } catch (IOException e) {
        throw new ElasticsearchException("Couldn't convert task result to XContent for [{}]", e, taskResult.getTask());
    }
    doStoreResult(STORE_BACKOFF_POLICY.iterator(), index, listener);
}

private void doStoreResult(Iterator<TimeValue> backoff, IndexRequestBuilder index, ActionListener<Void> listener) {
    index.execute(new ActionListener<IndexResponse>() {
        @Override
        public void onResponse(IndexResponse indexResponse) {
            listener.onResponse(null);
        }

        @Override
        public void onFailure(Exception e) {
            if (false == (e instanceof EsRejectedExecutionException) || false == backoff.hasNext()) {
                listener.onFailure(e);
            } else {
                TimeValue wait = backoff.next();
                logger.warn(() -> new ParameterizedMessage("failed to store task result, retrying in [{}]", wait), e);
                threadPool.schedule(() -> doStoreResult(backoff, index, listener), wait, ThreadPool.Names.SAME);
            }
        }
    });
}
```
调用持久化的方式，是在广播事件中调用，当任务完成是会去调用很多广播事件，其中有一个是持久化结果相关的事件，这里失败调失败的持久化操作，成功调用成功的持久化操作
```
private static class TaskResultStoringActionListener<Response extends ActionResponse> implements ActionListener<Response> {
    private final ActionListener<Response> delegate;
    private final Task task;
    private final TaskManager taskManager;

    private TaskResultStoringActionListener(TaskManager taskManager, Task task, ActionListener<Response> delegate) {
        this.taskManager = taskManager;
        this.task = task;
        this.delegate = delegate;
    }

    @Override
    public void onResponse(Response response) {
        try {
            taskManager.storeResult(task, response, delegate);
        } catch (Exception e) {
            delegate.onFailure(e);
        }
    }

    @Override
    public void onFailure(Exception e) {
        try {
            taskManager.storeResult(task, e, delegate);
        } catch (Exception inner) {
            inner.addSuppressed(e);
            delegate.onFailure(inner);
        }
    }
}
```
持久化的时间是在action中注册的，当执行请求之前，会根据条件来判断是否需要持久化，如果需要1持久化就会创建一个持久化事件，当任务处理完毕会执行这个事件，判断的条件是用request.getShouldStoreResult
```
public final void execute(Task task, Request request, ActionListener<Response> listener) {
    final ActionRequestValidationException validationException;
    try {
        validationException = request.validate();
    } catch (Exception e) {
        assert false : new AssertionError("validating of request [" + request + "] threw exception", e);
        logger.warn("validating of request [" + request + "] threw exception", e);
        listener.onFailure(e);
        return;
    }
    if (validationException != null) {
        listener.onFailure(validationException);
        return;
    }
    if (task != null && request.getShouldStoreResult()) {
        listener = new TaskResultStoringActionListener<>(taskManager, task, listener);
    }

    RequestFilterChain<Request, Response> requestFilterChain = new RequestFilterChain<>(this, logger);
    requestFilterChain.proceed(task, actionName, request, listener);
}
```
getShouldStoreResult的赋值优一个set方法，也就是说，setShouldStoreResult(true) 这个task的结果就会被持久化
```
public Self setShouldStoreResult(boolean shouldStoreResult) {
    this.shouldStoreResult = shouldStoreResult;
    return self();
}
```
下面是其中一个action设置true的地方，条件是wait_for_completion=false， 也就是只有部分任务结果会被持久化，并且wait_for_completion必须为true
```
public RestChannelConsumer prepareRequest(final RestRequest request, final NodeClient client) throws IOException {
        final ForceMergeRequest mergeRequest = new ForceMergeRequest(Strings.splitStringByCommaToArray(request.param("index")));
        mergeRequest.indicesOptions(IndicesOptions.fromRequest(request, mergeRequest.indicesOptions()));
        mergeRequest.maxNumSegments(request.paramAsInt("max_num_segments", mergeRequest.maxNumSegments()));
        mergeRequest.onlyExpungeDeletes(request.paramAsBoolean("only_expunge_deletes", mergeRequest.onlyExpungeDeletes()));
        mergeRequest.flush(request.paramAsBoolean("flush", mergeRequest.flush()));
        if (request.paramAsBoolean("wait_for_completion", true)) {
            return channel -> client.admin().indices().forceMerge(mergeRequest, new RestToXContentListener<>(channel));
        } else {
            mergeRequest.setShouldStoreResult(true);
            /*
             * Let's try and validate before forking so the user gets some error. The
             * task can't totally validate until it starts but this is better than
             * nothing.
             */
            ActionRequestValidationException validationException = mergeRequest.validate();
            if (validationException != null) {
                throw validationException;
            }
            final var responseFuture = new ListenableActionFuture<ForceMergeResponse>();
            final var task = client.executeLocally(ForceMergeAction.INSTANCE, mergeRequest, responseFuture);
            responseFuture.addListener(new LoggingTaskListener<>(task));
            return sendTask(client.getLocalNodeId(), task);
        }
    }
```
持久化流程总结: 任务开始执行 -> 判断是否需要持久化(部分请求 and wait_for_completion=false) -> 如果需要持久化就创建监听事件 -> 任务直接完成广播事件 -> 触发持久化 -> 构建文档 -> 写入index(名称tasks)

#### 等待
wait_for_completion这个参数的意思是是否等待任务完成，等待的方法在也在taskManager中实现
这里逻辑是在timeout时间内不段的取tasks列表和cancellableTasks列表中去获取对应的task, 如果能获取到task就说明，任务还没有完成(因为任务完成会注销task，会从对应的列表中移除task), 任务没有完成就让线程休眠100毫秒, 然后不能获取到，就说明任务已经完成
```
public void waitForTaskCompletion(Task task, long untilInNanos) {
    while (System.nanoTime() - untilInNanos < 0) {
        if (getTask(task.getId()) == null) {
            return;
        }
        try {
            Thread.sleep(WAIT_FOR_COMPLETION_POLL.millis());
        } catch (InterruptedException e) {
            throw new ElasticsearchException("Interrupted waiting for completion of [{}]", e, task);
        }
    }
    throw new ElasticsearchTimeoutException("Timed out waiting for completion of [{}]", task);
}

public Task getTask(long id) {
    Task task = tasks.get(id);
    if (task != null) {
        return task;
    } else {
        return getCancellableTask(id);
    }
}

public CancellableTask getCancellableTask(long id) {
    CancellableTaskHolder holder = cancellableTasks.get(id);
    if (holder != null) {
        return holder.getTask();
    } else {
        return null;
    }
}
```

#### 查询
查询task， 我们调用Get /_tasks方法能获取到全部tasks, 这里其实所有tasks都在存在中，所以直接将内存的中对象返回就行。这里因为任务完成会注销task， 所有这里获取到的就正在执行中的task
```
public Map<Long, Task> getTasks() {
    HashMap<Long, Task> taskHashMap = new HashMap<>(this.tasks);
    for (CancellableTaskHolder holder : cancellableTasks.values()) {
        taskHashMap.put(holder.getTask().getId(), holder.getTask());
    }
    return Collections.unmodifiableMap(taskHashMap);
}

public Map<Long, CancellableTask> getCancellableTasks() {
    HashMap<Long, CancellableTask> taskHashMap = new HashMap<>();
    for (CancellableTaskHolder holder : cancellableTasks.values()) {
        taskHashMap.put(holder.getTask().getId(), holder.getTask());
    }
    return Collections.unmodifiableMap(taskHashMap);
}
```

#### Persistent Task
ES中还有一种被称为持久性任务(persistent task)的任务，这些任务通常是长期存在的任务，并存储在集群state中，允许在集群完全重新启动后恢复任务。每次创建持久性任务时，主节点负责将任务分配给集群的其他节点，然后分配的节点将获取任务并在本地执行。将持久性任务分配给节点的过程由以下属性控制，这些属性可以动态更新：
- cluster.persistent_tasks.allocation.enable，启用或禁用持久任务的分配：
    - all -（默认）允许将持久性任务分配给节点
    - none - 不允许为任何类型的持久性任务分配

此设置不会影响已执行的持久性任务。只有新创建的持久性任务或必须重新分配的任务（例如，在节点离开集群之后）才受此设置的影响
- cluster.persistent_tasks.allocation.recheck_interval，当集群状态发生显著变化时，主节点将自动检查是否需要分配持久性任务。但是，可能还有其他因素（例如内存使用）影响持久性任务是否可以分配给节点，但不会导致集群状态更改。此设置控制执行分配检查以响应这些因素的频率，默认值为30秒，最小允许值为10秒

持久性任务不用自己创建，只能有插件创建，有自己的执行逻辑和执行位置，这里用的不多，不做详细介绍

### 总结
- Elasticsearch中几乎所有的事情都是一项任务（搜索、索引操作、节点统计请求）
- 一些任务在后台运行，并将它们的状态同步到.tasks这样的专用索引中
- 有些任务不是从外部触发的，这些都是持久的任务
- 可以通过api获取到task的信息
- 可以通过api取消task的执行
- 请求被处理之前会注册task，请求结束后会注销task