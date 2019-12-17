.. _pip-tutorial.rst:

=================
Pip 使用手册
=================

:download:`pip-cheatsheet.pdf </files/Pip-Cheatsheet.pdf>`


Pip 是 Python 官方的包管理工具。

::

    $ pip
    Usage:
      pip <command> [options]

    Commands:
      install Install packages.
      download Download packages.
      uninstall Uninstall packages.
      freeze Output installed packages in requirements format.
      list List installed packages.
      show Show information about installed packages.
      check Verify installed packages have compatible dependencies.
      search Search PyPI for packages.
      wheel Build wheels from your requirements.
      hash Compute hashes of package archives.
      completion A helper command used for command completion.
      help Show help for commands.

最常用的命令有：``install``, ``uninstall``, ``search``, ``show``, ``freeze``。

列出已安装的包
=================

``pip list`` 用于列出已安装的包，一般使用 ``--format=(legacy|columns)`` 参数设置显示样式：

::

    $ pip list --format=legacy
    absl-py (0.2.1)
    acora (2.0)
    alabaster (0.7.9)
    anaconda-clean (1.0)
    ...

    $ pip list --format=columns
    Package                            Version
    ---------------------------------- -----------
    absl-py                            0.2.1
    acora                              2.0
    alabaster                          0.7.9
    anaconda-clean                     1.0
    ...

.. tip::

    如果不想每次输入 ``--format``，参考 :ref:`更换 PyPI 镜像 <pypi-registry>` 。

也可以简单使用 ``pip freeze`` 以 requirements 文件样式列出安装的包，使用重定向可方便导出 requirements：

::

    $ pip freeze > requirements.txt

使用 list 的 ``-o`` 参数查看可升级的包：

::

    $ pip list -o


搜索包
=============

从 PyPI 镜像搜索包：

::

    $ pip search tensorflow
    tensorflow (1.8.0) - TensorFlow helps the tensors flow
    tensorflow-qndex (0.0.22) - tensorflow-qnd x tensorflow-extenteten
    tensorflow-lattice (0.9.6) - TensorFlow Lattice provides lattice models in TensorFlow
    ...


安装包
==============

使用 ``pip install`` 在线安装或安装本地包，默认安装路径为 ``site-packages``。

从 PyPI 安装
---------------

默认从官方 PyPI 源安装包，如果设置了国内镜像源，则从指定源安装。

::

    $ pip install tensorflow

安装时可以指定包的版本，如果不指定则安装最新版本：

::

    $ pip install tensorflow==1.7.0  # 指定版本
    $ pip install tensorflow>=1.7.0  # 最低版本
    $ pip install tensorflow<=1.7.0  # 最高版本

可以简单的使用 ``-i`` 指定获取包的软件源，如果不想每次都手动指定，参考 :ref:`更换 PyPI 镜像 <pypi-registry>`：

::

    $ pip install tensorflow==1.7.0 -i https://pypi.tuna.tsinghua.edu.cn/simple

从 Github 安装
---------------------

使用格式 ``git+URL`` 从 github 安装包，一般只用于未上传至 PyPI 或想要尝试最新开发版。

::

    $ pip install git+https://github.com/fxsjy/jieba

从本地安装
-------------

pip 也可以安装本地下载或本地打包的 Python 包，通常用于自己写的包和下载编译好的 wheel 包。比如安装 windows 平台编译好的 `twisted`_ ：

::

    $ pip install Twisted‑18.4.0‑cp35‑cp35m‑win_amd64.whl

.. _twisted: https://www.lfd.uci.edu/~gohlke/pythonlibs/#twisted

从 requirements 安装
------------------------

在配置生产环境时，需要安装的包较多且对版本有要求，如果一个个使用 ``pip install`` 去安装太费劲。pip 提供了从文本读取包名进行安装，只需要事先准备好 requirements.txt 文件。格式如：

::

    tqdm==4.14.0
    tornado==4.4.1
    scikit-learn==0.18.1
    tensorflow-gpu==1.0.1

安装时使用 ``-r`` 选项指定 requirements.txt：

::

    $ pip install -r requirements.txt

不使用缓存
---------------

pip 默认使用缓存安装，如果包已经下载在本地，则直接使用本地下载过的包安装。如果不想使用缓存安装，使用选项 ``--no-cache-dir``：

::

    $ pip install --no-cache-dir tensorflow



卸载包
==============

卸载包使用 ``pip uninstall``：

::

    $ pip uninstall jieba

升级包
==============

使用 ``pip install -U`` 进行包升级：

::

    $ pip install -U tensorflow

查看包的安装信息
=====================

使用 ``pip show`` 查看已安装的包的相关信息，如版本，目录等。

::

    $ pip show jieba
    Name: jieba
    Version: 0.39
    Summary: Chinese Words Segementation Utilities
    Home-page: https://github.com/fxsjy/jieba
    Author: Sun, Junyi
    Author-email: ccnusjy@gmail.com
    License: MIT
    Location: /home/chenfei/.conda/envs/tf1.7/lib/python3.5/site-packages
    Requires:

.. _pypi-registry:

更换 PyPI 镜像
==================

在 Linux 下新建/修改 ``~/.config/pip/pip.conf``：

::

    [global]
    index-url = https://pypi.tuna.tsinghua.edu.cn/simple
    format = columns

.. note::

    Mac 和 Windows 下配置文件地址参见：`configuration`_ 。

.. _configuration: https://pip.pypa.io/en/stable/user_guide/#configuration

.. tip::

    推荐 PyPI 镜像源：

    - `清华 PyPI 源`_ : ``https://pypi.tuna.tsinghua.edu.cn/simple``

    - `中科大 PyPI 源`_ : ``https://mirrors.ustc.edu.cn/pypi/web/simple``

.. _清华 PyPI 源: https://mirrors.tuna.tsinghua.edu.cn/help/pypi/
.. _中科大 PyPI 源: https://mirrors.ustc.edu.cn/help/pypi.html

如果使用豆瓣之类的 HTTP 镜像，需要使用 ``--trusted-host`` 添加信任：

::

    $ pip install jieba -i http://pypi.douban.com/simple --trusted-host pypi.douban.com

不想每次指定 ``--trusted-host`` 就在 ``pip.conf`` 添加：

::

    [global]
    index-url = http://pypi.douban.com/simple
    format = columns
    [install]
    trusted-host=pypi.douban.com


