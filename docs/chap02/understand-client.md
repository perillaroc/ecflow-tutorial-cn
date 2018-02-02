# 理解客户端

所有与 `ecflow_server` 的通讯都需要通过 `ecflow_client`。任何与服务器的通讯都需要知道服务器的地址和端口。同一台主机中可能运行多个服务，每个服务都有唯一的端口号。

本教程将会给出通过 shell 和在Python 脚本中使用客户端的例子。

## 客户端命令行接口

客户端命令的列表：

```
$ecflow_client --help

Client/server based work flow package:

Ecflow version(4.8.0) boost(1.53.0) compiler(gcc 4.8.5) protocol(TEXT_ARCHIVE) Compiled on Jan 18 2018 13:37:13

ecflow_client provides the command line interface, for interacting with the server:
Try:

   ecflow_client --help=all       # List all commands, verbosely
   ecflow_client --help=summary   # One line summary of all commands
   ecflow_client --help=child     # One line summary of child commands
   ecflow_client --help=user      # One line summary of user command
   ecflow_client --help=<cmd>     # Detailed help on each command

Commands:

   abort                alter                begin                ch_add               ch_auto_add
   ch_drop              ch_drop_user         ch_register          ch_rem               ch_suites
   check                checkJobGenOnly      check_pt             complete             debug
   debug_server_off     debug_server_on      delete               edit_history         edit_script
   event                file                 force                force-dep-eval       free-dep
   get                  get_state            group                halt                 help
   host                 init                 job_gen              kill                 label
   load                 log                  meter                migrate              msg
   news                 order                ping                 plug                 port
   reloadwsfile         replace              requeue              restart              restore_from_checkpt
   resume               rid                  run                  server_load          server_version
   show                 shutdown             stats                stats_reset          status
   suites               suspend              sync                 sync_full            terminate
   version              wait                 why                  zombie_adopt         zombie_block
   zombie_fail          zombie_fob           zombie_get           zombie_kill          zombie_remove
```

上面提到过，使用 `ecflow_client` 与服务器通讯需要设置 `host` 和 `port`，如下方法确定：

默认主机和端口为：`localhost:3141`

默认值被 `ECF_NODE` 和 `ECF_PORT` 环境变量覆盖。

环境变量被命令行参数 `-–port` 和 `-–host` 覆盖，并且可以用于 `--help`参数显示的任意 shell 层命令。

例如在命令行 ping 一个服务可以输入命令：

```bash
ecflow_client --ping --host=machineX --port=4141
```

运行结果如下：

```
$ecflow_client --ping --host=login05 --port=33083
ping server(login05:33083) succeeded in 00:00:00.023347  ~23 milliseconds
```

## 客户端 Python 接口

`ecflow_client` 提供的功能可以通过 [Client Server API](https://software.ecmwf.int/wiki/display/ECFLOW/ecFlow+Python+Api#client-server-python-api) 实现。
Python 接口使用相同的算法确定 `host` 和 `port`，允许显式设定 `host` 和 `port`。参看 [ecflow.Client](https://software.ecmwf.int/wiki/display/ECFLOW/ecFlow+Python+Api#ecflow.Client)

下面给出 python 的示例

```py
import ecflow

try:
    # When no arguments specified uses ECF_HOST and/or ECF_PORT,
    # otherwise defaults to localhost:3141
    ci = ecflow.Client()  # inherit from shell variables
    ci.ping()

except RuntimeError as e:
    print("ping failed: " + str(e))

try:
    # Explicitly set host and port using the same client
    # For alternative argument list see ecflow.Client.set_host_port()
    ci.set_host_port("login05:33083")  # actually set the host and port (change to your host and port)
    ci.ping()

except RuntimeError as e:
    print("ping failed: " + str(e))

try:
    # Create a new client, Explicitly setting host and port.
    # For alternative argument list see ecflow.Client
    ci = ecflow.Client("localhost:1000")  # another server
    ci.ping()
except RuntimeError as e:
    print("ping failed: " + str(e))
```

上面只有第二次 ping 成功，执行结果如下所示：

```
$python3 use_client.py
ping failed: [13:31:17 30.1.2018] Request( --ping :wangdp ), Failed to connect to localhost:3141. After 2 attempts. Is the server running ?

ping failed: [13:31:18 30.1.2018] Request( --ping :wangdp ), Failed to connect to localhost:1000. After 2 attempts. Is the server running ?
```

## 需要做什么

如果你的 `ecflow_server` 是用 `ecflow_start.sh` 启动的，并且需要用到 shell 接口，那么请设置 `ECF_NODE` 和 `ECF_PORT` 环境变量。
请注意，如果服务器通过 `ecflow_start.sh` 脚本 启动，那么默认的服务地址 `localhost:3141`就不正确。

如果服务运行在本地主机上，可以使用 `netstat` 确定端口号。

```
netstat -lnptu
```

可以通过该命令检查 ecflow 服务的端口是否正在正常。

```text
$netstat -lnptu
...
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
...
tcp        0      0 0.0.0.0:33083           0.0.0.0:*               LISTEN      127321/ecflow_serve
...
```
