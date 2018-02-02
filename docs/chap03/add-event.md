# 添加事件

有时，等待一个任务结束还不够。如果任务产生多个结果，另一个任务或许需要在第一个结果生成时就开始运行。

ecFlow 引入事件 event 概念。event 是任务运行时发送给 `ecflow_server` 的一个消息，每个任务可以设置多个事件。event 是 trigger 的一种。

## Ecf Script

我们将创建新的 task（t3，t4），它们被 t2 发出的事件触发。
通过拷贝 t1 创建 t3 和 t4 的 ecf script。
为了通知 `ecflow_server`，task（下面示例中的 t2）必须调用 `ecflow_client --event`。

### t2.ecf

```bash
%include <head.h>
echo "I will now sleep for %SLEEP% seconds"
sleep %SLEEP%
ecflow_client --event a       # Set the first event
sleep %SLEEP%                 # Sleep a bit more
ecflow_client --event b       # Set the second event
sleep %SLEEP%                 # A last nap...
%include <tail.h>
```

## Suite Definition

### Text

```bash

# Definition of the suite test.
suite test
   edit ECF_INCLUDE "$ECF_HOME"  # replace '$ECF_HOME' with the path to your ECF_HOME directory
   edit ECF_HOME    "$ECF_HOME"
   family f1
     edit SLEEP 20
     task t1
     task t2
         trigger t1 eq complete
         event a
         event b
     task t3
         trigger t2:a
     task t4
         trigger t2:b
   endfamily
endsuite
```

### Python

```python
import os
from pathlib import Path
from ecflow import Defs, Suite, Task, Family, Edit, Trigger, Event


def create_family_f1():
    return Family(
        "f1",
        Edit(SLEEP=20),
        Task("t1"),
        Task(
            "t2",
            Trigger("t1 == complete"),
            Event('a'),
            Event('b')),
        Task(
            "t3",
            Trigger("t2:a"),
            Trigger("t2:b"))
    )


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

运行结果：

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
      trigger t1 == complete
      event a
      event b
    task t3
      trigger t2:a
    task t4
      trigger t2:b
  endfamily
endsuite

Checking job creation: .ecf -> .job0

Saving definition to file 'test.def'
```

## 任务

1. 更新 `test.def` 或 `test.py`
2. 编辑 `t2.ecf`，添加 `ecflow_client –event` 调用
3. 拷贝 `t1.ecf` 为 `t3.ecf` 和 `t4.ecf`
4. 替换 suite
5. 在 ecflow_ui 中观察任务

    ![](./asset/add_event_replace.png)

6. 查看 t3 的触发器

    ![](./asset/add_event_t2.png)

7. 查看 t2 的触发器

    ![](./asset/add_event_t3.png)
