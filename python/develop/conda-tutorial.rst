.. _conda-tutorial:

=====================
Conda 使用手册
=====================

:download:`conda-cheatsheet.pdf </files/conda-cheatsheet.pdf>`


Conda 是针对于 Python 的环境和包管理工具。可以安装 miniconda 或 anaconda 进行安装，前者是简化版本，只包含 conda 和其依赖。

Conda 有 Python3.x 和 Python2.x 系列两个版本，其实都没有关系，因为你在使用 conda 进行创建环境时，可以指定 Python 的版本。

安装 conda
=============

以 miniconda 为例，进入 https://conda.io/miniconda.html 选择对应的版本下载并安装。

.. tip::

    也可以使用清华 miniconda 源：https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/

查看 conda 帮助
=======================

所有关于 conda 的使用都可以从帮助信息获取，在什么也不知道的情况下就可以使用 ``conda -h/--help`` 查看帮助信息：

::

    $ conda -h
    conda is a tool for managing and deploying applications, environments and packages.

    Options:

    positional arguments:
      command
        info Display information about current conda install.
        help Displays a list of available conda commands and their help
                     strings.
        list List linked packages in a conda environment.
        ......

可以看到，conda 有很多命令，要查看具体命令帮助，同样使用 ``-h`` 选项，如：

::

    $ conda info -h
    usage: conda info [-h] [--json] [--debug] [--verbose] [--offline] [-a] [-e]
                      [-l] [-s] [--root] [--unsafe-channels]
                      [packages [packages ...]]

    Display information about current conda install.

    Options:

    positional arguments:
      packages Display information about packages.

    optional arguments:
      -h, --help Show this help message and exit.
      --json Report all output as json. Suitable for using conda
                         programmatically.
      ......

conda 常用命令
=================

conda info
----------------

显示当前 conda 安装的所有信息。通常使用 ``-e/--envs`` 选项查看创建的虚拟环境：

::

    $ conda info -e
    # conda environments:
    #
    root * /opt/anaconda3


包管理命令
-------------------

一般来说 conda 仓库的软件没有 PyPI 更新快和全。所以推荐 conda 只用来创建虚拟环境（后面介绍），包的安装管理仍然使用 pip。但在 windows 下由于依赖不好处理，所以在使用 pip 失败时可以查询是否有编译好的 conda 包。

conda list
~~~~~~~~~~~~~~~~~

列出当前环境下所有安装的 conda 包。

::

    $ conda list
    # packages in environment at /opt/anaconda3:
    #
    _license 1.1 py35_1
    _nb_ext_conf 0.3.0 py35_0
    acora 2.0 <pip>
    alabaster 0.7.9 py35_0
    ......


查询包
~~~~~~~~~~~~

使用 ``conda search`` 查询 conda 包，只有经过 conda 重新编译入库的才能使用 ``conda search`` 查询得到，具体可以进 `Anaconda package lists`_ 查看。

::

    $ conda search scrapy
    Loading channels: done
    Name Version Build Channel
    ......
    scrapy 1.4.0 py27h060f748_1 defaults
    scrapy 1.4.0 py35h03cf01c_1 defaults
    scrapy 1.4.0 py36h5dd8a1d_1 defaults
    scrapy 1.5.0 py27_0 defaults
    scrapy 1.5.0 py35_0 defaults
    scrapy 1.5.0 py36_0 defaults

.. _Anaconda package lists: https://docs.anaconda.com/anaconda/packages/pkg-docs

安装包
~~~~~~~~~~~~~

使用 ``conda install`` 安装 conda 包，会自动处理包之间的依赖。

::

    $ conda install scrapy

使用 conda 安装指定版本包，既可以使用类似 pip 的 ``==``，也可以直接使用 ``=``：

::

    $ conda install scrapy=1.5.0

更新包
~~~~~~~~~~~

使用 ``conda update`` 更新 conda 包到最新版本，也可使用 ``conda upgrade``。

::

    $ conda update scrapy

卸载包
~~~~~~~~~~~~

使用 ``conda remove`` 卸载 conda 包，也可使用 ``conda uninstall``。

::

    $ conda remove scrapy

环境管理命令
----------------

创建环境
~~~~~~~~~~~~~~

创建虚拟环境，使用 ``-n/--name`` 指定环境名称。可以在创建环境的同时安装包。由于 conda 将 Python 也作为包，所以可以像其他包一样安装。

::

    $ conda create --name tf python=3.5.2 tensorflow

现在使用 ``conda info -e`` 查看环境（也可使用命令 ``conda env list``）：

::

    $ conda info -e
    # conda environments:
    #
    base * /opt/anaconda3
    tf /home/${user}/.conda/envs/tf

.. note::

    用户创建的虚拟环境保存在 ``~/.conda/envs`` 下。

激活环境
~~~~~~~~~~~~~~~

默认处于 base 环境，进入其他环境需要使用 ``source activate`` 手动切换：

::

    $ source activate tf

.. attention::

    Windows 下建议使用 git-bash 操作（安装 windows 版本 git 自动创建），或者在 cmd/powershell 下使用 ``activate ${env-name}`` 激活（没有 ``source``）。

激活成功会在命令行提示符前面标识出当前环境：

::

    (tf) ➜ ~

若要退出激活当前环境，使用 ``source deactivate``，默认回到 base 环境：

::

    $ source deactivate

删除环境
~~~~~~~~~~~~~~~

删除环境也使用 ``conda remove`` 命令，不过加上参数 ``--all`` 并使用 ``-n/--name`` 指定要删除的环境名。

::

    $ conda remove -n tf --all

也可以使用命令 ``conda env remove -n tf``。

拷贝环境
~~~~~~~~~~~~~~~

在创建环境时可以使用 ``--clone`` 从已存在的环境进行拷贝。

::

    $ conda create --clone tensorflow --name tf

在指定环境中管理包
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    如果和我一样使用 pip 进行包管理，可以忽略这部分。o(∩_∩)o

一种方法是激活该环境，然后按照上面包管理进行操作；另一种方法是使用 conda 进行包管理时指定环境：

::

    $ conda install --name myenv redis
    $ conda remove --name myenv redis

添加 conda 镜像源
===============================

同样，使用 pip 管理包的忽略。conda 会在每个用户家目录下创建 ``.conda`` 目录，用于管理创建的环境，而配置文件存放于 ``.condarc`` （没有可以新建）。

.. tip::

    国内源比较好的有：`清华 Conda 源`_ ，`中科大 Conda 源`_ 。

.. _清华 Conda 源: https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/
.. _中科大 Conda 源: https://mirrors.ustc.edu.cn/help/anaconda.html

使用方法：

::

    $ conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
    $ conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
    $ conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
    $ conda config --set show_channel_urls yes

若要使用官方仓库，删除 ``~/.condarc``。

.. tip::

    以上添加镜像源只针对默认 ``base`` 环境生效，因为 ``~/.condarc`` 只服务于 base。

    如果想在自己创建环境中生效，则需要在激活的环境内使用 ``conda config`` 选项 ``--env`` 重复上面添加操作。如：

    ::

        $ conda config --env --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/

    这样，会在 ``~/.conda/ennvs/${env-name}`` 下创建 ``.condarc``。

