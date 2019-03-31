---
title: 使用 python 脚本
weight: 10
---

前面已经看到，ecFlow 有 [ecFlow Python Api](https://software.ecmwf.int/wiki/display/ECFLOW/ecFlow+Python+Api#python-api)：

```py
import ecflow
```

允许我们使用 python 构建 suite definition，也可以使用 python 与 `ecflow_server` 通讯。
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
    suite = Suite(name)
    for i in range(1, 7) :
        fam = suite.add_family("f" + str(i))
        for t in ( "a", "b", "c", "d", "e" ) :
            fam.add_task(t)
    return suite
```

或者使用更新的构造函数方法：

```py
def create_suite(name) :
    return Suite(name,
            [ Family("f{0}".format(i),
                [ Task(t) for t in ( "a", "b", "c", "d", "e") ])
              for i in range(1,7) ])
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

## 添加节点属性

有多种添加节点属性的方式，属性包括：

- Repeat
- Time
- Today
- Date
- Day
- Cron
- Clock
- DefStatus
- Meter
- Event
- Variable
- Label
- Trigger
- Complete
- Limit
- InLimit
- Zombie
- Late

### 函数方式

使用添加特定属性的函数：

```py
# Functional style
node.add_variable(home,'COURSE')       # c++ style
node.add_limit('limitX',10)            # c++ style
```
 
使用 `add` 函数添加任意属性：

```py
# Using <node>.add(<attributes>)
node.add(Edit(home=COURSE),                       # Notice that add() allows you adjust the indentation
         Limit('limitX',10))                      # node.add(<attributes>) 
```

### 原地添加

创建节点时，将属性作为额外的参数（**推荐方式**）。支持类似文本方式的缩进。

```py
# in place. When creating a Node, attributes are additional arguments (preferred)
# This also allows indentation.
#   Task(name,<attributes>)
#   Family(name,Node | <attributes>)
#   Suite(name,Node  | <attributes>)
 node = Family('t1',                              
           Edit(home=COURSE),                  
           Limit('limitX',10),
           Task('t1,
              Event('e')))
```

### 表达式

```py
# Using <node> += <attribute>     adding a single attribute                       
node += Edit(home=COURSE)                           
 
# Using <node> += [ <attributes> ]  - use list to add multiple attributes
node += [ Edit(home=COURSE), Limit('limitX',10), Event(1) ]    
 
# Using node + <attributes>  - A node container(suite | family) must appear on the left hand side. Use brackets to control scope.
node + Edit(home=COURSE) + Limit('limitX',10)  
 
# In this example, variable 'name' is added to suite 's/' and not task 't3'    
suite = Suite("s") + Family("f") + Family("f2") + Task("t3") + Edit(name="value")
```

生成的 def 文件如下：

```bash
suite s
  edit name 'value'
  family f
  endfamily
  family f2
  endfamily
  task t3
endsuite
```

可以使用括号控制变量添加到哪个节点。

```py
# here we use parenthesis to control where the variable get added.
suite = Suite("s") + Family("f") + Family("f2") + (Task("t3") + Edit(name="value"))
```

生成的 def 文件如下：

```py
 suite s
  family f
  endfamily
  family f2
  endfamily
  task t3
    edit name 'value'
endsuite
```
