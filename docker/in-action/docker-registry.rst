.. _docker-registry:

========================
搭建 Docker 私有仓库
========================


搭建私有仓库
=================

使用 `registry`_ 镜像搭建本地私有仓库，compose 文件如下：

::

    version: '3'
    services:
      registry:
        image: registry
        restart: always
        container_name: registry
        volumes:
          - $PWD/docker-registry/registry:/var/lib/registry
        ports:
          - 5000:5000

.. _registry: https://store.docker.com/images/registry

使用如下命令启动仓库镜像：

::

    $ docker-compose up -d


开启 HTTP 访问
==================

.. note::

    由于 Registry 为了安全性考虑，默认是需要 HTTPS 证书支持的。而启动的 docker registry 未采用 HTTPS 服务，所以需要把客户端的请求也改为 HTTP。

在 ``/etc/docker/daemon.json`` 加入如下内容（没有就新建文件，注意 IP 换成自己私有仓库的）：

::

    {
        "insecure-registries" : ["1.1.1.1:5000"]
    }

重启 docker：

::

    $ sudo systemctl restart docker

推送镜像
==============

首先将本地镜像以仓库 URL 为前缀重命名：

::

    $ docker tag datascience/tensorflow-gpu:1.7.0 1.1.1.1:5000/datascience/tensorflow-gpu:1.7.0

推送到私有仓库：

::

    $ docker push 1.1.1.1:5000/datascience/tensorflow-gpu:1.7.0

如果没有按照上面配置 HTTP 访问，则推送时报错：

::

    The push refers to a repository [1.1.1.1:5000/datascience/tensorflow-gpu]
    Get https://1.1.1.1:5000/v2/: http: server gave HTTP response to HTTPS client



查看私有仓库
=================

查看所有镜像，使用 ``GET /v2/_catalog``：

::

    $ curl http://1.1.1.1:5000/v2/_catalog
    {"repositories":["datascience/base","datascience/cuda","datascience/tensorflow-gpu"]}

查看具体镜像，使用 ``GET /v2/<name>/tags/list``：

::

    $ curl http://1.1.1.1:5000/v2/datascience/tensorflow-gpu/tags/list
    {"name":"datascience/tensorflow-gpu","tags":["1.7.0","1.0.1"]}


拉取镜像
=============

::

    $ docker pull 1.1.1.1:5000/datascience/tensorflow-gpu:1.7.0

.. tip::

    初始镜像名较长，推荐重命名（``docker tag``）后使用。


