---
title: 「ElasticSearch」Kibana Timelion语法函数总结
date: 2023-07-29 20:05:35
tags: [大数据, ElasticSearch, Kibana]
---
工作中需要在Kibana上做图，但是对kibana上使用的Timelion语法一知半解的，遇到问题总数很难解决，故总结一下Timelion里面的函数，还可以给其它网友做个参考, 信息来自<a href='https://github.com/elastic/timelion/blob/master/FUNCTIONS.md' target='_blank'>官方文档</a>

### es(或elasticsearch)
从ElastSearch中提取数据, Timelion函数始终以点开始，后跟函数名称，后跟括号，其中包含函数的所有参数
参数:
*   q: es的查询语句
*   metric: 值必须是一个聚合函数，后跟一个冒号，后跟一个字段名, 需要展示的指标字段
*   split: 值必须是一个字段名，后跟一个冒号，后跟一个数字值,提取指定字段的前n个值
*   index: 要查询的索引名称, 如果为空则查询高级设置中的es.default_index
*   timefield: 时间戳字段, 如果空则使用高级设置中的es.timefield
*   offset: 时间偏移，offset=-1h 往前偏移一小时
*   kibana: 尊重Kibana仪表板上的过滤器。只有在Kibana仪表板上使用时才有效果, 布尔值, true/false
*   fit: 用于将序列拟合到目标时间跨度和间隔的算法, 填充空值
    -   average: 用空值边上的两个值取平均数1
    -   carry: 在空值边上的两个点处上升或下降
    -   none: 在空值边上的两个点的中间位置上升或下降
    -   nearest: 使用距离最近的点的值
    -   scale: 直接将附件的两个点直接连接

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response',
    fit='none'
)
```

### static(或value)
以静态值作为数据源输入
参数:
*   value: 要显示的单个值，您也可以传递多个值，它将在您的时间范围内均匀地对它们进行显示
*   label: 显示的标签

示例:
```
.static(10, "测试")
```

### graphite
从graphite中提取数据。在timelion.json中配置石墨服务器
参数: 
*   metric: 一个字段名, 需要展示的指标字段
*   offset: 时间偏移，offset=-1h 往前偏移一小时
*   fit: 用于将序列拟合到目标时间跨度和间隔的算法, 填充空值
    -   average: 用空值边上的两个值取平均数1
    -   carry: 在空值边上的两个点处上升或下降
    -   none: 在空值边上的两个点的中间位置上升或下降
    -   nearest: 使用距离最近的点的值
    -   scale: 直接将附件的两个点直接连接

示例:
```
.graphite(metric='_test-data.users.*.data')
```

### quandl
使用quandl代码从quandl.com中提取数据。将您的免费API密钥放在timelion.json中。API在没有密钥的情况下是受费率限制的
参数:
*   code: 要绘制的quandl代码。你可以在quandl.com上找到这些
*   position: 需要展示的列序号
*   offset: 时间偏移，offset=-1h 往前偏移一小时
*   fit: 用于将序列拟合到目标时间跨度和间隔的算法, 填充空值
    -   average: 用空值边上的两个值取平均数1
    -   carry: 在空值边上的两个点处上升或下降
    -   none: 在空值边上的两个点的中间位置上升或下降
    -   nearest: 使用距离最近的点的值
    -   scale: 直接将附件的两个点直接连接

示例:
```
.quandl()
```

### worldbank_indicators
从中提取数据 http://data.worldbank.org 使用国家名称和指示符。世界银行主要提供年度数据，通常没有当年的数据。如果您没有得到最近时间范围的数据，请尝试偏移量=-1y。
参数: 
*   country: 世界银行国家标识符，通常是国家的2个字母代码
*   indicator: 要展示的指标, 可以再data.worldbank.org上查询, 例如SP.POP.TOTL是人口
*   offset: 时间偏移，offset=-1h 往前偏移一小时
*   fit: 用于将序列拟合到目标时间跨度和间隔的算法, 填充空值
    -   average: 用空值边上的两个值取平均数1
    -   carry: 在空值边上的两个点处上升或下降
    -   none: 在空值边上的两个点的中间位置上升或下降
    -   nearest: 使用距离最近的点的值
    -   scale: 直接将附件的两个点直接连接

示例:
```
.worldbank_indicators(
    country=GH,
    indicator="VC.IHR.PSRC.P5"
)
```

### worldbank
从中提取数据 http://data.worldbank.org 世界银行主要提供年度数据，通常没有当年的数据。如果您没有得到最近时间范围的数据，请尝试偏移量=-1y。
参数: 
*   code: 世界银行API路径。这通常是域之后、querystring之前的所有内容。例如：/en/国家/ind；chn/indicators/DPANUSSPF。
*   offset: 时间偏移，offset=-1h 往前偏移一小时
*   fit: 用于将序列拟合到目标时间跨度和间隔的算法, 填充空值
    -   average: 用空值边上的两个值取平均数1
    -   carry: 在空值边上的两个点处上升或下降
    -   none: 在空值边上的两个点的中间位置上升或下降
    -   nearest: 使用距离最近的点的值
    -   scale: 直接将附件的两个点直接连接

示例:
```
.worldbank(
    code='/indicator/VC.IHR.PSRC.P5?locations=BT-GH'
)
```

### abs
返回序列列表中每个值的绝对值
参数: 无
示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response'
)
.abs()
```

### add(或plus 或sum)
将seriesList中的一个或多个系列的值添加到每个系列的每个位置
参数:
*   term: 将被叠加的值
    -   number: 一个固定的数字
    -   seriesList: 将另一个seriesList同叠加当前seriesList

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response'
)
.add(1993)

.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response'
)
.add(
    .es(
        index='bigdata-hostlog-squid-*', 
        timefield='timestamp',
        metric='sum:req_num'
    )
)
```

### aggregate
根据处理序列中所有点的结果创建静态线
参数:
*   function: 使用一个函数
    -   avg: 平均值
    -   cardinality: 基数
    -   min: 最小值
    -   max: 最大值
    -   last: 最后值
    -   first: 第一个值
    -   sum: 累计值

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response'
)
.add(
    .es(
        index='bigdata-hostlog-squid-*', 
        timefield='timestamp',
        metric='sum:req_num'
    )
)
.aggregate(cardinality)
```

### bars
将图表显示为条形图
参数:
*   width: 条形图的宽度
    -   number: 一个数据，指示条形图多宽
    -   null: 空值, 自适应
*   stack:  是否堆叠
    -   bool: 布尔值, 表示是否
    -   null: 空值, 自适应

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response'
)
.add(
    .es(
        index='bigdata-hostlog-squid-*', 
        timefield='timestamp',
        metric='sum:req_num'
    )
)
.bars(100)
```

### color
更改图表的颜色
参数:
*   color: 需要更改到的颜色
    -   rgb: 用rgb表示颜色, 例: #44ff33
    -   string: 用颜色英文名, 例: red

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response'
)
.add(
    .es(
        index='bigdata-hostlog-squid-*', 
        timefield='timestamp',
        metric='sum:req_num'
    )
)
.bars(100)
.color(#ff0000)
```

### cusum
返回从基数开始的序列的累计和, 在特定时间点的值是所有先前点的值累计得到, 累计和仅从图形的开始时间计算
参数:
*   base: 指定基数
    -   number: 一个数字作为基数

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response'
)
.cusum(100)
```

### derivative
计算时间序列的导数，即曲线的斜率
参数: 无
示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response'
)
.derivative()
```

### divide
将seriesList中一个或多个系列的值除以输入系列List中每个系列的每个位置
参数:
*   divisor: 除数
    -   seriesList: 一个list作为除数
    -   number: 一个固定的数字作为除数

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response'
)
.divide(  
    .es(
        index='bigdata-hostlog-squid-*', 
        timefield='timestamp',
        metric='sum:req_num'
    )
)
```

### fit
用一个拟合数字来填充空值
参数:
*   fit: 用于将序列拟合到目标时间跨度和间隔的算法, 填充空值
    -   average: 用空值边上的两个值取平均数1
    -   carry: 在空值边上的两个点处上升或下降
    -   none: 在空值边上的两个点的中间位置上升或下降
    -   nearest: 使用距离最近的点的值
    -   scale: 直接将附件的两个点直接连接

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response',
)
.divide(  
    .es(
        index='bigdata-hostlog-squid-*', 
        timefield='timestamp',
        metric='sum:req_num',
    )
)
.fit(scale)
```

### hide
设置图标数据是否隐藏
参数:
*   hide: 是否隐藏
    -   bool: true/false
    _   null: 默认 true
示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response',
)
.hide(true)
```

### holt
使用开头的几个数据，预测未来的数据趋势。即预测还未发送的时间的数据
参数:
*   alpha: 	将权重从0平滑到1。增加alpha将使新系列更接近原始系列。降低它将使序列更平滑
*   beta: 趋势权重从0到1。增加贝塔系数将使上升/下降线持续上升/下降的时间更长。降低它将使函数更快地了解新趋势
*   gamma: 季节性重量从0到1。你的数据看起来像波浪吗？增加这个值将使最近的季节更加重要，从而更快地改变波形。降低它将降低新赛季的重要性，使历史变得更加重要
*   season: 季节有多长，例如，如果你的模式每周重复一次，则为1w
*   sample: 季节序列中开始“预测”之前要采样的季节数

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response'
),
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response'
)
.holt(0.9, 0.1, 0.9, 0.5h)
```

### if(或condition)
对数据进行if表达式测试，如果通过使用某一个值，否则使用另一个值
参数:
*   operator: 判断表达式
    -   lt: 小于
    -   lte: 小于等于
    -   eq: 等于
    -   gt: 大于
    -   gte: 大于等于
*   if: 判断值， > xxx 里面的xxx
*   then: 如果满足条件，则使用这个值
*   else: 如果不满足调整，则使用这个值

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response',
)
.if(lt, 10000000, 20, 30)
```

### label
更改序列的标签。使用$引用现有标签
参数:
*   label: 显示标签的文字
    -   string: 一个固定的字符串
    -   $1: 取正则表达式里面的分组
*   regex: 正则表达式

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response',
)
.label("测试")
```

### legend
设置图例在绘图上的位置和样式
参数:
*   position: 图例的位置
    -   nw: 左上角
    -   ne: 右上角
    -   se: 右下角
    -   sw: 左下角
    -   false: 隐藏图例
*   columns: 每一行多少列
*   showTime: 是否显示时间(在图标中滑动可以查询当前点时间), 是否可选择隐藏显示分组数据
*   timeFormat: 时间格式, <a href='https://momentjs.com/docs/#/displaying/format' target="_blank">格式查看这里</a>

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response',
    split='http_code:4'
)
.legend(nw, 4, true)
```

### lines(
将seriesList显示为折线图
参数:
*   width: 线条粗细
*   fill: 数字介于0和10之间，用于制作面积图, 数值表示颜色深度
*   stack: 是否将线条显示未堆叠图
*   show: 是否显示线条
*   steps: 步长, 例如，点之间不要有数据

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response',
    split='http_code:4'
)
.lines(1.2, 2, false, true, 200)
```

### log
叫数据列表中的数据计算对数
参数:
*   base: 对数的基数, 即log2 里面的2

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response',
    split='http_code:4'
)
.log(2)
```

### max
将点的现有值与传递值比较，取其中较大的值
参数:
*   value: 传递的将要比较的值
    -   number: 与一个固定的数字进行比较
    -   seriesList: 与一个时间序列比较，对应点的数据进行比较

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response',
    split='http_code:4'
)
.max(20)
```

### min
将点的现有值与传递值比较，取其中较小的值
参数:
*   value: 传递的将要比较的值
    -   number: 与一个固定的数字进行比较
    -   seriesList: 与一个时间序列比较，对应点的数据进行比较

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response',
    split='http_code:4'
)
.min(20)
```

### multiply
将seriesList中一个或多个系列的值乘以输入系列List中每个系列的每个位置
参数:
*   multiplier: 除数
    -   seriesList: 一个list作为乘数
    -   number: 一个固定的数字作为乘数

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response'
)
.multiply(  
    .es(
        index='bigdata-hostlog-squid-*', 
        timefield='timestamp',
        metric='sum:req_num'
    )
)
```

### mvavg
计算给定窗口上的移动平均值
参数:
*   window: 时间窗口长度
    -   string: 1d, 1m 等日期数学表达式
    -   number: 平均值的点数
*   position: 时间窗口的位置
    -   left: 时间窗口在点的左边
    -   right: 时间窗口在点的右边
    -   center: 时间窗口居中

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response',
    split='http_code:4'
)
.mvavg(2m, right)
```

### mvstd
计算给定窗口上的移动标准偏差。使用朴素的两步算法
参数:
*   window: 时间窗口长度
    -   number: 平均值的点数
*   position: 时间窗口的位置
    -   left: 时间窗口在点的左边
    -   right: 时间窗口在点的右边
    -   center: 时间窗口居中

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response',
    split='http_code:4'
)
.mvstd(5, right)
```

### points
将图标显示成点状图
参数:
*   radius: 点的半径大小
*   weight: 点线的宽度
*   fill: 点填充的不透明度，0到10
*   fillColor: 填充的颜色, rgb值
*   symbol: 点的形状
    -   triangle: 三角形
    -   cross: 十字形
    -   square: 正方形
    -   diamond: 菱形
    -   circle: 圆形
*   show: 是否显示点状图

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response',
    split='http_code:4'
)
.points(10, 1, 5, #f4f4f4, cross, true)
```

### precision
保留小数的位数，四舍五入
参数:
*   precision: 要保留的位数

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response',
    split='http_code:4'
)
.precision(2)
```

### range
更改序列中的最大值和最小值
参数:
*   min: 最小值
*   max: 最大值

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response',
    split='http_code:4'
)
.range(2, 5)
```

### scale_interval
将值计算为频率，byte/s 这种数据
参数:
*   interval: 时间间隔, 例如: e.g., 1s, 1m, 5m, 1w, 1M, 1y

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response'
)
.scale_interval(20m)
```

### subtract
将seriesList中一个或多个系列的值减去输入系列List中每个系列的每个位置
参数:
*   term: 除数
    -   seriesList: 一个list作为减数
    -   number: 一个固定的数字作为减数

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response'
)
.subtract(  
    .es(
        index='bigdata-hostlog-squid-*', 
        timefield='timestamp',
        metric='sum:req_num'
    )
)
```

### title:
在图标的顶部显示一个标题
参数:
*   title: 标题内容

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response',
    split='http_code:4'
)
.title("测试")
```

### trend
使用指定的回归算法绘制趋势线
参数:
*   mode: 指定算法
    -   linear: 线型
    -   log: 对数
*   start: 开始位置不计算的点数, 正数, 例如 +15
*   end: 结束位置不计算的点数, 负数, 例如 -15

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response'
),
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response'
)
.trend(log, 2, -3)
```


### trim
将系列开始或结束处的N个桶设置为空
参数:
*   start: 开始位置置空的点的个数
*   end: 结束位置置空的点的个数

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response'
)
.trim(10, 2)
```

### yaxis
配置各种y轴选项，最重要的可能是能够添加第N个（例如第2个）y轴
参数:
*   position: y轴的位置, left和right
*   color: y轴的标签的颜色
*   label: y轴的标签
*   min: y轴的最小值
*   max: y轴的最大值
*   tickDecimals: y轴刻度的小数位数
*   units: y轴刻度的单位, bits/s, bytes, bytes/s, currency(:ISO 4217 currency code), percent, custom(:prefix:suffix)
*   yaxis: y轴的1编号，可以绘制多个y轴, 如果不绘制多个.es，则无意义

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response'
)
.yaxis(
    position=right,
    color=red,
    min=10,
    max=200000,
    label='测试',
    tickDecimals=2,
    units=percent,
    yaxis=1
),
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:req_num'
)
.yaxis(
    position=left,
    color=blue,
    min=-10,
    max=2000,
    label='测试2',
    tickDecimals=2,
    yaxis=2
)
```

### props
设置系列的任意属性
参数:
*   global: 设置任意属性

示例:
```
.es(
    index='bigdata-hostlog-squid-*', 
    timefield='timestamp',
    metric='sum:time_response',
)
.props(label='123')
```
