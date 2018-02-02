# Late 属性

有时任务没有按期望运行，我们希望在这种情况发生时得到通知。可以使用 `late` 属性实现。

每个节点只能有 1 个 `late` 属性，通常放在 task 节点上。可以在 suite 或 family 节点中定义，子节点可以继承该属性。
底层定义的 `late` 属性会覆盖掉上层定义的 `late` 属性。

- `-s submitted`：节点可能保持在 `submitted` 状态的时间（格式:`[+]hh:mm`）。sumitted 总是相对的，所以 `+` 被忽略。如果节点在 `submitted` 状态超过给定时间，`late` 标志则会被设置。
- `-a Active`：节点必须启动（状态为 `active`）的时间（格式：`hh:mm`）。如果节点还处于 `queued` 或 `submitted` 状态，则会设置 `late` 标志。
- `-c Complete`：节点必须完成（状态为 `complete`）的时间（格式：`{+}hh:mm`）。如果是相对时间，时间从节点开始运行后计算，否则节点必须在给定时间点时完成。

示例

```bash
task t1
   late -s +00:15 -a 20:00 -c +02:00
```

上面的设置含义是：节点保持在 `submitted` 状态不能超过 15 分钟，必须在 20:00 前启动，运行时间不能超过 2 小时。

本教程中，我们只为运行时间添加 `late` 属性。

## Ecf 脚本

```bash
%include <head.h>
echo "I will now sleep for %SLEEP% seconds"
sleep %SLEEP%
%include <tail.h>
```

## Suite 定义

### 文本方式

```
# Definition of the suite test.
suite test
 edit ECF_INCLUDE "$ECF_HOME"    # replace $ECF_HOME with the path to your ECF_HOME directory
 edit ECF_HOME    "$ECF_HOME"
 
 family f6
      edit SLEEP 120
      task t1
           late -c +00:01 # set late flag if task take longer than a minute
 endfamily
endsuite
```

### Python

```py
import os
from pathlib import Path
from ecflow import Defs, Suite, Task, Family, Edit, Trigger, \
    Event, Complete, Meter, Time, Day, Date, Cron, Label, \
    RepeatString, RepeatInteger, RepeatDate, Limit, InLimit, \
    Late

# ...skip...

def create_family_f6():
    return Family("f6",
                  Edit(SLEEP=120),
                  Task('t1',
                       Late(complete="+00:01")))


print("Creating suite definition")
home = os.path.abspath(Path(Path(__file__).parent, "../../../build/course"))
defs = Defs(
    Suite('test',
          Edit(ECF_INCLUDE=home, ECF_HOME=home),
          Limit("l1", 2),
          create_family_f1(),
          create_family_house_keeping(),
          create_family_f3(),
          create_family_f4(),
          create_family_f5(),
          create_family_f6()))
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
  # ... skip ...
  family f6
    edit SLEEP '120'
    task t1
      late -c +00:01
  endfamily
endsuite

Checking job creation: .ecf -> .job0

Saving definition to file 'test.def'
```

## 任务

1. 修改
2. 替换 suite 定义
3. 运行 suite，在 ecflow_ui 中应该可以看到 task 的 late 标签。

    ![](./asset/add_late.png)
