# 定义新的 suite

有多种定义 suite definition 的方法，参看 [Definition creation strategies](https://software.ecmwf.int/wiki/display/ECFLOW/Definition+creation+strategies#strategy)。

本教程介绍下面两种方式：

- 文本方法
- Python 方法

## 文本方法

创建文本文件 `test.def`，内容如下：

```bash
# Definition of the suite test
suite test
   edit ECF_HOME "$ECF_HOME"  # replace '$ECF_HOME' with the path to your ECF_HOME directory
   task t1
endsuite
```

> 译者注：与 SMS 的定义方式相同，只需要修改变量名，将 `SMS_XXX` 修改为 `ECF_XXX`。

上述文件包含一个名为 `test` 的 suite 的 suite definition，该 suite 包含一个名为 `t1` 的 task。

下面逐行解释含义

1. 该行为注释。在 `#` 后到行尾之间的所有字符都会被忽略。
2. 定义一个名为 test 的新的 suite。
3. 定义一个 ecflow 变量（[variable](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-variable)），叫做 `ECF_HOME`。该变量定义定义名为 test 的 suite 可以在哪里找到所有的 unix 文件。余下的课程中，所有的文件名都相对于该目录。确保用你的 `ECF_HOME` 目录替换 `$ECF_HOME`。
4. 定义一个名为 t1 的 task。
5. [endsuite](https://software.ecmwf.int/wiki/display/ECFLOW/Definition+file+Grammar#grammar-token-endsuite) 结束名为 test 的 suite 的定义。

## python方法

创建一个 python 文件，例如命名为 `test.py`:

```py
import os
from ecflow import Defs, Suite, Task, Edit

print("Creating suite definition")
home = os.path.join(os.getenv("HOME"), "course")
defs = Defs(
    Suite('test',
          Edit(ECF_HOME=home),
          Task('t1')))
print(defs)
```

运行脚本

```
$ python test.py 
Creating suite definition
# 4.8.0
suite test
  edit ECF_HOME '/g1/u/wangdp/course'
  task t1
endsuite
```

附：下面时 2017 版的 suite 定义脚本，可以看到更新后的 ecflow 提供更简洁的 suite 定义方式。

```py
#!/usr/bin/env python2.7
import os
import ecflow 
   
print "Creating suite definition"   
defs = ecflow.Defs()
suite = defs.add_suite("test")
suite.add_variable("ECF_HOME", os.path.join(os.getenv("HOME"),  "course"))
suite.add_task("t1")
```

接下来的所有 Python 例子都应该用这种方式运行。

## 任务

1. 尝试测试两种方法，后续例子将只用 python 接口
2. 创建并编辑 suite definition 文件
