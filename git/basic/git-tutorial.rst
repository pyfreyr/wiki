.. _git-tutorial:

===============
Git 基础教程
===============



Git 安装配置
===============


进入 Git 官网 https://git-scm.com/ 下载对应平台版本进行安装。



配置用户信息：

::

    $ git config --global user.name "you name"
    $ git config --global user.email "you_email@example.com"



使用了 ``--global`` 参数，表示本机所有的 git 仓库都会使用这个配置，当然也可以对各个仓库指定不同的用户名和邮件地址（不使用 ``--global``）。



Git 默认使用操作系统默认编辑器，通常是 Vim，如果想使用不同的文本编辑器（如 emacs）：

::

    $ git config --global core.editor emacs



使用 ``git config --list`` 列出所有 git 能找到的配置：

::

    $ git config --list

    core.symlinks=false

    core.autocrlf=true

    core.fscache=true

    color.diff=auto

    color.status=auto

    color.branch=auto





Git 配置变量存储在三个不同位置，分别有不同作用域（更小作用域有更高优先级）：



- ``/etc/gitconfig``：系统作用域，包含系统上每一个用户及他们仓库的通用配置。 如果使用 ``--system`` 选项，它会从此文件读写配置变量。

- ``~/.gitconfig`` 或 ``~/.config/git/config``：用户作用域，只针对当前用户。 可以传递 ``--global`` 选项让 git 读写此文件。

- ``.git/config``：项目作用域，只针对该仓库有效。




基本概念
============

工作区
    工作区就是要进行版本管理的目录，也就是普通的文件夹。


版本库
    当使用 ``git init`` 创建一个关于工作区的版本库时，工作区内会新建一个隐藏目录 ``.git``，这个不算工作区，而是 git 的版本库。


暂存区
    Git 的版本库里存了很多东西，其中 ``index`` 文件被称为暂存区（stage），有时也叫做索引（index）。


.. attention::

    首次 ``git add`` 之后才会出现 index 文件。



工作区、暂存区和版本库的关系
-------------------------------


暂存区是工作区和版本库的桥梁，是一个临时存储的地方，所有的修改经由暂存区，再提交到版本库。好处是可以在真正提交到版本库之前做任意的操作，在需要真正提交的时候提交到版本库。


Git 的三种状态
-----------------------

Git 有三种状态，你的文件可能处于其中之一：

- 已修改(modified)
- 已暂存(staged)
- 已提交(committed)

通过上面工作区、暂存区和版本库的概念，可以很容易理解：

- **已修改** 表示修改了工作区的文件，但还没保存到版本库中；
- **已暂存** 表示对一个已修改文件的当前版本做了标记，将文件的快照放入暂存区；
- **已提交** 表示暂存区的文件已经永久的保存在本地版本库中。

使用 git 时文件的生命周期如图：


.. image:: /images/git-lifetime.png



基础操作
=============


创建版本库
---------------


在需要版本控制的任何目录内使用命令 ``git init``，以 learning-git 目录为例：

::

    $ git init

    Initialized empty Git repository in /Users/freyr/selfRepo/learning-git/.git/



在空目录内创建 git 仓库，结果显示如上: initialized empty Git repository。



查看工作区和暂存区状态
--------------------------



``git status`` 命令用于显示工作区和暂存区的状态，能看到哪些修改被暂存，哪些文件没有被追踪（tracked，如新建的文件）：

::

    $ git status

    On branch master



    No commits yet



    nothing to commit (create/copy files and use "git add" to track)



因为版本库新创建，工作区和暂存区都为空，所以没有修改需要提交。


.. tip::

    使用 ``git status`` 的 ``-s`` 参数简洁输出状态。


    ::

        $ git status -s

         M README.md

        MM Rakefile

        A lib/git.rb

        M lib/simplegit.rb

        ?? LICENSE.txt



把文件修改添加到版本库
---------------------------



在工作区进行一些操作，比如创建文件 README.md：

::

    $ touch README.md



查看状态，提示有未追踪的文件（Untracked files）：

::

    $ git status

    On branch master



    No commits yet



    Untracked files:

      (use "git add <file>..." to include in what will be committed)



     README.md



    nothing added to commit but untracked files present (use "git add" to track)



Git 进行版本控制的操作分两步：



第一步，把 **文件修改** 添加进暂存区，使用命令 ``git add``:


.. image:: /images/git-add.png

::

    $ git add README.md  # 如果有多个文件，可以使用 git add . 添加整个目录内所有文件



查看状态未出现 Untracked，说明文件已经被追踪：

::

    $ git status

    On branch master



    No commits yet



    Changes to be committed:

      (use "git rm --cached <file>..." to unstage)



     new file: README.md




第二步，把暂存区的内容提交到当前分支（Git 默认自动创建 master 分支，分支概念后面介绍），使用命令 ``git commit``：


.. image:: /images/git-commit.png

::

    $ git commit -m 'add README.md'  # -m 参数添加本次提交的说明，方便以后查找到本次改动

    [master (root-commit) 3b83f4f] add README.md

     1 file changed, 0 insertions(+), 0 deletions(-)

     create mode 100644 README.md



显示一个文件被改动（1 changed，因为未有内容增删，所以 insertions/deletions 都为 0），查看状态显示工作区已干净。

::

    $ git status

    On branch master

    nothing to commit, working tree clean



.. note::

    最终提交的是文件修改，因为 git 跟踪管理的是修改，而非文件。比如新增一行、删除一行、更改某些字符、新增文件、删除文件等这些都是修改。


.. tip::

    1. 可以多次 add 不同的文件到暂存区，然后一次 commit 提交。
    2. 使用 ``git commit -am`` 同时添加暂存并提交（相当于 ``git add`` + ``git commit``）



到此，文件已经被成功提交到版本库，可以使用命令 ``git log`` 查看提交记录：

::

    $ git log

    commit 3b83f4fc79f99d2749c058a713233b6928400435 (HEAD -> master)

    Author: freyr <mrchan3030@foxmail.com>

    Date: Sun May 27 11:35:55 2018 +0800



        add README.md



可以看到提交的 ID、分支名、作者、时间和提交说明。



撤销修改
--------------



使用版本库的第一个好处是：可以随时撤销修改，恢复到上一次提交版本库的文件状态。



由于工作区、暂存区、版本库的存在，对应的修改也会发生在这三个地方。



撤销工作区修改
~~~~~~~~~~~~~~~~~~


.. note::

    **场景**：工作区修改没有 add，没有 commit



在 README 中增加一行内容：

::

    咸豆脑好吃



再查看下版本库：

::

    $ git status

    On branch master

    Changes not staged for commit:

      (use "git add <file>..." to update what will be committed)

      (use "git checkout -- <file>..." to discard changes in working directory)



        modified: README.md



    no changes added to commit (use "git add" and/or "git commit -a")



显示文件已经发生修改，可以使用 ``git checkout --`` 撤销修改：

::

    $ git checkout -- README.md



再查看状态可以看到干净的工作区。


.. attention::

    对于工作区单个文件，checkout 后的 ``--`` 不能少！（不然就变成切换分支）


.. tip::

    对于工作区的修改：

    - 撤销 tracked 的单个文件使用 ``git checkout -- <file>``；
    - 撤销 tracked 的所有文件使用 ``git checkout .``；
    - 撤销 untracked 文件使用 ``git clean -df``。





撤销暂存区修改
~~~~~~~~~~~~~~~~~~


.. note::

    **场景**：工作区修改已经 add，但还没有 commit



在 README 中新增如下一行，并 add 到暂存区：

::

    坚决拥护甜的！！表示无法想象咸豆脑是神马样的！



查看状态：

::

    $ git status

    On branch master

    Changes to be committed:

      (use "git reset HEAD <file>..." to unstage)



        modified: README.md



提示显示可以使用 ``git reset HEAD`` 把暂存区的修改退回到工作区：

::

    $ git reset HEAD README.md

    Unstaged changes after reset:

    M README.md



此时暂存区的修改已经撤销，修改只存在于工作区（回到了上一个场景），如果需要继续撤销工作区修改，则参考上一小节。

.. tip::

    要撤销已提交到暂存区和工作区的修改，需要分两步：

    1. ``git reset HEAD`` 撤销暂存区修改；
    2. ``git checkout --`` 撤销工作区修改。



撤销版本库修改: 版本回退
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


.. note::

    **场景**：修改已经 add，且 commit 到版本库



在 README 中添加如下内容，并提交到 git 仓库。

::

    为了构建社会主义和谐社会，建议豆腐脑加打卤加白糖姜汁，各自让一步。



查看提交日志：

::

    $ git log

    commit 466938fc4266e0ed6061ae7c831d61c0936ebd4e (HEAD -> master)

    Author: freyr <mrchan3030@foxmail.com>

    Date: Sun May 27 12:25:44 2018 +0800



        update README.md



    commit 3b83f4fc79f99d2749c058a713233b6928400435

    Author: freyr <mrchan3030@foxmail.com>

    Date: Sun May 27 11:35:55 2018 +0800



        add README.md



对于这种看似最棘手的状况，其实处理起来反而很简单。只需要使用 ``git reset`` 进行版本回退！

::

    $ git reset --hard HEAD^

    HEAD is now at 3b83f4f add README.md



.. tip::

    1. 一个 ``^`` 表示一个版本，可以多个（如 ``HEAD^^``）；另外也可以使用 ``HEAD~n`` 形式；或者直接指定 commit id；

    2. reset 选项：

        - ``--mixed``: reset HEAD and index(default)
        - ``--soft``: reset only HEAD
        - ``--hard``: reset HEAD, index and working tree





远程仓库：GitHub
======================



目前我们使用到的 git 命令都是在本地执行，如果你想通过 Git 分享你的代码或者与其他开发人员合作，就需要将数据放到一台其他开发人员能够连接的服务器上。



Github 是一个基于 git 的代码托管平台，付费用户可以建私人仓库，一般的免费用户只能使用公共仓库，也就是代码要公开。




创建远程库
----------------



进入 https://github.com/ 注册账号，回到主页点击 **Start a project** 进入代码仓库创建页面：


.. image:: /images/github-create.png


依次填写、勾选、确认，完成创建。

.. image:: /images/github-done.png



注意上面的 URL 就是远程仓库的地址。




添加 SSH Key
----------------



在家目录下创建 SSH Key：

::

    $ ssh-keygen -t rsa -C "you_email@example.com"



一路回车使用默认值，在家目录的 ``.ssh`` 子目录里新增 ``id_rsa`` 和 ``id_rsa.pub`` 两个文件（SSH Key 密钥对），``id_rsa`` 是私钥，不能泄露出去；``id_rsa.pub`` 是公钥，可以放心告诉别人。



登录 Github，进入 settings —— SSH and GPG keys —— New SSH key，将 ``id_rsa.pub`` 的内容粘贴到 Key 内完成添加 SSH key。

.. image:: /images/github-ssh.png



使用如下目录验证是够成功：

::

    $ ssh -T git@github.com

提示 “You've successfully authenticated, but GitHub does not provide shell access.” 表示成功连上 github。


与远程库关联
-----------------



在要上传的仓库内，使用如下命令添加远程地址（替换为自己的仓库地址）：

::

    $ git remote add origin https://github.com/ifreyr/learning-git.git



查看远程分支
-----------------



使用 ``git remote`` 查看远程库信息：

::

    $ git remote

    origin



可以使用 ``-v`` 显示详细信息：

::

    $ git remote -v

    origin	https://github.com/ifreyr/learning-git.git (fetch)

    origin	https://github.com/ifreyr/learning-git.git (push)




推送本地分支到远程
----------------------



然后使用 ``git push`` 将本地分支的更新推送到远程：

::

    $ git push -u origin master

    Counting objects: 3, done.

    Writing objects: 100% (3/3), 217 bytes | 217.00 KiB/s, done.

    Total 3 (delta 0), reused 0 (delta 0)

    To https://github.com/ifreyr/learning-git.git

     * [new branch] master -> master

    Branch master set up to track remote branch master from origin.


.. tip::

    使用 ``-u`` 选项指定一个默认主机（因为本地分支可能与多个主机存在关系），这样以后就可以不加任何参数使用 ``git push`` 了：

::

    $ git push origin master



如果当前分支只有一个追踪分支，主机名和分支名可以省略：

::

    $ git push



现在，Github 项目主页就和本地仓库一样了：


.. image:: /images/github-master.png



拉取远程分支到本地
-----------------------



在多人协作下，远程分支更新通常都超前于本地分支，所以需要使用 ``git pull`` 从远程拉取最新更新到本地：

::

    $ git pull origin master



同样在只有一个追踪分支时可以省略主机和分支名：

::

    $ git pull


.. note::

    ``git pull`` 命令的作用是：取回远程主机某个分支的更新，再与本地的指定分支合并。



在默认模式下，``git pull`` 使用给定的参数运行 ``git fetch``，并调用 ``git merge`` 将检索到的分支头合并到当前分支中。 使用 ``--rebase``，它运行 ``git rebase`` 而不是 ``git merge``。



在实际使用中，``git fetch`` 更安全一些，因为在 merge 前，我们可以查看更新情况，然后再决定是否合并。


.. note::

    git fetch 和 git pull 区别：

    1. ``git fetch`` 从远程获取最新版本到本地，不会自动合并。

        ::

            $ git fetch origin master

            # 下载 origin 的 master 主分支到本地 ``origin/master`` 分支上



    2. ``git pull`` 从远程获取最新版本并 merge 到本地

        ::

            $ git pull origin master

            # 下载远程 master 分支到本地 ``origin/master`` 并合并到本地 master





从远程库克隆
-----------------



前面介绍了先有本地库，后有远程库，如何关联本地到远程。



如果项目从零开始，最好的方式是先创建远程库，然后从远程库克隆。使用命令 ``git clone``：

::

    $ git clone https://github.com/ifreyr/learning-git.git





远程操作示意图
-------------------



.. image:: /images/git-remote.jpg




多人协作：分支
==================



分支就是科幻电影里面的平行宇宙，当你正在电脑前努力学习 git 的时候，另一个你正在另一个平行宇宙里努力学习 SVN。



如果两个平行宇宙互不干扰，那对现在的你也没啥影响。不过，在某个时间点，两个平行宇宙合并了，结果，你既学会了 git 又学会了 SVN！


.. image:: /images/git-svn.png


几乎每一种版本控制系统都以某种形式支持分支。使用分支意味着你可以从开发主线上分离开来，然后在不影响主线的同时继续工作。例如，我们发布了 1.0 版本的产品，可能需要创建一个分支，以便将 2.0 功能的开发与 1.0 版本中错误修复分开。

每次提交，git 都把它们串成一条时间线。这条时间线就是一个分支。当创建版本库的时候，默认就创建了主分支 ``master``。Git 使用 ``master`` 指向最新的提交，再用 ``HEAD`` 指向 ``master``，就能确定当前的分支，以及当前分支的提交点：

.. image:: /images/git-branch01.png

每次提交，``master`` 分支都会向前移动一步，随着不断的提交，``master`` 分支的时间线也越来越长。

列出分支
-----------

使用 ``git branch`` 列出本地分支：

::

    $ git branch
    * master

显示目前只有一个分支 ``master``，且 ``*`` 表示当前处于 ``master`` 分支。

.. tip::

    使用 ``-a`` 选项能同时显示本地和远程分支。


创建分支
--------------



使用命令 ``git branch`` 创建分支：

::

    $ git branch dev



列出分支：

::

    $ git branch

      dev

    * master



当创建新分支 ``dev`` 时，git 新建了一个指针 ``dev``，指向 ``master`` 相同的提交：



.. image:: /images/git-branch02.png



切换分支
-------------



使用命令 ``git checkout`` 切换分支：

::

    $ git checkout dev

    Switched to branch 'dev'


.. tip::

    可以使用 ``git checkout -b`` 创建并切换到该分支（相当于 ``git branch`` + ``git checkout``）。



查看当前分支：

::

    $ git branch

    * dev

      master



当从 ``master`` 分支切换到 ``dev`` 分支时，git 把 ``HEAD`` 指向 ``dev``，就表示当前分支在 ``dev`` 上：

.. image:: /images/git-branch03.png



Git 创建和切换分支很快，因为除了增加一个 ``dev`` 指针，修改下 ``HEAD`` 的指向，工作区的文件没有任何变化。

从现在开始，对工作区的修改和提交都是针对 ``dev`` 分支，比如新提交一次后，``dev`` 指针往前移动一步，而 ``master`` 指针不变：

.. image:: /images/git-branch04.png



合并分支
--------------



如果在 ``dev`` 上工作完成，就可以把 ``dev`` 合并到 ``master`` 上。具体操作分两步：



第一步，切换回 ``master`` 分支：

::

    $ git checkout master

    Switched to branch 'master'

    Your branch is up-to-date with 'origin/master'.



第二步，合并 ``dev`` 分支到 ``master``：

::

    $ git merge dev

    Updating 3b83f4f..78c9a3c

    Fast-forward

     README.md | 1 +

     1 file changed, 1 insertion(+)



以上合并操作直接把 ``master`` 指向 ``dev`` 的当前提交：

.. image:: /images/git-branch05.png



所以，git 合并分支也很快，就改改指针，工作区内容也不变。

合并完成后，就可以使用 ``git branch -d`` 删除 dev 分支（还是 ``git branch`` 命令，加了 ``-d`` 选项）：

::

    $ git branch -d dev
    Deleted branch dev (was 78c9a3c).

删除分支就是把 ``dev`` 指针给删掉，删掉后就只剩下一条 ``master`` 分支：


.. image:: /images/git-branch06.png



解决冲突
------------



合并分支往往也不是一帆风顺，当多分支同时修改统一文件，合并时就会出现冲突。



创建分支 ``feature1`` 并修改 README 内容为：

::

    甜豆脑好吃



而 master 分支修改 README 内容为：

::

    咸豆脑好吃



现在，``master`` 和 ``feature1`` 分支各自都有新的提交：


.. image:: /images/git-branch07.png



这种情况下，git 只能试图把各自的修改合并起来，但这种合并就可能会有冲突：

::

    $ git merge feature1

    Auto-merging README.md

    CONFLICT (content): Merge conflict in README.md

    Automatic merge failed; fix conflicts and then commit the result.



提示有冲突，查看 README：

::

    $ cat README.md

    <<<<<<< HEAD

    咸豆脑好吃

    =======

    甜豆脑好吃

    >>>>>>> feature1



Git用 ``<<<<<<<``，``=======``，``>>>>>>>`` 标记出不同分支的内容。修改 README 如下：

::

    咸豆脑好吃

    甜豆脑也好吃



解决好冲突再提交：

::

    $ git commit -am 'conflict fixed'

    [master d88e6b6] conflict fixed



现在，``master`` 和 ``feature1`` 分支变成：


.. image:: /images/git-branch08.png

.. tip::

    合并分支时，加上 ``--no-ff`` 参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而默认使用的 ``fast forward`` 合并就看不出来曾经做过合并。


删除分支
--------------

删除本地分支：

::

    $ git branch -d ${branch-name}

删除远程分支：

::

    $ git  push origin -d ${branch-name}



分支管理策略
----------------



在实际开发中，我们应该按照几个基本原则进行分支管理：


1. ``master`` 分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；
2. ``develop`` 分支是不稳定的，到某个时候，比如 1.0 版本发布时，再把 ``develop`` 分支合并到 ``master`` 上，在 ``master`` 分支发布 1.0 版本；
3. 团队成员在开发时只与 ``develop`` 分支打交道，在 ``develop`` 上创建各种分支（``feature``, ``bugfix``, ``hotfix``）。在开发完成后合并到 ``develop`` 分支并删除功能分支。



所以，团队合作的分支看起来就像这样：


.. image:: /images/git-flow.png





多人协作
--------------



多人协作的工作模式通常是这样：


1. 试图用 ``git push`` 推送自己的修改；
2. 如果推送失败，则因为远程分支比你的本地更新，需要先用 ``git pull`` 试图合并；
3. 如果合并有冲突，则解决冲突，并在本地提交；
4. 没有冲突或者解决掉冲突后，再用 ``git push`` 推送就能成功。


在本地创建和远程分支对应分支
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~



本地和远程分支名称最好一样。

::

    $ git checkout -b branch-name origin/branch-name



关联本地分支和远程分支
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~



如果 ``git pull`` 提示 **no tracking information**，则说明本地分支和远程分支的链接关系没有创建：

::

    $ git branch --set-upstream branch-name origin/branch-name






版本库快照：标签
======================



发布一个版本时，我们通常先在版本库中打一个标签（tag），这样，就唯一确定了打标签时刻的版本。将来无论什么时候，取某个标签的版本，就是把那个打标签的时刻的历史版本取出来。所以，标签也是版本库的一个快照。



Git 的标签虽然是版本库的快照，但其实它就是指向某个 commit 的指针（跟分支很像，但是分支可以移动，标签不能移动），所以，创建和删除标签都是瞬间完成的。



标签就是一个让人容易记住的有意义的名字，它跟某个 commit 绑在一起。因为标签总是和某个 commit 挂钩。如果这个 commit 既出现在两个分支上，那么在这两个分支上都可以看到这个标签。



创建标签
---------------

在需要打标签的分支上使用 ``git tag`` 创建标签：

::

    $ git tag v1.0



以上默认在最新提交上打标签，可以使用 commit id 将标签打在历史提交上：

::

    $ git tag v0.1 78c9



可以创建带有说明的标签，用 ``-a`` 指定标签名，``-m`` 指定说明文字：

::

    $ git tag -a v0.2 -m 'version 0.2 released' d88e



查看标签
-------------

使用命令 ``git tag`` 查看所有标签：

::

    $ git tag

    v0.1

    v0.2

    v1.0


.. attention::

    标签不是按时间顺序列出，而是按字母排序的。



可以用 ``git show`` 查看标签详细信息：

::

    $ git show v0.1

    commit 78c9a3c63ed6c62cd4175cbf1ed7c29cd64b300a (tag: v0.1)

    Author: freyr <mrchan3030@foxmail.com>

    Date: Sun May 27 21:54:18 2018 +0800



        add title to README.md



    diff --git a/README.md b/README.md

    index e69de29..a125ae6 100644

    --- a/README.md

    +++ b/README.md

    @@ -0,0 +1 @@





推送标签到远程
-------------------



使用命令 ``git push`` 推送标签到远程：

::

    $ git push origin v1.0

    Counting objects: 13, done.

    Delta compression using up to 8 threads.

    Compressing objects: 100% (5/5), done.

    Writing objects: 100% (13/13), 1.01 KiB | 1.01 MiB/s, done.

    Total 13 (delta 2), reused 0 (delta 0)

    remote: Resolving deltas: 100% (2/2), done.

    To https://github.com/ifreyr/learning-git.git

     * [new tag] v1.0 -> v1.0


.. tip::

    多个未推送的本地标签，可以一次性推送到远程：

    ::

        $ git push origin --tags

        Counting objects: 1, done.

        Writing objects: 100% (1/1), 166 bytes | 166.00 KiB/s, done.

        Total 1 (delta 0), reused 0 (delta 0)

        To https://github.com/ifreyr/learning-git.git

         * [new tag] v0.1 -> v0.1

         * [new tag] v0.2 -> v0.2





删除标签
------------

删除本地标签
~~~~~~~~~~~~~~~~



如果标签打错了，可以使用 ``git tag -d`` 删除：

::

    $ git tag -d v0.1

    Deleted tag 'v0.1' (was 78c9a3c)



删除远程标签
~~~~~~~~~~~~~~~~~~



删除远程标签需要两步操作：

第一步，删除本地标签：

::

    $ git tag -d v0.1



第二步，使用 ``git push`` 删除远程标签：

::

    $ git push origin :refs/tags/v0.1

    To https://github.com/ifreyr/learning-git.git

     - [deleted] v0.1





其他主题
==============



储藏
--------------



经常有这样的事情发生，当你正在进行项目中某一部分的工作，里面的东西处于一个比较杂乱的状态，而你想转到其他分支上进行一些工作。问题是，你不想提交进行了一半的工作，否则以后你无法回到这个工作点。解决这个问题的办法就是 ``git stash`` 命令。



“‘储藏”可以获取你工作目录的中间状态——也就是你修改过的被追踪的文件和暂存的变更——并将它保存到一个未完结变更的堆栈中，随时可以重新应用。

::

    $ git stash

    Saved working directory and index state \

      "WIP on master: 049d078 added the index file"

    HEAD is now at 049d078 added the index file

    (To restore them type "git stash apply")



这时，工作区就干净了，你可以方便地切换到其他分支工作；你的变更都保存在栈上。要查看现有的储藏，你可以使用 ``git stash list``：

::

    $ git stash list

    stash@{0}: WIP on master: 049d078 added the index file

    stash@{1}: WIP on master: c264051 Revert "added file_size"

    stash@{2}: WIP on master: 21d80a5 added number to log



使用命令 ``git stash apply`` 就可以重新应用你刚刚实施的储藏：

::

    $ git stash apply

    # On branch master

    # Changes not staged for commit:

    # (use "git add <file>..." to update what will be committed)

    #

    # modified: index.html

    # modified: lib/simplegit.rb

    #





如果你想应用更早的储藏，你可以通过名字指定它，如果你不指明，git 默认使用最近一次的储藏：

::

    $ git stash apply stash@{2}



``apply`` 选项只尝试应用储藏的工作，储藏的内容仍然在栈上。可以使用 ``git stash drop`` 从栈上移除：

::

    $ git stash drop stash@{0}

    Dropped stash@{0} (364e91f3f268f0900bc3ee613f9f733e82aaed43)


.. tip::

    可以使用 ``git stash pop`` 来重新应用储藏，同时立刻将其从堆栈中移走。



忽略特殊文件
---------------



有些时候，你必须把某些文件放到Git工作目录中，但又不能提交它们，比如保存了数据库密码的配置文件。在 Git 工作区的根目录下创建一个特殊的 ``.gitignore`` 文件，然后把要忽略的文件名填进去，Git 就会自动忽略这些文件。



不需要从头写 ``.gitignore``，Github 已经准备了各种配置文件：



- `github/gitignore`_

- `Python.gitignore`_

.. _github/gitignore: https://github.com/github/gitignore

.. _Python.gitignore: https://github.com/github/gitignore/blob/master/Python.gitignore)



有些时候，你想添加一个文件到 git，但发现添加不了，原因可能是这个文件被 ``.gitignore`` 忽略了，可以使用 ``git check-ignore`` 检查看看：

::

    $ git check-ignore -v conf.json

    .gitignore:3:*.json conf.json



可以修改 ``.gitignore`` 文件或者使用 ``-f`` 强制添加：

::

    $ git add -f conf.json





git rebase
--------------


.. tip::

    本小节图片箭头和通常时间线方向相反，如果不适可以找找其他教程。



假设你现在基于远程分支 origin ，创建一个叫 mywork 的分支：

.. image:: /images/git-rebase01.png


现在在 ``origin`` 和 ``mywork`` 两个分支上分别开发：


.. image:: /images/git-rebase02.png


通常使用 ``git merge`` 合并，结果如图：

.. image:: /images/git-rebase03.png


但是，如果你想让 ``mywork`` 分支历史看起来像没有经过任何合并一样，也可以用 ``git rebase``，当 ``mywork`` 分支更新之后，它会指向这些新创建的提交，而那些老的提交会被丢弃。：

::

    $ git checkout mywork  # 在 mywork 上执行 rebase

    $ git rebase origin

.. image:: /images/git-rebase04.png
.. image:: /images/git-rebase05.png
.. image:: /images/git-rebase06.png
.. image:: /images/git-rebase07.png


在 rebase 的过程中，也许会出现冲突。在这种情况，git 会停止 rebase 并会让你去解决冲突；在解决完冲突后，用 ``git add`` 命令去更新这些内容的索引(index), 然后，你无需执行 ``git commit``，只要执行：

::

    $ git rebase --continue



参考资料
==============

1. `Git Documentation`_

2. `Git Book 中文版`_

3. `Git 教程（廖雪峰）`_

4. `Git 教程（易百教程）`_

5. `Git 教程（菜鸟教程）`_


.. _Git Documentation: https://git-scm.com/docs
.. _Git Book 中文版: https://git-scm.com/book/zh/v2
.. _Git 教程（廖雪峰）: https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000
.. _Git 教程（易百教程）: https://www.yiibai.com/git/
.. _Git 教程（菜鸟教程）: http://www.runoob.com/git/git-tutorial.html



























