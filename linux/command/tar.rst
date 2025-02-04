.. _tar:

=======
tar
=======

tar 命令可以为 linux 的文件和目录创建档案。利用 tar，可以为某一特定文件创建档案（备份文件），也可以在档案中改变文件，或者向档案中加入新的文件。

首先要弄清两个概念——打包和压缩：

- 打包是指将一大堆文件或目录变成一个总的文件；
- 压缩则是将一个大的文件通过一些压缩算法变成一个小文件。

为什么要区分这两个概念呢？这源于 Linux 中很多压缩程序只能针对一个文件进行压缩，这样当你想要压缩一大堆文件时，你得先将这一大堆文件先打成一个包（tar 命令），然后再用压缩程序进行压缩（gzip/bzip2 命令）。

语法
========

::

    $ tar [OPTION...] [FILE]...

选项
========


-A, -catenate    新增文件到以存在的备份文件
-B    设置区块大小
-c, --create    建立新的备份文件
-C <目录>    这个选项用在解压缩，若要在特定目录解压缩，可以使用这个选项
-d    记录文件的差别
-x, --extract, --get    从备份文件中还原文件
-t, --list    列出备份文件的内容
-z, --gzip, --ungzip    通过gzip指令处理备份文件
-Z, --compress, --uncompress    通过compress指令处理备份文件
-f <备份文件>, --file=<备份文件>    指定备份文件
-v, --verbose    显示指令执行过程
-r    添加文件到已经压缩的文件
-u    添加改变了和现有的文件到已经存在的压缩文件
-j    支持bzip2解压文件
-v    显示操作过程
-l    文件系统边界设置
-k    保留原有文件不覆盖
-m    保留文件不被覆盖
-w    确认压缩文件的正确性
-p, --same-permissions    用原来的文件权限还原文件
-P, --absolute-names    文件名使用绝对名称，不移除文件名称前的“/”号
-N <日期格式>, --newer=<日期时间>    只将较指定日期更新的文件保存到备份文件里
--exclude=<范本样式>    排除符合范本样式的文件



打包/压缩
=============

打包使用参数 ``-c``，压缩使用参数 ``-z/-j``：

::

    $ tar -cvf log.tar *.log  # 仅打包，不压缩！
    $ tar -zcvf log.tar.gz *.log  # 打包后，以 gzip 压缩
    $ tar -jcvf log.tar.bz2 *.log  # 打包后，以 bzip2 压缩

查看包内容
=============

::

    $ tar -ztvf log.tar.gz

使用 ``-t`` 参数查看包内容，如果压缩则需要带上相应压缩参数 ``-z/-j``。

解包/解压
=============

解压缩使用参数 ``-x``:

::

    $ tar -zxvf log.tar.gz

如果要指定解压的目录（目录要求存在），使用 ``-C`` 参数：

::

    $ tar -zxvf log.tar.gz -C ~/logs

以上是解压所有文件，如果只想解压部分文件在最后指定文件列表（可以先使用 ``-t`` 查看包内容）：

::

    $ tar -zxvf log.tar.gz log2012.log

