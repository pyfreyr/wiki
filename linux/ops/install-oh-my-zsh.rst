.. _install-oh-my-zsh:

========================
oh-my-zsh 安装配置
========================

安装 zsh
=============

::

    $ sudo yum install zsh

安装 oh-my-zsh
==================

::

    $ sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"


命令自动提示
===============

::

    $ cd ~/.oh-my-zsh/custom/plugins
    $ git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions

更改显示颜色：

::

    $ vim zsh-autosuggestions/zsh-autosuggestions.zsh

    ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE='fg=10'

更新 ``.zshrc``：

::

    plugins=(zsh-autosuggestions git)


语法高亮
============

::

    $  git clone https://github.com/zsh-users/zsh-syntax-highlighting.git

更新 ``.zshrc``：

::

    $ vim ~/.zshrc

    source $ZSH_CUSTOM/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh

.. attention::

    使用 ``source ~/.zshrc`` 使配置即刻生效。
