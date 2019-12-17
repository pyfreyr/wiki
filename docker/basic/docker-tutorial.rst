.. _docker-tutorial:


====================
Docker 入门指南
====================



本文介绍 Docker 的基本使用方法和常用命令，包括容器的创建、查看、停止和删除。通过这些操作可以实现单个容器的使用和应用部署。

至于 Docker 镜像构建、Dockerfile 书写和编配工具 Compose 的使用等进阶话题，移步 :ref:`Docker 基础<docker-basic>` 。

安装和启动 Docker
==================

安装和启动参考 :ref:`Docker 安装教程 <install-docker>` 。

如果没有启动 docker，则需要手动启动 Docker 守护进程：

::

    $ systemctl start docker


镜像和容器
================

.. image:: /images/image-and-container.jpg
   :align: center


**镜像** （Image）就是一堆只读层（read-only layer）的统一视角。

.. image:: /images/docker-image.jpg
   :align: center


**容器** （container）的定义和镜像（image）几乎一模一样，也是一堆层的统一视角，**唯一区别在于容器的最上面那一层是可读可写的** 。

.. image:: /images/docker-container.jpg
   :align: center

详细的说明以及 Docker 命令的理解参考：`Docker objects <https://docs.docker.com/engine/docker-overview/#docker-objects>`_ 和 `Docker 容器和镜像的区别`_ 。

.. _Docker 容器和镜像的区别: https://www.cnblogs.com/bethal/p/5942369.html


运行第一个容器
===============


Docker 操作的对象就是容器，具体操作包括创建、启动、查看、关闭、删除等。

启动一个容器：

::

    $ docker run -it --name myubuntu ubuntu /bin/bash

其中：

- ``docker run`` 为 Docker 命令；
- ``-it``、``--name`` 为命令参数，``-i`` 表示开启容器 ``stdin``，``-t`` 表示创建伪 tty 终端，只有指定这两个参数才能提供交互式 shell；``--name`` 用于指定要创建容器的名字，这里取为 myubuntu，若不指定容器名称则自动生成随机名称。容器名称必须唯一！
- ``ubuntu`` 指定创建容器所需要的镜像名，每个容器都依赖于镜像创建，镜像概念后续更新。
- ``/bin/bash`` 指定了容器启动后执行的命令，这里表示启动 ubuntu 系统内的 bash。

不出意外，成功进入 ubuntu：

::

    root@22c17371eeae:/#

.. note::

    Docker 命令的使用帮助：

        1. 查看 Docker 支持的命令 ``docker --help``

        2. 查看具体命令的使用 ``docker run --help``


容器的使用
============

上面已经成功以 root 用户启动容器 myubuntu 并进入 bash 终端，这是一个完整的 ubuntu 系统，可以用来做任何事情。其中容器的 ID 为 22c17371eeae 同时也是容器的主机名。

::

    root@22c17371eeae:/# hostname
    22c17371eeae

我们可以尝试在系统内安装 vim：

::

    root@22c17371eeae:/# apt update -y && apt install -y vim
    Get:1 http://archive.ubuntu.com/ubuntu xenial InRelease [247 kB]
    Get:2 http://security.ubuntu.com/ubuntu xenial-security InRelease [107 kB]
    Get:3 http://security.ubuntu.com/ubuntu xenial-security/universe Sources [87.2 kB]
    ...


再看看看容器的进程：

::

    root@22c17371eeae:/# ps -ef
    UID        PID  PPID  C STIME TTY          TIME CMD
    root         1     0  0 12:04 pts/0    00:00:00 /bin/bash
    root       423     1  0 12:12 pts/0    00:00:00 ps -ef


你可以继续做任何想做的，如安装 python3， 部署 flask ...，甚至 ``rm -rf /`` o(∩_∩)o 。不用担心，创建容器是 ``docker run`` 那么容易。

现在是时候退出了，输入 ``exit`` 并回车（或者 ``Ctrl-D``）：

::

    root@22c17371eeae:/# exit
    exit
    $


一旦退出容器，容器也随之停止（如果不想容器退出就停止，参考下面 :ref:`创建后台容器 <container-daemon>`）。我们又回到了主机，初次的体验完毕。


查看容器
===========

退出容器回到宿主机，容器虽然停止，但是仍然存在，可以再次使用。

使用 ``docker ps`` 查看当前系统的容器列表（``-a`` 显示所有容器，包括运行和停止的，不加则只显示运行的）

::

    $ docker ps -a
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
    22c17371eeae        ubuntu              "/bin/bash"         12 minutes ago      Exited (0) 3 minutes ago                       myubuntu


其中：

- ``CONTAINER ID``，容器 id
- ``IMAGE``，创建容器使用的镜像
- ``COMMAND``，容器最后执行的命令
- ``CREATED``，容器的创建时间
- ``STATUS``，容器的退出状态
- ``PORTS``，容器和主机的端口映射
- ``NAMES``，容器名称

.. tip::

    ``docker inspect`` 可以查看容器的配置信息，如名称、网络等。

重新启动已停止的容器
========================

上面使用 ``exit`` 退出了容器，容器也随之停止。也许以后某个时候，我们还需要再次使用（当然可以再创建一个新的，这样的话，不用的容器记得删除，删除方法见本文最后），则可以使用 ``docker start`` 重新启动停止的容器：

::

    $ docker start myubuntu

一般的，容器名和容器 ID 可以替换使用，如上面操作也可以用：

::

    $ docker start 22c17371eeae

.. tip::

    不需要使用完整的 ID，开始的 3 - 4 位足矣！

类似相关命令有：``docker restart``, ``docker create``，如何学习？``docker --help`` 。

有没有发现执行了上述命令，好像没有看到什么变化？ ``docker ps`` 一下：

::

    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    22c17371eeae        ubuntu              "/bin/bash"         16 minutes ago      Up 3 seconds                            myubuntu

注意 **STATUS**，已经显示为 **UP**。

那么问题来了，如何再次进入终端呢？有两种方法（科普一下茴香豆的茴有4种写法：茴 ，回，囘，囬）：


1. 附着到容器（``docker attach`` 执行后需要 Enter 直到进入）

::

        $ docker attach myubuntu

2. 在容器内重新运行新进程（``docker exec`` 登场）

::

        $ docker exec -it myubuntu /bin/bash

在第二种方法中，我们新启动了 bash 进程：

::

    root@22c17371eeae:/# ps -ef
    UID        PID  PPID  C STIME TTY          TIME CMD
    root         1     0  0 12:20 pts/0    00:00:00 /bin/bash
    root        11     0  0 12:23 pts/1    00:00:00 /bin/bash
    root        21    11  0 12:23 pts/1    00:00:00 ps -ef

.. attention::

   注意上面操作都要求容器先启动！

同样，``exit`` 退出。

.. _container-daemon:

创建后台容器
================



前面创建的容器都是交互式运行的，更多时候我们需要创建长期运行的容器来运行服务，而且不需要交互式会话（比如数据库）。

容器创建仍旧使用 ``docker run`` 命令，但这次使用 ``-d`` 参数：

::

    $ docker run --name myredis -d -p 9527:6379 redis



这里我使用 redis 镜像 并添加 ``-d`` 将任务放到了后台，所以执行命令后仍然回到主机而没有进入容器。至于 ``-p`` 参数，用于容器与主机的端口映射（上面例子将容器内 redis 端口映射到主机的 9527），这次 ``docker ps`` 可以看到 **PORTS** 有了内容。

::

    $ docker ps
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
    e88992f6a3e0        redis               "docker-entrypoint.s…"   5 seconds ago       Up 4 seconds        0.0.0.0:9527->6379/tcp   myredis

.. tip::

    查看某容器的端口可直接使用命令 ``docker port``：

    ::

        $ docker port myredis
        6379/tcp -> 0.0.0.0:9527


通过 ``docker ps`` 可以看到容器已经成功在后台运行，如何与该容器交互使用呢？比如这里如何连接上 redis？其实很简单，只要使用主机的 ip 和 容器映射的主机端口就可以正常使用。

::

    $ redis-cli -h ${your-IP} -p 9527

如果想研究容器内部都做了些什么，可以使用 ``docker logs`` 来读取容器日志。命令可以添加参数 ``-f`` 进行滚动查看（效果类似 ``tail -f``），参数 ``-t`` 为 每条日志添加时间戳。退出日志跟踪使用 ``Ctrl-C`` 。

::

    $ docker logs -ft myredis
    2018-08-13T12:25:28.424333397Z 1:C 13 Aug 12:25:28.423 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
    2018-08-13T12:25:28.424390765Z 1:C 13 Aug 12:25:28.424 # Redis version=4.0.9, bits=64, commit=00000000, modified=0, pid=1, just started
    ...

如果想进入容器，使用上面的 ``docker exec`` 命令。

更多容器状态检测命令，查阅：``docker top`` , ``docker stats`` 。

停止后台容器
==============

只需要执行 ``docker stop`` 命令：

::

    $ docker stop myredis

还有个命令 ``docker kill`` 直接向容器发送 ``SIGKILL`` 信号停止容器。

删除容器
============

如果容器不再使用，可以使用 ``docker rm`` 删除。

::

    $ docker rm myredis

.. tip::

    若要删除运行中的容器，使用 ``-f`` 参数。但最好先 stop 然后 rm。

若要删除所有容器：

::

    $ docker rm ``docker ps -aq``
    e88992f6a3e0
    22c17371eeae


总结
========

Docker 最基本常用的命令就这些：

- ``docker run`` 启动容器；
- ``docker ps`` 查看容器状态；
- ``docker stop`` 停止容器；
- ``docker rm`` 删除容器。

有了 Docker，你可以通过 ``docker search`` 查询镜像仓库是否已经有人做好相关镜像，如果有则直接 ``docker run`` 创建容器使用，否则需要自己构建镜像。作为入门篇，这里就不多说了。

Docker 最直观的好处，就是可以无痛快速尝试新的技术，这些往往别人已经做好镜像配置好环境（对于学习这些环境足够了），初学者不必自己去处理繁琐的运维配置。而且启动容器的代价非常小，资源隔离保证主机安全下满足好奇心，折腾坏了直接删除新建一个就可以。


