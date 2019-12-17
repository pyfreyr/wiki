.. _docker-tcp:

====================================
开启 Docker 远程访问
====================================

CentOS 7
=============

.. note::

    在 CentOS 中没有 ``/etc/default/docker``，另外也没有找到 ``/etc/sysconfig/docker`` 这个配置文件。


在 ``/usr/lib/systemd/system/docker.service`` 配置远程访问，主要是在 ``[Service]`` 部分，加上下面两个参数：

::

    [Service]
    ExecStart=
    ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock

重启 docker 服务

::

    $ systemctl daemon-reload
    $ systemctl restart docker


在另一台机器上使用命令检查是否成功：

::

    $ docker -H ${your-IP} version




