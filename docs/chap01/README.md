# 介绍

这篇教程的目的是通过一个简单的示例介绍 ecFlow 的功能。

每个章节介绍一个新概念，并提供一系列任务。大部分章节都附带在线 ecflow 文档的链接，便于读者查看。

## 使用 ecFlow 的步骤

### 1. 编写 suite 定义

多个 [task](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-task) 可以组成 family，
family 可以属于另外的 family 或者属于 [suite](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-suite)。
所有的实体（task，family，suite）都叫做 [node](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-node)，
构成一个层次树。

有两个主要方法向 [ecflow_server](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-ecflow-server) 描述 suite 定义（[suite definition](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-suite-definition)）：文本文件，python API接口

**文本文件**

* 不支持条件语句，不支持函数
* 可以使用各种语言生成文本文件
* 与 SMS/CDP 类似，便于移植

语法参见《[Definition file Grammar](https://software.ecmwf.int/wiki/display/ECFLOW/Definition+file+Grammar)》

**Python接口**

推荐方式，提供更多功能

参见 [ecFlow Python Api](https://software.ecmwf.int/wiki/display/ECFLOW/ecFlow+Python+Api#python-api)

> 译者注：强烈建议使用 Python 接口，这也是 ecFlow 的优势之一。文本定义语法编写判断、循环及定义函数的能力很弱，def 文件很难复用。

### 2.编写脚本

[ecf script](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-ecf-script) 是对应 suite 定义中 task 的文本文件。
脚本定义任务重需要运行的主要工作，包含 [child command](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-child-command) 调用，特殊注释，以及为用户提供信息的说明段落。

child command 是 [ecflow_client](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-ecflow-client) 的命令子集，
实现与 `ecflow_server` 的通讯。
这些命令通知服务器某任务已经开始，完成，出错，或设置某些事件（[event](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-event)）。

### 3.启动 ecflow 服务器

`ecflow_server` 启动后，可以加载 suite definition

用户接着启动 `ecflow_server` 中的调度器（[scheduling](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-scheduling)）

调度器会每分钟检查 suite definition 的依赖关系（[dependencies](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-dependencies)）。
如果满足依赖关系，服务器会提交任务（task）。
这个过程叫做作业生成（[job creation](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-job-creation)）。
任务（task）对应的运行过程被称为作业（job）。

运行中的作业使用 child command 与服务器通信。这些会导致：

* 服务器中保存的节点（node）状态（[status](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-status)）改变
* 更新节点的属性（比如事件（event），标尺（[meter](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-meter)），标签（[label](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-label)））

### 4. 使用 GUI 交互

ecFlow 有专用的 GUI 客户端，叫做 ecflow_ui，用于可视化和监控：

* suite definition 的树形结构（suite，family，task）
* 节点和服务器的状态改变
* 节点的属性和任意依赖关系
* ecf 脚本文件和扩展的作业文件（job file）

`ecflow_ui` 提供一系列使用 ecflow_client 命令的工具与服务器交互。
