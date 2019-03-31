---
title: 数据获取
weight: 1
---

## 要求

每小时，从 Exeter，Toulouse 和 Offenbach 接收数据。
每三小时，从 Washington 接收数据
每天，从 Tokeyo 接收数据
每周一，从 Melbourne 接收数据
每月第一天，从 Montreal 接收数据

接收的数据种类：

* 观测
* GRIB 场
* 卫星图片

接收数据需要三个步骤：

* 从外部接收数据
* 处理数据
* 将数据保存到数据库中

每天从数据库中检索前一天接收的数据并写入归档文件。

## 任务

1. 编写 suite 的 suite definition

有用的参考：

* [Add Trigger](https://software.ecmwf.int/wiki/display/ECFLOW/Add+Trigger#add-trigger)
* [Dates and Clocks](https://software.ecmwf.int/wiki/display/ECFLOW/Dates+and+Clocks#dates-and-clocks)
* [time]()
* [date or day](https://software.ecmwf.int/wiki/display/ECFLOW/Time+Dependencies#date-or-day)
* [Repeat](https://software.ecmwf.int/wiki/display/ECFLOW/Repeat#repeat)
* [Suite Definhition API](https://software.ecmwf.int/wiki/display/ECFLOW/ecFlow+Python+Api#suite-definition-python-api)

因为没有标准 unix 日期操作命令，可以使用 `ecf_date`。

参考答案《[Data acquisition solution](https://software.ecmwf.int/wiki/display/ECFLOW/Data+acquisition+solution#data-acquisition-soln)》