---
name: time-helper
description: 时间工具，获取多时区当前时间、Unix时间戳、时区互相转换。支持 HKT、UTC、GMT、CET、IST(印度)、BST(英国夏令)等时区。
user-invocable: true
---

# 技能目标
提供时间相关能力：
1. 获取多个指定时区的当前本地时间
2. 获取当前Unix时间戳（秒、毫秒两种格式）
3. 将某个时间字符串，从源时区转换到目标时区
4. 输出标准可读格式：`yyyy-MM-dd HH:mm:ss`，附带时区标识

# 支持时区缩写清单
- UTC：世界协调时间
- HKT：香港时间 UTC+8
- CST：中国标准时间 UTC+8
- BST：英国夏令时 UTC+1（UK夏令）
- GMT：格林尼治标准 UTC+0
- CET：中欧时间 UTC+1
- IST：印度标准时间 UTC+5:30
- EST：北美东部时间
- PST：太平洋时间

# 输入格式约定
用户输入支持几种形式：
1. 直接查询：`当前 HKT,IST,BST 时间`
2. 查询时间戳：`获取当前时间戳（秒/毫秒）`
3. 时区转换：`2026-09-16 10:00 HKT 转 IST`

# 输出格式