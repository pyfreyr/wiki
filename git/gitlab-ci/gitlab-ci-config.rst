.. _gitlab-ci-config:

===============================
Gitlab CI 配置文件使用指南
===============================


Gitlab CI 使用 ``.gitlab-ci.yml`` 文件进行持续集成管理，通过使用一组预定义的指令完成所有控制。

执行单元 Job
===============

Job 是 gitlab-ci 的最小执行单元，所有的操作都被合理划分在各个 job 内，通过 ``script`` 指令控制执行。

.. attention::

    Job 必须包含 script 指令。

下面定义了两个任务：

::

    job1:
      script: "execute-script-for-job1"

    job2:
      script: "execute-script-for-job2"

.. note::

    Job 名称可以随意取，除了 gitlab 保留字：image, services, stages, types, before_script, after_script, variables, cache。


script
============

在 ``script`` 指令中，可以使用纯 shell 命令或执行 shell 脚本。这也是集成工作 **最核心** 的部分（考察的是 shell 脚本能力）。

::

    job1:
      script: "bundle exec rspec"

    job2:
      script:
        - uname -a
        - bundle exec rspec

通过定义 job 并写好 script，我们已经能完成了最基本的集成工作。下面的指令都是为了增加扩展和功能。

before_script & after_script
================================

如果多个 jobs 有一些共同的操作，除了在各自 ``script`` 里都添加之外，我们有更好的方法：

- 使用 ``before_script`` 添加预处理，如环境的配置
- 使用 ``after_script`` 完成后处理，如资源的清理

.. note::

    `before_script/after_script` 可同时用于全局或 Job 之内，Job 内会覆盖全局的设置。

    ::

        before_script:
          - global before script

        job:
          before_script:
            - execute this instead of global before script
          script:
            - my command
          after_script:
            - execute this after my script

stages & stage
==================

为了使多个 Jobs 好管理，一种好的实践是使用 ``stages`` 将任务按照操作流程进行划分。Gitlab 默认提供了 ``build``, ``test``, ``deploy`` 三种 stage，我们自己可以根据需要进行扩充。

在全局定义 ``stages``：

::

    stages:
      - build
      - test
      - deploy

.. attention::

    ``stages`` 是有顺序的：

        - 靠前 ``stage`` 的 jobs 会优先执行，同一级的 jobs 会并行执行
        - 默认情形下只有前面的 jobs 都执行成功才执行下一级（如 ``build`` 的所有 jobs 都成功才执行 ``test`` 的 jobs，只要有一个失败集成任务就失败了）。

定义好的 ``stages`` 可以在各个 jobs 内通过 ``stage`` 指令进行使用：

::

    stages:
      - build
      - test
      - deploy

    job 1:
      stage: build
      script: make build dependencies

    job 2:
      stage: build
      script: make build artifacts

    job 3:
      stage: test
      script: make test

    job 4:
      stage: deploy
      script: make deploy

tags
=========

Tags 用于指定本项目的执行 Runner，对 gitlab-runner 的使用可参考 :ref:`Gitlab Runner 安装使用 <gitlab-runner-tutorial>` 。

下面任务指定了同时标记有 ruby 和 postgres 的 Runner 执行集成：

::

    job:
      tags:
        - ruby
        - postgres

variables
===============

可以在 ``.gitlab-ci.yml`` 中定义变量进行重复使用，这就需要 ``variables`` 指令。同样的，可以在全局和 Job 内使用，且 Job 会覆盖全局相同的变量。

::

    variables:
      VERSION_SUFFIX: v1.0.0

    job:
      script:
        - VERSION="${VERSION_SUFFIX}.${CI_JOB_ID}"

.. note::

    gitlab-ci 也预定义了一些 `环境变量`_ ，上面的 ``CI_JOB_ID`` 就是其一。

.. _环境变量: https://docs.gitlab.com/ee/ci/variables/README.html#predefined-variables-environment-variables

allow_failure
===============

在 ``stages`` 部分提到只有前一个 stage 的所有任务都成功，才会执行下一 stage 的任务。这在绝大部分时候是合适的，但有些时候我们需要弱化一下规则，忽略一些不太重要的任务（如某个测试）。

``allow_failure`` 允许整个 pipeline 出现一些小的失败，下面例子中，job1 和 job2 同时执行，但如果 job1 失败，job3 仍然能够继续执行：

::

    job1:
      stage: test
      script:
        - execute_script_that_will_fail
      allow_failure: true

    job2:
      stage: test
      script:
        - execute_script_that_will_succeed

    job3:
      stage: deploy
      script:
        - deploy_to_staging

when
========

``allow_failure`` 在自己 job 内控制对外的影响（自己出错了要不要影响他人），类似的可以使用 ``when`` 控制自己对上层 jobs 的响应（他人出错了自己如何面对）。

``when`` 指令可选值包括：

- ``on_success``，只有上层 jobs 全成功才执行
- ``on_failure``，只要上层有一个失败就执行
- ``always``，忽略上层执行结果，总是执行
- ``manual``，手动控制任务的执行（一般用在项目部署）

下面例子中，cleanup_build_job 只有在 build_job 失败的时候才会执行，cleanup_job 总是会执行，而 deploy_job 手动执行：

::

    stages:
      - build
      - cleanup_build
      - test
      - deploy
      - cleanup

    build_job:
      stage: build
      script:
        - make build

    cleanup_build_job:
      stage: cleanup_build
      script:
        - cleanup build when failed
      when: on_failure

    test_job:
      stage: test
      script:
        - make test

    deploy_job:
      stage: deploy
      script:
        - make deploy
      when: manual

    cleanup_job:
      stage: cleanup
      script:
        - cleanup after jobs
      when: always

retry
==========

当任务执行失败，可以使用 ``retry`` 进行重试：

::

    test:
      script: rspec
      retry: 2


only & except
=================

``only`` 和 ``except`` 用于控制代码分支和 tag 的选择：

- ``only`` 圈定的分支和 tags 会执行该任务
- ``except`` 圈定的分支和 tags 不会执行该任务


二者使用的一些规则：

- ``only`` 和 ``except`` 可以同时使用，最终取条件的交集
- ``only`` 和 ``except`` 都支持正则表达式
- ``only`` 和 ``except`` 可以指定代码仓库地址

示例：

::

    job1:
      # 只允许以 issue 开始的 refs，忽略所有分支
      only:
        - /^issue-.*$/
      # use special keyword
      except:
        - branches

    job2:
      # 运行 gitlab-org/gitlab-ce 上所有分支，排除 master 分支
      only:
        - branches@gitlab-org/gitlab-ce
      except:
        - master@gitlab-org/gitlab-ce

image & services
===================

``image`` 指定镜像名称，``services`` 提供其他镜像（通常为数据库），使得运行任务时主镜像能连接到这些镜像。``image`` 和 ``services`` 支持全局和 Job 级定义。

::

    before_script:
      - bundle install

    test:2.1:
      image: ruby:2.1
      services:
      - postgres:9.3
      script:
      - bundle exec rake spec

    test:2.2:
      image: ruby:2.2
      services:
      - postgres:9.4
      script:
      - bundle exec rake spec

其他指令
============

Gitlab 提供的其他指令，并不太常用，有需要的可以自行查看文档 `Configuration`_ 。

- environment
- cache
- artifacts
- dependencies
- coverage
- include

.. _Configuration: https://docs.gitlab.com/ee/ci/yaml/README.html

一个较完整的 .gitlab-ci.yml
==============================

::

    image: docker:stable

    variables:
      PROJECT: kerberos
      REPOSITORY: username/${PROJECT}
      REGISTRY_USER: username
      REGISTRY_TOKEN: password
      VERSION_SUFFIX: py35-centos7-v1.0.0

    before_script:
      - docker login -u ${REGISTRY_USER} -p ${REGISTRY_TOKEN}

    stages:
      - build

    kerberos:
      stage: build
      tags:
        - docker
      script:
        - VERSION="${VERSION_SUFFIX}.${CI_JOB_ID}"
        - IMAGE_NAME="${REPOSITORY}:${VERSION}"
        - cd kerberos/py35/centos7
        - docker build --network=host -t ${IMAGE_NAME} .
        - docker push ${IMAGE_NAME}
        - docker rmi ${IMAGE_NAME}

    after_script:
      - docker logout











