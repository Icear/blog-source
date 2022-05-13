---
title: 使用 supervisor 来管理 WSL 上的服务并实现开机自启动 
date: 2020-01-17 17:46:53
updated: 2020-01-17 17:46:53
tags: [Supervisor, WSL, Service Management]
---

WSL 是 Windows 下的 Linux 子系统，通过兼容层实现在 Windows 平台上模拟 Linux 环境，不像 WSL2 拥有完整的 Linux 内核。这使得很多依赖内核机制的应用都无法使用，比如说 system-init 和 docker。没有 system-init 支持之后常用的服务管理工具 systemctl 也就失效了，虽然可以通过 service 命令来操作，但是遇到需要自定义服务的时候，service 就显得有些简陋了：需要手写较长的脚本配置外加不支持自动重启等。此时就需要 supervisor 了，基于子进程的服务管理方式不需要内核支持，能够在 WSL 环境很好的工作

## 相关细节
supervisor 可以通过系统自带包管理器安装，又因为是 python 编写，支持通过 pip 安装。但是两者安装的版本上有一些区别。
- 通过包管理器安装的 supervisor 会向 /etc/init.d 目录释放用于 service 命令管理的配置文件（其实就是个脚本文件，所以说有点麻烦写），但是它安装的 supervisor 所依赖的 python 版本会根据当前系统的初始版本变化。比如我这里使用的是 Ubuntu 系统，系统默认为 python2 ，虽然我已经手动切换到了 python 3，但是所安装的 supervisor 依然是基于 python2 版本（这个依赖应该是由 Ubuntu 的包所决定的，大概制作包的时候就是以 python2 为基准制作的吧，并没有根据环境自动选择），于是在安装 python2 版本的 supervisor 而运行环境为 python3 时，运行 supervisor 就会出现如下错误，找不到对应包：
```
Traceback (most recent call last):
  File "/usr/bin/supervisord", line 6, in <module>
    from pkg_resources import load_entry_point
  File "/usr/lib/python3/dist-packages/pkg_resources/__init__.py", line 3088, in <module>
    @_call_aside
  File "/usr/lib/python3/dist-packages/pkg_resources/__init__.py", line 3072, in _call_aside
    f(*args, **kwargs)
  File "/usr/lib/python3/dist-packages/pkg_resources/__init__.py", line 3101, in _initialize_master_working_set
    working_set = WorkingSet._build_master()
  File "/usr/lib/python3/dist-packages/pkg_resources/__init__.py", line 574, in _build_master
    ws.require(__requires__)
  File "/usr/lib/python3/dist-packages/pkg_resources/__init__.py", line 892, in require
    needed = self.resolve(parse_requirements(requirements))
  File "/usr/lib/python3/dist-packages/pkg_resources/__init__.py", line 778, in resolve
    raise DistributionNotFound(req, requirers)
pkg_resources.DistributionNotFound: The 'supervisor==3.3.1' distribution was not found and is required by the application
```
- 通过 pip 安装的 supervisor 能够正常匹配所使用的 python 版本<del>（废话）</del>，但是
    - 使用当前用户安装时程序会被放置到家目录的 .local/lib/python*/site-packages/ 目录下，虽然能够调用但是不是常规位置，不便于管理
    - 使用 sudo 或 root 权限安装时会安装到 /usr/local/lib/ 目录下，可执行文件在 /usr/local/bin/ 目录中，比较正常

## 最终方案
配置安装 supervisor 并使其支持通过 service 命令控制启停，然后在 Windows 下设置开机时启动 supervisor 进行，借此启动注册在 supervisor 下的各种服务

- 使用默认 python 版本的话可以直接从包管理器安装 supervisor，之后通过以下命令来启动进程，相关配置文件在 /etc/supervisor/ 下
    ```
    sudo service supervisor start
    ```

## 配置步骤

### 安装 supervisor 并配置
- 使用 python3 但系统默认版本为 python2 的情况，使用 pip 进行全局安装，但配置文件使用包管理器自带版本，即 /etc/init.d/ 目录下相关服务管理脚本与 /etc/supervisor/ 配置文件目录。这里为了部署方便我会直接贴出从包管理器安装版本提取的服务管理脚本以及 supervisor 配置文件。以下为部署步骤：
    - 使用 pip 全局安装 supervisor 
        ```
        sudo pip install supervisor
        ```
    - 创建服务管理脚本
        ```
        sudo vi /etc/init.d/supervisor
        ```
        以下为脚本内容，已修改程序路径
        ```
        #! /bin/sh
        #
        # skeleton      example file to build /etc/init.d/ scripts.
        #               This file should be used to construct scripts for /etc/init.d.
        #
        #               Written by Miquel van Smoorenburg <miquels@cistron.nl>.
        #               Modified for Debian
        #               by Ian Murdock <imurdock@gnu.ai.mit.edu>.
        #               Further changes by Javier Fernandez-Sanguino <jfs@debian.org>
        #
        # Version:      @(#)skeleton  1.9  26-Feb-2001  miquels@cistron.nl
        #
        ### BEGIN INIT INFO
        # Provides:          supervisor
        # Required-Start:    $remote_fs $network $named
        # Required-Stop:     $remote_fs $network $named
        # Default-Start:     2 3 4 5
        # Default-Stop:      0 1 6
        # Short-Description: Start/stop supervisor
        # Description:       Start/stop supervisor daemon and its configured
        #                    subprocesses.
        ### END INIT INFO

        . /lib/lsb/init-functions

        PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
        DAEMON=/usr/local/bin/supervisord
        NAME=supervisord
        DESC=supervisor

        test -x $DAEMON || exit 0

        RETRY=TERM/30/KILL/5
        LOGDIR=/var/log/supervisor
        PIDFILE=/var/run/$NAME.pid
        DODTIME=5                   # Time to wait for the server to die, in seconds
                                    # If this value is set too low you might not
                                    # let some servers to die gracefully and
                                    # 'restart' will not work

        # Include supervisor defaults if available
        if [ -f /etc/default/supervisor ] ; then
                . /etc/default/supervisor
        fi
        DAEMON_OPTS="-c /etc/supervisor/supervisord.conf $DAEMON_OPTS"

        set -e

        running_pid()
        {
            # Check if a given process pid's cmdline matches a given name
            pid=$1
            name=$2
            [ -z "$pid" ] && return 1
            [ ! -d /proc/$pid ] &&  return 1
            (cat /proc/$pid/cmdline | tr "\000" "\n"|grep -q $name) || return 1
            return 0
        }

        running()
        {
        # Check if the process is running looking at /proc
        # (works for all users)

            # No pidfile, probably no daemon present
            [ ! -f "$PIDFILE" ] && return 1
            # Obtain the pid and check it against the binary name
            pid=`cat $PIDFILE`
            running_pid $pid $DAEMON || return 1
            return 0
        }

        case "$1" in
        start)
                echo -n "Starting $DESC: "
                start-stop-daemon --start --quiet --pidfile $PIDFILE \
                        --startas $DAEMON -- $DAEMON_OPTS
                test -f $PIDFILE || sleep 1
                if running ; then
                    echo "$NAME."
                else
                    echo " ERROR."
                fi
                ;;
        stop)
                echo -n "Stopping $DESC: "
                # [ -f $PIDFILE ] && killproc $prog || success $"$prog shutdown"
                start-stop-daemon --stop --quiet --oknodo --pidfile $PIDFILE
                echo "$NAME."
                ;;
        #reload)
                #
                #       If the daemon can reload its config files on the fly
                #       for example by sending it SIGHUP, do it here.
                #
                #       If the daemon responds to changes in its config file
                #       directly anyway, make this a do-nothing entry.
                #
                # echo "Reloading $DESC configuration files."
                # start-stop-daemon --stop --signal 1 --quiet --pidfile \
                #       /var/run/$NAME.pid --exec $DAEMON
        #;;
        force-reload)
                #
                #       If the "reload" option is implemented, move the "force-reload"
                #       option to the "reload" entry above. If not, "force-reload" is
                #       just the same as "restart" except that it does nothing if the
                #   daemon isn't already running.
                # check wether $DAEMON is running. If so, restart
                start-stop-daemon --stop --test --quiet --pidfile $PIDFILE \
                --startas $DAEMON \
                && $0 restart \
                || exit 0
                ;;
        restart)
            echo -n "Restarting $DESC: "
            start-stop-daemon --stop --retry=$RETRY --quiet --oknodo --pidfile $PIDFILE
                echo "$NAME."
                ;;
        status)
            echo -n "$NAME is "
            if running ;  then
                echo "running"
            else
                echo " not running."
                exit 1
            fi
            ;;
        *)
                N=/etc/init.d/$NAME
                # echo "Usage: $N {start|stop|restart|reload|force-reload}" >&2
                echo "Usage: $N {start|stop|restart|force-reload|status}" >&2
                exit 1
                ;;
        esac

        exit 0
        ```
    - 创建配置目录
        ```
        sudo mkdir /etc/supervisor
        sudo mkdir /etc/supervisor/conf.d
        sudo vi /etc/supervisor/supervisord.conf
        ```
        以下为脚本内容
        ```
        ; supervisor config file

        [unix_http_server]
        file=/var/run/supervisor.sock   ; (the path to the socket file)
        chmod=0700                       ; sockef file mode (default 0700)

        [supervisord]
        logfile=/var/log/supervisor/supervisord.log ; (main log file;default $CWD/supervisord.log)
        pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
        childlogdir=/var/log/supervisor            ; ('AUTO' child log dir, default $TEMP)

        ; the below section must remain in the config file for RPC
        ; (supervisorctl/web interface) to work, additional interfaces may be
        ; added by defining them in separate rpcinterface: sections
        [rpcinterface:supervisor]
        supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

        [supervisorctl]
        serverurl=unix:///var/run/supervisor.sock ; use a unix:// URL  for a unix socket

        ; The [include] section can just contain the "files" setting.  This
        ; setting can list multiple files (separated by whitespace or
        ; newlines).  It can also contain wildcards.  The filenames are
        ; interpreted as relative to this file.  Included files *cannot*
        ; include files themselves.

        [include]
        files = /etc/supervisor/conf.d/*.conf
        ```
    -   至此配置结束，启动服务的命令为 
        ```
        sudo service supervisor start
        ```

### 设置自启动
- 使用 Windows + R 快捷键调出运行窗口，输入 
    ```
    shell:startup
    ```
- 在弹出的文件目录下创建一个快捷方式，名称任意，其执行命令设为 
    ```
    C:\Windows\System32\wsl.exe -u root service supervisor start
    ``` 

至此所有的操作就完成了，向 supervisor 配置的各种服务也能够正常运作，具体的配置方式可以查阅官方文档。