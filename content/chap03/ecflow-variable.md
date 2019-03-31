---
title: ecFlow变量
weight: 4
---

我们已经看到 [ecFlow](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-ecflow) 使用一些变量
（[variable](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-variable)），比如 `ECF_HOME`，`ECF_INCLUDE`。

共有三种变量：

* ecFlow 使用的变量，例如 `ECF_HOME`
* 用户定义的变量，不应该以 `ECF` 开头，推荐使用**大写字母**来定义变量。
* ecFlow 生成的变量，可以在 job 中使用，例如包含 suite 的日期 `ECF_DATE`。

## Ecf脚本

之前的例子中，我们复制 `t1.ecf` 为 `t2.ecf`。
编辑这两个文件，以用户定义的变量 `SLEEP` 为参数调用 unix 的 `sleep` 命令。

```bash
%include <head.h>
echo "I will now sleep for %SLEEP% seconds"
sleep %SLEEP%
%include <tail.h>
```

## suite definition

添加变量到 [suite definition](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-suite-definition)

### Text

```bash
# Definition of the suite test.
suite test
   edit ECF_INCLUDE "$ECF_HOME"   # replace '$ECF_HOME' with the path to your ECF_HOME directory
   edit ECF_HOME    "$ECF_HOME"
   family f1
      task t1
         edit SLEEP 20
      task t2
         edit SLEEP 20
   endfamily
endsuite
```

### Python

```py
import os
from pathlib import Path
from ecflow import Defs, Suite, Task, Family, Edit


def create_family_f1():
    return Family(
        "f1",
        Task("t1", Edit(SLEEP=20)),
        Task("t2", Edit(SLEEP=20)))


print("Creating suite definition")
home = os.path.abspath(Path(Path(__file__).parent, "../../../build/course"))
defs = Defs(
    Suite('test',
          Edit(ECF_INCLUDE=home, ECF_HOME=home),
          create_family_f1()))
print(defs)

print("Checking job creation: .ecf -> .job0")
print(defs.check_job_creation())

print("Saving definition to file 'test.def'")
defs.save_as_defs(str(Path(home, "test.def")))

# To restore the definition from file 'test.def' we can use:
# restored_defs = ecflow.Defs("test.def")
```

运行脚本：

```
$python test.py
Creating suite definition
# 4.8.0
suite test
  edit ECF_INCLUDE '/g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course'
  edit ECF_HOME '/g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course'
  family f1
    task t1
      edit SLEEP '20'
    task t2
      edit SLEEP '20'
  endfamily
endsuite

Checking job creation: .ecf -> .job0

Saving definition to file 'test.def'
```

## 任务

1. 修改文件
2. 替换 [suite](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-suite)
3. 查看 ecflow_ui，将看到任务处于 [active](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-active) 状态 20 秒，查看 job output。

![](asset/add_variables.png)

