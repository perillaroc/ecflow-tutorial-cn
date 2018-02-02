# 缩进

> 译者注：2017年 ecflow 加入新的 API 接口，使用构造函数方式实现缩进定义，已不需要此章节的方法。</br>
    保留本节，仅做参考。如果后续有更合适的缩进定义方式，会更新到本文中。

文本方式支持首行缩进。但 Python 中缩进也是语法结构，会影响程序的含义。

Python 方式需要使用 with 语句实现缩进。

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

另外一种方式是使用 ecf.py，但没找到这个脚本。

```python
#!/usr/bin/env python2.7
import os
import sys
sys.path.append('/home/ma/emos/def/o/def')
from ecf import *
print "Creating suite definition" 
defs = Defs().add(# Stream like definition
    Suite("test").add(
        Variables({ # a dictionnary to detect duplicated variables
                "ECF_INCLUDE": os.getenv("HOME") + "/course",
                "ECF_HOME":    os.getenv("HOME") + "/course",}),
        Family("f1").add(
            Variable("SLEEP", "20"),
            Task("t1").add(Meter("progress", 1, 100, 90)),
            Task("t2").add(
                Trigger("t1 eq complete"),
                Event("a"),
                Event("b")),
            Task("t3").add(Trigger("t2:a")),
            Task("t4").add(Trigger("t2 eq complete"),
                           Complete("t2:b")),
            Task("t5").add(Trigger("t1:progress ge 30")),
            Task("t6").add(Trigger("t1:progress ge 60")),
            Task("t7").add(Trigger("t1:progress ge 90")),),
        Family("f2").add(
            Variable("SLEEP", "20"),
            Task("t1").add(Time( "00:30 23:30 00:30" )),
            Task("t2").add(Day( "sunday" )),
            Task("t3").add(Date("1.*.*"),
                           Time("12:00")),
            Task("t4").add(Time("+00:02")),
            Task("t5").add(Time("00:02")))))
             
out = file("test.def", "w")
print >>out, defs
```
