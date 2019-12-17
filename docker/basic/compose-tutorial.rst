.. _compose-tutorial:

========================
Compose 入门指南
========================



`Compose`_ is a tool for defining and running multi-container Docker applications.

.. _Compose: https://docs.docker.com/compose/

Docker Compose 是 Docker 的编配工具，用于完成自动配置、协作和管理服务等过程。

Compose 安装
========================

因为 docker-compose 使用 Python 编写，所以可以使用 pip 直接安装：

::

    $ pip install -U docker-compose

.. tip::

    ``docker-compose`` 命令较长，可以使用 alias 指定别名：

    ::

        # .bashrc
        alias dc="docker-compose"


使用 ``docker-compose -h`` 可以查看命令帮助，常用命令：

- ``build``
- ``logs``
- ``ps``
- ``rm``
- ``stop``
- ``up``








运行 Compose
================

这里同时启动 8 个 spider 服务：

::

    $ docker-compose up -d --scale spider=8


Compose 命令
=======================

使用命令 ``docker-compose --help`` 可查看命令工具支持的命令和参数，``docker-compose help [COMMAND]`` 查看命令详情。

::

    Define and run multi-container applications with Docker.

    Usage:
      docker-compose [-f <arg>...] [options] [COMMAND] [ARGS...]
      docker-compose -h|--help

    Options:
      -f, --file FILE            Specify an alternate Compose file (default: docker-compose.yml)
      -p, --project-name NAME    Specify an alternate project name (default: directory name)
      --verbose                  Show more output
      --no-ansi                  Do not print ANSI control characters
      -v, --version              Print version and exit
      -H, --host HOST            Daemon socket to connect to

      --tls                      Use TLS; implied by --tlsverify
      --tlscacert CA_PATH        Trust certs signed only by this CA
      --tlscert CLIENT_CERT_PATH  Path to TLS certificate file
      --tlskey TLS_KEY_PATH      Path to TLS key file
      --tlsverify                Use TLS and verify the remote
      --skip-hostname-check      Don't check the daemon's hostname against the name specified
                                  in the client certificate (for example if your docker host
                                  is an IP address)
      --project-directory PATH    Specify an alternate working directory
                                  (default: the path of the Compose file)

    Commands:
      build              Build or rebuild services
      bundle            Generate a Docker bundle from the Compose file
      config            Validate and view the Compose file
      create            Create services
      down              Stop and remove containers, networks, images, and volumes
      events            Receive real time events from containers
      exec              Execute a command in a running container
      help              Get help on a command
      images            List images
      kill              Kill containers
      logs              View output from containers
      pause              Pause services
      port              Print the public port for a port binding
      ps                List containers
      pull              Pull service images
      push              Push service images
      restart            Restart services
      rm                Remove stopped containers
      run                Run a one-off command
      scale              Set number of containers for a service
      start              Start services
      stop              Stop services
      top                Display the running processes
      unpause            Unpause services
      up                Create and start containers
      version            Show the Docker-Compose version information

其中，最常用的命令为：

build
----------

``build`` 根据 docker-compose 文件指定的镜像和选项构建镜像，如果事先使用 Dockerfile 编译好镜像则无需 ``build``，直接使用 ``up`` 启动。

::

    $ docker-compose build

up
------

类似 Docker，Compose 使用 ``up`` 启动容器，使用制定的参数来执行，并将所有的日志输出合并到一起。

::

    $ docker-compose up

如果启动时指定 ``-d`` 标志，则以守护进程模式运行服务。

::

    $ docker-compose up -d

如果要批量启动服务（如启动 8 个 Scrapy），则在 ``--scale`` 选项指定服务的个数：

::

    $ docker-compose up -d --scale spider=8

ps
-----

``ps`` 列出本地 docker-compose.yml 文件里定义的正在运行的所有服务。

::

    $ docker-compose ps

logs
-------

``logs`` 查看服务的日志，这个命令会追踪服务的日志文件，类似 ``tail -f`` 命令，使用 ``Ctrl+C`` 退出。

::

    $ docker-compose logs

stop
-------

``stop`` 停止所有服务，如果服务没有停止，可以使用 ``docker-compose kill`` 强制杀死服务。

::

    $ docker-compose stop

rm
------

``rm`` 删除所有服务。

::

    $ docker-compose rm


一个例子
================

最后，以一个官方 docker-compose.yml 结束：

::

    version: "3"
    services:

      redis:
        image: redis:alpine
        ports:
          - "6379"
        networks:
          - frontend
        deploy:
          replicas: 2
          update_config:
            parallelism: 2
            delay: 10s
          restart_policy:
            condition: on-failure
      db:
        image: postgres:9.4
        volumes:
          - db-data:/var/lib/postgresql/data
        networks:
          - backend
        deploy:
          placement:
            constraints: [node.role == manager]
      vote:
        image: dockersamples/examplevotingapp_vote:before
        ports:
          - 5000:80
        networks:
          - frontend
        depends_on:
          - redis
        deploy:
          replicas: 2
          update_config:
            parallelism: 2
          restart_policy:
            condition: on-failure
      result:
        image: dockersamples/examplevotingapp_result:before
        ports:
          - 5001:80
        networks:
          - backend
        depends_on:
          - db
        deploy:
          replicas: 1
          update_config:
            parallelism: 2
            delay: 10s
          restart_policy:
            condition: on-failure

      worker:
        image: dockersamples/examplevotingapp_worker
        networks:
          - frontend
          - backend
        deploy:
          mode: replicated
          replicas: 1
          labels: [APP=VOTING]
          restart_policy:
            condition: on-failure
            delay: 10s
            max_attempts: 3
            window: 120s
          placement:
            constraints: [node.role == manager]

      visualizer:
        image: dockersamples/visualizer:stable
        ports:
          - "8080:8080"
        stop_grace_period: 1m30s
        volumes:
          - "/var/run/docker.sock:/var/run/docker.sock"
        deploy:
          placement:
            constraints: [node.role == manager]

    networks:
      frontend:
      backend:

    volumes:
      db-data:
