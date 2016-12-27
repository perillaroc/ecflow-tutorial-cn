# 添加说明

[manual page](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-manual-page) 使 ecf script 中的文档可以在 ecflowview 中看到。

说明页是指令 %manual 和 %end 之前的所有文本的组合。

## 修改 t2.ec

```bash
%manual
Manual for task t2
Operations: if this task fails, set it to complete and report next working day
Analyst: Check something ?
%end

%include “../head.h”
echo “I am part of a suite that lives in %ECF_HOME%”
%include “../tail.h”

%manual
There can be multiple manual pages in the same file.
When viewed they are simply concatenated.
%end
```

## 任务

1. 修改 $HOME/course/test/t2.ecf 脚本
2. 在 ecflowview 中查看 task t2 的说明页

![](./asset/manual.jpg)