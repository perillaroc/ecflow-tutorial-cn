# 启动suite

`ecflow_start.sh` 脚本会自动启动 `ecflow_server`。
手动启动 ecFlow 后，服务器处于 [halted](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-halted) 状态，需要 restart 服务器。
halted 状态的服务器不会调度任务。

## 文本方式

检查服务器的状态，输入下列 unix 命令

```
ecflow_client --stats
```

stats 命令的运行结果：

```
$ecflow_client --host=login05 --port=33083 --stats
Server statistics
   Version                         Ecflow version(4.7.1) boost(1.53.0) compiler(gcc 4.8.5) protocol(TEXT_ARCHIVE) Compiled on Jan 18 2018 01:15:48
   Status                          RUNNING
   Host                            a303r6n1
   Port                            33083
   Up since                        2018-Jan-30 04:31:46
   Job sub' interval               60s
   ECF_HOME                        /g3/wangdp/ecf_home
   ECF_LOG                         /g3/wangdp/ecf_home/a303r6n1.33083.ecf.log
   ECF_CHECK                       /g3/wangdp/ecf_home/a303r6n1.33083.check
   Check pt interval               120s
   Check pt mode                   CHECK_ON_TIME
   Check pt save time alarm        20s
   Number of Suites                1
   Request's per 1,5,15,30,60 min  0.02 0.01 0.03 0.02 0.01

   Restart server                  1
   Ping                            7
   Sync                            9
   News                            20

   Task init                       2
   Task complete                   2

   Load definition                 3
   Begin                           2
   Node delete                     2
   Run                             1
   Suites                          3
   stats cmd                       8
```

如果 `ecflow_server` 处于 halted 状态，需要重新启动服务器

手动进入 `halted` 状态

```
$ecflow_client --host=login05 --port=33083 --halt
Are you sure you want to halt the server ? y
```

![](assert/ecflowui_start_suite_halt.png)

重新启动

```
$ecflow_client --host=login05 --port=33083 --restart
```

一旦 `ecflow_server` 处于 [running](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-running) 状态，就可以启动 suite

```
$ecflow_client --host=login05 --port=33083 --begin=/test
```

检查状态

![](assert/ecflowui_start_suite_begin.png)

## Python方式

重新启动和开始调度 suite 可以通过 Client Server API 完成，修改 client.py 并重新运行。

译者注：如果之前加载过 suite，则注释掉 `ci.load(...)` 行。

```python
import os
from pathlib import Path
import ecflow

home = os.path.abspath(Path(Path(__file__).parent, "../../../build/course"))
try:
    print("Loading definition in 'test.def' into the server")
    ci = ecflow.Client('login05', '33083')
    ci.load(str(Path(home, "test.def")))  # read definition from disk and load into the server

    print("Restarting the server. This starts job scheduling")
    ci.restart_server()

    print("Begin the suite named 'test'")
    ci.begin_suite("test")

except RuntimeError as e:
    print("Failed:", e)
```

在运行上述代码前，需要先手动删除之前加载的 test：

```
$ecflow_client --host=login05 --port=33083 --delete=/test
Are you sure want to delete nodes at paths:
  /test ? y
$python test_client.py
Loading definition in 'test.def' into the server
Restarting the server. This starts job scheduling
Begin the suite named 'test'
```

## 任务

1. 重新启动 ecflow_server
2. 开始调度 suite
