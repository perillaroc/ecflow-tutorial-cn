# 变量继承

之前的章节中，我们看到如何为 [task](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-task) 定义变量（[variable](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-variable)）。
当同一 [family](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-family) 下的所有 task 都共享同一个变量值时，
该值可以定义在 family 层。这就是变量继承（[variable inheritance](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-variable-inheritance)）。

下面的例子中，也可将变量定义在 [suite](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-suite) 层，得到相同的结果。

变量从父节点继承。子节点（[node](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-node)）可以重新定义变量，这种情况下使用新的变量值。
**生成的变量**（generated variables）也可以重新定义，但不推荐这么做，除非你很清楚可能出现的后果。

## Suite Definition

### Text

```bash
# Definition of the suite test.
suite test
   edit ECF_INCLUDE "$ECF_HOME"   # replace '$ECF_HOME' with the path to your ECF_HOME directory
   edit ECF_HOME    "$ECF_HOME"
   family f1
      edit SLEEP 20
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
        Edit(SLEEP=20),
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

生成的 def 文件

```
$python test.py
Creating suite definition
# 4.8.0
suite test
  edit ECF_INCLUDE '/g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course'
  edit ECF_HOME '/g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course'
  family f1
    edit SLEEP '20'
    task t1
    task t2
  endfamily
endsuite

Checking job creation: .ecf -> .job0

Saving definition to file 'test.def'
```

![](./asset/variable_inheritance.png)

## 测试

如下的 suite definition

```bash
suite test
   edit SLEEP 100
   family f1
      edit SLEEP 80
      task t1
      task t2
         edit SLEEP 9
      family g1
          edit SLEEP 89
          task x1
              edit SLEEP 10
          task x2
      endfamily
   endfamily
   family f2
     task t1
     task t2
         edit SLEEP 77
     family g2
          task x1
              edit SLEEP 12
          task x2
      endfamily
   endfamily
endsuite
```

上面 suite 的 SLEEP 值

| node | SLEEP |
| ------------- | ------------- |
| /test/f1/t1  | 	80  |
| /test/f1/t2  | 	9 |
| /test/f1/g1/x1  | 10 |
| /test/f1/g1/x2  | 89 |
| /test/f2/t1  | 100 |
| /test/f2/t2 | 77 |
| /test/f2/g2/x1  | 12 |
| /test/f2/g2/x2 | 100 |

![](./asset/variable-inheritance-test.jpg)

## 任务

1. 修改
2. 替换 suite
3. 在 ecflow_ui 中查看修改后的 suite test
