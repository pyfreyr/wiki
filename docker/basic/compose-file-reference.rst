.. _compose-file-reference:

============================
Compose file 参考手册
============================




.. note::

    详细 Compose file 选项参见：`Compose file version 3 reference <https://docs.docker.com/compose/compose-file/>`_ 。




第一个 Compose file
=======================

::

    version: '3'
    services:
      spider:
        build:
          context: .
          dockerfile: Dockerfile
        volumes:
          - $PWD:/code
          - /data1/datascience/scrapy-data:/data
        command: scrapy crawl comment

说明：

- ``version`` Compose file 的版本，最新的版本为 3.x；
- ``services`` 定义服务，这里定义了一个爬虫服务 spider；
- ``build`` 构建镜像上下文、Dockerfile 文件和 ARG 等；
- ``volumes`` 用于创建卷并挂载，这里挂载了源码目录和数据存储目录；
- ``command`` 服务启动时执行的命令。


Compose file 通过 YAML 文件定义服务（services），网络（networks）和卷（volumes）：

::

    version: "3"
    services:
      web:
        # replace username/repo:tag with your name and image details
        image: username/repo:tag
        deploy:
          replicas: 5
          restart_policy:
            condition: on-failure
          resources:
            limits:
              cpus: "0.1"
              memory: 50M
        ports:
          - "80:80"
        networks:
          - webnet
      visualizer:
        image: dockersamples/visualizer:stable
        ports:
          - "8080:8080"
        volumes:
          - "/var/run/docker.sock:/var/run/docker.sock"
        deploy:
          placement:
            constraints: [node.role == manager]
        networks:
          - webnet
      redis:
        image: redis
        ports:
          - "6379:6379"
        volumes:
          - "/home/docker/data:/data"
        deploy:
          placement:
            constraints: [node.role == manager]
        command: redis-server --appendonly yes
        networks:
          - webnet
    networks:
      webnet:

services
===============

每个 service 定义了本服务启动时传递给容器的配置信息，类似于命令行操作 ``docker container create``。

同 ``docker container create`` 一样，已经在 Dockerfile 中指定的选项（如 ``CMD``, ``EXPOST``, ``VOLUME``, ``ENV``）不需要重复设置，会作为默认值使用。

build
------------

构建镜像时配置选项。

服务除了可以基于指定的镜像，还可以基于一份 Dockerfile，在使用 ``up`` 启动之时执行构建任务，这个构建标签就是 ``build``，它可以指定 Dockerfile 所在文件夹的路径。Compose 将会利用它自动构建这个镜像，然后使用这个镜像启动服务容器。

::

    version: '3'
    services:
      webapp:
        build: ./dir

如果需要更细粒度的配置，需要使用 ``context``, ``dockerfile``, ``args``, ``labels`` 等选项。

::

    version: '3'
    services:
      webapp:
        build:
          context: ./dir
          dockerfile: Dockerfile-alternate
          args:
            buildno: 1

在执行镜像构建时，Compose 通过 ``image`` 指定构建出的镜像名称：

::

    build: ./dir
    image: webapp:tag

.. note::

    使用 Swarm 部署服务时会忽略 ``build`` 选项，``docker stack`` 只接受预先构建好的镜像。


context
~~~~~~~~~~~~

构建上下文，可以是本地目录或者 git 仓库 URL。

::

    build:
      context: ./dir


dockerfile
~~~~~~~~~~~~~

使用此 Dockerfile 文件来构建，必须指定构建路径。

::

    build:
      context: .
      dockerfile: Dockerfile-alternate

args
~~~~~~~

添加构建参数，这些参数是仅在构建过程中可访问的环境变量。

首先， 在 Dockerfile 中指定参数：

::

    ARG buildno
    ARG gitcommithash

    RUN echo "Build number: $buildno"
    RUN echo "Based on commit: $gitcommithash"

然后指定 ``build`` 下的参数，可以传递映射或列表：

::

    build:
      context: .
      args:
        buildno: 1
        gitcommithash: cdc3b19

也可以使用 Compose 构建时手动传入，此时不用设置默认值：

::

    args:
      - buildno
      - gitcommithash

.. note::

    YAML 的布尔值（``true/false``, ``yes/no``, ``on/off``）必须使用引号括起来才能准确解析为字符串。


cache_from
~~~~~~~~~~~~~

用于指定缓存解析镜像列表。

::

    build:
      context: .
      cache_from:
        - alpine:latest
        - corp/web_app:3.14

.. _sub_labels:

labels
~~~~~~~~~~~~

使用 Docker 标签将元数据添加到生成的镜像中，可以使用数组或字典。建议使用反向 DNS 标记来防止签名与其他软件所使用的签名冲突。

::

    build:
      context: .
      labels:
        com.example.description: "Accounting webapp"
        com.example.department: "Finance"
        com.example.label-with-empty-value: ""

    build:
      context: .
      labels:
        - "com.example.description=Accounting webapp"
        - "com.example.department=Finance"
        - "com.example.label-with-empty-value"


shm_size
~~~~~~~~~~~~~

用于设置 ``/dev/shm`` 分区大小，值为表示字节的整数值或表示字符的字符串，使用数值时单位 byte。

::

    build:
      context: .
      shm_size: '2gb'


    build:
      context: .
      shm_size: 10000000

进一步了解 `/dev/shm <https://www.jb51.net/article/105946.htm>`_ 。


target
~~~~~~~~~~

在多层级镜像构建时，用于构建指定镜像：

::

    # Dockerfile
    FROM golang:1.7.3 as builder
    WORKDIR /go/src/github.com/alexellis/href-counter/
    RUN go get -d -v golang.org/x/net/html
    COPY app.go    .
    RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

    FROM alpine:latest
    RUN apk --no-cache add ca-certificates
    WORKDIR /root/
    COPY --from=builder /go/src/github.com/alexellis/href-counter/app .
    CMD ["./app"]

    # docker-compose.yml
    build:
      context: .
      target: builder


cap_add, cap_drop
----------------------

增加或删除容器系统功能。默认情况下，docker 的容器中的 root 的权限是有严格限制的，比如，网络管理（NET_ADMIN）等很多权限都是没有的。

::

    cap_add:
      - ALL

    cap_drop:
      - NET_ADMIN
      - SYS_ADMIN

进一步阅读 `capabilities <http://man7.org/linux/man-pages/man7/capabilities.7.html>`_ 。

.. note::

    ``cap_add`` 和 ``cap_drop`` 在 Swarm 部署时被忽略。


command
-------------

覆盖容器启动后默认执行的命令：

::

    command: bundle exec thin -p 3000

    command: ["bundle", "exec", "thin", "-p", "3000"]


config
------------

使用服务 `configs <https://docs.docker.com/compose/compose-file/#configs>`_ 配置为每个服务赋予相应的访问权限，支持两种不同的语法。

.. note::

    配置必须存在或在 configs 此堆栈文件的顶层中定义，否则堆栈部署失效。

SHORT syntax
~~~~~~~~~~~~~~~~~

SHORT 语法只能指定配置名称，这允许容器访问配置并将其安装在 ``/<config_name>`` 容器内，源名称和目标装入点都设为配置名称。

::

    version: "3.3"
    services:
      redis:
        image: redis:latest
        deploy:
          replicas: 1
        configs:
          - my_config
          - my_other_config
    configs:
      my_config:
        file: ./my_config.txt
      my_other_config:
        external: true

以上实例使用 SHORT 语法将 redis 服务访问授予 ``my_config`` 和 ``my_other_config`` ，并被 ``my_other_config`` 定义为外部资源，这意味着它已经在 Docker 中定义。可以通过 ``docker config create`` 命令或通过另一个堆栈部署。如果外部部署配置都不存在，则堆栈部署会失败并出现 ``config not found`` 错误。



LONG syntax
~~~~~~~~~~~~~~~~~~

LONG 语法提供了创建服务配置的更加详细的信息

- ``source``：Docker 中存在的配置的名称
- ``target``：要在服务的任务中装载的文件的路径或名称。如果未指定则默认为 ``/<source>``
- ``uid`` 和 ``gid``：在服务的任务容器中拥有安装的配置文件的数字 UID 或 GID。如果未指定，则默认为在 Linux 上，Windows 不支持。
- ``mode``：在服务的任务容器中安装的文件的权限，以八进制表示法。例如，``0444`` 代表文件可读的。默认是 ``0444``。如果配置文件无法写入，是因为它们安装在临时文件系统中，所以如果设置了可写位，它将被忽略。可执行位可以设置。

下面示例在容器中将 ``my_config`` 名称设置为 ``redis_config``，将模式设置为 ``0440``（group-readable）并将用户和组设置为 ``103``。该 ``redis`` 服务无法访问 ``my_other_config`` 配置。

::

    version: "3.3"
    services:
      redis:
        image: redis:latest
        deploy:
          replicas: 1
        configs:
          - source: my_config
            target: /redis_config
            uid: '103'
            gid: '103'
            mode: 0440
    configs:
      my_config:
        file: ./my_config.txt
      my_other_config:
        external: true

可以同时授予多个配置的服务相应的访问权限，也可以混合使用 LONG 和 SHORT 语法。定义配置并不意味着授予服务访问权限。


cgroup_parent
------------------

可以为容器选择一个可选的父 cgroup。

::

    cgroup_parent: m-executor-abcd


.. note::

    ``cgroup_parent`` 在 Swarm 模式下无效。


container_name
---------------

为自定义的容器指定一个名称，而不是使用默认的名称。因为 docker 容器名称必须是唯一的，所以如果指定了一个自定义的名称，不能扩展一个服务超过 1 个容器。

::

    container_name: my-web-container

.. note::

    ``container_name`` 在 Swarm 模式中无效。


deploy
-------------

指定与部署和运行服务相关的配置，只在 Swarm 模式下生效，在 ``docker-compose up/run`` 时无效。

::

    version: '3'
    services:
      redis:
        image: redis:alpine
        deploy:
          replicas: 6
          update_config:
            parallelism: 2
            delay: 10s
          restart_policy:
            condition: on-failure

endpoint_mode
~~~~~~~~~~~~~~~~~

指定连接到群组外部客户端服务发现方法，支持两种模式：

- ``endpoint_mode:vip`` （virtual IP），默认模式，Docker 为该服务分配了一个虚拟 IP，作为客户端的 “前端” 部位用于访问网络上的服务；
- ``dnsrr`` （DNS round-robin），DNS 轮询服务发现不使用单个虚拟 IP。Docker 为服务设置 DNS 条目，使得服务名称的 DNS 查询返回一个 IP 地址列表，并且客户端直接连接到其中的一个。如果想使用自己的负载平衡器，或者混合 Windows 和 Linux 应用程序，则 DNS 轮询调度（round-robin）功能就非常实用。

::

    version: "3.3"

    services:
      wordpress:
        image: wordpress
        ports:
          - "8080:80"
        networks:
          - overlay
        deploy:
          mode: replicated
          replicas: 2
          endpoint_mode: vip

      mysql:
        image: mysql
        volumes:
           - db-data:/var/lib/mysql/data
        networks:
           - overlay
        deploy:
          mode: replicated
          replicas: 2
          endpoint_mode: dnsrr

    volumes:
      db-data:

    networks:
      overlay:


labels
~~~~~~~~~~

指定服务的标签，这些标签仅在服务上设置（而非服务的容器上）。

::

    version: "3"
    services:
      web:
        image: web
        deploy:
          labels:
            com.example.description: "This label will appear on the web service"

通过将 deploy 外面的 :ref:`labels <labels>` 标签来设置容器上的 ``labels``：

::

    version: "3"
    services:
      web:
        image: web
        labels:
          com.example.description: "This label will appear on all containers for the web service"

mode
~~~~~~~~~

- ``global``：每个集节点只有一个容器
- ``replicated``：指定容器数量（默认）

::

    version: '3'
    services:
      worker:
        image: dockersamples/examplevotingapp_worker
        deploy:
          mode: global


placement
~~~~~~~~~~

指定 constraints 和 preferences，参考 `docker service create <https://docs.docker.com/engine/reference/commandline/service_create/>`_ :

::

    version: '3.3'
    services:
      db:
        image: postgres
        deploy:
          placement:
            constraints:
              - node.role == manager
              - engine.labels.operatingsystem == ubuntu 14.04
            preferences:
              - spread: node.labels.zone


replicas
~~~~~~~~~~~~

如果服务是 ``replicated`` （默认)，需要指定运行的容器数量：

::

    version: '3'
    services:
      worker:
        image: dockersamples/examplevotingapp_worker
        networks:
          - frontend
          - backend
        deploy:
          mode: replicated
          replicas: 6

resources
~~~~~~~~~~~~~

配置资源限制，类似 `docker service create <https://docs.docker.com/engine/reference/commandline/service_create/>`_ 。

::

    version: '3'
    services:
      redis:
        image: redis:alpine
        deploy:
          resources:
            limits:
              cpus: '0.50'
              memory: 50M
            reservations:
              cpus: '0.25'
              memory: 20M

此例子中，redis 服务限制使用不超过 50M 的内存和 0.50（50％）可用处理时间（CPU），并且 保留 20M 了内存和 0.25 CPU 时间


.. _restart_policy:

restart_policy
~~~~~~~~~~~~~~~~~

配置容器的重新启动，代替 ``restart``。

- ``condition``：值可以为 ``none`` 、``on-failure`` 以及 ``any`` (默认)
- ``delay``：尝试重启的等待时间，默认为 0
- ``max_attempts``：在放弃之前尝试重新启动容器次数（默认：从不放弃）。如果 max_attempts 值为 2，并且第一次尝试重新启动失败，则可能会尝试重新启动两次以上。
- ``windows``：在决定重新启动是否成功之前的等时间，指定为持续时间（默认值：立即决定）。

::

    version: "3"
    services:
      redis:
        image: redis:alpine
        deploy:
          restart_policy:
            condition: on-failure
            delay: 5s
            max_attempts: 3
            window: 120s

.. _update_config:

update_config
~~~~~~~~~~~~~~~~

配置更新服务，用于无缝更新应用（rolling update)。

- ``parallelism``：一次性更新的容器数量，如果设置为 0，所有容器同时操作。
- ``delay``：更新一组容器之间的等待时间，默认 0s。
- ``failure_action``：如果更新失败，可以执行的的是 ``continue``、``rollback`` 或 ``pause`` （默认）
- ``monitor``：每次任务更新后监视失败的时间( ``ns|us|ms|s|m|h`` )（默认为0）
- ``max_failure_ratio``：在更新期间能接受的失败率
- ``order``：更新次序设置，``top-first`` （旧的任务在开始新任务之前停止）、``start-first`` （新的任务首先启动，并且正在运行的任务短暂重叠）（默认 ``stop-first``）


rollback_config
~~~~~~~~~~~~~~~~~~~

服务升级失败时回滚配置。选项参考 :ref:`update_config <update_config>` 。

.. note::

    不支持 ``docker stack deploy`` 的子选项（支持 ``docker-compose up/run`` ）：

    - ``build``
    - ``cgroup_parent``
    - ``container_name``
    - ``devices``
    - ``tmpfs``
    - ``external_links``
    - ``links``
    - ``network_mode``
    - ``restart``
    - ``security_opt``
    - ``stop_signal``
    - ``sysctls``
    - ``userns_mode``





device
----------

设置映射列表，与 Docker 客户端的 ``--device`` 参数类似 :

::

    devices:
      - "/dev/ttyUSB0:/dev/ttyUSB0"

.. note::

    ``device`` 在 Swarm 模式无效。


depends_on
----------------

此选项解决了启动顺序的问题。

在使用 Compose 时，最大的好处就是少打启动命令，但是一般项目容器启动的顺序是有要求的，如果直接从上到下启动容器，必然会因为容器依赖问题而启动失败。例如在没启动数据库容器的时候启动了应用容器，这时候应用容器会因为找不到数据库而退出，为了避免这种情况我们需要加入一个标签，就是 ``depends_on``，这个标签解决了容器的依赖、启动先后的问题。

指定服务之间的依赖关系，有两种效果：

- ``docker-compose up`` 以依赖顺序启动服务，下面例子中 ``redis`` 和 ``db`` 服务在 ``web`` 启动前启动
- ``docker-compose up SERVICE`` 自动包含 ``SERVICE`` 的依赖性，下面例子中会先启动 ``redis`` 和 ``db`` 两个服务，最后才启动 ``web`` 服务：

::

    version: '3'
    services:
      web:
        build: .
        depends_on:
          - db
          - redis
      redis:
        image: redis
      db:
        image: postgres

注意的是，默认情况下使用 ``docker-compose up web`` 这样的方式启动 ``web`` 服务时，也会启动 ``redis`` 和 ``db`` 两个服务，因为在配置文件中定义了依赖关系。


.. note::

    ``depends_on`` 在 Swarm 模式无效。

dns
---------

自定义 DNS 服务器，与 ``--dns`` 具有一样的用途，可以是单个值或列表。

::

    dns: 8.8.8.8
    dns:
      - 8.8.8.8
      - 9.9.9.9

dns_search
-----------


自定义 DNS 搜索域，可以是单个值或列表。

::

    dns_search: example.com
    dns_search:
      - dc1.example.com
      - dc2.example.com

tmpfs
-------

挂载临时文件目录到容器内部，``size`` 以 bytes 指定大小，默认无限制。

::

    - type: tmpfs
         target: /app
         tmpfs:
           size: 1000

entrypoint
------------

在 Dockerfile 中有一个指令叫做 ``ENTRYPOINT`` 指令，用于指定接入点。在 docker-compose.yml 中可以定义接入点，覆盖 Dockerfile 中的定义：

::

    entrypoint: /code/entrypoint.sh

``entrypoint`` 也可以是一个列表：

::

    entrypoint:
        - php
        - -d
        - zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20100525/xdebug.so
        - -d
        - memory_limit=-1
        - vendor/bin/phpunit

.. note::

    ``entrypoint`` 会覆盖 ``ENTRYPOINT`` 并忽略 ``CMD``。


env_file
------------

从文件中添加环境变量，可以是单个值或是列表。

如果已经用 ``docker-compose -f FILE`` 指定了 Compose 文件，那么 ``env_file`` 路径值为相对于该文件所在的目录。

但 :ref:`environment <environment>` 环境中的设置的变量会会覆盖这些值，无论这些值为空还是未定义。

::

    env_file: .env

    env_file:
      - ./common.env
      - ./apps/web.env
      - /opt/secrets.env

环境配置文件 ``env_file`` 中的声明每行都是以 ``VAR=VAL`` 格式，其中以 ``#`` 开头的被解析为注释而被忽略：

::

    # Set Rails/Rack environment
    RACK_ENV=development

.. note::

    如果在配置文件中有 ``build`` 操作，环境配置文件中的变量并不会进入构建过程中，如果要在构建中使用变量还是首选 ``arg`` 标签。


以下例子中，``$VAR`` 的值为 ``hello``。

::

    services:
      some-service:
        env_file:
          - a.env
          - b.env

    # a.env
    VAR=1

    # b.env
    VAR=hello


.. _environment:

environment
---------------

添加环境变量，可以使用数组或字典。与上面的 ``env_file`` 选项完全不同，反而和 ``arg`` 有几分类似，这个标签的作用是设置镜像变量，它可以保存变量到镜像里面，也就是说启动的容器也会包含这些变量设置，这是与 ``arg`` 最大的不同。

一般 ``arg`` 标签的变量仅用在构建过程中。而 ``environment`` 和 Dockerfile 中的 ``ENV`` 指令一样会把变量一直保存在镜像、容器中，类似 ``docker run -e`` 的效果

::

    environment:
      RACK_ENV: development
      SHOW: 'true'
      SESSION_SECRET:

    environment:
      - RACK_ENV=development
      - SHOW=true
      - SESSION_SECRET

.. note::

    如果在配置文件中有 ``build`` 操作，``environment`` 定义的变量并不会进入构建过程中，如果要在构建中使用变量请使用 ``arg`` 标签。


expose
------------

暴露端口，但不映射到宿主机，只被连接的服务访问。这个标签与 Dockerfile 中的 ``EXPOSE`` 指令一样，用于指定暴露的端口，但是只是作为一种参考，实际上 docker-compose.yml 的端口映射还得 ``ports`` 这样的标签。

::

    expose:
     - "3000"
     - "8000"

external_links
-------------------

链接到 docker-compose.yml 外部的容器，甚至并非 Compose 项目文件管理的容器。参数格式跟 ``links`` 类似。

在使用 Docker 过程中，会有许多单独使用 ``docker run`` 启动的容器的情况，为了使 Compose 能够连接这些不在docker-compose.yml 配置文件中定义的容器，那么就需要一个特殊的标签，就是 ``external_links``，它可以让 Compose 项目里面的容器连接到那些项目配置外部的容器（前提是外部容器中必须至少有一个容器是连接到与项目内的服务的同一个网络里面）。


::

    external_links:
     - redis_1
     - project_db_1:mysql
     - project_db_1:postgresql

.. note::

    1. ``link`` 已经弃用，优先使用 ``networks``；
    2. ``external_links`` 在 Swarm 模式无效。


extra_hosts
-------------

添加主机名的标签，就是往 ``/etc/hosts`` 文件中添加一些记录，与 Docker 客户端 中的 ``--add-host`` 类似：

::

    extra_hosts:
     - "somehost:162.242.195.82"
     - "otherhost:50.31.209.229"

具有 IP 地址和主机名的条目在 ``/etc/hosts`` 内部容器中创建。启动之后查看容器内部 hosts，例如：

::

    162.242.195.82  somehost
    50.31.209.229   otherhost


healthcheck
------------------

用于检查测试服务使用的容器是否正常。

::

    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 1m30s
      timeout: 10s
      retries: 3
      start_period: 40s

``interval``，``timeout`` 以及 ``start_period`` 都定为持续时间。

``test`` 必须是字符串或列表，如果它是一个列表，第一项必须是 ``NONE``，``CMD`` 或 ``CMD-SHELL`` ；如果它是一个字符串，则相当于指定 ``CMD-SHELL`` 后跟该字符串。

::

    # Hit the local web app
    test: ["CMD", "curl", "-f", "http://localhost"]

    # As above, but wrapped in /bin/sh. Both forms below are equivalent.
    test: ["CMD-SHELL", "curl -f http://localhost || exit 1"]
    test: curl -f https://localhost || exit 1

如果需要禁用镜像的所有检查项目，可以使用 ``disable:true`` ,相当于 ``test:["NONE"]`` 。

::

    healthcheck:
      disable: true



image
----------

从指定的镜像中启动容器，可以是存储仓库、标签以及镜像 ID。如果镜像在本地不存在，Compose 将会尝试从官方仓库拉取这个镜像。

::

    image: redis
    image: ubuntu:14.04
    image: tutum/influxdb
    image: example-registry.com:4000/postgresql
    image: a4bc65fd

如果使用 `build` 构建镜像，则 `image` 为构建的镜像名。

init
----------

Run an init inside the container that forwards signals and reaps processes. Either set a boolean value to use the default ``init``, or specify a path to a custom one.

::

    version: '3.7'
    services:
      web:
        image: alpine:latest
        init: true


    version: '2.2'
    services:
      web:
        image: alpine:latest
        init: /usr/libexec/docker-init

isolation
-------------

Specify a container’s isolation technology. On Linux, the only supported value is ``default``.


.. _labels:

labels
----------

使用 Docker 标签将元数据添加到容器，可以使用数组或字典。与 Dockerfile 中的 ``LABELS`` 类似：

::

    labels:
      com.example.description: "Accounting webapp"
      com.example.department: "Finance"
      com.example.label-with-empty-value: ""

    labels:
      - "com.example.description=Accounting webapp"
      - "com.example.department=Finance"
      - "com.example.label-with-empty-value"

links
-------------

.. note::

    未来会弃用，建议使用 :ref:`networks <networks>` 。

链接到其它服务的中的容器，可以指定服务名称也可以指定链接别名（ ``SERVICE：ALIAS`` )，与 Docker 客户端的 ``--link`` 有一样效果，会连接到其它服务中的容器。

::

    web:
      links:
       - db
       - db:database
       - redis

使用的别名将会自动在服务容器中的 ``/etc/hosts`` 里创建，相应的环境变量也将被创建。例如：

::

    172.12.2.186  db
    172.12.2.186  database
    172.12.2.187  redis


logging
------------

配置日志服务。

::

    logging:
      driver: syslog
      options:
        syslog-address: "tcp://192.168.0.42:123"

``driver`` 值是指定服务器的日志记录驱动程序，默认值为 ``json-file``，与 ``--log-diver`` 选项一样：

::

    driver: "json-file"
    driver: "syslog"
    driver: "none"

.. note::

    只有驱动程序 ``json-file`` 和 ``journald`` 驱动程序可以直接从 ``docker-compose up`` 和 ``docker-compose logs`` 获取日志。使用任何其他方式不会显示任何日志。

对于可选值，可以使用 ``options`` 指定日志记录中的日志记录选项：

::

    driver: "syslog"
    options:
      syslog-address: "tcp://192.168.0.42:123"

默认驱动程序 ``json-file`` 具有限制存储日志量的选项，所以，使用键值对来获得最大存储大小以及最小存储数量：

::

    options:
      max-size: "200k"
      max-file: "10"

上面实例将存储日志文件，直到它们达到 ``max-size`` （200kB），存储的单个日志文件的数量由该 ``max-file`` 值指定。随着日志增长超出最大限制，旧日志文件将被删除以存储新日志。

``docker-compose.yml`` 限制日志存储的示例：

::

    services:
      some-service:
        image: some-service
        logging:
          driver: "json-file"
          options:
            max-size: "200k"
            max-file: "10"

.. _network_mode:

network_mode
----------------

网络模式，用法类似于 Docker 客户端的 ``--network`` 选项，格式为：``service:[service name]``。

::

    network_mode: "bridge"
    network_mode: "host"
    network_mode: "none"
    network_mode: "service:[service name]"
    network_mode: "container:[container name/id]"


.. note::

    1. ``network_mode`` 在 Swarm 模式无效。
    2. ``network_mode: "host"`` 不能与 ``links`` 混合使用。


networks
------------

加入指定网络，可选网络来自顶层 :ref:`networks <networks>` 。

::

    services:
      some-service:
        networks:
         - some-network
         - other-network

aliases
~~~~~~~~~~

Aliases（alternative hostnames），同一网络上的其他容器可以使用服务器名称或别名（也是 hostname）来连接到其他服务的容器。


::

    services:
      some-service:
        networks:
          some-network:
            aliases:
             - alias1
             - alias3
          other-network:
            aliases:
             - alias2

下面实例中，提供 ``web`` 、``worker`` 以及 ``db`` 三个服务和两个网络 （ ``new`` 和 ``legacy`` ），相同的服务可以在不同的网络有不同的别名。

::

    version: '2'

    services:
      web:
        build: ./web
        networks:
          - new

      worker:
        build: ./worker
        networks:
          - legacy

      db:
        image: mysql
        networks:
          new:
            aliases:
              - database
          legacy:
            aliases:
              - mysql

    networks:
      new:
      legacy:

其他服务可以使用以下方式连接到服务 ``db``：

- 直接使用 ``db``
- 同在 ``new`` 网络上的服务可以使用 ``database``
- 同在 ``legacy`` 网络上的服务可以使用 ``mysql``


ipv4_address、ipv6_address
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

为服务的容器指定一个静态 IP 地址。

::

    version: '2.1'

    services:
      app:
        image: busybox
        command: ifconfig
        networks:
          app_net:
            ipv4_address: 172.16.238.10
            ipv6_address: 2001:3984:3989::10

    networks:
      app_net:
        driver: bridge
        enable_ipv6: true
        ipam:
          driver: default
          config:
          -
            subnet: 172.16.238.0/24
          -
            subnet: 2001:3984:3989::/64


.. note::

    启用 IPV6 需要设置 ``enable_ipv6``，且只在 2.x 版本 Compose file 下适用，本选项在 Swarm 模式无效。


pid
----------

将 PID 模式设置为主机 PID 模式，可以打开容器与主机操作系统之间的共享 PID 地址空间。使用此标志启动的容器可以访问和操作宿主机的其他容器，反之亦然。

::

    pid: "host"


ports
----------

映射端口。

.. note::

    端口映射和 ``network_mode: host`` 不兼容。


SHORT syntax
~~~~~~~~~~~~~~~~~

可以使用 ``HOST:CONTAINER`` 的方式指定端口，也可以指定容器端口（选择临时主机端口），宿主机会随机映射端口。

::

    ports:
     - "3000"
     - "3000-3005"
     - "8000:8000"
     - "9090-9091:8080-8081"
     - "49100:22"
     - "127.0.0.1:8001:8001"
     - "127.0.0.1:5000-5010:5000-5010"
     - "6060:6060/udp"

.. note::

    当使用 ``HOST:CONTAINER`` 格式来映射端口时，如果使用的容器端口小于 60 可能会得到错误得结果，因为 YAML 将会解析 ``xx:yy`` 这种数字格式为 60 进制，所以建议采用字符串格式。


LONG syntax
~~~~~~~~~~~~~~~~

LONG 语法支持 SHORT 语法不支持的附加字段：

- ``target`` ：容器内的端口
- ``published`` ：公开的端口
- ``protocol`` ： 端口协议（ ``tcp`` 或 ``udp`` ）
- ``mode`` ：通过 ``host`` 用在每个节点还是哪个发布的主机端口或使用 ingress 用于集群模式端口进行平衡负载

::

    ports:
      - target: 80
        published: 8080
        protocol: tcp
        mode: host

secrets
------------

通过 ``secrets`` 为每个服务授予相应的访问权限，选项必须在顶层 :ref:`secrets <secrets>` 定义。

SHORT syntax
~~~~~~~~~~~~~~~~~~

仅用于指定 secret 名称，允许容器进入 secret 并挂载到 ``/run/secrets/>secret_name>``。

::

    version: "3.1"
    services:
      redis:
        image: redis:latest
        deploy:
          replicas: 1
        secrets:
          - my_secret
          - my_other_secret
    secrets:
      my_secret:
        file: ./my_secret.txt
      my_other_secret:
        external: true

LONG syntax
~~~~~~~~~~~~~~~~~~

LONG 语法可以添加其他选项：

- ``source`` ：secret 名称
- ``target`` ：在服务任务容器中需要装载在 ``/run/secrets/`` 中的文件名称，如果 ``source`` 未定义，那么默认为此值
- ``uid/gid`` ：在服务的任务容器中拥有该文件的 UID 或 GID 。如果未指定，两者都默认为 0。
- ``mode`` ：以八进制表示法将文件装载到服务的任务容器中 ``/run/secrets/`` 的权限。例如，``0444`` 代表可读。

::

    version: "3.1"
    services:
      redis:
        image: redis:latest
        deploy:
          replicas: 1
        secrets:
          - source: my_secret
            target: redis_secret
            uid: '103'
            gid: '103'
            mode: 0440
    secrets:
      my_secret:
        file: ./my_secret.txt
      my_other_secret:
        external: true

security_opt
--------------

为每个容器覆盖默认的标签。简单说来就是管理全部服务的标签。

::

    security_opt:
      - label:user:USER
      - label:role:ROLE

.. note::

    ``security_opt`` 在 Swarm 模式无效。


stop_grace_period
-----------------------

在发送 SIGKILL 之前 ，如果试图停止容器（如果它没有处理 SIGTERM 或使用 ``stop_signal`` 指定的任何停止信号），则需要等待的时间。

::

    stop_grace_period: 1s
    stop_grace_period: 1m30s

默认情况下，``stop`` 在发送 SIGKILL 之前等待 10 秒钟容器退出。


stop_signal
----------------

设置另一个信号来停止容器。在默认情况下使用的 SIGTERM 来停止容器。设置另一个信号可以使用 ``stop_signal`` 标签：

::

    stop_signal: SIGUSR1

.. note::

    `stop_signal` 在 Swarm 模式无效。


sysctls
--------------

在容器中设置的内核参数，可以为数组或字典。

::

    sysctls:
      net.core.somaxconn: 1024
      net.ipv4.tcp_syncookies: 0

    sysctls:
      - net.core.somaxconn=1024
      - net.ipv4.tcp_syncookies=0

.. note::

    `sysctls` 在 Swarm 模式无效。


ulimits
--------------

覆盖容器的默认 ulimits，可以单一地将限制值设为一个整数，也可以将 soft/hard 限制指定为映射

::

    ulimits:
      nproc: 65535
      nofile:
        soft: 20000
        hard: 40000

userns_mode
-----------------

如果 docker deamon 设置了用户命名空间，可以使用 ``userns_mode`` 禁用。

::

    userns_mode: "host"


.. note::

    ``userns_mode`` 在 Swarm 模式无效。


volumes
----------

挂载主机目录或数据卷。

如果直接挂载主机目录并作为单一服务使用，无需在顶层 :ref:`volumes <volumes>` 预定义后引用；如果要做多服务间复用，则需要先定义。


下面例子 ``web`` 服务使用 named volume （ ``mydata`` ），``db`` 服务使用了两个挂载：bind mount （``postgres.sock`` ）和 named volume（ ``dbdata`` ）。

::

    version: "3.2"
    services:
      web:
        image: nginx:alpine
        volumes:
          - type: volume
            source: mydata
            target: /data
            volume:
              nocopy: true
          - type: bind
            source: ./static
            target: /opt/app/static

      db:
        image: postgres:latest
        volumes:
          - "/var/run/postgres/postgres.sock:/var/run/postgres/postgres.sock"
          - "dbdata:/var/lib/postgresql/data"

    volumes:
      mydata:
      dbdata:


SHORT syntax
~~~~~~~~~~~~~~~~~~

挂载主机目录可以直接使用 ``HOST:CONTAINER`` 这样的格式，或者使用 ``HOST:CONTAINER:ro`` 这样的格式，后者对于容器来说，数据卷是只读的，这样可以有效保护宿主机的文件系统。

可以在主机上挂载相对路径，该路径相对于正在使用的 Compose 配置文件的目录进行扩展。相对路径应始终以 ``.`` 或 ``..`` 开头

::

    volumes:
      # Just specify a path and let the Engine create a volume
      # 只是指定一个路径，Docker 会自动在创建一个数据卷（这个路径是容器内部的）。
      - /var/lib/mysql

      # Specify an absolute path mapping
      - /opt/data:/var/lib/mysql

      # Path on the host, relative to the Compose file
      - ./cache:/tmp/cache

      # User-relative path
      - ~/configs:/etc/configs/:ro

      # Named volume
      - datavolume:/var/lib/mysql


LONG syntax
~~~~~~~~~~~~~~~

LONG 语法有些附加字段：

- ``type`` ：安装类型，可以为 ``volume``、``bind`` 或 ``tmpfs``
- ``source`` ：挂载源路径，主机路径或定义在顶级 ``volumes`` 中卷的名称，不适用于 tmpfs 类型安装。
- ``target`` ：挂载在容器中的路径
- ``read_only`` ：将卷设置为只读
- ``bind`` ：配置额外的绑定选项
    - ``propagation`` ：用于绑定的传播模式
- ``volume`` ：配置额外的卷选项
    - ``nocopy`` ：创建卷时禁止从容器复制数据的标志
- ``tmpfs`` ：配置额外的 tmpfs 选项
    - ``size`` ：tmpfs 的大小，以 byte 为单位

::

    version: "3.2"
    services:
      web:
        image: nginx:alpine
        ports:
          - "80:80"
        volumes:
          - type: volume
            source: mydata
            target: /data
            volume:
              nocopy: true
          - type: bind
            source: ./static
            target: /opt/app/static

    networks:
      webnet:

    volumes:
      mydata:

用于服务、群集以及堆栈文件的卷
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在使用服务，群集和 ``docker-stack.yml`` 文件时，请记住支持服务的任务（容器）可以部署在群集中的任何节点上，并且每次更新服务时都可能是不同的节点。

在缺少指定源的命名卷的情况下，Docker 为支持服务的每个任务创建一个匿名卷。关联的容器被移除后，匿名卷不会保留。

如果希望数据持久存在，请使用可识别多主机的命名卷和卷驱动程序，以便可以从任何节点访问数据。或者，对该服务设置约束，以便将其任务部署在具有该卷的节点上。

下面一个例子中定义了一个 ``db`` 的服务。它被配置为一个命名卷来保存群体上的数据，并且仅限于在节点上运行。下面是来自该文件的部分内容：

::

    version: "3"
    services:
      db:
        image: postgres:9.4
        volumes:
          - db-data:/var/lib/postgresql/data
        networks:
          - backend
        deploy:
          placement:
            constraints: [node.role == manager]


restart
--------------

默认值为 ``no`` ，即在任何情况下都不会重新启动容器；当值为 ``always`` 时，容器总是重新启动；当值为 ``on-failure`` 时，当出现 on-failure 报错容器退出时，容器重新启动。

::

    restart: "no"
    restart: always
    restart: on-failure
    restart: unless-stopped

.. note::

    ``restart`` 在 Swarm 模式无效，请使用 :ref:`restart_policy <restart_policy>` 。


domainname
--------------

::

    domainname: foo.com

类似于使用 `docker run <https://docs.docker.com/engine/reference/run/>`_ 。

hostname
--------------

::

    hostname: foo

类似于使用 `docker run <https://docs.docker.com/engine/reference/run/>`_ 。


ipc
-----------

::

    ipc: host

类似于使用 `docker run <https://docs.docker.com/engine/reference/run/>`_ 。

mac_address
--------------

::

    mac_address: 02:42:ac:11:65:43

类似于使用 `docker run <https://docs.docker.com/engine/reference/run/>`_ 。

privileged
------------

::

    privileged: true

类似于使用 `docker run <https://docs.docker.com/engine/reference/run/>`_ 。

read_only
-------------

::

    read_only: true

类似于使用 `docker run <https://docs.docker.com/engine/reference/run/>`_ 。

shm_size
-----------

::

    shm_size: 64M

类似于使用 `docker run <https://docs.docker.com/engine/reference/run/>`_ 。

stdin_open
--------------

::

    stdin_open: true

类似于使用 `docker run <https://docs.docker.com/engine/reference/run/>`_ 。

tty
--------------

::

    tty: true

类似于使用 `docker run <https://docs.docker.com/engine/reference/run/>`_ 。

user
---------------

::

    user: postgresql

类似于使用 `docker run <https://docs.docker.com/engine/reference/run/>`_ 。


working_dir
-------------------

::

    working_dir: /code

类似于使用 `docker run <https://docs.docker.com/engine/reference/run/>`_ 。


durations
============

某些配置选项如 ``check`` 的子选项 ``interval`` 以及 ``timeout`` 的设置格式，支持的单位有 ``us``、``ms``、``s``、``m`` 以及 ``h`` 。

::

    2.5s
    10s
    1m30s
    2h32m
    5h34m56s

byte values
================

某些选项如 ``bulid`` 的子选项 ``shm_size``。

::

    2b
    1024kb
    2048k
    300m
    1gb

支持的单位是 ``b``，``k``，``m``，``g``，或 ``kb``，``mb`` 和 ``gb`` 。目前不支持十进制值。

.. _volumes:

volumes
===========

命名卷可以跨服务复用，下面两服务共享同一个目录用作数据备份。

::

    version: "3"

    services:
      db:
        image: db
        volumes:
          - data-volume:/var/lib/db
      backup:
        image: backup-service
        volumes:
          - data-volume:/var/lib/backup/data

    volumes:
      data-volume:


driver
-----------

``volumes`` 定义的卷可以为空，此时使用默认驱动（通常是 ``local`` ），也可以手动指定驱动。但如果驱动不可以，``docker-compose up`` 启动时创建卷失败，如：

::

    driver: foobar

.. _driver_opts:

driver_opts
---------------

指定驱动选项。

::

    driver_opts:
      foo: "bar"
      baz: 1

external
-------------

如果指定为 ``true``，说明该卷在 Compose 之外创建，``docker-compose up`` 时不会创建，但如果不存在则报错。


``external`` 不能喝其他卷配置一起使用（ ``driver``，``driver_opts`` ）。


::

    version: '2'

    services:
      db:
        image: postgres
        volumes:
          - data:/var/lib/postgresql/data

    volumes:
      data:
        external: true

可使用 ``name`` 指定外部卷实际名称：

::

    volumes:
      data:
        external:
          name: actual-name-of-volume

.. note::

    在 Swarm 模式下，外部卷不存在则创建。

labels
-----------

参考 :ref:`labels <sub_labels>` 。

name
------------

设置挂载卷名称，可用于引用包含特殊字符的卷。

::

    version: '3.4'
    volumes:
      data:
        name: my-app-data

``name`` 可以和 ``external`` 结合使用：

::

    version: '3.4'
    volumes:
      data:
        external: true
        name: my-app-data


.. _networks:

networks
===========

driver
----------

用于指定网络驱动。默认驱动通常为 ``bridge`` （single host）或 ``overlay`` （swarm）。

如果驱动不可用则报错。

::

    driver: overlay

bridge
~~~~~~~~~~~~

单节点上默认使用 ``bridge`` 网络。

overlay
~~~~~~~~~~~

在 Swarm 多节点间使用 ``overlay`` 驱动创建网络。

host or none
~~~~~~~~~~~~~~

使用主机网络，或不使用网络。只在 ``docker stack`` 下有效，等价于：

::

    docker run --net=host

    docker run --net=none

如果使用 ``docker-compose``，请使用 :ref:`network_mode <network_mode>` 。

内置网络（Docker 自动创建） ``host`` 和 ``none`` 的使用需要创建别名（如 ``hostnet/nonet``），然后才能在 Compose 这使用内置网络：

::

    services:
      web:
        ...
        networks:
          hostnet: {}

    networks:
      hostnet:
        external: true
        name: host

::

    services:
      web:
        ...
        networks:
          nonet: {}

    networks:
      nonet:
        external: true
        name: none

driver_opts
----------------

参考 :ref:`driver_opts <driver_opts>` 。

attachable
-------------

参考 `attachable <https://docs.docker.com/compose/compose-file/#attachable>`_ 。

ipam
--------

参考 `ipam <https://docs.docker.com/compose/compose-file/#ipam>`_ 。

internal
--------------

默认情况下，Docker 使用 bridge 网络提供外界连接，如果要创建与外界隔离的 overlay 网络，将 internal 设置为 ``true`` 。

labels
--------

参考 :ref:`labels <sub_labels>` 。


external
------------

如果为 ``true``，指明本网络为 Compose 外部创建，``docker-compose up`` 时不会创建，但如果不存在则报错。

``external`` 不能与其他网络配置混合使用（ ``driver``, ``driver_opts``, ``ipam``, ``internal`` ）。

::

    version: '2'

    services:
      proxy:
        build: ./proxy
        networks:
          - outside
          - default
      app:
        build: ./app
        networks:
          - default

    networks:
      outside:
        external: true

使用 ``name`` 设置实际网络名称：

::

    networks:
      outside:
        external:
          name: actual-name-of-network


name
----------

设置网络实际名称，可包含特殊字符。

::

    version: '3.5'
    networks:
      network1:
        name: my-app-net

``name`` 可与 ``external`` 一起使用：

::

    version: '3.5'
    networks:
      network1:
        external: true
        name: my-app-net


.. _configs:

configs
===========

``configs`` 可被同一 stack 内的服务共享，来源只能为 ``file`` 和 ``external`` 。

- ``file`` ：使用指定路径文件创建配置
- ``external`` ：如果值为 true，指明配置已经创建，如果不存在报错。
- ``name`` ：配置对象名称，可以包含特殊字符。

::

    configs:
      my_first_config:
        file: ./config_data
      my_second_config:
        external: true

::

    configs:
      my_first_config:
        file: ./config_data
      my_second_config:
        external:
          name: redis_config


.. _secrets:

secrets
===========

选项 ``file``，``external``，``name`` 类似 :ref:`configs <configs>` 。

::

    secrets:
      my_first_secret:
        file: ./secret_data
      my_second_secret:
        external: true

::

    secrets:
      my_first_secret:
        file: ./secret_data
      my_second_secret:
        external:
          name: redis_secret

variable substitution
========================

Compose 变量能使用环境变量，如定义环境变量 ``POSTGRES_VERSION=9.3``，在 Compose 中使用：

::

    db:
      image: "postgres:${POSTGRES_VERSION}"


当使用 ``docker-compose up`` 时，Compose 将 ``image`` 解析为 ``postgres:9.3``，然后运行。

如果环境变量没有设置，Compose 替换为空字符串，如上面 ``image`` 解析为 ``postgres:`` 。

在 ``.env`` 中设置的环境变量会被 shell 环境变量覆盖，且只在 ``docker-compose up`` 中有效，在 ``docker stack deploy`` 中无效。

变量取值支持 ``$VARIABLE`` 和 ``${VARIABLE}`` 两种语法，后一种支持使用默认值：

- ``${VARIABLE:-default}`` ：如果 ``VARIABLE`` 不存在或为空时，使用默认值 ``default``
- ``${VARIABLE-default}`` ：如果 ``VARIABLE`` 不存在时，使用默认值 ``default``
- ``${VARIABLE:?err}`` ：如果 ``VARIABLE`` 不存在或为空时，以包含 ``err`` 错误消息退出
- ``${VARIABLE?err}`` ：如果 ``VARIABLE`` 不存在时，以包含 ``err`` 错误消息退出

如果配置需要使用美元符号 $，需要使用 ``$$`` 引用：

::

    web:
      build: .
      command: "$$VAR_NOT_INTERPOLATED_BY_COMPOSE"























