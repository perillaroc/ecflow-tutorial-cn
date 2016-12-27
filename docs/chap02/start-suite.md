# 启动suite

ecflow_start.sh 脚本会自动启动 ecflow_server。
手动启动 ecFlow 后，服务器处于 [halted](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-halted) 状态，需要 restart 服务器。
halted 状态的服务器不会调度任务。

## 文本方式

检查服务器的状态，输入下列 unix 命令

```bash
ecflow_client --stats
```

运行结果

```bash
windroc@ubuntu:~/course$ ecflow_client --stats
Server statistics
   Version                         Ecflow version(4.0.9) boost(1.53.0) compiler(gcc 4.9.2) protocol(TEXT_ARCHIVE) Compiled on Jan 12 2016 05:57:30
   Status                          RUNNING
   Host                            ubuntu
   Port                            2500
   Up since                        2016-Jan-21 03:11:41
   Job sub' interval               60s
   ECF_HOME                        /home/windroc/course
   ECF_LOG                         /home/windroc/course/ubuntu.2500.ecf.log
   ECF_CHECK                       /home/windroc/course/ubuntu.2500.check
   Check pt interval               120s
   Check pt mode                   CHECK_ON_TIME
   Check pt save time alarm        30s
   Number of Suites                1
   Request's per 1,5,15,30,60 min  0.00 0.00 0.00 0.00 0.00

   Restart server                  1
   Ping                            10
   Server version                  8
   Sync                            12
   News                            623

   Load definition                 4
   Node delete                     2
   stats cmd                       1
```

如果 ecflow_server 处于 halted 状态，需要重新启动服务器

手动进入 halted 状态

```bash
windroc@ubuntu:~/course$ ecflow_client --halt
Are you sure you want to halt the server ? y
```

重新启动

```bash
windroc@ubuntu:~/course$ ecflow_client --restart
```

一旦 ecflow_server 处于 [running](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-running) 状态，就可以启动 suite

```bash
windroc@ubuntu:~/course$ ecflow_client --begin=test
```

检查状态

![](./asset/ecflowview-start-suite.jpg)

## Python方式

重新启动和开始调度 suite 可以通过 Client Server API 完成，修改 client.py 并重新运行。

译者注：如果之前加载过 suite，则注释掉 ci.load(..) 行。

```python
#!/usr/bin/env python2.7
import ecflow

try:
    print "Loading definition in 'test.def' into the server"
    ci = ecflow.Client()
    ci.load("test.def")

    print "Restarting the server. This starts job scheduling"
    ci.restart_server()

    print "Begin the suite named 'test'"
    ci.begin_suite("test")

except RuntimeError as e:
    print "Failed:",    e
```

测试：

```bash
windroc@ubuntu:~/course$ ecflow_client --delete=_all_
Are you sure you want to delete all the suites ? y
windroc@ubuntu:~/course$ ecflow_client --halt
Are you sure you want to halt the server ? y
windroc@ubuntu:~/course$ python client.py
Loading definition in 'test.def' into server
Restarting the server. This starts job scheduling
Begin the suite named 'test'
```

## 任务

1. 重新启动 ecflow_server
2. 开始调度 suite
