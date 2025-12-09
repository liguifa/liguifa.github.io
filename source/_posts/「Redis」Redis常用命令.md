---
title: 「Redis」Redis常用命令
date: 2023-07-27 19:53:36
tags: [Redis, 数据库]
---
| # | 命令 | 参数 | 例子 | 功能 | 返回值 | 原理 |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | keys | pattern: 键名配置模式 | keys \* | 输出所有键 | 返回键列表 | 遍历所有键，和输入的模式做匹配，时间复杂度为O(n)，当Redis保存大量键时，线上环境禁止使用 |
| 2 | dbsize |  | dbsize | 获取键总数 | 返回键总数 | 直接获取Redis内置键总数变量，时间复杂度为O(1) |
| 3 | exists | key: 键名 | exists key | 检查键是否存在 | 1：键存在 0：键不存在 |  |
| 4 | del | key: 键名列表 | del key1 key2 | 删除键 | 1：删除成功 0：键不存在 |  |
| 5 | expire | key: 键名 seconds:过期时间 | expire key1 10 | 键过期 | 1：设置成功 0：键不存在 |  |
| 6 | ttl | key:键名 | ttl key1 | 获取剩余过期时间 | \>=0：键剩余的过期时间 -1：键没有设置过期时间 -2：键不存在 |  |
| 7 | type | key: 键名 | type key1 | 获取键的数据类型 | 返回键类型 如果键不存在则返回none |  |
| 8 | set | key: 键名 value: 值 | set name hello | 设置键 |  |  |
| 9 | get | key:键名 | get name | 获取键值 | 键值 |  |