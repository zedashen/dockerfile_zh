# Shell形式的ENTRYPOINT例子

你可以为`ENTRYPOINT`指定一个字符串，并且他会在`/bin/sh -c`中被执行。这种形式会利用shell去处理环境变量，并且会无视所有`CMD`或者`docker run`中的环境变量。为了确保`ENTRYPOINT`的执行程序能够收到正确的`docker stop`信号，你需要去使用`exec`启动他：

```
FROM ubuntu
ENTRYPOINT exec top -b
```

当你run这个镜像，你会看到一个`PID 1`的进程：

```
$ docker run -it --rm --name test top
Mem: 1704520K used, 352148K free, 0K shrd, 0K buff, 140368121167873K cached
CPU:   5% usr   0% sys   0% nic  94% idle   0% io   0% irq   0% sirq
Load average: 0.08 0.03 0.05 2/98 6
  PID  PPID USER     STAT   VSZ %VSZ %CPU COMMAND
    1     0 root     R     3164   0%   0% top -b
```

使用`docker stop`可以正确的关闭：

```
$ /usr/bin/time docker stop test
test
real	0m 0.20s
user	0m 0.02s
sys	0m 0.04s
```

如果你忘记在你的`ENTRYPOINT` 前面添加`exec`：

```
FROM ubuntu
ENTRYPOINT top -b
CMD --ignored-param1
```

你接着可以运行它：

```
$ docker run -it --name test top --ignored-param2
Mem: 1704184K used, 352484K free, 0K shrd, 0K buff, 140621524238337K cached
CPU:   9% usr   2% sys   0% nic  88% idle   0% io   0% irq   0% sirq
Load average: 0.01 0.02 0.05 2/101 7
  PID  PPID USER     STAT   VSZ %VSZ %CPU COMMAND
    1     0 root     S     3168   0%   0% /bin/sh -c top -b cmd cmd2
    7     1 root     R     3164   0%   0% top -b
```

你可以从`top`的输出中看到，指定的`ENTRYPOINT`不是`PID 1`。

如果你运行`docker run test`，那个容器不会完全退出，`stop`命令会在超时过后发送一个SIGKILL命令。

```
$ docker exec -it test ps aux
PID   USER     COMMAND
    1 root     /bin/sh -c top -b cmd cmd2
    7 root     top -b
    8 root     ps aux
$ /usr/bin/time docker stop test
test
real	0m 10.19s
user	0m 0.04s
sys	0m 0.03s
```

