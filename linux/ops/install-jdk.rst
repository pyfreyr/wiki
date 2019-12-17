.. _install-jdk:

===================
Linux 安装 JDK
===================

下载 JDK
===========

前往 `Java SE Downloads`_ 下载 JDK（如 `jdk-8u181-linux-x64.tar.gz`_ ）。

.. _Java SE Downloads: http://www.oracle.com/technetwork/java/javase/downloads/index.html
.. _jdk-8u181-linux-x64.tar.gz: http://download.oracle.com/otn-pub/java/jdk/8u181-b13/96a7b8442fe848ef90c96a2fad6ed6d1/jdk-8u181-linux-x64.tar.gz

卸载 OpenJDK
==============

使用 ``java -version`` 查看是否已经安装 OpenJDK，如果已经安装先卸载。

查看安装位置：

::

    $ rpm -aq | grep java

卸载：

::

    $ rpm -e --nodeps java-1.8.0-openjdk-1.8.0.171-8.b10.el7_5.x86_64
    ...

安装 JDK
=============

解压 JDK 安装包：

::

    $ tar -zxvf jdk-8u181-linux-x64.tar.gz -C /opt/jdk-8u181

配置环境变量
==============

新建文件 ``/etc/profile.d/java.sh``，添加如下内容：

::

    export JAVA_HOME=/opt/jdk-8u181
    export JRE_HOME=$JAVA_HOME/jre
    export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
    export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH

修改文件权限：

::

    $ sudo chmod 755 /etc/profile.d/java.sh

.. attention::

    重启电脑生效，使用 ``java -version`` 查看是否成功。

