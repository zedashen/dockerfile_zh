# Exec形式的ENTRYPOINT例子

你可以使用*exec*形式的`ENTRYPOINT` 去设置相当稳定的命令和参数，然后使用`CMD`去设置可能会改变的更多默认命令。

```
FROM ubuntu
ENTRYPOINT ["top", "-b"]
CMD ["-c"]
```

当你运行容器，你会看到`top`是唯一的进程：

```
$ docker run -it --rm --name test  top -H
top - 08:25:00 up  7:27,  0 users,  load average: 0.00, 0.01, 0.05
Threads:   1 total,   1 running,   0 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.1 us,  0.1 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem:   2056668 total,  1616832 used,   439836 free,    99352 buffers
KiB Swap:  1441840 total,        0 used,  1441840 free.  1324440 cached Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
    1 root      20   0   19744   2336   2080 R  0.0  0.1   0:00.04 top
```

为了进一步检查结果，你可以用`docker exec`：

```
$ docker exec -it test ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  2.6  0.1  19752  2352 ?        Ss+  08:24   0:00 top -b -H
root         7  0.0  0.1  15572  2164 ?        R+   08:25   0:00 ps aux
```

并且你可以优雅的关闭`top`请求使用`docker stop test`。

下面的`Dockerfile`展示了使用`ENTRYPOINT`去在前台运行Apache(即，作为`PID 1`)：

```
FROM debian:stable
RUN apt-get update && apt-get install -y --force-yes apache2
EXPOSE 80 443
VOLUME ["/var/www", "/var/log/apache2", "/etc/apache2"]
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

如果你需要去编写一个启动脚本，那么你可以通过使用`exec`和`gosu`命令来接收Unix信号。

```
#!/bin/bash
set -e

if [ "$1" = 'postgres' ]; then
    chown -R postgres "$PGDATA"

    if [ -z "$(ls -A "$PGDATA")" ]; then
        gosu postgres initdb
    fi

    exec gosu postgres "$@"
fi

exec "$@"
```

最后，如果你需要在关机的时候进行一些额外的清理(或者通知其他容器)，或者协调多个可执行的程序，你需要确定`ENTRYPOINT` 脚本收到了Unix信号，传输他们，然后做一些更多的工作：

```
#!/bin/sh
# Note: I've written this using sh so it works in the busybox container too

# USE the trap if you need to also do manual cleanup after the service is stopped,
#     or need to start multiple services in the one container
trap "echo TRAPed signal" HUP INT QUIT TERM

# start service in background here
/usr/sbin/apachectl start

echo "[hit enter key to exit] or run 'docker stop <container>'"
read

# stop service and clean up here
echo "stopping apache"
/usr/sbin/apachectl stop

echo "exited $0"
```

如果你用`docker run -it --rm -p 80:80 --name test apache`运行了一个镜像，你可以检查容器的进程使用`docker exec`或者`docker top` ，并且使用脚本去停止Apache：

```
$ docker exec -it test ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.1  0.0   4448   692 ?        Ss+  00:42   0:00 /bin/sh /run.sh 123 cmd cmd2
root        19  0.0  0.2  71304  4440 ?        Ss   00:42   0:00 /usr/sbin/apache2 -k start
www-data    20  0.2  0.2 360468  6004 ?        Sl   00:42   0:00 /usr/sbin/apache2 -k start
www-data    21  0.2  0.2 360468  6000 ?        Sl   00:42   0:00 /usr/sbin/apache2 -k start
root        81  0.0  0.1  15572  2140 ?        R+   00:44   0:00 ps aux
$ docker top test
PID                 USER                COMMAND
10035               root                {run.sh} /bin/sh /run.sh 123 cmd cmd2
10054               root                /usr/sbin/apache2 -k start
10055               33                  /usr/sbin/apache2 -k start
10056               33                  /usr/sbin/apache2 -k start
$ /usr/bin/time docker stop test
test
real	0m 0.27s
user	0m 0.03s
sys	0m 0.03s
```

>**注意** ：你可以覆盖`ENTRYPOINT` 使用`--entrypoint` ，但是这个可以设置二进制数据去执行(没有`sh -c`会被使用)。



>**注意** ：*exec*形式会被解析为一个JSON数组，这意味着你必须使用双引号(")包裹单词而不是单引号(')。



>**注意** ：不同于*shell*形式，*exec*形式不会唤起shell。这意味着一般的shell进程是不会发生的。例如，`ENTRYPOINT [ "echo", "$HOME" ]`不会做`$HOME`的变量替换。如果你希望使用shell那么使用*shell*形式或者直接执行一个shell，例如：`ENTRYPOINT [ "sh", "-c", "echo $HOME" ]`。当使用exec形式直接执行一个shell，就像shell表单一样，他会使用执行环境中的变量替换，而不是docker的环境变量替换。