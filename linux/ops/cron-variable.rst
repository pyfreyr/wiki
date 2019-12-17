.. _cron-variable:

========================
crontab 配置环境变量
========================

.. note::

    当使用 crontab 部署的任务没执行时怎么办？


首先收集 crontab 日志，在任务最后加上 ``2> log.txt``，然后更改下执行时间下一分钟生效，在日志中找到出错原因。（也可直接查看 cron 的运行日志 ``/var/log/cron`` ）

如果是因为环境变量缺失，那就是本文要解决的问题。

系统 cron 任务
==================

系统 cron 任务存在于 ``/etc/crontab`` 文件，这个文件是系统安装时设置好的自动安排的进程任务的 crontab 文件。这为系统管理员安排 cron 任务提供了方便。

CentOS 默认的 ``/etc/crontab`` 文件的内容为：

::

    SHELL=/bin/bash
    PATH=/sbin:/bin:/usr/sbin:/usr/bin
    MAILTO=root
    HOME=/
    # run-parts
    01 * * * * root run-parts /etc/cron.hourly
    02 4 * * * root run-parts /etc/cron.daily
    22 4 * * 0 root run-parts /etc/cron.weekly
    42 4 1 * * root run-parts /etc/cron.monthly

``/etc/cron.daily``、``/etc/cron.monthly``、``/etc/cron.weekly`` 和 ``/etc/cron.hourly`` 是四个目录，分别放置系统每天、每个月、每周和每个小时要执行的任务的脚本文件。

.. note::

    可以看到 crontab 文件最前面是可以设置环境变量的！

用户 cron 任务添加环境变量
==============================

用户 cron 任务存在于 ``/var/spool/cron`` 目录下用户的 crontab 文件。当任务依赖于环境变量时可以仿照系统 crontab 文件在用于 crontab 文件最开始添加环境变量。


比如：

::

    SHELL=/bin/zsh
    HOME=/data1/datascience/scrapy-docker
    PYTHONPATH=/data1/datascience/scrapy-docker
    PATH=/opt/anaconda3/bin:/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin

    # ftp data to cluster
    */5 * * * * cd crawler_tools && python sftp.py

其中，

- ``SHELL``，指定使用的 shell
- ``HOME``，指定 cron 工作目录，所有任务默认在该目录下执行。非默认目录执行需要手动使用 ``cd`` 切换（支持相对路径）。
- ``PATH``，默认只搜索系统 crontab 的四个目录，上面添加了 anaconda3
- 其他环境变量，如 PYTHONPATH。


在 shell 脚本添加环境变量
============================

如果任务是 shell 脚本，也可以在脚本开头加上：

::

    source /etc/profile

来添加环境变量。``/etc/profile`` 可换成 ``~/.bashrc``, ``~/.bash_profile`` 等，根据环境变量定义的位置进行替换。