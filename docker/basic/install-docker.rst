.. _install-docker:

=================
Docker 安装教程
=================



CentOS 7
==============



本文参考 https://store.docker.com/search?type=edition&offering=community。

系统依赖
-------------

``centos-extras`` 仓库必须开启（``/etc/yum.repos.d/Centos-Base.repo`` 内）。

卸载旧版（可选）
-------------------

如果安装老版本（以前叫做 docker 或 docker-engine，现在拆分为 docker-ee 和 docker-ce）则需要先卸载：

::

    $ yum remove docker \
          docker-client \
          docker-client-latest \
          docker-common \
          docker-latest \
          docker-latest-logrotate \
          docker-logrotate \
          docker-selinux \
          docker-engine-selinux \
          docker-engine
    $ rm -rf /var/lib/docker


安装 docker-ce
-----------------------

使用 yum 安装
~~~~~~~~~~~~~~~~~~~

1. 添加仓库

    ::

        $ yum install -y yum-utils \
              device-mapper-persistent-data \
              lvm2
        $ yum-config-manager \
            --add-repo \
            https://download.docker.com/linux/centos/docker-ce.repo
2. 安装

    使用以下命令直接安装最新版：

    ::

        $ yum install docker-ce

    如果要安装指定版本，先查看可用版本：

    ::

        $ yum list docker-ce --showduplicates | sort -r
        docker-ce.x86_64 18.06.1.ce-3.el7 docker-ce-stable
        docker-ce.x86_64 18.06.0.ce-3.el7 docker-ce-stable
        docker-ce.x86_64 18.03.1.ce-1.el7.centos docker-ce-stable
        ......

    然后指定版本安装：

    ::

        $ yum install docker-ce-18.03.1.ce

更新使用 ``yum install`` 指定版本即可。

使用 rpm 安装
~~~~~~~~~~~~~~~~~~

进入 https://download.docker.com/linux/centos/7/x86_64/stable/Packages/ 下载安装包，执行以下命令安装：

::

    $ yum install docker-ce-18.06.0.ce-3.el7.x86_64.rpm

更新时使用 ``yum upgrade`` 指定新 rpm 包。

查看版本
---------------

::

    $ docker version
    Client:
     Version: 17.09.0-ce
     API version: 1.32
     Go version: go1.8.3
     Git commit: afdb6d4
     Built: Tue Sep 26 22:41:23 2017
     OS/Arch: linux/amd64

此时还没启动 docker daemon，只能看到客户端信息。

启动 docker
-----------------

启动 docker daemon：

::

    $ systemctl start docker

现在可以查看服务端信息：

::

    $ docker version
    Client:
     Version: 17.09.0-ce
     API version: 1.32
     Go version: go1.8.3
     Git commit: afdb6d4
     Built: Tue Sep 26 22:41:23 2017
     OS/Arch: linux/amd64

    Server:
     Version: 17.09.0-ce
     API version: 1.32 (minimum version 1.12)
     Go version: go1.8.3
     Git commit: afdb6d4
     Built: Tue Sep 26 22:42:49 2017
     OS/Arch: linux/amd64
     Experimental: false

开机自启
--------------

::

    $ systemctl enable docker


非 root 用户使用 docker
---------------------------

::

    $ usermod -aG docker ${username}

    $ systemctl daemon-reload
    $ systemctl restart docker

.. _docker-mirror:

更换 docker 镜像仓库
-------------------------

中科大源教程：http://mirrors.ustc.edu.cn/help/dockerhub.html

更新 ``/etc/docker/daemon.json`` 为：

::

    {
        "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/"]
    }



.. _docker-uninstall:

卸载 docker
---------------------

::

    $ yum remove docker-ce
    $ rm -rf /var/lib/docker



Ubuntu 16.04
====================

本节参考 https://docs.docker.com/install/linux/docker-ce/ubuntu/。

卸载旧版（可选）
---------------------

如果安装过较老版本的 docker 或 docker-engine，执行删除：

::

    $ apt-get remove docker docker-engine docker.io
    $ rm -rf /var/lib/docker

安装 docker-ce
---------------------

使用 apt 安装
~~~~~~~~~~~~~~~~~~~~~

1. 添加仓库

    ::

        $ apt-get update

        $ apt-get install \
              apt-transport-https \
              ca-certificates \
              curl \
              software-properties-common

        $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

        $ apt-key fingerprint 0EBFCD88

        $ add-apt-repository \
              "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
              $(lsb_release -cs) \
              stable"

2. 安装

    安装最新版本：

    ::

        $ apt-get update
        $ apt-get install docker-ce

    安装特定版本：

    ::

        $ apt-cache madison docker-ce
        docker-ce | 18.06.1~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
        docker-ce | 18.06.0~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
        docker-ce | 18.03.1~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
        ......

        $ sudo apt-get install docker-ce=18.06.1~ce~3-0~ubuntu

3. 升级

    先 update 系统，然后指定版本 install。参考上面安装。

使用 deb 安装
~~~~~~~~~~~~~~~~~~~

前往 https://download.docker.com/linux/ubuntu/dists/xenial/pool/stable/amd64/ 下载安装包，执行：

::

    $ dpkg -i docker-ce_17.09.1~ce-0~ubuntu_amd64.deb

升级下载新 deb 包安装。

启动 docker
-----------------

::

    $ service docker start

开机自启
---------------

.. warning::

    Unknown, to be continued...

非 root 用户使用 docker
-------------------------

::

    $ usermod -aG docker ${username}

    $ service docker restart

更换 docker 镜像仓库
----------------------

参考 :ref:`更换 docker 镜像仓库 <docker-mirror>` 。


卸载 docker
----------------------

::

    $ apt-get purge docker-ce
    $ rm -rf /var/lib/docker




