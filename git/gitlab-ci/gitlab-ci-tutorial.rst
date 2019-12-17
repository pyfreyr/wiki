.. _gitlab-ci-tutorial:

=====================
Gitlab CI 简介
=====================

.. note::

    本文只对 gitlab-ci 进行概览，不涉及集成运行环境 ``gitlab-runner`` 的搭建使用以及配置文件 ``.gitlab-ci.yml`` 的语法说明，后续会有相关专题介绍。

什么是持续集成
=================

持续集成是一种软件开发实践，即团队开发成员经常集成他们的工作，通过每个成员每天至少集成一次，也就意味着每天可能会发生多次集成。每次集成都通过自动化的构建（包括编译，发布，自动化测试）来验证，从而尽早地发现集成错误。

软件开发一般流程：编码 —— 测试 —— 封装/部署，除了代码自己实现，测试，封装和部署都可以自动化。

如何集成
=================

有各种集成工具可以使用，主流的 travis(github 默认集成工具）和 jekins，而 gitlab 自带 `Gitlab CI/CD`_ 。

.. _Gitlab CI/CD: https://docs.gitlab.com/ee/ci/README.html

.. image:: /images/gitlab-ci-pipeline.png

使用 gitlab-ci 只要自己编写 ``.gitlab-ci.yml`` 就可以，例如简单的 Python:Miniconda 环境构建：

::

    before_script:
      - docker -v

    python-3.5-centos7:
      script:
        - cd python/3.5/centos7
        - docker build -t python:3.5-centos7 .

.. attention::

    1. ``.gitlab-ci.yml`` 要放在 git 仓库的根目录，具体语法参考：`.gitlab-ci.yml 语法`_ 。
    2. 集成任务的实际运行依赖于 ``gitlab-runner``，否则任务一直处于 stuck (挂起）状态。具体参考：`gitlab-runner 安装`_ 和 `gitlab-runner 配置`_ 。

.. _.gitlab-ci.yml 语法: https://docs.gitlab.com/ee/ci/yaml/README.html
.. _gitlab-runner 安装: https://docs.gitlab.com/runner/install/
.. _gitlab-runner 配置: https://docs.gitlab.com/ee/ci/runners/README.html


.. image:: /images/gitlab-job.png



一个例子
================

gitlab-ci 集成 docker 镜像构建

仓库目录结构如下：

::

    .
    ├── .gitignore
    ├── .gitlab-ci.yml
    ├── python
    │   └── 3.5
    │       ├── centos7
    │       │   ├── docker-compose.yml
    │       │   ├── Dockerfile
    │       │   └── pip.conf
    │       ├── debian9
    │       │   ├── docker-compose.yml
    │       │   ├── Dockerfile
    │       │   └── pip.conf
    │       └── ubuntu16.04
    │           ├── docker-compose.yml
    │           ├── Dockerfile
    │           └── pip.conf
    └── README.md


只要在根目录添加 ``.gitlab-ci.yml`` 文件，分支的任何修改（提交或合并），就会触发 gitlab-ci 集成任务。可以在 CI/CD —— jobs 下查看任务执行。

.. image:: /images/gitlab-job1.png



点击 ``status`` 图标可以查看任务细节：

.. image:: /images/gitlab-job2.png
.. image:: /images/gitlab-job3.png


构建成功则 status 图标为绿色 ``passed``，否则为红色 ``failed``，同时发送错误邮件。

.. image:: /images/gitlab-job4.png










