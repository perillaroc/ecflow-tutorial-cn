# 文件位置

目前位置，我们看到 ecflow_server 在特定的位置寻找需要的文件。

使用如下变量控制寻找文件的位置：

* ECF_INCLUDE：include 文件
* ECF_FILES：如果 ecf 脚本没在默认位置，则 ecflow_server 在该位置寻找ecf脚本
* ECF_OUT：作业输出文件位置

如果两个 task 使用相同的 ecf script，区别只是简单地位某个变量设置不同的值，那我们不需要维护多个相同文件的拷贝。可以在 suite 中的不同位置用同一个名称使用同一个脚本，将该脚本放在公用目录下，并将变量 ECF_FILES 设为该位置。

许多用户只是用一个目录，并将 ECF_FILES 指向该目录。

如果 task 有不同名字，可以使用 unix 命令 ln -s 创建链接。

总结下：

* 不同作业有相同名字，使用同一个文件
* 不同作业有不同名字，链接到同一个文件

## 作业

想象在我们的示例 suite 中如何使用 ECF_FILES 和 ln 来减少脚本数目。

译者注：测试使用 ECF_FILES 和 ln