.. _config-vmware-and-centos:


===============================
Vmware 安装 CentOS7 后配置
===============================


配置虚拟机网络
==================

首先设置虚拟机网络连接为 **NAT 模式**：

.. image:: /images/vmware-nat.png



然后进入 "编辑——虚拟机网络编辑器"，配置 VMnet8 网络：

.. image:: /images/vmware-nat1.png
.. image:: /images/vmware-nat2.png

配置静态 IP
===============

编辑文件 ``/etc/sysconfig/network-scripts/ifcfg-ens33`` （文件名可能因系统不同而不同，如可能为 ``ifcfg-eth0`` ）:

::

    TYPE=Ethernet
    PROXY_METHOD=none
    BROWSER_ONLY=no
    BOOTPROTO=static
    DEFROUTE=yes
    IPV4_FAILURE_FATAL=no
    IPV6INIT=yes
    IPV6_AUTOCONF=yes
    IPV6_DEFROUTE=yes
    IPV6_FAILURE_FATAL=no
    IPV6_ADDR_GEN_MODE=stable-privacy
    NAME=ens33
    UUID=4f321d80-7fb7-4e16-879e-9897971b5ae5
    DEVICE=ens33
    ONBOOT=yes
    NETMASK=255.255.255.0
    GATEWAY=192.168.70.2
    IPADDR=192.168.70.33
    DNS1=114.114.114.114
    DNS2=202.106.0.20

其中，``GATEWAY`` 配置为上一步操作中的 网关 IP；``IPADDR`` 为静态 IP，可依据网关自由修改。

.. tip::

    使用 ``ifconfig`` 查看自己 IP 地址。

重启网络服务：

::

    $ service network restart


.. tip::

    可以使用 ``ping www.baidu.com`` 查看网络是否已通。

    宿主机如果为 Windows，可以在 Xshell 中配置使用上面设置的静态 IP 进行登录。

更换 yum 源
================

可以使用国内镜像源进行系统更新和软件下载的加速，以 aliyun 为例：


进入 yum 配置文件目录，备份默认源配置：

::

    $ cd /etc/yum.repos.d
    $ mv CentOS-Base.repo CentOS-Base.repo.bak

下载 aliyun 配置：

::

    $ wget -O CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

下载 epel 源（可选）：

::

    $ wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

更新缓存：

::

    $ yum clearn all
    $ yum makecache

更新系统：

::

    $ yum update


解决 SMBus 错误
====================

在 ``/etc/modprobe.d/blacklist.conf`` 中添加黑名单：

::

    blacklist i2c_piix4