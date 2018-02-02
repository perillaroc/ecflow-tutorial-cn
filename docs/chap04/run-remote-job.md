# 运行远程作业

ecFlow 使用 `ECF_JOB_CMD` 变量值提交作业。修改该变量可以控制在哪里如何运行作业。
该变量应该与 `ECF_JOB` 和 `ECF_JOBOUT` 变量同时使用。

* `ECF_JOB` 是作业文件（[job file](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-job-file)）的路径
* `ECF_JOBOUT` 是标准输出流的文件位置

默认的命令如下：

```bash
ECF_JOB_CMD = %ECF_JOB% 1> %ECF_JOBOUT% 2>&1 &
```

接下来，我们将在远程主机上运行程序。需要使用 UNIX 命令 `ssh`。

我们使用 `HOST` 变量定义远程主机的名字，我们假设所有远程主机上的文件都可见（例如使用 NFS）。

下面的例子中将字符串 `??????` 替换为你的实际的主机名。

> 注意：远程运行任务的主机环境可能与本地运行的环境不同。这取决于你的系统如何设置。
`head.h` 中应该使用设置正确的 `PATH`，可以直接调用 child command。<br/>
如果没有设置，在 `head.h` 中调用 `ecflow_client --init` 前添加下面的语句：

```bash
export PATH=$PATH:/usr/local/apps/ecflow/%ECF_VERSION%/bin
```

使用 `ssh` 需要远程主机上配置好 public key。
检查不用密码是否能登陆到远程主机。如果需要输入密码，则需要将你的 pulic key 添加到远程机器上。
执行下面的命令：

```bash
REMOTE_HOST=??????
ssh $USER@$REMOTE_HOST mkdir -p \$HOME/.ssh
cat $HOME/.ssh/id_rsa.pub || ssh-keygen -t rsa -b 2048
cat $HOME/.ssh/id_rsa.pub | ssh $USER@$REMOTE_HOST 'cat >> $HOME/.ssh/authorized_keys'
```

修改 family f5，是所有任务都在远程服务器上运行。本教程中的 ecflow 服务运行在 `login05` 节点中，下面使用 `login08` 节点运行 f5 下的所有作业。

## Suite Definition

### Text

```bash
# Definition of the suite test
suite test
 edit ECF_INCLUDE "$ECF_HOME"
 edit ECF_HOME    "$ECF_HOME"
 limit l1 2
 
 family f5
     edit HOST ??????
     edit ECF_OUT /tmp/$USER
     edit ECF_JOB_CMD "ssh %HOST% 'mkdir -p %ECF_OUT%/%SUITE%/%FAMILY% && %ECF_JOB% > %ECF_JOBOUT% 2>&1 &'"
     inlimit l1
     edit SLEEP 20
     task t1
     task t2
     task t3
     task t4
     task t5
     task t6
     task t7
     task t8
     task t9
 endfamily
endsuite
```

如果 login shell 是 csh，应该这样定义 `ECF_JOB_CMD`：

```bash
edit ECF_JOB_CMD "ssh %HOST% 'mkdir -p %ECF_OUT%/%SUITE%/%FAMILY%; %ECF_JOB% >& %ECF_JOBOUT%'"
```

### Python

修改前面创建的 `create_family_f5()` 函数。

```python
import os
from pathlib import Path
from ecflow import Defs, Suite, Task, Family, Edit, Trigger, \
    Event, Complete, Meter, Time, Day, Date, Cron, Label, \
    RepeatString, RepeatInteger, RepeatDate, Limit, InLimit, \
    Late

# ...skip...

def create_family_f5():
    return Family("f5",
                  InLimit("l1"),
                  Edit(SLEEP=20,
                       HOST='login08',
                       ECF_LOGHOST="%HOST%",
                       ECF_LOGPORT="33084",
                       ECF_JOB_CMD="ssh %HOST% '%ECF_JOB% >& %ECF_JOBOUT%'"),
                  [Task('t{}'.format(i)) for i in range(1, 10)])

# ...skip...

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

运行脚本：

```
$python test.py
Creating suite definition
# 4.8.0
suite test
  edit ECF_INCLUDE '/g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course'
  edit ECF_HOME '/g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course'
  limit l1 2
  # ... skip ...
  family f5
    edit SLEEP '20'
    edit HOST 'login08'
    edit ECF_LOGHOST '%HOST%'
    edit ECF_LOGPORT '33084'
    edit ECF_JOB_CMD 'ssh %HOST% '%ECF_JOB% >& %ECF_JOBOUT%''
    inlimit l1
    task t1
    task t2
    task t3
    task t4
    task t5
    task t6
    task t7
    task t8
    task t9
  endfamily
  # ... skip ...
endsuite

Checking job creation: .ecf -> .job0

Saving definition to file 'test.def'
```

## Logserver

我们可以通过使用一个日志服务器查看远程服务器上的输出文件。

假设已定义变量 `ECF_LOGHOST` 和 `ECF_LOGPORT`。

在远程服务器上运行 logserver：

```bash
ssh $USER@class01 /usr/local/apps/ecflow/4.8.0/bin/start_logserver -d /tmp/$USER -m /tmp/$USER:/tmp/$USER
```

> 译者注：尚未实验该功能，后续添加。

## 任务

1. 修改 `head.h` 中的环境变量

2. 修改 suite definitino

3. 替换 suite definition

4. 可能不会立即生效，查看日志文件 `$ECF_HOME/host.port.ecf.log` 寻找原因。

5. 在 ecf script 脚本中添加 `hostname` 检查任务运行在哪台主机

    ![](./asset/remote_run.png)

6. 如何才能让 `/test/f5/t9` 运行在另外一台主机上？实验你的方法。

7. 创建一个 log 服务器，访问远程输出。