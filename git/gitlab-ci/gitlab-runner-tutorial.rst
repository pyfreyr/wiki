.. _gitlab-runner-tutorial:

===========================
Gitlab Runner 安装使用
===========================


安装 Runner
=================

本文使用 docker 运行 Runner，所以只需要启动容器即可工作，其他安装方式参考：`Install GitLab Runner`_ 。

.. _Install GitLab Runner: https://docs.gitlab.com/runner/install/

.. tip::

    Docker 的基础知识和使用参考：:ref:`Docker 基础 <docker-basic>` 。

为方便容器管理，使用 docker-compose 管理容器，compose 文件如下：

::

    version: '3'
    services:
      gitlab-runner:
        image: gitlab/gitlab-runner
        container_name: gitlab-runner
        volumes:
          - $PWD/gitlab-runner/config:/etc/gitlab-runner
          - /var/run/docker.sock:/var/run/docker.sock
        privileged: true
        restart: always

使用如下命令启动 gitlab-runner：

::

    $ docker-compose up -d


获取 Gitlab token
========================

根据 token 的来源权限不同，gitlab-runner 对应的有三种类型（`Configuring GitLab Runners`_ ）：

- ``Specific Runner``: 一个 Runner 对应一个 Project，只能在本项目内使用
- ``Group Runner``: 一个 Runner 对应一个 Group，群组内的项目都可以使用
- ``Shared Runner``: 可应用于所有项目

以 Specific Runner 为例，进入项目内 Settings —— CI/CD —— Runner settings 即可看到 URL 和 token：

.. image:: /images/gitlab-runner-token.png

.. _Configuring GitLab Runners: https://docs.gitlab.com/ee/ci/runners/README.html


注册 Runner
===================

项目只有注册了 Runner 才能完成实际执行，使用命令 ``gitlab-runner register`` 进行注册，在我们的 docker 环境下使用：

::

    $ docker exec -it gitlab-runner gitlab-runner register

完成几个问题交互，即完成注册：

.. image:: /images/gitlab-runner-register.png

其中，``URL`` 和 ``token`` 就是上一步在项目内得到的，``description`` 和 ``tag`` 可以随意填写，``executor`` 根据实际需要选择，这里选择使用 docker，所以下一步的默认镜像使用 ``docker:stable``。

.. note::

    也可以在 register 的时候使用参数指定所有设置，``gitlab-runner register -h`` 查看可用选项。

    如果设置了 tag，以后的 ``.gitlab-ci.yml`` 使用中都必须通过显式指定 tags 来选择执行 runner。

正常的注册到此就结束，但 docker 还需要进一步的修改配置。


使用 docker 的额外配置
=========================

gitlab-runner 的配置文件为 ``/config/config.toml``，需要修改 ``privileged`` 和 ``volumes``：



修改后的配置如下：

::

    [[runners]]
      name = "docker runner"
      url = "https://gitlab.*****.com/"
      token = "34ac*******************"
      executor = "docker"
      [runners.docker]
        tls_verify = false
        image = "docker:stable"
        privileged = true
        disable_cache = false
        volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock"]
        shm_size = 0
      [runners.cache]

最后需要重启 runner 容器生效：


::

    $ docker-compose up -d

现在，在 Runner settings 里面就能看到注册好的 Runner 了：

.. image:: /images/gitlab-runner-activated.png











