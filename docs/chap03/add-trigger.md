# 添加触发器

前一个练习中，我们看到两个 task 同时运行。我们想确保 t2 只在 t1 完成后再运行，因此需要定义触发器 [trigger](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-trigger)。

Trigger 用来声明两个任务间的依赖关系（[dependencies](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-dependencies)），
例如，二号任务可能需要一号任务生成的数据。
当 ecFlow 尝试启动一个任务时，它会检查 trigger 表达式。如果条件满足，任务启动，相反任务保持 [queued](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-queued) 状态。
Trigger 可以用在任务间、family 间或者两者的混合。

记住下面两条规则：

* 所有任务都完成时，family 才完成
* 任务的 trigger 和所有父节点的 trigger 都满足时，任务才会启动。

每个节点只能有一个 trigger 表达式，但可以构建非常复杂的表达式（记住父节点的 trigger 也是隐含的 trigger）。

有时 trigger 用于防止同一时间运行过多的任务。这种情况下更好的方法就是使用 [limit](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-limit) （后面将会介绍 limit）。

trigger 中的表达式可以以使用节点的全名，例如

* /test/f1/t1 表示 [task](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-task) t1
* /test/f1 表示 [family](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-family) f1

```bash
trigger /test/f1/t1 == complete
```

一些情况下，ecFlow 接受相对名称，例如../t1

Trigger 可以非常复杂，ecFlow 支持所有的条件语句（not、and、or 等等），并且 trigger 可以引用节点属性，
例如 [event](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-event), 
[meter](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-meter), 
[variable](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-variable), 
[repeat](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-repeat)，limits 和生成变量。

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
         trigger t1 eq complete
   endfamily
endsuite
```

### Python

```python
import os
from pathlib import Path
from ecflow import Defs, Suite, Task, Family, Edit, Trigger


def create_family_f1():
    return Family(
        "f1",
        Edit(SLEEP=20),
        Task("t1"),
        Task("t2", Trigger("t1 == complete")))


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
    edit SLEEP '20'
    task t1
    task t2
      trigger t1 == complete
  endfamily
endsuite

Checking job creation: .ecf -> .job0

Saving definition to file 'test.def'
```

![](./asset/add_trigger.png)

## 任务

1. 编辑 [suite definition](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-suite-definition)，添加 trigger
2. 替换 [suite](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-suite)
3. 在 ecflow_ui 中观察任务。
4. 在 trigger 标签中查看 t1 和 t2 的 trigger。

![](./asset/add_trigger_t1.png)

![](./asset/add_trigger_t2.png)

5. 搜索t1

![](./asset/add_trigger_search_t1.png)
