---
title: 业务Suite
weight: 2
---

> Operation Suite

业务 suite 包含两个循环，00 和 12.

每个循环包括三个部分；

分析

* 获取观测资料

* 运行分析模式

* 后处理数据

预报

* 准备输入数据

* 运行预报模式。预报模式每6小时输出数据，00 循环输出24小时，12 循环输出 240 小时。

归档

* 保存分析结果

* 当可用时，保存预报结果

## 任务

1. 编写 suite definition

2. ecf script 该如何组织

有用的提示

* [Add Trigger](https://software.ecmwf.int/wiki/display/ECFLOW/Add+Trigger#add-trigger)
* [Add a meter](https://software.ecmwf.int/wiki/display/ECFLOW/Add+a+meter#add-meter)
* [ecFlow variables](https://software.ecmwf.int/wiki/display/ECFLOW/ecFlow+variables#add-variable)
* [Using python scripting](https://software.ecmwf.int/wiki/display/ECFLOW/Using+python+scripting#using-python-scripting)
* [File location for ECF_FILES](https://software.ecmwf.int/wiki/display/ECFLOW/File+location#file-location)
* [Suite Definition API](https://software.ecmwf.int/wiki/display/ECFLOW/ecFlow+Python+Api#suite-definition-python-api)

参考答案《[Operational Suite Solution](https://software.ecmwf.int/wiki/display/ECFLOW/Operational+Suite+Solution#operational-suite-soln)》
