# 使用 python 脚本

以前已经看到，ecFlow 有 [ecFlow Python Api](https://software.ecmwf.int/wiki/display/ECFLOW/ecFlow+Python+Api#python-api)：

```python
#!/usr/bin/env python2.7
import ecflow
```

允许我们使用 python 构建 suite definition，也可以使用 python 与 ecflow_server 通讯。
这是个强大的功能，可以帮助我们以相对简化的方式定义复杂的 suite。

考虑下面的 suite：

```bash
suite test
 family f1
     task a
     task b
     task c
     task d
     task e
 endfamily
 family f2
     task a
     task b
     task c
     task d
     task e
 endfamily
 family f3
     task a
     task b
     task c
     task d
     task e
 endfamily
 family f4
     task a
     task b
     task c
     task d
     task e
 endfamily
 family f5
     task a
     task b
     task c
     task d
     task e
 endfamily
 family f6
     task a
     task b
     task c
     task d
     task e
 endfamily
endsuite
```

用 python 可以写成：

```python
def create_suite(name) : 
    suite = ecflow.Suite(name)
    for i in range(1, 7) :
        fam = suite.add_family("f" + str(i))
        for t in ( "a", "b", "c", "d", "e" ) :
            fam.add_task(t)
    return suite
```

Python 变量可以用于生成 trigger dependencies。

想象我们将 family f1 到 family f6 串联到一起，f2 在 f1 后运行，f3 接着 f2 运行，以此类推。下面实现这个功能。

```python
def create_sequential_suite(name) :
    suite = ecflow.Suite(name)
    for i in range(1, 7) :
        fam = suite.add_family("f" + str(i))
        if i != 1: 
            fam.add_trigger("f" + str(i-1) + " == complete")  # or fam.add_family( "f%d == complete" % (i-1) )
        for t in ( "a", "b", "c", "d", "e" ) :
            fam.add_task(t) 
    return suite
```

接下来的 python 代码显示向节点树添加多种属性的例子。请查看用户手册获取更详细的解释。

```python
# provides *examples* of add adding node attributes using the python API
# hence does *not* represent a real suite definition
from ecflow import *
    
if __name__ == "__main__":

    # 添加变量
    suite = Suite("s1");
    suite.add_variable(Variable("ECF_HOME", "/tmp/"))
    suite.add_variable("ECF_URL_CMD", "${BROWSER:=firefox} -remote 'openURL(%ECF_URL_BASE%/%ECF_URL%)'")
    suite.add_variable("NAME", 10)
    
    a_dict = { "name":"value", "name2":"value2", "name3":"value3", "name4":"value4" }
    suite.add_variable(a_dict)

     
    # 添加 limit
    suite.add_limit(  Limit("limitName1", 10) )
    suite.add_limit(  "limitName3", 10 )
  
    # 添加 inlimits
    suite.add_inlimit( InLimit("limitName1", "/s1/f1", 2) )
    suite.add_inlimit( "limitName3", "/s1/f1", 2)
 
         # 添加短的 trigger 和 complete
    task = Task("task")
    task.add_trigger( "t2 == active" )
    task.add_complete( "t2 == complete" )
      
         # 添加长的 trigger 和 complete。下面的例子中，True 表示 AND，False 表示 OR。
    task = Task("trigger")
    task.add_part_trigger(  "t1 == complete" )
    task.add_part_trigger(  "t2 == active", True )  # 对于长的 and/or 表达式,子序列表达式必须为 and/or
    task.add_part_complete( "t3 == complete" )
    task.add_part_complete( "t4 == active", False)  # 对于长的 and/or 表达式,子序列表达式必须为 and/or
    
    # 添加 events
    task.add_event( Event(1) )
    task.add_event( 2 )
    task.add_event( Event(10, "Eventname") )
    task.add_event( 10, "Eventname2" )
    task.add_event( "fred" )

    # 添加 meter
    task.add_meter( Meter("metername1", 0, 100, 50) )
    task.add_meter( "metername3", 0, 100 )
  
    # 添加 label
    task.add_label( Label("label_name1", "value") )
    task.add_label( "label_name3", "value" )
 
    # 添加 Repeat.单个及诶单只能有一个 repeat，因此我们在添加另一个 repeat 之前删掉前面的 repeat。
    task.add_repeat( RepeatInteger("integer", 0, 100, 2) )  
    task.delete_repeat()     
    task.add_repeat( RepeatEnumerated("enum", ["red", "green", "blue" ] ) )
    task.delete_repeat()     
    task.add_repeat( RepeatDate("date", 20100111, 20100115, 2) )
    task.delete_repeat()         
    task.add_repeat( RepeatString("string", ["a", "b", "c" ] ) )
    task.delete_repeat()         
    
    # 创建时间序列，用于添加 time、today 和 cron
    start = TimeSlot(0, 0)
    finish = TimeSlot(23, 0)
    incr = TimeSlot(0, 30)
    time_series = TimeSeries( start, finish, incr, True) # True means relative to suite start
  
    # 添加 today
    task.add_today( "00:30" )
    task.add_today( "+00:30" )
    task.add_today( "+00:30 20:00 01:00" )
    task.add_today( Today( time_series) )
    task.add_today( Today( 0, 10 ))
    task.add_today( 0, 59, True )
    task.add_today( Today(TimeSlot(20, 10)) )
    task.add_today( Today(TimeSlot(20, 20), False)) 
                    
    # 添加 time
    task.add_time( "00:30" )
    task.add_time( "+00:30" )
    task.add_time( "+00:30 20:00 01:00" )
    task.add_time( Time(time_series ))
    task.add_time( Time( 0, 10 ))
    task.add_time( 0, 59, True)
    task.add_time( Time(TimeSlot(20, 10)) )
    task.add_time( Time(TimeSlot(20, 20), False)) 
 
    # 添加 date
    for i in [ 1, 2, 4, 8, 16 ] :
        task.add_date( i, 0, 0)    # day,month,year, where corresponding 0 means any possible day,month, year
    task.add_date( Date(1, 1, 2010))

    # 添加 day
    task.add_day( Day(Days.sunday))
    task.add_day( Days.monday)
    task.add_day( "tuesday")
    
    # 创建 cron, 以不同的方式添加时间，并设置 cron 属性。
    cron = Cron()
    cron.set_week_days( [0, 1, 2, 3, 4, 5, 6]  )
    cron.set_days_of_month( [1, 2, 3, 4, 5, 6] )
    cron.set_months( [1, 2, 3, 4, 5, 6] )
    start = TimeSlot(0, 0)
    finish = TimeSlot(23, 0)
    incr = TimeSlot(0, 30)
    ts = TimeSeries( start, finish, incr, True)  # True means relative to suite start
    cron.set_time_series( ts )

    cron1 = Cron()
    cron1.set_week_days( [0, 1, 2, 3, 4, 5, 6] )
    cron1.set_time_series( 1, 30,  True )
    
    cron2 = Cron()
    cron2.set_week_days(  [0, 1, 2, 3, 4, 5, 6] )
    cron2.set_time_series( "00:30 01:30 00:01" )
    
    cron3 = Cron()
    cron3.set_week_days( [0, 1, 2, 3, 4, 5, 6] )
    cron3.set_time_series( "+00:30" )
    
    # 添加 auto cancel
    t1 = Task("t1");
    t3 = Task("t3")
    t4 = Task("t4")
    t5 = Task("t5")
    t1.add_autocancel( 3 )                       # 3 days
    t3.add_autocancel( 20, 10, True )            # hour,minutes,relative
    t4.add_autocancel( TimeSlot(10, 10), True )  # hour,minutes,relative
    t5.add_autocancel( Autocancel(1, 10, True) ) # hour,minutes,relative
    
    # 添加 late
    late = Late()
    late.submitted( TimeSlot(20, 10))
    late.active(    TimeSlot(20, 10))
    late.complete(  TimeSlot(20, 10), True)
    task.add_late(  late )
    
    late = Late()
    late.submitted( 20, 10 )
    late.active(    20, 10 )
    late.complete(  20, 10, True)
    t1.add_late( late )
    
    # 添加 defstatus, 最后的设置生效
    task.add_defstatus( DState.complete )
    task.add_defstatus( DState.queued )
    task.add_defstatus( DState.aborted )
    task.add_defstatus( DState.submitted )
    task.add_defstatus( DState.suspended )
    task.add_defstatus( DState.active )
    
    # 添加 clock
    clock = Clock(1, 1, 2010, False)    # day,month, year, hybrid(true), real(False)
    clock.set_gain(1, 10, True)         # True means positive gain
    suite =  Suite("suite")
    suite.add_clock(clock)
    
    clock = Clock(False)                # Use the current time and date with real time clock
    clock.set_gain_in_seconds(12, True)
    s1 = Suite("s1")
    s1.add_clock(clock)
```
