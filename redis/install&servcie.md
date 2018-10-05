#### `Redis`安装以及自动启动服务

#### `Redis`安装

[参考官网安装](https://redis.io/download)

~~~sh
wget http://download.redis.io/releases/redis-4.0.11.tar.gz
tar xzf redis-4.0.11.tar.gz	
cd redis-4.0.11
make
~~~

官网这样说的安装，也不能说有问题，只是说后续操作`redis`比较麻烦，每次都要切换到`src`下才行

可以直接将最后一步make修改为make install

~~~sh
wget http://download.redis.io/releases/redis-4.0.11.tar.gz
tar xzf redis-4.0.11.tar.gz	
cd redis-4.0.11
make & make install
~~~

#### 自动启动服务

- `redis`已经给我准备好了启动的脚本，在解压后的文件`./utils/redis_init_script`文件，我们只需在这个文件中的第二行加一个`chkconfig: 2345 90 10`，内容如下

~~~sh
#!/bin/sh
# chkconfig: 2345 90 10
#
# Simple Redis init.d script conceived to work on Linux systems
# as it does use of the /proc filesystem.

### BEGIN INIT INFO
# Provides:     redis_6379
# Default-Start:        2 3 4 5
# Default-Stop:         0 1 6
# Short-Description:    Redis data structure server
# Description:          Redis data structure server. See https://redis.io
### END INIT INFO

REDISPORT=6379
EXEC=/usr/local/bin/redis-server
CLIEXEC=/usr/local/bin/redis-cli

PIDFILE=/var/run/redis_${REDISPORT}.pid
CONF="/etc/redis/${REDISPORT}.conf"

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $CLIEXEC -p $REDISPORT shutdown
                while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    *)
        echo "Please use start or stop as first argument"
        ;;
esac

~~~

- 在将文件拷贝到`/etc/init.d`下,最好重新命名下。文件名看着别扭，我这边是就是`redis`。

  `chmod +x redis`,如果没法启动 `chkconfig --add redis`，将其加到服务中

- `CONF="/etc/redis/${REDISPORT}.conf"`这个默认在目录下没有，需要自己创建一个。

  `cp redis.conf /etc/redis/6397.conf` ,最后还是需要设置权限`chmod +x 6397.conf`

- 设置开机启动`chkconfig redis on`

`service redis start`启动服务

`service redis stop`停止服务

