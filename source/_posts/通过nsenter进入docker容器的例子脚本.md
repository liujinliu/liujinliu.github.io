---
title: 通过nsenter进入docker容器的例子脚本
date: 2018-01-18
categories: 运维
---
下面是一个通过nsenter进入docker容器的例子脚本：
文件名字：ns
使用方法：将文件放入系统PATH路径下，进入容器方式
```
ns <container-name/container-id>
```

```
#!/bin/bash
if [ -e $(dirname "$0")/nsenter ]; then
NSENTER=$(dirname "$0")/nsenter
else
NSENTER=nsenter
fi
if [ -z "$1" ]; then
echo "Usage: `basename "$0"` CONTAINER [COMMAND [ARG]...]"
echo ""
echo "Enters the Docker CONTAINER and executes the specified COMMAND."
echo "If COMMAND is not specified, runs an interactive shell in CONTAINER."
else
PID=$(docker inspect -f "{{.State.Pid}}" "$1")
if [ -z "$PID" ]; then
exit 1
fi
shift
OPTS="--target $PID --mount --uts --ipc --net --pid --"
if [ -z "$1" ]; then
"$NSENTER" $OPTS su - root
else
"$NSENTER" $OPTS env --ignore-environment -- "$@"
fi
fi
```
