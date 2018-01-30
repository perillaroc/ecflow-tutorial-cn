# 检查job生成

前面章节我们已经实现第一个 task（t1.ecf 文件）。t1.ecf 脚本需要经过预处理生成 jobs file。这个过程由 `ecflow_server` 在将要运行 task 时自动完成。

我们还可以在 suite definition 加载到 `ecflow_server` 前检查 job creation。

## 文本方式

检查脚本生成仅在 Python 方式下可用。

如果 `ecflow_server` 无法定位 ecf script，请参看 [ecf file location algorithm](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-ecf-file-location-algorithm)。

## Python

在 suite 定义加载到服务器前可以检查作业生成过程，检查包括：

1. 定位 ecf 脚本文件，对应 suite 定义中的每个 task
2. 进行预处理

当 suite definition 较长且包含许多 ecf script 时，这种检查可以节省大量时间。

检查 job creation 时需要注意一下几点：

1. 检查独立于 `ecflow_server`，所以 `ECF_PORT` 和 `ECF_NODE` 将被设为默认值。
2. job 文件扩展名为 `.job0`，服务器生成的 job 文件扩展名为 `.job<1-n>`，[ECF_TRYNO](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-ecf-tryno)将不为0.
3. 默认 job 文件将在 ecf 脚本同样目录下生成，请查看词汇表 [ECF_JOB](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-ecf-job)。

使用 [ecflow.Defs.check_job_creation](https://software.ecmwf.int/wiki/display/ECFLOW/ecFlow+Python+Api#ecflow.Defs.check_job_creation) 进行检查，修改 test.py

```py
import os
from ecflow import Defs,Suite,Task,Edit
    
print("Creating suite definition")
home = os.path.join(os.getenv("HOME"),  "course")
defs = Defs(
        Suite('test',
            Edit(ECF_HOME=home),
            Task('t1')))
print(defs)
 
print("Checking job creation: .ecf -> .job0") 
print(defs.check_job_creation())
 
# We can assert, so that we only progress once job creation works
# assert len(defs.check_job_creation()) == 0, "Job generation failed"
```

运行该脚本：

```
$python3 test.py
Creating suite definition
# 4.8.0
suite test
  edit ECF_HOME '../../../build/course'
  task t1
endsuite

Checking job creation: .ecf -> .job0
```

运行上述脚本后，会在 `test` 目录下生成的 `t1.job0`，文件内容如下：

```bash
#!/bin/ksh
set -e          # stop the shell on first error
set -u          # fail when using an undefined variable
set -x          # echo script lines as they are executed
set -o pipefail # fail if last(rightmost) command exits with a non-zero status

# Defines the variables that are needed for any communication with ECF
export ECF_PORT=3141    # The server port number
export ECF_HOST=localhost    # The host name where the server is running
export ECF_NAME=/test/t1    # The name of this current task
export ECF_PASS=vOW08rvF    # A unique password
export ECF_TRYNO=0  # Current try number of the task
export ECF_RID=$$             # record the process id. Also used for zombie detection

# Define the path where to find ecflow_client
# make sure client and server use the *same* version.
# Important when there are multiple versions of ecFlow
export PATH=/usr/local/apps/ecflow/4.8.0/bin:$PATH

# Tell ecFlow we have started
ecflow_client --init=$$


# Define a error handler
ERROR() {
   set +e                      # Clear -e flag, so we don't fail
   wait                        # wait for background process to stop
   ecflow_client --abort=trap  # Notify ecFlow that something went wrong, using 'trap' as the reason
   trap 0                      # Remove the trap
   exit 0                      # End the script
}


# Trap any calls to exit and errors caught by the -e flag
trap ERROR 0


# Trap any signal that may cause the script to fail
trap '{ echo "Killed by a signal"; ERROR ; }' 1 2 3 4 5 6 7 8 10 12 13 15
echo "I am part of a suite that lives in ../../../build/course"
wait                      # wait for background process to stop
ecflow_client --complete  # Notify ecFlow of a normal end
trap 0                    # Remove all traps
exit 0                    # End the shell
```
强烈建议随后的例子中使用 job creation 检查脚本。

## 任务

1. 添加 job creation 检查

2. 查看job文件 $HOME/course/test/t1.job0