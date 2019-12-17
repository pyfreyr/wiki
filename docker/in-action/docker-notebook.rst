.. _docker-notebook:

====================================
Docker 运行 IPython Notebook
====================================


使用 Docker 构建 notebook 镜像，Dockerfile 如下：

::

    FROM registry.cn-beijing.aliyuncs.com/freyr/python:ubuntu16.04

    RUN pip install jupyter

    ADD requirements.txt .

    RUN pip install -r requirements.txt && rm -rf requirements.txt

    ADD jupyter_notebook_config.py .

    RUN mkdir ~/.jupyter && mv jupyter_notebook_config.py ~/.jupyter

    WORKDIR /notebook

    CMD ["/bin/bash"]


其中，``jupyter_notebook_config.py`` 为 jupyter 配置文件，可通过以下命令在本地生成（路径 ``~/.jupyter/``）

::

    $ jupyter notebook --generate-config

主要修改参数：

::

    c.NotebookApp.allow_root = True
    c.NotebookApp.ip = '*'
    c.NotebookApp.notebook_dir = '/notebook'
    c.NotebookApp.token = ''

Docker-compose 文件如下：

::

    version: '3'
    services:
      notebook:
        build: .
        image: notebook
        container_name: notebook
        ports:
          - 8888:8888
        volumes:
          - $PWD/notebooks:/notebook
        command: jupyter notebook

使用如下命令启动容器：

::

    $ docker-compose up -d