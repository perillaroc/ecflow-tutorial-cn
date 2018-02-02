# 缩进

文本方式支持首行缩进。但 Python 中缩进也是语法结构，会影响程序的含义。

然而，从之前的例子中我们已经看到，可以使用节点构造函数来提供缩进，用以显示定义结构。

```py
import os
from ecflow import Defs,Suite,Family,Task,Edit,Trigger,Complete,Event,Meter,Time,Day,Date
 
print("Creating suite definition")
home = os.path.join(os.getenv("HOME"), "course")
defs = Defs(
        Suite("test",
            Edit(ECF_INCLUDE=home,ECF_HOME=home),
            Family("f1",
                Edit(SLEEP=20),
                Task("t1", Meter("progress", 1, 100, 90)),
                Task("t2", Trigger("t1 == complete"),Event("a"),Event("b")),
                Task("t3", Trigger("t2:a")),
                Task("t4", Trigger("t2 == complete"), Complete("t2:b")),
                Task("t5", Trigger("t1:progress ge 30")),
                Task("t6", Trigger("t1:progress ge 60")),
                Task("t7", Trigger("t1:progress ge 90"))),
            Family("f2",
                Edit(SLEEP=20),
                Task("t1", Time( "00:30 23:30 00:30" )),
                Task("t2", Day( "sunday" )),
                Task("t3", Date("1.*.*"), Time("12:00")),
                Task("t4", Time("+00:02")),
                Task("t5", Time("00:02")))))
print(defs)
 
print("Checking job creation: .ecf -> .job0") 
print(defs.check_job_creation())
 
print("Checking trigger expressions")
assert len(defs.check()) == 0, defs.check()
 
print("Saving definition to file 'test.def'")
defs.save_as_defs("test.def")
```

我们还可以利用 Python 的 `with` 语句实现缩进。

下面将前一个例子用 with 语句改写。

```python
#!/usr/bin/env python2.7
import os
import ecflow
import sys

version = sys.version_info 
if version[1] < 7: 
    print "This example requires python version 2.7, but found : " + str(version)
    exit(0)

print "Creating suite definition"  
with ecflow.Defs() as defs: 
    
    with defs.add_suite("test") as suite:
        
        suite.add_variable("ECF_INCLUDE", os.path.join(os.getenv("HOME"),  "course"))
        suite.add_variable("ECF_HOME",    os.path.join(os.getenv("HOME"),  "course"))
        
        with suite.add_family("f1") as f1:
            f1.add_variable("SLEEP", 20)
            f1.add_task("t1").add_meter("progress", 1, 100, 90)
            f1.add_task("t2").add_trigger("t1 eq complete").add_event("a").add_event("b")
            f1.add_task("t3").add_trigger("t2:a")  
            f1.add_task("t4").add_trigger("t2 eq complete").add_complete("t2:b")  
            f1.add_task("t5").add_trigger("t1:progress ge 30")  
            f1.add_task("t6").add_trigger("t1:progress ge 60")  
            f1.add_task("t7").add_trigger("t1:progress ge 90") 
    
        with suite.add_family("f2") as f2:        
            f2.add_variable("SLEEP", 20)
            f2.add_task("t1").add_time("00:30 23:30 00:30")
            f2.add_task("t2").add_day("sunday")
            f2.add_task("t3").add_date(1, 0, 0).add_time(12, 0)
            f2.add_task("t4").add_time(0, 2, True)
            f2.add_task("t5").add_time(0, 2)
            
    print defs

    print "Checking job creation: .ecf -> .job0"   
    print defs.check_job_creation()

    print "Checking trigger expressions"
    print defs.check()

    print "Saving definition to file 'test.def'"
    defs.save_as_defs("test.def")
```

另一种实现方式：

```python
from ecflow import Defs,Suite,Family,Task,Edit,Trigger,Complete,Event,Meter,Time,Day,Date
import os
import sys
 
version = sys.version_info
if version[1] < 7:
    print "This example requires python version 2.7, but found : " + str(version)
    exit(0)
 
print "Creating suite definition" 
with Defs() as defs:
     
    with defs.add_suite("test") as suite:
         
        suite += Edit(ECF_HOME=os.path.join(os.getenv("HOME"),  "course"))
        suite += Edit(ECF_INCLUDE =os.path.join(os.getenv("HOME"),  "course"))
         
    with suite.add_family("f1") as f1:
        f1 += Edit(SLEEP=20)
        f1 += Task("t1", Meter("progress", 1, 100, 90))
        f1 += Task("t2", Trigger("t1 == complete"), Event("a"), Event("b"))
        f1 += Task("t3", Trigger("t2:a"))
        f1 += Task("t4", Trigger("t2 == complete"), Complete("t2:b"))
        f1 += Task("t5", Trigger("t1:progress ge 30"))
        f1 += Task("t6", Trigger("t1:progress ge 60"))
        f1 += Task("t7", Trigger("t1:progress ge 90"))
     
    with suite.add_family("f2") as f2:       
        f2 += Edit(SLEEP=20)
        f2 += Task("t1", Time("00:30 23:30 00:30"))
        f2 += Task("t2", Day("sunday"))
        f2 += Task("t3", Date(1, 0, 0), Time(12, 0))
        f2 += Task("t4", Time(0, 2, True))
        f2 += Task("t5", Time(0, 2))
             
print(defs)
 
print("Checking job creation: .ecf -> .job0")
print(defs.check_job_creation())
 
print("Checking trigger expressions")
print(defs.check())
 
print("Saving definition to file 'test.def'")
defs.save_as_defs("test.def")
```

> 译者注：因为业务尚未大规模使用 ecFlow，所以还没有标准化的编写方法。个人感觉使用 `with` 语句更具有扩展性。
