---
title: maven打包dubbo接口之最佳实践
author: Bridge Li
type: post
date: 2015-12-20T13:56:50+00:00

categories:
  - Maven
tags:
  - Dubbo
  - maven
  - packing

---
之前刚开始学习dubbo的时候，曾写过一个入门的[小例子][1]，当时生产者也是用tomcat去跑的，其实dubbo只需要提供service层接口就好了，并不需要和http相关的东西，所以其实并不需要用tomcat去跑，我们完全打成其他的包直接去跑，这样dubbo接口也不会tomcat性能的限制，而打包可以说是maven最擅长的事情之一，今天就记录一下我们公司的实际项目中使用maven-assembly-plugin打包的方法。

1. 首先在pom文件中，添加maven-assembly-plugin插件

```
<plugin>
  <artifactId>maven-assembly-plugin</artifactId>
  <configuration>
    <descriptor>src/main/assembly/assembly.xml</descriptor>
  </configuration>
  <executions>
    <execution>
      <id>make-assembly</id>
      <phase>package</phase>
      <goals>
        <goal>single </goal>
      </goals>
    </execution>
  </executions>
</plugin>

```

在该插件的第四行我们指定了一个assembly.xml文件，下面我们就看看assembly.xml的内容

2. assembly.xml文件

```
<assembly>
  <id>assembly</id>
  <formats>
    <format>tar.gz</format>
  </formats>
  <includeBaseDirectory>true</includeBaseDirectory>
  <fileSets>
    <fileSet>
      <outputDirectory>/</outputDirectory>
      <includes>
        <include>README.txt</include>
      </includes>
    </fileSet>
    <fileSet>
      <directory>src/main/scripts</directory>
      <outputDirectory>/bin</outputDirectory>
    </fileSet>
  </fileSets>
  <dependencySets>
    <dependencySet>
      <useProjectArtifact>true</useProjectArtifact>
      <outputDirectory>lib</outputDirectory>
    </dependencySet>
  </dependencySets>
</assembly>

```

该文件的第四行中的tar.gz指的就是打包的文件格式，对于Linux用户，对这个格式一定非常熟悉，当然大家也可以指定为zip格式，另外在该文件的第十五行，指定了一个scripts文件夹，那么这里面放的又是什么呢？我们知道打包之后的系统我们要跑起来才能用，那么这里面放的就是对我们的系统操作的一些脚本，打包之后，我们的系统都是一些jar文件，放在了倒数第四行指定的lib文件中，而这些脚本则放在了和lib同级的bin文件中，下面就让我们一一看看scripts中几个文件的内容

3. scripts文件夹

①. start.bat

```
@echo off
setlocal enabledelayedexpansion

:: 初始化变量
set "LIB_JARS="

:: 切换到 lib 目录并拼接 jar 包路径
cd ..\lib
for %%i in (*) do (
    set "LIB_JARS=!LIB_JARS!;..\lib\%%i"
)
cd ..\bin

:: 判断启动参数
if "%1" == "debug" goto debug
if "%1" == "jmx" goto jmx

:: 默认启动模式
java -Xms64m -Xmx1024m -XX:MaxPermSize=64M -classpath ..\conf;%LIB_JARS% com.alibaba.dubbo.container.Main
goto end

:debug
:: Debug 模式启动
java -Xms64m -Xmx1024m -XX:MaxPermSize=64M -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,address=8000,server=y,suspend=n -classpath ..\conf;%LIB_JARS% com.alibaba.dubbo.container.Main
goto end

:jmx
:: JMX 模式启动
java -Xms64m -Xmx1024m -XX:MaxPermSize=64M -Dcom.sun.management.jmxremote.port=1099 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -classpath ..\conf;%LIB_JARS% com.alibaba.dubbo.container.Main

:end
pause
```

②. start.sh

```
#!/bin/bash

# 获取脚本所在目录
cd "$(dirname "$0")"
BIN_DIR=$(pwd)
cd ..
DEPLOY_DIR=$(pwd)
CONF_DIR="$DEPLOY_DIR/conf"

# 运行用户与组配置
USER=www
GROUP=www

# --- 配置读取 (默认留空，可根据需要开启 sed 读取逻辑) ---
# SERVER_NAME=$(sed '/dubbo.application.name/!d;s/.*=//' conf/dubbo.properties | tr -d '\r')
# SERVER_PROTOCOL=$(sed '/dubbo.protocol.name/!d;s/.*=//' conf/dubbo.properties | tr -d '\r')
# SERVER_PORT=$(sed '/dubbo.protocol.port/!d;s/.*=//' conf/dubbo.properties | tr -d '\r')
# LOGS_FILE=$(sed '/dubbo.log4j.file/!d;s/.*=//' conf/dubbo.properties | tr -d '\r')

SERVER_NAME=""
SERVER_PROTOCOL=""
SERVER_PORT=""
LOGS_FILE=""

# 如果未配置应用名称，使用主机名
if [ -z "$SERVER_NAME" ]; then
    SERVER_NAME=$(hostname)
fi

# --- 1. 检查进程是否已启动 ---
PIDS=$(ps -f | grep java | grep "$CONF_DIR" | awk '{print $2}')
if [ -n "$PIDS" ]; then
    echo "ERROR: The $SERVER_NAME already started!"
    echo "PID: $PIDS"
    exit 1
fi

# --- 2. 检查端口是否被占用 ---
if [ -n "$SERVER_PORT" ]; then
    SERVER_PORT_COUNT=$(netstat -tln | grep "$SERVER_PORT" | wc -l)
    if [ "$SERVER_PORT_COUNT" -gt 0 ]; then
        echo "ERROR: The $SERVER_NAME port $SERVER_PORT already used!"
        exit 1
    fi
fi

# --- 3. 准备日志目录 ---
LOGS_DIR="/data/logs/$(basename "$DEPLOY_DIR")"

if [ ! -d "$LOGS_DIR" ]; then
    mkdir -p "$LOGS_DIR"
    chown -R "$USER"."$GROUP" "$LOGS_DIR"
fi
STDOUT_FILE="$LOGS_DIR/$(basename "$DEPLOY_DIR").log"

# --- 4. 构建 Classpath ---
LIB_DIR="$DEPLOY_DIR/lib"
# 将 lib 目录下所有 jar 包拼接为 classpath 字符串
LIB_JARS=$(ls "$LIB_DIR" | grep .jar | awk '{print "'"$LIB_DIR"'/"$0}' | tr "\n" ":")

# --- 5. 设置 Java 启动参数 ---
JAVA_OPTS=" -Djava.awt.headless=true -Djava.net.preferIPv4Stack=true "
JAVA_DEBUG_OPTS=""
JAVA_JMX_OPTS=""
JAVA_MEM_OPTS=""

# 调试模式参数
if [ "$1" = "debug" ]; then
    JAVA_DEBUG_OPTS=" -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,address=8000,server=y,suspend=n "
fi

# JMX 模式参数
if [ "$1" = "jmx" ]; then
    JAVA_JMX_OPTS=" -Dcom.sun.management.jmxremote.port=1099 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false "
fi

# 内存参数 (根据 JDK 位数自动调整)
BITS=$(java -version 2>&1 | grep -i 64-bit)
if [ -n "$BITS" ]; then
    # 64位系统配置
    JAVA_MEM_OPTS=" -server -Xmx2g -Xms2g -Xmn720m -XX:PermSize=128m -Xss256k -XX:+DisableExplicitGC -XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled -XX:+UseCMSCompactAtFullCollection -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=70 "
else
    # 32位系统配置
    JAVA_MEM_OPTS=" -server -Xms2g -Xmx2g -XX:PermSize=128m -XX:SurvivorRatio=2 -XX:+UseParallelGC "
fi

# --- 6. 启动服务 ---
echo -e "Starting the $SERVER_NAME ...\c"
nohup java $JAVA_OPTS $JAVA_MEM_OPTS $JAVA_DEBUG_OPTS $JAVA_JMX_OPTS \
    -classpath "$CONF_DIR:$LIB_JARS" \
    com.alibaba.dubbo.container.Main \
    > "$STDOUT_FILE" 2>&1 &

# --- 7. 检查启动状态 ---
COUNT=0
while [ $COUNT -lt 1 ]; do
    echo -e ".\c"
    sleep 1

    # 检查逻辑：优先检查端口，其次检查进程
    if [ -n "$SERVER_PORT" ]; then
        if [ "$SERVER_PROTOCOL" == "dubbo" ]; then
            # Dubbo 协议特有的检查方式
            COUNT=$(echo status | nc -i 1 127.0.0.1 "$SERVER_PORT" | grep -c OK)
        else
            # 普通端口检查
            COUNT=$(netstat -an | grep "$SERVER_PORT" | wc -l)
        fi
    else
        # 无端口信息时检查进程数
        COUNT=$(ps -f | grep java | grep "$DEPLOY_DIR" | awk '{print $2}' | wc -l)
    fi

    if [ "$COUNT" -gt 0 ]; then
        break
    fi
done

echo "OK!"
PIDS=$(ps -f | grep java | grep "$DEPLOY_DIR" | awk '{print $2}')
echo "PID: $PIDS"
echo "STDOUT: $STDOUT_FILE"
```

③. stop.sh

```
#!/bin/bash

# 1. 获取脚本所在目录
cd "$(dirname "$0")"
BIN_DIR=$(pwd)

# 2. 获取部署根目录
cd ..
DEPLOY_DIR=$(pwd)
CONF_DIR="$DEPLOY_DIR/conf"

# 3. 读取应用名称
# 注意：原代码中的 'r' 可能是 '\r' 的误写，这里修正为去除回车符
SERVER_NAME=$(sed '/dubbo.application.name/!d;s/.*=//' conf/dubbo.properties | tr -d '\r')

if [ -z "$SERVER_NAME" ]; then
    SERVER_NAME=$(hostname)
fi

# 4. 查找进程 ID
PIDS=$(ps -f | grep java | grep "$CONF_DIR" | awk '{print $2}')

if [ -z "$PIDS" ]; then
    echo "ERROR: The $SERVER_NAME does not started!"
    exit 1
fi

# 5. 停止前 Dump (除非传入 skip 参数)
if [ "$1" != "skip" ]; then
    "$BIN_DIR/dump.sh"
fi

# 6. 发送停止信号
echo -e "Stopping the $SERVER_NAME ...\c"
for PID in $PIDS; do
    kill "$PID" > /dev/null 2>&1
done

# 7. 轮询等待进程完全消失
COUNT=0
while [ $COUNT -lt 1 ]; do
    echo -e ".\c"
    sleep 1
    COUNT=1 # 假设所有进程已停止

    for PID in $PIDS; do
        # 检查该 PID 是否还存在
        PID_EXIST=$(ps -f -p "$PID" | grep java)
        if [ -n "$PID_EXIST" ]; then
            COUNT=0 # 进程仍存在，继续循环
            break
        fi
    done
done

echo "OK!"
echo "PID: $PIDS"
```

④. restart.sh

```
#!/bin/bash

# 1. 切换到脚本所在目录
# 使用 $(dirname "$0") 获取脚本所在路径，确保后续执行 stop.sh 和 start.sh 时能找到文件
cd "$(dirname "$0")"

# 2. 执行停止脚本
./stop.sh

# 3. 执行启动脚本
./start.sh
```

⑤. server.sh

```
#!/bin/bash

# 1. 切换到脚本所在目录
cd "$(dirname "$0")"

# 2. 参数判断与执行
if [ "$1" = "start" ]; then
    ./start.sh

elif [ "$1" = "stop" ]; then
    ./stop.sh

elif [ "$1" = "debug" ]; then
    ./start.sh debug

elif [ "$1" = "restart" ]; then
    ./restart.sh

elif [ "$1" = "dump" ]; then
    ./dump.sh

else
    echo "ERROR: Please input argument: start or stop or debug or restart or dump"
    exit 1
fi
```

⑥. dump.sh

```
#!/bin/bash

# 1. 获取脚本所在目录与部署目录
cd "$(dirname "$0")"
BIN_DIR=$(pwd)
cd ..
DEPLOY_DIR=$(pwd)
CONF_DIR="$DEPLOY_DIR/conf"

# 2. 读取配置信息
# 注意：原代码中的 'r' 修正为 '\r' 以去除 Windows 换行符
SERVER_NAME=$(sed '/dubbo.application.name/!d;s/.*=//' conf/dubbo.properties | tr -d '\r')
LOGS_FILE=$(sed '/dubbo.log4j.file/!d;s/.*=//' conf/dubbo.properties | tr -d '\r')

if [ -z "$SERVER_NAME" ]; then
    SERVER_NAME=$(hostname)
fi

# 3. 检查进程是否启动
PIDS=$(ps -f | grep java | grep "$CONF_DIR" | awk '{print $2}')
if [ -z "$PIDS" ]; then
    echo "ERROR: The $SERVER_NAME does not started!"
    exit 1
fi

# 4. 准备日志与导出目录
LOGS_DIR=""
if [ -n "$LOGS_FILE" ]; then
    LOGS_DIR=$(dirname "$LOGS_FILE")
else
    LOGS_DIR="$DEPLOY_DIR/logs"
fi

if [ ! -d "$LOGS_DIR" ]; then
    mkdir -p "$LOGS_DIR"
fi

DUMP_DIR="$LOGS_DIR/dump"
if [ ! -d "$DUMP_DIR" ]; then
    mkdir -p "$DUMP_DIR"
fi

DUMP_DATE=$(date +%Y%m%d%H%M%S)
DATE_DIR="$DUMP_DIR/$DUMP_DATE"
if [ ! -d "$DATE_DIR" ]; then
    mkdir -p "$DATE_DIR"
fi

# 5. 执行诊断命令收集信息
echo -e "Dumping the $SERVER_NAME ...\c"

for PID in $PIDS; do
    # JVM 线程堆栈
    jstack "$PID" > "$DATE_DIR/jstack-$PID.dump" 2>&1
    echo -e ".\c"

    # JVM 配置信息
    jinfo "$PID" > "$DATE_DIR/jinfo-$PID.dump" 2>&1
    echo -e ".\c"

    # GC 统计信息
    jstat -gcutil "$PID" > "$DATE_DIR/jstat-gcutil-$PID.dump" 2>&1
    echo -e ".\c"

    # GC 容量信息
    jstat -gccapacity "$PID" > "$DATE_DIR/jstat-gccapacity-$PID.dump" 2>&1
    echo -e ".\c"

    # 堆内存快照 (注意：原脚本使用的是 jmap <pid>，通常用于生成 dump 文件需加 -dump 参数，此处保持原脚本逻辑)
    jmap "$PID" > "$DATE_DIR/jmap-$PID.dump" 2>&1
    echo -e ".\c"

    # 堆内存详细信息
    jmap -heap "$PID" > "$DATE_DIR/jmap-heap-$PID.dump" 2>&1
    echo -e ".\c"

    # 堆对象直方图
    jmap -histo "$PID" > "$DATE_DIR/jmap-histo-$PID.dump" 2>&1
    echo -e ".\c"

    # 进程打开的文件句柄
    if [ -r /usr/sbin/lsof ]; then
        /usr/sbin/lsof -p "$PID" > "$DATE_DIR/lsof-$PID.dump"
        echo -e ".\c"
    fi
done

# 6. 收集系统级信息
if [ -r /bin/netstat ]; then
    /bin/netstat -an > "$DATE_DIR/netstat.dump" 2>&1
    echo -e ".\c"
fi

if [ -r /usr/bin/iostat ]; then
    /usr/bin/iostat > "$DATE_DIR/iostat.dump" 2>&1
    echo -e ".\c"
fi

if [ -r /usr/bin/mpstat ]; then
    /usr/bin/mpstat > "$DATE_DIR/mpstat.dump" 2>&1
    echo -e ".\c"
fi

if [ -r /usr/bin/vmstat ]; then
    /usr/bin/vmstat > "$DATE_DIR/vmstat.dump" 2>&1
    echo -e ".\c"
fi

if [ -r /usr/bin/free ]; then
    /usr/bin/free -t > "$DATE_DIR/free.dump" 2>&1
    echo -e ".\c"
fi

if [ -r /usr/bin/sar ]; then
    /usr/bin/sar > "$DATE_DIR/sar.dump" 2>&1
    echo -e ".\c"
fi

if [ -r /usr/bin/uptime ]; then
    /usr/bin/uptime > "$DATE_DIR/uptime.dump" 2>&1
    echo -e ".\c"
fi

echo "OK!"
echo "DUMP: $DATE_DIR"
```

至于这些文件是干嘛的，相信作为一个程序猿不用老夫一一解释了，聪明如您，一定一眼就能看出来了，如果您的shell水平目前还不够，可以先看看老夫的[这篇文章][2]，看完之后再看这几个脚本可以说完全无压力。

 [1]: https://www.bridgeli.cn/archives/196 "Dubbo和zookeeper入门实例"
 [2]: https://www.bridgeli.cn/archives/163 "Shell编程入门"