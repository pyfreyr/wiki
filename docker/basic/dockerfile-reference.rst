.. _dockerfile-reference:

========================
Dockerfile 参考手册
========================


第一个 Dockerfile
=====================

创建一个名为 web 的目录并在里面创建初始的 Dockerfile，这个目录就是我们的构建环境（build environment），Docker 称此环境为**上下文**（context）。Docker 会在构建镜像时将上下文和其中的文件及目录上传到 Docker 守护进程，这样守护进程就能直接访问用户想在镜像中存储的任何代码、文件或数据。

.. tip::

    如果在构建镜像时不想把某些目录或文件（如数据目录）上传到 docker daemon，可以添加进 ``.dockerignore`` 进行忽略。具体语法参见 `.dockerignore file <https://docs.docker.com/engine/reference/builder/#dockerignore-file>`_ 。

::

    FROM ubuntu:16.04
    RUN apt-get update && apt-get install -y nginx
    RUN echo 'Hi, I am in your container' > /usr/share/nginx/html/index.html
    EXPOSE 80

Dockerfile 由一系列指令和参数组成，指令会按顺序从上到下执行（所以需要合理安排指令的顺序）。

每条指令都会创建一个新的镜像层并提交，大致执行流程：

1. 从基础镜像运行一个容器
2. 执行一条指令，并对容器做出修改
3. 提交新的镜像层
4. 再基于刚提交的镜像运行一个新的容器
5. 执行下一条指令，直到所有指令执行完毕

可以看出，如果 Dockerfile 由于某些原因（如某条指令失败）没有正常结束，也能得到一个可以使用的镜像，这对调试非常有帮助。

使用 Dockerfile 构建新镜像
------------------------------------

在上下文环境中（这里即 web 目录内）执行 ``docker build`` ：

::

    $ docker build -t "datascience/web" .

其中，``-t`` 选项指定了新镜像的仓库和镜像名，方便追踪和管理。此外，可以同时为镜像设置标签（不指定默认为 ``latest`` ），如：

::

    $ docker build -t "datascience/web:v1" .

.. attention::

    上面构建命令的最后有个 ``.``，用于告诉 Docker 到本地目录中去找 Dockerfile。也可以通过 ``-f`` 选项指定 Dockerfile 位置（文件名可以不为 Dockerfile），如：

    ::

        $ docker build -t "datascience/web:v1"  -f path/to/dockerfile

.. tip::

    由于每一步构建过程都将结果提交为镜像，所以可将之前的镜像层看做缓存。如本例如果新的应用只修改了 Dockerfile 的第 4 步，则构建时直接从第 4 步开始。需要忽略缓存构建可使用选项 ``--no-cache``：

    ::

        $ docker build --no-cache -t "datascience/web" .

    以上的镜像名称的引号可以省略。

查看新镜像
-----------------

::

    $ docker images

如果想了解镜像构建的流程，可使用 ``docker history``，可以看到镜像的每一层和创建该层的指令。

从新镜像启动容器
-------------------------

::

    $ docker run -d -p 80:80 --name web datascience/web nginx -g "daemon off;"

``docker run`` 的使用方法参见：:ref:`Docker 入门指南 <docker-tutorial>` 。



下面介绍 Dockerfile 指令：

FROM
==========

::

    syntax:

        FROM <image>[:<tag>] [AS <name>]



每个 Dockerfile 的第一条指令必须是 `FROM <https://docs.docker.com/engine/reference/builder/#from>`_ ，用于为后续指令指定基础镜像（base image）。Dockerfile 中可以包含多条 ``FROM`` （多层级构建），这样会在构建时产出多个镜像，或者使其中一些作为其他构建的依赖。

参数 ``tag`` 和 ``name`` 是可选的，不指定 ``tag`` 默认使用 ``latest``；``name`` 在多层级构建时会有用。



RUN
==========

::

    syntax:

        RUN <command>                             # shell form, the command is run in a shell,
                                                  # default is `/bin/sh -c`
        RUN ["executable", "param1", "param2"]    # exec form, using `"`



`RUN <https://docs.docker.com/engine/reference/builder/#run>`_ 执行命令并创建新的镜像层，经常用于安装软件包。每条 RUN 指令都会创建一个新的镜像层。

在 shell 模式下，默认使用 sh，也可以自己指定：

::

    RUN /bin/bash -c 'echo hello'

同样的操作，在 exec 模式下为：

::

    RUN ["/bin/bash", "-c", "echo hello"]

EXPOSE
==========

::

    syntax:

        EXPOSE <port> [<port>/<protocol>...]

`EXPOSE <https://docs.docker.com/engine/reference/builder/#expose>`_ 通知 Docker 该容器运行时监听的网络端口，可以设置 TCP 或 UDP 协议，默认使用 TCP。

::

    EXPOSE 80/tcp
    EXPOSE 80/udp

``EXPOSE`` 是镜像构建者和使用者之间关于端口映射的桥梁，实际使用时需要使用 ``-p`` 选项映射宿主机与容器对应端口：

::

    docker run -p 80:80/tcp -p 80:80/udp ...


CMD
=============

::

    syntax:

        CMD command param1 param2               # shell form
        CMD ["executable","param1","param2"]    # exec form
        CMD ["param1","param2"]                 # as default params to ENTRYPOINT


`CMD <https://docs.docker.com/engine/reference/builder/#cmd>`_ 用于指定一个容器启动时要运行的命令或参数，，类似于 ``RUN``，区别在于：

- ``RUN`` 是指定镜像构建时要运行的命令；
- ``CMD`` 指定容器启动时要运行的命令或参数。

.. attention::

    1. 使用 ``docker run`` 命令可以覆盖 ``CMD`` 指令。

    2. 在 Dockerfile 中只能指定一条 ``CMD`` 指令，多条只有最后一条生效。


LABEL
=========

::

    syntax:

        LABEL <key>=<value> <key>=<value> <key>=<value> ...

`LABEL <https://docs.docker.com/engine/reference/builder/#label>`_ 用于为镜像添加元数据，元数据以键值对展现。

::

    LABEL "com.example.vendor"="ACME Incorporated"
    LABEL com.example.label-with-value="foo"
    LABEL version="1.0"
    LABEL description="This text illustrates \
    that label-values can span multiple lines."



.. note::

    1. 标签会从基础镜像中继承，如果更新了同名变量值，则覆盖父镜像。

    2. ``docker inspect`` 可以查看镜像中的标签信息。

    3. ``MAINTAINER`` 指令已经废弃，改为使用标签：

        ::

            LABEL maintainer="SvenDowideit"


ENV
========

::

    syntax:

        ENV <key> <value>
        ENV <key>=<value> ...

`ENV <https://docs.docker.com/engine/reference/builder/#env>`_ 用于在镜像构建过程中设置环境变量，环境变量被持久保存在镜像创建的任何容器中，可以在后续指令中使用。

::

    ENV TARGET_DIR /opt/app
    ENV RVM_PATH=/home/rvm RVM_ARCHFLAGS="-arch_i486"
    WORKDIR $TARGET_DIR

.. note::

    ``docker run`` 选项 ``--env/-e`` 可以运行时传递环境变量，但只会在运行时有效。


ADD
========

::

    syntax:

        ADD [--chown=<user>:<group>] <src>... <dest>
        ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]    # if path contains whitespace

`ADD <https://docs.docker.com/engine/reference/builder/#add>`_ 用来将构建环境下的文件和目录添加到镜像中。


``<src>`` 支持通配符筛选文件：

::

    ADD hom* /mydir/
    ADD hom?.txt /mydir/


新添加的文件或目录默认使用 UID=0, GID=0（即 root），可以使用 ``--chown`` 指定用户和用户组（username/groupname 或 UID/GID，使用名称时必须确保已存在于宿主机的 ``/etc/passwd`` 和 ``/etc/group``，UID/GID 则无此要求）：

::

    ADD --chown=55:mygroup files* /somedir/
    ADD --chown=bin files* /somedir/
    ADD --chown=1 files* /somedir/
    ADD --chown=10:11 files* /somedir/

.. note::

    1. ``<src>`` 必须位于上下文（指构建目录本身，但内部目录可以）之中；
    2. ``<src>`` 为目录时，目录下所有内容将被复制，而不包含目录本身；
    3. ``<src>`` 为本地压缩文件时，``ADD`` 会自动解压 gzip、bzip2、xz；
    4. ``<src>`` 也可以为网络资源，若为压缩文件不会被解压；
    5. ``<dest>`` 可以是相对路径（相对 ``WORKDIR`` ）或绝对路径；
    6. 如果 ``<src>`` 为多个文件，要求 ``<dest>`` 为目录（以 ``/`` 结尾）；
    7. 如果 ``<dest>`` 不存在，会自动创建缺失的目录。


COPY
============

::

    syntax:

        COPY [--chown=<user>:<group>] <src>... <dest>
        COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]    # if path contains whitespace

`COPY <https://docs.docker.com/engine/reference/builder/#copy>`_ 类似 ``ADD`` （相应规则参考 ``ADD`` ），但不会对文件解压。同样如果目的位置不存在会自动创建。

::

    COPY conf.d/ /etc/apache2/


ENTRYPOINT
================

::

    syntax:

        ENTRYPOINT ["executable", "param1", "param2"]    # exec form, preferred
        ENTRYPOINT command param1 param2                 # shell form

`ENTRYPOINT <https://docs.docker.com/engine/reference/builder/#entrypoint>`_ 用于执行指令，和 ``CMD`` 非常类似，但 ``ENTRYPOINT`` 提供的命令不会在容器启动时被覆盖！实际上，``docker run`` 指定的任何参数都会被当做参数传递给 ``ENTRYPOINT`` 指令中指定的命令。

::

    ENTRYPOINT ["/usr/sbin/nginx", "-g", "daemon off;"]

可以组合使用 ``ENTRYPOINT`` 和 ``CMD`` 来完成一些巧妙的工作（如果启动容器时没有指定参数，则在 ``CMD`` 中指定的 ``-h`` 会被传递给 ``Nginx``）：

::

    ENTRYPOINT ["/usr/sbin/nginx"]
    CMD ["-h"]

.. note::

    ``docker run`` 选项 ``--entrypoint`` 可以覆盖 ``ENTRYPOINT`` 指令。


VOLUME
============

::

    syntax:

        VOLUME ["dir1", "dir2", ...]

`VOLUME <https://docs.docker.com/engine/reference/builder/#volume>`_ 用来向基于镜像创建的容器添加卷（挂载点）。一个卷可以存在于一个或多个容器内特定的目录，这个目录可以绕过联合文件系统，用于数据共享或数据持久化。

卷功能让我们可以将数据（如源码）、数据库或者其他内容挂载到镜像中，而不是提交到镜像中，并且运行在多个容器间共享这些内容。


.. note::

    ``docker run`` 选项 ``-v`` 用于在运行时挂载本地目录到容器内挂载点。

    ``docker cp`` 允许从容器复制文件和复制文件到容器上。


USER
========

::

    syntax:

        USER <user>[:<group>]
        USER <UID>[:<GID>]

`USER <https://docs.docker.com/engine/reference/builder/#user>`_ 用于指定镜像以什么用户去运行，如果不指定默认为 root:root。

::

    USER user
    USER user:group
    USER uid
    USER uid:gid
    USER user:gid
    USER uid:group

.. note::

    ``docker run`` 选项 ``-u`` 可在运行时覆盖 ``USER`` 。



WORKDIR
===========

`WORKDIR <https://docs.docker.com/engine/reference/builder/#workdir>`_ 创建新容器时，在容器内部设置一个工作目录，``RUN``, ``CMD``, ``ENTRYPOINT``, ``COPY`` 和 ``ADD`` 指令会在这个目录下执行。

::

    WORKDIR /opt/webapp/db
    RUN bundle install
    WORKDIR /opt/webapp
    ENTRYPOINT ["rackuo"]

.. note::

    1. ``WORKDIR`` 可以使用相对路径或绝对路径；
    2. 路径不存在时自动创建；
    3. ``WORKDIR`` 可以使用 ``ENV`` 指定的环境变量；
    4. ``docker run`` 选项 ``-w`` 可在运行时覆盖工作目录。


ARG
===========

::

    syntax:

        ARG <name>[=<default value>]


`ARG <https://docs.docker.com/engine/reference/builder/#arg>`_ 用来定义可以在 ``docker build`` 运行时传递的变量。只需要在构建时使用 ``--build-arg``。当指定 Dockerfile 中未定义过的参数时，会输出警告（未定义说明在 Dockerfile 中不会使用）。

::

    ARG user
    ARG buildno=1

    $ docker build --build-arg user=someuser -t datascience/web .


``ARG`` 变量的作用域从声明开始：

- 示例一

    ::

        FROM busybox
        USER ${user:-some_user}
        ARG user
        USER $user

        $ docker build --build-arg user=aaa .

    ``user`` 的值分别为：``some_user``, ``aaa``, ``aaa``。

- 示例二

    ::

        FROM ubuntu
        ARG CONT_IMG_VER
        ENV CONT_IMG_VER v1.0.0
        RUN echo $CONT_IMG_VER

        $ docker build --build-arg CONT_IMG_VER=v2.0.1 .

    ``CONT_IMG_VER`` 值分别为 ``v2.0.1``, ``v1.0.0``, ``v1.0.0``。


``ARG`` 只在当前构建层级（前一个 ``FROM`` 和后一个 ``FROM`` 之间，服务于前一个 ``FROM``）生效，如果多级构建都需要变量则需要分别设置：

::

    FROM busybox
    ARG SETTINGS
    RUN ./run/setup $SETTINGS

    FROM busybox
    ARG SETTINGS
    RUN ./run/other $SETTINGS

.. note::

    Docker 预定义的 ``ARG`` 变量：

    - ``HTTP_PROXY``
    - ``http_proxy``
    - ``HTTPS_PROXY``
    - ``https_proxy``
    - ``FTP_PROXY``
    - ``ftp_proxy``
    - ``NO_PROXY``
    - ``no_proxy``

STOPSIGNAL
================

::

    syntax:

        STOPSIGNAL signal

`STOPSIGNAL <https://docs.docker.com/engine/reference/builder/#stopsignal>`_ 用来设置停止容器时发送什么系统调用信号给容器，如 ``SIGKILL`` 。

ONBUILD
==============

::

    syntax:

        ONBUILD [INSTRUCTION]

`ONBUILD <https://docs.docker.com/engine/reference/builder/#onbuild>`_ 为镜像添加触发器，广泛用在制作基础镜像：用构建的镜像创建其他镜像时，才会触发 ``ONBUILD`` 的命令。

当我们在一个 Dockerfile 文件中加上 ``ONBUILD`` 指令，该指令对利用该 Dockerfile 构建镜像（比如 image A）不会产生实质性影响。

但是当我们编写一个新的 Dockerfile 文件来基于A镜像构建一个镜像（比如 image B）时，这时构造 A 镜像的 Dockerfile 文件中的 ``ONBUILD`` 指令就生效了，在构建 B 镜像的过程中，首先会执行 ``ONBUILD`` 指定的指令，然后才会执行其它指令（如 ``FROM``）。

::

    ...
    ONBUILD ADD . /app/src
    ONBUILD RUN /usr/local/bin/python-build --dir /app/src
    ...

.. warning::

    1. ``ONBUILD`` 指定的指令不能是 ``ONBUILD`` 和 ``FROM``；
    2. 触发器无法被继承，imageA -> imageB -> imageC 时，C 不会执行 A 中的触发器。


HEALTHCHECK
================

::

    syntax:

        HEALTHCHECK [OPTIONS] CMD command    # check health by running a command inside the container
        HEALTHCHECK NONE                     # disable any healthcheck inherited from the base image

`HEALTHCHECK <https://docs.docker.com/engine/reference/builder/#healthcheck>`_ 指令声明了健康检测命令，用这个命令来判断容器主进程的服务状态是否正常，从而比较真实的反应容器实际状态。

在 ``CMD`` 之前可选项有：

- ``--interval=DURATION`` (default: 30s)，两次健康检查的间隔；
- ``--timeout=DURATION`` (default: 30s)，健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败；
- ``--start-period=DURATION`` (default: 0s)，应用启动的初始化时间，在启动过程中的健康检查失效不会计入；
- ``--retries=N`` (default: 3)，当连续失败指定次数后，则将容器状态视为 unhealthy。


在 HEALTHCHECK [选项] CMD 后面命令的返回值决定了该次健康检查的成功与否：

::

    HEALTHCHECK --interval=5m --timeout=3s \
      CMD curl -f http://localhost/ || exit 1

容器启动之后，初始状态会为 starting (启动中)。Docker Engine会等待 interval 时间，开始执行健康检查命令，并周期性执行。如果单次检查返回值非0或者运行需要比指定 timeout 时间还长，则本次检查被认为失败。如果健康检查连续失败超过了 retries 重试次数，状态就会变为 unhealthy (不健康)。

.. note::

    1. 在 Dockerfile 中 ``HEALTHCHECK`` 只可以出现一次，如果写了多个，只有最后一个生效。
    2. 一旦有一次健康检查成功，Docker会将容器置回 healthy (健康)状态
    3. 当容器的健康状态发生变化时，Docker Engine会发出一个 health_status 事件。

SHELL
========

::

    syntax:

        SHELL ["executable", "parameters"]

`SHELL <https://docs.docker.com/engine/reference/builder/#shell>`_ 用于设置默认 shell。









