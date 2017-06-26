# ENTRYPOINT

ENTRYPOINT有两种形式：

* `ENTRYPOINT ["executable", "param1", "param2"]` (*exec* 形式，推荐)
* `ENTRYPOINT command param1 param2` (*shell* 形式)

`ENTRYPOINT` 允许你配置一个容器作为可执行的。

例如，下面的会使用默认的内容启动nginx，监听80端口：

`docker run -i -t --rm -p 80:80 nginx`

`docker run <image>` 的命令行参数会附加到`ENTRYPOINT` 的*exec* 形式后，并且会覆盖所用使用`CMD` 指定的元素。这允许把参数传入到entry point。例如，`docker run <image> -d` 会传输`-d` 参数到entry point。你可以覆写`ENTRYPOINT` 指令使用`docker run --entrypoint ` 命令。

*shell*形式会保证任何`CMD`或`run`的命令行参数不会被使用。但是有不好的一点，你的`ENTRYPOINT` 会被作为`/bin/sh -c`的子命令，是不会传输信号的。这以为着你执行的东西不是`PID 1` ，并且不会收到Unix的信号。因此你执行的东西是不会收到`docker stop <container>`的`SIGTERM` 信号。

只有`Dockerfile`中的最后一个`ENTRYPOINT`指令会生效。