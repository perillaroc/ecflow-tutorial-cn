# 运行远程作业

ecFlow 使用 ECF_JOB_CMD 变量值提交作业。修改该变量可以控制在哪里如何运行作业。该变量应该与 ECF_JOB 和 ECF_JOBOUT变量同时使用。

* ECF_JOB 是作业文件（[job file](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-job-file)）的路径
* ECF_JOBOUT 是标准输出流的文件位置

默认的命令

```bash
ECF_JOB_CMD = %ECF_JOB% 1> %ECF_JOBOUT% 2>&1 &
```

接下来，我们将在远程主机上运行程序。需要使用 UNIX 命令 ssh。

我们喜欢使用 HOST 变量定义远程主机的名字，我们假设所有远程主机上的文件都可见（例如使用 NFS）。

下面的例子中将字符串 ?????? 替换为你的实际的主机名。

注意：远程运行任务的主机环境可能与本地运行的环境不同。这取决于你的系统如何设置。
head.h 中已经设置正确的 PATH，可以使用 child command。

如果没有这事，在 head.h 中调用 ecflow_client --init 前添加下面的行：

```bash
export PATH=$PATH:/usr/local/apps/ecflow/%ECF_VERSION%/bin
```

使用 ssh 需要远程主机上配置好 public key。检查不用密码是否能登陆到远程主机。如果需要输入密码，则需要将你的 pulic key 添加到远程机器上。执行下面的命令：

```bash
REMOTE_HOST=??????
ssh $USER@$REMOTE_HOST mkdir -p \$HOME/.ssh
cat $HOME/.ssh/id_rsa.pub || ssh-keygen -t rsa -b 2048
cat $HOME/.ssh/id_rsa.pub | ssh $USER@$REMOTE_HOST 'cat >> $HOME/.ssh/authorized_keys'
```

修改 family f5，是所有任务都在远程服务器上运行。

## Suite Definition

### Text

```bash
# Definition of the suite test
suite test
 edit ECF_INCLUDE "$HOME/course"
 edit ECF_HOME    "$HOME/course"
 limit l1 2
 
 family f5
     edit HOST ??????
     edit ECF_JOB_CMD "ssh %HOST% '%ECF_JOB% > %ECF_JOBOUT% 2>&1 &'"
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

如果 login shell 是 csh，应该这样定义 ECF_JOB_CMD：

```bash
edit ECF_JOB_CMD "ssh %HOST% '%ECF_JOB%' >& %ECF_JOBOUT%'"
```

### Python

修改前面创建的 create_family_f5() 函数。

```python
#!/usr/bin/env python2.7
import os
import ecflow

def create_family_f5() :
    f5 = ecflow.Family("f5")
    f5.add_inlimit("l1")
    f5.add_variable("HOST", "??????")
    f5.add_variable("ECF_JOB_CMD", "ssh %HOST% '%ECF_JOB% > %ECF_JOBOUT% 2>&1 &'")
    f5.add_variable("SLEEP", 20)
    for i in range(1, 10):
        f5.add_task( "t" + str(i) )
    return f5
          
print "Creating suite definition"   
defs = ecflow.Defs()
suite = defs.add_suite("test")
suite.add_variable("ECF_INCLUDE", os.path.join(os.getenv("HOME"),  "course"))
suite.add_variable("ECF_HOME",    os.path.join(os.getenv("HOME"),  "course"))

suite.add_limit("l1", 2)
suite.add_family( create_family_f5() )
print defs

print "Checking job creation: .ecf -> .job0"   
print defs.check_job_creation()

print "Saving definition to file 'test.def'"
defs.save_as_defs("test.def")
```

## 任务

1. 修改 head.h 中的环境变量

2. 修改 suite definitino

3. 替换 suite definition

4. 可能不会立即生效，查看日志文件 $HOME/course/host.port.ecf.log 寻找原因。

5. 在 ecf script 脚本中添加 uname -n 检查任务运行在哪台主机

6. 如何才能让 /test/f5/t9 运行在另外一台主机上？实验你的方法。
