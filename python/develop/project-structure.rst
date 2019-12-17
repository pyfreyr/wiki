.. _python-project-structure:


===================
Python 项目结构
===================



“项目目录结构” 也属于 “可读性和可维护性” 的范畴，我们设计一个层次清晰的目录结构，就是为了达到以下两点:

- **可读性高**: 不熟悉这个项目的代码的人，一眼就能看懂目录结构，知道程序启动脚本是哪个，测试目录在哪儿，配置文件在哪儿等等。从而非常快速的了解这个项目。
- **可维护性高**: 定义好组织规则后，维护者就能很明确地知道，新增的哪个文件和代码应该放在什么目录之下。这个好处是，随着时间的推移，代码/配置的规模增加，项目结构不会混乱，仍然能够组织良好。

基础目录结构
==================

比较方便快捷的目录结构如下：

::

    myproject
    ├── myproject
    │   └── __init__.py
    ├── data
    │   └── data1.dat
    ├── scripts
    │   └── rpm_install.sh
    ├── docs
    │   └── abc.rst
    ├── examples
    │   └── app.py
    ├── tests
    │   └── test_myproject.py
    ├── LICENSE
    ├── MANIFEST.in
    ├── README.rst
    ├── requirements.txt
    ├── setup.cfg
    └── setup.py

目录结构说明
=================

data/
------------

数据目录，Python 项目很少以 resources 命名数据目录。

scripts/
-----------

项目部署依赖的系统环境安装脚本。

docs/
-----------

项目文档目录，一般以 reStructureText 格式书写，最终使用 Sphinx 自动生成文档，并选择是否托管到 `readthedocs`_ 。

.. _readthedocs: https://readthedocs.org/

examples/
-------------

示例程序目录。

tests/
-------------

测试目录。

requirements.txt
--------------------

软件外部依赖 Python 包列表。

LICENSE/MANIFEST.in/setup.py/setup.cfg
------------------------------------------

项目打包，参考 `Python 打包分发工具 setuptools`_ 。

.. _Python 打包分发工具 setuptools: http://d6aa891c.wiz03.com/share/s/3mGEAs0Nqh7y2FJV1W2R6SqK2OdHe803BAmw24tTRa0wFr7Z

README.rst
---------------

项目说明文件。

.. tip::

    目录结构学习推荐项目：`django`_ ，`scrapy`_ ，`flask`_ 。

.. _django: https://github.com/django/django
.. _scrapy: https://github.com/scrapy/scrapy
.. _flask: https://github.com/pallets/flask)



