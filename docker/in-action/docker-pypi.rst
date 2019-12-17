.. _docker-pypi:

================================
Docker 搭建 PyPI 服务器
================================


运行 pypi-server 容器
======================

使用镜像 `codekoala/pypi`_ 启动 pypi-server 服务器，docker-compose 如下：

::

    version: '3'
    services:
      pypi:
        image: codekoala/pypi
        volumes:
          - $PWD:/srv/pypi
        container_name: pypi
        ports:
          - 9527:80

使用如下命令启动：

::

    $ docker-compose up -d

.. _codekoala/pypi: https://hub.docker.com/r/codekoala/pypi/

添加用户
==============

使用 ``htpasswd`` 添加用户：

::

    $ htpasswd -c -s /htpasswd ${username}

两次输入密码后创建用户成功。

此时已经可以访问服务器：``http://${your-IP}:9527``。注意要把 IP 换成自己的服务器 IP。

.. tip::

    如果在 CentOS7 下没有 ``htpasswd``，则需要使用 ``sudo yum install httpd-tools`` 进行安装。


使用方法
==============

上传 package
-------------------

配置 ``~/.pypirc``，为了不用每次上传输入账号密码和仓库 URL。格式如下：

::

    [distutils]
    index-servers =
        pypi
        pypitest
        internal

    [pypi]
    username: xxxxx
    password: xxxxx

    [pypitest]
    repository: https://test.pypi.org/legacy/
    username: xxxxx
    password: xxxxx

    [internal]
    repository: http://${your-IP}:9527
    username: xxxxx
    password: xxxxx

使用 twine 上传

::

    $ twine upload dist/* -r internal


使用仓库安装 package
-----------------------

::

    $ pip install ${package-name} -i http://${your-IP}:9527 --trusted-host ${your-IP}

因为是 HTTP 连接，所以需要选项 ``--trusted-host``。














