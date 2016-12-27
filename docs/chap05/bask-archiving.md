# Back Archiving

该例子只运行一次。

## 说明

某个旧归档系统中有从 1990-01-01 到 1995-07-12 的数据。

这些数据需要拷贝到新的归档系统中。

在写入新归档系统前需要处理数据。

数据可以被单独拷贝。

旧归档系统中的数据按日期组织。系统应该拷贝一天的数据。

同一时刻只有两个任务能访问旧归档系统。

## 任务

1. 编写 suite definition

2. 设计 suite，使新的数据类型可以方便地添加其中。

有用的提示：

* [Limits](https://software.ecmwf.int/wiki/display/ECFLOW/Limits#limits)
* [inlimit](https://software.ecmwf.int/wiki/display/ECFLOW/Limits#inlimit)
* [ecFlow variables](https://software.ecmwf.int/wiki/display/ECFLOW/ecFlow+variables#add-variable)
* [Add Trigger](https://software.ecmwf.int/wiki/display/ECFLOW/Add+Trigger#add-trigger)
* [Repeat](https://software.ecmwf.int/wiki/display/ECFLOW/Repeat#repeat)
* [Using python scripting](https://software.ecmwf.int/wiki/display/ECFLOW/Using+python+scripting#using-python-scripting)
* [Suite Definition API](https://software.ecmwf.int/wiki/display/ECFLOW/ecFlow+Python+Api#suite-definition-python-api)

参考答案《[Back archiving solution](https://software.ecmwf.int/wiki/display/ECFLOW/Back+archiving+solution#back-archiving-soln)》