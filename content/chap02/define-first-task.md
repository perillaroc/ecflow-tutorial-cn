---
title: 定义第一个task
weight: 3
---

接下来，我们需要为 task t1 编写 ecf script。

默认情况下，ecFlow 认为文件放在 `ECF_HOME` 目录下的一个目录结构中，该层次结构反映 suite 的层次关系。task t1 在 suite test 中，对应的 ecf script 应该在子目录 `test` 下。

在 `ECF_HOME` 下创建 `test` 文件夹

```text
$ mkdir test
```

在 `test` 中，创建 `t1.ecf`，文件内容为

```bash
%include "../head.h"
echo "I am part of a suite that lives in %ECF_HOME%"
%include "../tail.h"
```

## 脚本创建

在提交任务前，服务会将 ecf script 转化成一个 [job file](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-job-file)。
这个过程叫做 [job creation](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-job-creation)。

包括在磁盘中定位 ecf script，然后预处理指令。这个步骤包括进行变量替换
（[variable substitution](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-variable-substitution)）。

这将创建一个以.job结尾的文件，`ecflow_server` 将该文件提交给你的系统。

在我们的例子中：

- `%include "../head.h"` 被 `head.h` 文件的内容替换。
- 注意文件路径相对于 `t1.ecf`，本例中的 `head.h` 位于 `t1.ecf` 所在目录的上一级目录中。`%ECF_HOME%` 被 `ECF_HOME` 变量的值替换
- `%include "../tail.h"` 被 `tail.h` 文件的内容替换。
