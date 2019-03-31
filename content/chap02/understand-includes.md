---
title: 理解头文件
weight: 2
---

前面的章节我们创建了一个 task。

每个 task 都有对应的 [ecf script](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-ecf-script)，定义需要执行那些操作。脚本类似于 UNIX shell 脚本。

但 ecf script 提供于 C 语言类似的预处理指令
（[pre-processing](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-pre-processing) 
[directive](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-directives)）和预定义变量。

suite definition 中定义的变量可以在 ecf script 中使用，提供一种配置机制。

默认人使用字符 `%` 表示预处理指令，其中一种就是 include 头文件。

头文件用于向脚本中注入代码（与 C 语言的 include 头文件一样），提供一种代码复用的机制。
如果相同的代码出现在不同的 ecf script 文件中，这些代码就应该放入一个头文件中。这样会提供一个单一的维护点。
例如，每个 task 都需要建立与 `ecflow_server` 的通讯，并告诉服务器任务已经开始。
这个样例代码就放在头文件中。

## head.h

放在 ecf script 开头，完成如下任务：

- 准备与 `ecflow_server` 通信的环境。
- 定义脚本错误处理函数。当错误发生，通知服务器该任务 [aborted](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-aborted)。
- 使用 child command 通知服务器作业已经开始。

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

放在 ecf script 文件结尾，通知服务器任务已经完成，使用 child command 中的 `complete` 命令。

```bash
wait                      # wait for background process to stop
ecflow_client --complete  # Notify ecFlow of a normal end
trap 0                    # Remove all traps
exit 0                    # End the shell
```

> 译者注：ecf script 也支持其它编程语言，保留头文件和预定义变量功能，不过需要在 def 文件中设置特殊的变量。<br/>
我在一篇博文中提到如何使用 python 编写 SMS 脚本，ecflow 也类似，后续博文会专门提到如何使用 python 编写 ecf script。

这两个头文件主要用于与 `ecflow_server` 通讯，建立通讯环境，在任务开始时通知服务器任务已经开始，
在任务结束时通知服务器任务已经完成，在任务出错时通知服务器任务发生错误。
可以使用其它方式实现这些基本通讯，比如使用单一的 python 脚本作为头文件。

## 任务

在 `$ECF_HOME` 目录中创建 `head.h` 和 `tail.h` 文件。
