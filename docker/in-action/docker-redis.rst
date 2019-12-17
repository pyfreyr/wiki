.. _docker-redis:


=====================
Docker 运行 Redis
=====================

本镜像用于启动 Redis 服务，并解决通常 redis-server 启动报警。

redis.conf
=============

示例配置：:download:`redis.conf </files/redis.conf>`

.. attention::

    注意不要使用后台启动，否则容器启动后马上关闭！


启动 Redis
=============

使用 docker-compose 启动 Redis，对应的 Compose 文件如下：

::

    version: '3'
    services:
      redis:
        image: redis
        sysctls:
          net.core.somaxconn: 511
        ports:
          - 6379:6379
        volumes:
          - $PWD/redis.conf:/etc/redis/redis.conf
          - $PWD/data:/data
        command: redis-server /etc/redis/redis.conf

启动命令如下：

::

    $ docker-compose up -d


WARNING 处理
==================

默认启动 redis-server 会出现几个警告，可按照以下操作配置宿主机解决。

maximum number of open files
--------------------------------

.. warning::

    Increased maximum number of open files to 10032 (it was originally set to 1024).

解决方法：

::

    $ echo '* soft nofile 65535' >> /etc/security/limits.conf
    $ echo '* hard nofile 65535' >> /etc/security/limits.conf

overcommit_memory
---------------------

.. warning::

    WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add
    'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.

解决方法：

::

    $ echo 1 > /proc/sys/vm/overcommit_memory
    $ echo "vm.overcommit_memory=1" >> /etc/sysctl.conf

backlog
-----------

.. note::

    如果使用 docker 启动可忽略，已经在 docker-compose.yml 中处理。

.. warning::

    WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.

解决方法：

::

    $ echo 511 >/proc/sys/net/core/somaxconn
    $ echo "net.core.somaxconn = 551" >> /etc/sysctl.conf

THP
--------

.. warning::

    WARNING you have Transparent Huge Pages (THP) support enabled in your kernel.
    This will create latency and memory usage issues with Redis.
    To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root,
    and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.

解决方法：

::

    $ echo never > /sys/kernel/mm/transparent_hugepage/enabled
    $ echo 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' >> /etc/rc.d/rc/local











