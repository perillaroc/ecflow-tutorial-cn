# Families

可以将任务组织成类似 unix 文件系统的树形结构，其中 task 类似文件，而 family 类似文件夹。

suite 是附带额外属性的 family （参看 [Dates and Clocks](https://software.ecmwf.int/wiki/display/ECFLOW/Dates+and+Clocks#dates-and-clocks)）。类似日暮，family 可以包含其它 family。类似目录，不同的 family 中的 task 可以重名。

除非指定文件位置，否则 ecFlow 认为 suite 的结构对应文件系统中任务文件的位置。

## Ecf脚本

在下面的 suite definition 中，我们将创建包含 task t1 和 task t2 的 family f1。需要创建目录 `$ECF_HOME/course/test/f1`，并将 `t1.ecf` 和 `t2.ecf` 移动到该目录中。ecflow job 文件和 output 文件将在该目录中创建。

因为我们将脚本移动到另外的目录中，ecFlow 无法找到脚本所在目录的父目录中的两个头文件 `head.h` 和 `tail.h`。可以修改脚本，在上两级目录搜索头文件，但这种方式太累赘。

解决方法就是定义一个特殊的 ecFlow 变量 `ECF_INCLUDE`。`ECF_INCLUDE` 变量指示包含头文件的目录。参看 [pre-prossing](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-pre-processing)。

使用尖括号时，ecFlow 首先检查 `ECF_INCLUDE` 变量是否被定义。如果存在，则检查 `%ECF_INCLUDE%/head.h` 是否存在，不存在则查找 `%ECF_HOME%/head.h` 是否存在。通过这种机制，可以讲特殊的头文件放到 `ECF_INCLUDE` 目录下，将通用的头文件放到 `ECF_HOME` 目录下。更多详细信息请参看 [directives](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-directives)。

我们需要对 ecf script 脚本作如下修改：

从

```bash
%include "../head.h"
echo "I am part of a suite that lives in %ECF_HOME%"
%include "../tail.h"
```

修改为

```bash
%include <head.h>
echo "I am part of a suite that lives in %ECF_HOME%"
%include <tail.h>
```

## 修改 suite definition

修改 suite definition，添加变量定义，下面给出文本和 Python 两种不同方式的示例。

### Text

```bash
# Definition of the suite test.
 suite test
   edit ECF_INCLUDE "$ECF_HOME/course"  # replace '$ECF_HOME' with the path to your ECF_HOME directory
   edit ECF_HOME    "$ECF_HOME/course"
   family f1
      task t1
      task t2
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
        Task("t1"),
        Task("t2"))


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

运行脚本

```
$python test.py
Creating suite definition
# 4.8.0
suite test
  edit ECF_INCLUDE '/g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course'
  edit ECF_HOME '/g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course'
  family f1
    task t1
    task t2
  endfamily
endsuite

Checking job creation: .ecf -> .job0

Saving definition to file 'test.def'
```

层次结构在 ecflowview 中用树型结构表示。

![](./asset/add_family.png)

## 任务

1. 更新 suite definition
2. 创建需要的目录，并移动 ecf script
3. 编辑脚本，从 `ECF_INCLUDE` 目录中包含头文件 `head.h` 和 `tail.h`。
4. 替换 suite
5. 在 ecflowview 中查看 suite，注意树形结构。可能需要展开 test 和 f1 来查看任务。
