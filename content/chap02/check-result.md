---
title: 检查结果
weight: 8
---

使用如下命令查看 suite 的运行状态：

```
$ecflow_client --host=login05 --port=33083 --get_state
# 4.8.0
defs_state STATE state>:complete flag:message state_change:84 modify_change:15
  edit ECF_MICRO '%' # server
  edit ECF_HOME '/g3/wangdp/ecf_home' # server
  edit ECF_JOB_CMD '%ECF_JOB% 1> %ECF_JOBOUT% 2>&1' # server
  edit ECF_KILL_CMD 'kill -15 %ECF_RID%' # server
  edit ECF_STATUS_CMD 'ps --sid %ECF_RID% -f' # server
  edit ECF_URL_CMD '${BROWSER:=firefox} -remote 'openURL(%ECF_URL_BASE%/%ECF_URL%)'' # server
  edit ECF_URL_BASE 'https://software.ecmwf.int' # server
  edit ECF_URL 'wiki/display/ECFLOW/Home' # server
  edit ECF_LOG '/g3/wangdp/ecf_home/a303r6n1.33083.ecf.log' # server
  edit ECF_INTERVAL '60' # server
  edit ECF_LISTS '/g3/wangdp/ecf_home/ecf.lists' # server
  edit ECF_CHECK '/g3/wangdp/ecf_home/a303r6n1.33083.check' # server
  edit ECF_CHECKOLD '/g3/wangdp/ecf_home/a303r6n1.33083.check.b' # server
  edit ECF_CHECKINTERVAL '120' # server
  edit ECF_CHECKMODE 'CHECK_ON_TIME' # server
  edit ECF_TRIES '2' # server
  edit ECF_VERSION '4.7.1' # server
  edit ECF_PORT '33083' # server
  edit ECF_NODE '%ECF_HOST%' # server
  edit ECF_HOST 'a303r6n1' # server
  edit ECF_PID '127321' # server
# server state: RUNNING
suite test #  begun:1 state:complete
  edit ECF_HOME '/g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course'
  # edit SUITE 'test'
  # edit ECF_DATE '20180131'
  # edit YYYY '2018'
  # edit DOW '3'
  # edit DOY '31'
  # edit DATE '31.01.2018'
  # edit DAY 'wednesday'
  # edit DD '31'
  # edit MM '01'
  # edit MONTH 'january'
  # edit ECF_CLOCK 'wednesday:january:3:31'
  # edit ECF_TIME '01:45'
  # edit ECF_JULIAN '2458150'
  # edit TIME '0145'
  calendar initTime:2018-Jan-30 14:23:09 suiteTime:2018-Jan-31 01:45:00 duration:11:21:51 initLocalTime:2018-Jan-30 14:23:09 lastTime:2018-Jan-31 01:45:00 calendarIncrement:00:01:00
  task t1 # try:1 state:complete
    # edit TASK 't1'
    # edit ECF_JOB '/g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course/test/t1.job1'
    # edit ECF_SCRIPT '/g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course/test/t1.ecf'
    # edit ECF_JOBOUT '/g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course/test/t1.1'
    # edit ECF_TRYNO '1'
    # edit ECF_RID ''
    # edit ECF_NAME '/test/t1'
    # edit ECF_PASS ''
endsuite
```

上述命令会从服务其中检索 suite definition ，并显示每个节点的状态。

查看 task t1，如果 `t1` 是 [complete](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-complete) 状态，
并且 suite 是 complete 状态，那么运行成功。如果不是这种情况，则可能会有 [aborted](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-aborted) 状态。

请检查 ecf script 的目录。服务器在 ecf script 相同的目录下创建 job file，名为 `t1.job1`。
比较 `t1.ecf`，`head.h`，`tail.h` 和 `t1.job1`。作业的输出文件也放在 ecf script 的目录下，名为 `t1.1`。

## t1.ecf

```bash
%include "../head.h"
echo "I am part of a suite that lives in %ECF_HOME%"
%include "../tail.h"
```

## head.h

```bash
#!%SHELL:/bin/ksh%
set -e          # stop the shell on first error
set -u          # fail when using an undefined variable
set -x          # echo script lines as they are executed
set -o pipefail # fail if last(rightmost) command exits with a non-zero status

# Defines the variables that are needed for any communication with ECF
export ECF_PORT=%ECF_PORT%    # The server port number
export ECF_HOST=%ECF_HOST%    # The host name where the server is running
export ECF_NAME=%ECF_NAME%    # The name of this current task
export ECF_PASS=%ECF_PASS%    # A unique password
export ECF_TRYNO=%ECF_TRYNO%  # Current try number of the task
export ECF_RID=$$             # record the process id. Also used for zombie detection

# Define the path where to find ecflow_client
# make sure client and server use the *same* version.
# Important when there are multiple versions of ecFlow
export PATH=/usr/local/apps/ecflow/%ECF_VERSION%/bin:$PATH

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
```

## tail.h

```bash
wait                      # wait for background process to stop
ecflow_client --complete  # Notify ecFlow of a normal end
trap 0                    # Remove all traps
exit 0                    # End the shell
```

## t1.job1

```bash
#!/bin/ksh
set -e          # stop the shell on first error
set -u          # fail when using an undefined variable
set -x          # echo script lines as they are executed
set -o pipefail # fail if last(rightmost) command exits with a non-zero status

# Defines the variables that are needed for any communication with ECF
export ECF_PORT=33083    # The server port number
export ECF_HOST=a303r6n1    # The host name where the server is running
export ECF_NAME=/test/t1    # The name of this current task
export ECF_PASS=0Fqbbc.7    # A unique password
export ECF_TRYNO=1  # Current try number of the task
export ECF_RID=$$             # record the process id. Also used for zombie detection

# Define the path where to find ecflow_client
# make sure client and server use the *same* version.
# Important when there are multiple versions of ecFlow
export PATH=/usr/local/apps/ecflow/4.7.1/bin:$PATH

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
echo "I am part of a suite that lives in /g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course"
wait                      # wait for background process to stop
ecflow_client --complete  # Notify ecFlow of a normal end
trap 0                    # Remove all traps
exit 0                    # End the shell
```

## t1.1

```bash
+ set -o pipefail
+ ECF_PORT=33083
+ export ECF_PORT
+ ECF_HOST=a303r6n1
+ export ECF_HOST
+ ECF_NAME=/test/t1
+ export ECF_NAME
+ ECF_PASS=0Fqbbc.7
+ export ECF_PASS
+ ECF_TRYNO=1
+ export ECF_TRYNO
+ ECF_RID=82186
+ export ECF_RID
+ PATH=/usr/local/apps/ecflow/4.7.1/bin:/g3/wangdp/usr/local/bin:/g1/app/mathlib/ncl_ncarg/6.4.0/gnu/bin:/g1/app//mathlib/netcdf/3.6.3/intel/bin:/g1/app/mathlib/hdf/4.2.13/intel/bin:/opt/mpi/intelmpi/2017.2.174/intel64/bin:/opt/compiler/intel/composer_xe_2017.2.174/bin/intel64:/usr/lib64/qt-3.3/bin:/g1/app/apps/perforce/bin:/g1/app/apps/perforce:/opt/gridview/slurm17/bin:/opt/gridview/slurm17/sbin:/opt/gridview/munge/bin:/opt/gridview/munge/sbin:/opt/clusconf/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/g1/app/apps/ecflow/4.7.1/bin:/opt/ibutils/bin:/g1/u/wangdp/.local/bin:/g1/u/wangdp/bin
+ export PATH
+ ecflow_client '--init=82186'
+ trap ERROR 0
+ trap '{ echo "Killed by a signal"; ERROR ; }' 1 2 3 4 5 6 7 8 10 12 13 15
+ echo 'I am part of a suite that lives in /g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course'
I am part of a suite that lives in /g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course
+ wait
+ ecflow_client --complete
+ trap 0
+ exit 0
```

## 获取 suite definition

获取可以解析的 suite definition 定义

```
$ecflow_client --host=login05 --port=33083 --get
# 4.8.0
suite test
  edit ECF_HOME '/g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course'
  task t1
endsuite
```

可以通过 Python API，编写如下文件 `get_suite.py`

```python
import ecflow

try:
    ci = ecflow.Client('login05', 33083)
    ci.sync_local()                                   # get server definition, by sync with client defs
    ecflow.PrintStyle.set_style( ecflow.Style.DEFS )  # set printing to show structure

    print(ci.get_defs())                              # print the returned suite definition
except RuntimeError as e:
    print("Failed:", e)
```

运行结果

```
$python get_def.py 
# 4.8.0
suite test
  edit ECF_HOME '/g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course'
  task t1
endsuite
```

可以使用 `ecflow_client --get_state` 获取 suite definition 并显示状态，见本节最开始的示例。

`ecflow_client --migrage` 命令将状态显示为注释，该命令的输出格式用在 check point 文件中。

```
$ecflow_client --host=login05 --port=33083 --migrate
# 4.8.0
defs_state MIGRATE state>:complete flag:message state_change:84 modify_change:15
  edit ECF_MICRO '%' # server
  edit ECF_HOME '/g3/wangdp/ecf_home' # server
  edit ECF_JOB_CMD '%ECF_JOB% 1> %ECF_JOBOUT% 2>&1' # server
  edit ECF_KILL_CMD 'kill -15 %ECF_RID%' # server
  edit ECF_STATUS_CMD 'ps --sid %ECF_RID% -f' # server
  edit ECF_URL_CMD '${BROWSER:=firefox} -remote 'openURL(%ECF_URL_BASE%/%ECF_URL%)'' # server
  edit ECF_URL_BASE 'https://software.ecmwf.int' # server
  edit ECF_URL 'wiki/display/ECFLOW/Home' # server
  edit ECF_LOG '/g3/wangdp/ecf_home/a303r6n1.33083.ecf.log' # server
  edit ECF_INTERVAL '60' # server
  edit ECF_LISTS '/g3/wangdp/ecf_home/ecf.lists' # server
  edit ECF_CHECK '/g3/wangdp/ecf_home/a303r6n1.33083.check' # server
  edit ECF_CHECKOLD '/g3/wangdp/ecf_home/a303r6n1.33083.check.b' # server
  edit ECF_CHECKINTERVAL '120' # server
  edit ECF_CHECKMODE 'CHECK_ON_TIME' # server
  edit ECF_TRIES '2' # server
  edit ECF_VERSION '4.7.1' # server
  edit ECF_PORT '33083' # server
  edit ECF_NODE '%ECF_HOST%' # server
  edit ECF_HOST 'a303r6n1' # server
  edit ECF_PID '127321' # server
suite test #  begun:1 state:complete
  edit ECF_HOME '/g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course'
  calendar initTime:2018-Jan-30 14:23:09 suiteTime:2018-Jan-31 02:01:00 duration:11:37:51 initLocalTime:2018-Jan-30 14:23:09 lastTime:2018-Jan-31 02:01:00 calendarIncrement:00:01:00
  task t1 # try:1 state:complete
endsuite
```

Python 接口也提供类似的功能：

```py
import ecflow

try:
    ci = ecflow.Client('login05', 33083)
    ci.sync_local()                                   # get server definition, by sync with client defs
    
    ecflow.PrintStyle.set_style( ecflow.Style.STATE )  # set printing to show structure
    print(ci.get_defs())                              # print the returned suite definition
    
    ecflow.PrintStyle.set_style(ecflow.Style.MIGRATE)  # set printing to show structure and state, and node history
    print(ci.get_defs())
    
except RuntimeError as e:
    print("Failed:", e)
```

运行脚本：

```
$python get_def_state.py 
# 4.8.0
defs_state STATE state>:complete flag:message state_change:84 modify_change:15
  edit ECF_MICRO '%' # server
  edit ECF_HOME '/g3/wangdp/ecf_home' # server
  edit ECF_JOB_CMD '%ECF_JOB% 1> %ECF_JOBOUT% 2>&1' # server
  edit ECF_KILL_CMD 'kill -15 %ECF_RID%' # server
  edit ECF_STATUS_CMD 'ps --sid %ECF_RID% -f' # server
  edit ECF_URL_CMD '${BROWSER:=firefox} -remote 'openURL(%ECF_URL_BASE%/%ECF_URL%)'' # server
  edit ECF_URL_BASE 'https://software.ecmwf.int' # server
  edit ECF_URL 'wiki/display/ECFLOW/Home' # server
  edit ECF_LOG '/g3/wangdp/ecf_home/a303r6n1.33083.ecf.log' # server
  edit ECF_INTERVAL '60' # server
  edit ECF_LISTS '/g3/wangdp/ecf_home/ecf.lists' # server
  edit ECF_CHECK '/g3/wangdp/ecf_home/a303r6n1.33083.check' # server
  edit ECF_CHECKOLD '/g3/wangdp/ecf_home/a303r6n1.33083.check.b' # server
  edit ECF_CHECKINTERVAL '120' # server
  edit ECF_CHECKMODE 'CHECK_ON_TIME' # server
  edit ECF_TRIES '2' # server
  edit ECF_VERSION '4.7.1' # server
  edit ECF_PORT '33083' # server
  edit ECF_NODE '%ECF_HOST%' # server
  edit ECF_HOST 'a303r6n1' # server
  edit ECF_PID '127321' # server
# server state: RUNNING
suite test #  begun:1 state:complete
  edit ECF_HOME '/g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course'
  # edit SUITE 'test'
  # edit ECF_DATE '20180130'
  # edit YYYY '2018'
  # edit DOW '2'
  # edit DOY '30'
  # edit DATE '30.01.2018'
  # edit DAY 'tuesday'
  # edit DD '30'
  # edit MM '01'
  # edit MONTH 'january'
  # edit ECF_CLOCK 'tuesday:january:2:30'
  # edit ECF_TIME '14:23'
  # edit ECF_JULIAN '2458149'
  # edit TIME '1423'
  calendar initTime:2018-Jan-30 14:23:09 suiteTime:2018-Jan-30 14:23:09 duration:00:00:00 initLocalTime:2018-Jan-30 14:23:09 lastTime:2018-Jan-30 14:23:09 calendarIncrement:00:01:00
  task t1 # try:1 state:complete
    # edit TASK 't1'
    # edit ECF_JOB '/g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course/test/t1.job1'
    # edit ECF_SCRIPT '/g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course/test/t1.ecf'
    # edit ECF_JOBOUT '/g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course/test/t1.1'
    # edit ECF_TRYNO '1'
    # edit ECF_RID ''
    # edit ECF_NAME '/test/t1'
    # edit ECF_PASS ''
endsuite

# 4.8.0
defs_state MIGRATE state>:complete flag:message state_change:84 modify_change:15
  edit ECF_MICRO '%' # server
  edit ECF_HOME '/g3/wangdp/ecf_home' # server
  edit ECF_JOB_CMD '%ECF_JOB% 1> %ECF_JOBOUT% 2>&1' # server
  edit ECF_KILL_CMD 'kill -15 %ECF_RID%' # server
  edit ECF_STATUS_CMD 'ps --sid %ECF_RID% -f' # server
  edit ECF_URL_CMD '${BROWSER:=firefox} -remote 'openURL(%ECF_URL_BASE%/%ECF_URL%)'' # server
  edit ECF_URL_BASE 'https://software.ecmwf.int' # server
  edit ECF_URL 'wiki/display/ECFLOW/Home' # server
  edit ECF_LOG '/g3/wangdp/ecf_home/a303r6n1.33083.ecf.log' # server
  edit ECF_INTERVAL '60' # server
  edit ECF_LISTS '/g3/wangdp/ecf_home/ecf.lists' # server
  edit ECF_CHECK '/g3/wangdp/ecf_home/a303r6n1.33083.check' # server
  edit ECF_CHECKOLD '/g3/wangdp/ecf_home/a303r6n1.33083.check.b' # server
  edit ECF_CHECKINTERVAL '120' # server
  edit ECF_CHECKMODE 'CHECK_ON_TIME' # server
  edit ECF_TRIES '2' # server
  edit ECF_VERSION '4.7.1' # server
  edit ECF_PORT '33083' # server
  edit ECF_NODE '%ECF_HOST%' # server
  edit ECF_HOST 'a303r6n1' # server
  edit ECF_PID '127321' # server
suite test #  begun:1 state:complete
  edit ECF_HOME '/g3/wangdp/project/study/ecflow/ecflow-tutorial-code/build/course'
  calendar initTime:2018-Jan-30 14:23:09 suiteTime:2018-Jan-30 14:23:09 duration:00:00:00 initLocalTime:2018-Jan-30 14:23:09 lastTime:2018-Jan-30 14:23:09 calendarIncrement:00:01:00
  task t1 # try:1 state:complete
endsuite
```

用 python 列出所有的节点和状态，请参看《[How can I access the path and task states ?](https://software.ecmwf.int/wiki/display/ECFLOW/How+can+I+access+the+path+and+task+states#print-all-states) 》

```py
import ecflow

try:
    # Create the client
    ci = ecflow.Client("login05", "33083")

    # Get the node tree suite definition as stored in the server
    # The definition is retrieved and stored on the variable 'ci'
    ci.sync_local()

    # access the definition retrieved from the server
    defs = ci.get_defs()

    if defs is None:
        print("The server has no definition")
        exit(1)

    # get the tasks, *alternatively* could use defs.get_all_nodes()
    # to include suites, families and tasks.
    task_vec = defs.get_all_tasks()

    # iterate over tasks and print path and state
    for task in task_vec:
        print(task.get_abs_node_path() + " " + str(task.get_state()))

except RuntimeError as e:
    print("Failed: ", str(e))
```

执行脚本：

```
$python get_tasks.py 
/test/t1 complete
```

将 `get_all_tasks` 替换为 `get_all_nodes` 可以获取所有节点的信息。

```
$python get_nodes.py
/test complete
/test/t1 complete
```

## 任务

1. 找到 job file 和输出文件
2. 查看从服务器获取的 suite definition。
