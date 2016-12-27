# 检查结果

查看我们的 suite 的运行状态，输入：

```bash
windroc@ubuntu:~/course$ ecflow_client --get_stat
# 4.0.9
defs_state STATE state>:complete flag:message state_change:72 modify_change:19
suite test #  begun:1 state:complete
  edit ECF_HOME '/home/windroc/course'
  calendar initTime:2016-Feb-02 07:26:50 suiteTime:2016-Feb-02 07:27:00 duration:00:00:10 initLocalTime:2016-Feb-02 07:26:50 lastTime:2016-Feb-02 07:27:00 calendarIncrement:00:00:10
  task t1 # try:1 state:complete
endsuite
```

上述命令会从服务其中检索 suite definition ，并显示每个节点的状态。

查看 task t1，如果t1是 [complete](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-complete) 状态，
并且 suite 是 complete 状态，那么运行成功。如果不是这种情况，则可能会有 [aborted](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-aborted) 状态。

请检查 ecf script 的目录。服务器在ecf script相同的目录下创建 job file，名为 t1.job1。
比较 t1.ecf，head.h，tail.h 和 t1.job1。作业的输出文件也放在ecf script 的目录下，名为 t1.1。

## t1.ecf

```bash
%include "../head.h"
echo "I am part of a suite that lives in %ECF_HOME%"
%include "../tail.h"
```

## head.h

```bash
#!/bin/ksh
set -e # stop the shell on first error
set -u # fail when using an undefined variable
set -x # echo script lines as they are executed


# Defines the variables that are needed for any communication with ECF
export ECF_PORT=%ECF_PORT%    # The server port number
export ECF_NODE=%ECF_NODE%    # The name of ecf host that issued this task
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
set -e # stop the shell on first error
set -u # fail when using an undefined variable
set -x # echo script lines as they are executed


# Defines the variables that are needed for any communication with ECF
export ECF_PORT=2500    # The server port number
export ECF_NODE=ubuntu    # The name of ecf host that issued this task
export ECF_NAME=/test/t1    # The name of this current task
export ECF_PASS=47v3ialj    # A unique password
export ECF_TRYNO=1  # Current try number of the task
export ECF_RID=$$             # record the process id. Also used for zombie detection

# Define the path where to find ecflow_client
# make sure client and server use the *same* version.
# Important when there are multiple versions of ecFlow
export PATH=/usr/local/apps/ecflow/4.0.9/bin:$PATH

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

echo "I am part of a suite that lives in /home/windroc/course"
wait                      # wait for background process to stop
ecflow_client --complete  # Notify ecFlow of a normal end
trap 0                    # Remove all traps
exit 0                    # End the shell
```

t1.1

```bash
+ ECF_PORT=2500
+ export ECF_PORT
+ ECF_NODE=ubuntu
+ export ECF_NODE
+ ECF_NAME=/test/t1
+ export ECF_NAME
+ ECF_PASS=47v3ialj
+ export ECF_PASS
+ ECF_TRYNO=1
+ export ECF_TRYNO
+ ECF_RID=63251
+ export ECF_RID
+ PATH=/usr/local/apps/ecflow/4.0.9/bin:/home/windroc/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
+ export PATH
+ ecflow_client '--init=63251'
+ trap ERROR 0
+ trap '{ echo "Killed by a signal"; ERROR ; }' 1 2 3 4 5 6 7 8 10 12 13 15
+ echo 'I am part of a suite that lives in /home/windroc/course'
I am part of a suite that lives in /home/windroc/course
+ wait
+ ecflow_client --complete
+ trap 0
+ exit 0
```

## 获取 suite definition

获取可以解析的 suite definition 定义

```bash
windroc@ubuntu:~/course$ ecflow_client --get
# 4.0.9
suite test
  edit ECF_HOME '/home/windroc/course'
  task t1
endsuite
```

可以通过 Python API，编写如下文件 get_suite.py

```python
#!/usr/bin/env python2.7
import ecflow
try:
    ci = ecflow.Client()
    ci.sync_local()                                   # get server definition, by sync with client defs
    ecflow.PrintStyle.set_style( ecflow.Style.DEFS )  # set printing to show structure
    print ci.get_defs()                               # print the returned suite definition
except RuntimeError as e:
    print "Failed:",   e
```

运行结果

```bash
windroc@ubuntu:~/course$ python get_suite.py 
# 4.0.9
suite test
  edit ECF_HOME '/home/windroc/course'
  task t1
endsuite
```

获取 suite definition 并显示状态：

命令行：

```bash
windroc@ubuntu:~/course$ ecflow_client --get_state
# 4.0.9
defs_state STATE state>:complete flag:message state_change:72 modify_change:19
suite test #  begun:1 state:complete
  edit ECF_HOME '/home/windroc/course'
  calendar initTime:2016-Feb-02 07:26:50 suiteTime:2016-Feb-02 08:09:00 duration:00:42:10 initLocalTime:2016-Feb-02 07:26:50 lastTime:2016-Feb-02 08:09:00 calendarIncrement:00:01:00
  task t1 # try:1 state:complete
endsuite
```

python：

```python
#!/usr/bin/env python2.7
import ecflow
try:
    ci = ecflow.Client()
    ci.sync_local()                                   # retrieve server definition, by sync with client defs
    ecflow.PrintStyle.set_style( ecflow.Style.STATE ) # set printing to show structure and state
    print ci.get_defs()                               # print the returned suite definition
except RuntimeError as e:
    print "Failed:",  e
```

显示结果

```bash
windroc@ubuntu:~/course$ python get_suite.py 
# 4.0.9
defs_state STATE state>:complete flag:message state_change:72 modify_change:19
suite test #  begun:1 state:complete
  edit ECF_HOME '/home/windroc/course'
  calendar initTime:2016-Feb-02 07:26:50 suiteTime:2016-Feb-02 07:26:50 duration:00:00:00 initLocalTime:2016-Feb-02 07:26:50 lastTime:2016-Feb-02 07:26:50 calendarIncrement:00:01:00
  task t1 #
```

用python列出所有的节点和状态，请参看《[How can I access the path and task states ?](https://software.ecmwf.int/wiki/display/ECFLOW/How+can+I+access+the+path+and+task+states#print-all-states) 》

```python
#!/usr/bin/env python2.7
import ecflow

try:
    # Create the client
    ci = ecflow.Client("localhost", "4143")
    
    # Get the node tree suite definition as stored in the server
    # The definition is retrieved and stored on the variable 'ci'
    ci.sync_local()

    # access the definition retrieved from the server
    defs = ci.get_defs()
    
    if defs == None :
        print "The server has no definition"
        exit(1)
    
    # get the tasks, *alternatively* could use defs.get_all_nodes() 
    # to include suites, families and tasks.
    task_vec = defs.get_all_tasks()
 
    # iterate over tasks and print path and state
    for task in task_vec:
        print task.get_abs_node_path()  + " "  + str(task.get_state())
        
except RuntimeError, e:
    print "Failed: " + str(e)
```

## 任务

1. 找到 job file 和输出文件
2. 查看从服务器获取的 suite definition。

