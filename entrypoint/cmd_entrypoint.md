# 了解CMD和ENTRYPOINT的相互作用

`CMD` 和`ENTRYPOINT`指令都定义了运行一个容器时要执行的命令。这里有一些规则来定义他们如何写作：

​    1.Dockerfile中最少指定一个`CMD`或`ENTRYPOINT`。

​    2.当把容器当做是一个可执行程序时，应该使用`ENTRYPOINT`。

​    3.`CMD`应该用来定义`ENTRYPOINT`指令的默认参数或者执行一个ad-hoc形式的命令在容器当中。

​    4.`CMD`会被覆盖当运行有显式参数的命令时。

下面表格告诉了我们不同的`ENTRYPOINT`/`CMD`的组合：

|                             | No ENTRYPOINT              | ENTRYPOINT exec_entry p1_entry | ENTRYPOINT [“exec_entry”, “p1_entry”]    |
| --------------------------- | -------------------------- | ------------------------------ | ---------------------------------------- |
| #No CMD                     | error, not allowed         | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry                      |
| #CMD [“exec_cmd”, “p1_cmd”] | exec_cmd p1_cmd            | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry exec_cmd p1_cmd      |
| #CMD [“p1_cmd”, “p2_cmd”]   | p1_cmd p2_cmd              | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry p1_cmd p2_cmd        |
| #CMD exec_cmd p1_cmd        | /bin/sh -c exec_cmd p1_cmd | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry /bin/sh -c exec_cmd p1_cmd |

