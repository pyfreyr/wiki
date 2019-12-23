=======================
reStructuredText Primer
=======================

Sphinx 默认使用 reStructuredText 标记语言，但在原 reST 基础上进行了语法扩展。


Paragraph
=========

段落（`Paragraph`_）是由空行分隔的一段文本。
和 Python 一样，对齐也是 reST 的操作符，因此同一段落的行都是左对齐的。

.. _Pargraph: https://docutils.sourceforge.io/docs/ref/rst/restructuredtext.html#paragraphs

::

    This is a paragraph.

    Paragraphs line up at their left
    edges, and are normally separated
    by blank lines.

Inline markup
=============

标准的 reST 行内标记（`Inline Markup`_）相当简单：

- ``*`` for emphasis（斜体）
- ``**`` for strong emphasis（粗体）
- ````` for interpreted text（用于程序标识符）
- `````` for code samples

.. _Inline Markup: https://docutils.sourceforge.io/docs/ref/rst/restructuredtext.html#inline-markup

::

    This is *emphasized text*.

    This is **strong text**.

    This is `interpreted text`.

    The regular expression ``[+-]?(\d+(\.\d*)?|\.\d+)`` matches
    floating-point numbers (without exponents).

.. note::

    - 行内标记不能嵌套
    - 标记内容首尾不能包含空白字符，如 ``* text*`` 是错误的
    - 使用反斜杠可以对标记符号进行转义


reST 也允许自定义 “文本解释角色”（`Role`_），这意味着可以以特定的方式解释文本。
Sphinx 以此方式提供语义标记及参考索引，操作符为 ``:rolename:`content```。

标准 reST 提供以下规则：

- ``emphasis``，如 ``:emphasis:`text``` 等价于 ``*emphasis*``
- ``strong``，如 ``:strong:`text``` 等价于 ``**text**``
- ``literal``，如 ``:literal:`text``` 等价于 ````text````
- ``subscript``，如 ``H\ :sub:`2`\ O``
- ``superscript``，如 ``E = mc\ :sup:`2```

.. _Role: https://docutils.sourceforge.io/docs/ref/rst/roles.html

Sphinx 扩展的角色参见 `Roles <http://www.sphinx-doc.org/en/master/usage/restructuredtext/roles.html>`_ 。


List
====

无序列表（`Bullet List`_）可以使用 ``- * +`` 来表示，
有序列表（`Enumerated List`_）支持数字、字母和罗马字符，还支持 ``#`` 自动编号。

.. _Bullet List: https://docutils.sourceforge.io/docs/ref/rst/restructuredtext.html#bullet-lists

.. _Enumerated List: https://docutils.sourceforge.io/docs/ref/rst/restructuredtext.html#enumerated-lists

::

    * This is a bulleted list.
    * It has two items, the second
    item uses two lines.

    1. This is a numbered list.
    2. It has two items too.

    #. This is a numbered list.
    #. It has two items too.

列表开始和结尾需要空行，同级列表项之间空行可选，但嵌套时上下级列表前后需要空行。

::

    * this is
    * a list

      * with a nested list
      * and some subitems

    * and here the parent list continues

列表标记也使用 ``\`` 进行转义，如：

::

    \A. Einstein was a really smart dude.

定义列表（`Definition List`_）用于名词解释。
条目占一行，解释文本要有缩进；多层可根据缩进实现。

.. _Definition List: https://docutils.sourceforge.io/docs/ref/rst/restructuredtext.html#definition-lists

::

    term (up to a line of text)
       Definition of the term, which must be indented

       and can even consist of multiple paragraphs

    next term
       Description.

字段列表（`Field List`_）用于字段显示，如 docstring 参数说明。

.. _Field List: https://docutils.sourceforge.io/docs/ref/rst/restructuredtext.html#field-lists

::

    :Authors:
        Tony J. (Tibs) Ibbs,
        David Goodger

        (and sundry other good-natured folks)

    :Version: 1.0 of 2001/08/08
    :Dedication: To my father.

..

.. code:: python

    def my_function(my_arg, my_other_arg):
        """A function just for me.

        :param my_arg: The first of my arguments.
        :param my_other_arg: The second of my arguments.

        :returns: A message (just for me, of course).
        """

选项列表（`Option List`_）是一个类似两列的表格，左边是参数，右边是描述信息。
当参数选项过长时，参数选项和描述信息各占一行。

选项与参数之间有一个空格，参数选项与描述信息之间至少有两个空格。

.. _Option List: https://docutils.sourceforge.io/docs/ref/rst/restructuredtext.html#option-lists

::

    -a            command-line option "a"
    -b file       options can have arguments
                  and long descriptions
    --long        options can be long also
    --input=file  long options can also have
                  arguments
    /V            DOS/VMS-style options too


Block
=====

字面代码块（`Literal Block`_）在段落的后面使用标记 ``::`` 引出。
代码块必须缩进（同段落需要与周围文本以空行分隔）:

.. _Literal Block: https://docutils.sourceforge.io/docs/ref/rst/restructuredtext.html#literal-blocks

::

    This is a normal text paragraph. The next paragraph is a code sample::

       It is not processed in any way, except
       that the indentation is removed.

       It can span multiple lines.

    This is a normal text paragraph again.

这个 ``::`` 标记很优雅：

- 如果作为独立段落存在，则整段都不会出现在文档里；
- 如果前面有空白，则标记被移除；
- 如果前面是非空白，则标记被一个冒号取代。



块引用（`Block Quote`_）就是缩进的段落，
可以使用空的注释 ``..`` 分隔上下的块引用（注释前后都要有空行）。

.. _Block Quote: https://docutils.sourceforge.io/docs/ref/rst/restructuredtext.html#block-quotes

::

    This is an ordinary paragraph, introducing a block quote.

        "It is my business to know things.  That is my trade."

        -- Sherlock Holmes

    ..

        Block quote 2.


行块（`Line Block`_）对于地址、诗句以及无装饰列表是非常有用的。
行块是以 ``|`` 开头，每一个行块可以是多段文本。

.. _Line Block: https://docutils.sourceforge.io/docs/ref/rst/restructuredtext.html#line-blocks

::

    Take it away, Eric the Orchestra Leader!

        | A one, two, a one two three four
        |
        | Half a bee, philosophically,
        |     must, *ipso facto*, half not be.
        | But half the bee has got to be,
        |     *vis a vis* its entity.  D'you see?
        |
        | But can a bee be said to be
        |     or not to be an entire bee,
        |         when half the bee is not a bee,
        |             due to some ancient injury?
        |
        | Singing...


文档测试块（`Doctest Block`_）是交互式的 Python 会话，以 ``>>>`` 开始，一个空行结束。

.. _Doctest Block: https://docutils.sourceforge.io/docs/ref/rst/restructuredtext.html#doctest-blocks

::

    This is an ordinary paragraph.

    >>> print('this is a Doctest block')
    this is a Doctest block

    The following is a literal block::

        >>> This is not recognized as a doctest block by
        reStructuredText.  It *will* be recognized by the doctest
        module, though!


Table
=====

网格表（`Grid Table`_）使用 ``-`` 用来分隔行， ``=`` 用来分隔表头和表体行，
``|`` 用来分隔列，``+`` 用来表示行和列相交的节点。

.. _Grid Table: https://docutils.sourceforge.io/docs/ref/rst/restructuredtext.html#grid-tables

::

    +------------------------+------------+----------+----------+
    | Header row, column 1   | Header 2   | Header 3 | Header 4 |
    | (header rows optional) |            |          |          |
    +========================+============+==========+==========+
    | body row 1, column 1   | column 2   | column 3 | column 4 |
    +------------------------+------------+----------+----------+
    | body row 2             | Cells may span columns.          |
    +------------------------+------------+---------------------+
    | body row 3             | Cells may  | - Table cells       |
    +------------------------+ span rows. | - contain           |
    | body row 4             |            | - body elements.    |
    +------------------------+------------+---------------------+

简单表（`Simple Table`_）书写简单，只用 ``-`` 和 ``=`` 表示表格。
但有一些限制：需要有多行，且第一列元素不能分行显示。

.. _Simple Table: https://docutils.sourceforge.io/docs/ref/rst/restructuredtext.html#simple-tables

::

    =====  =====  =======
    A      B    A and B
    =====  =====  =======
    False  False  False
    True   False  False
    False  True   False
    True   True   True
    =====  =====  =======


Hyperlink
=========

如果链接文本是网址，则不需要特别标记，分析器会自动发现文本里的链接或邮件地址。


最简单的外链在 ``<>`` 内放入链接::

    External hyperlinks, like `Python <http://www.python.org/>`_.

可以将链接和标签分开：

::

    External hyperlinks, like Python_.

    .. _Python: http://www.python.org/

如果标记文字包含空格或符号，需要使用 ````` 标记，如::

    See the `Python home page`_ for info.

    .. _Python home page: http://www.python.org


内链通过在文内设置锚点以支持跳转，如::

    Internal crossreferences, like example_.

    .. _example:

    This is an example crossreference target.

匿名超链接不通过标记引用，而按顺序引用，使用 ``__`` 标记，如::

    See `the web site of my favorite programming language`__.

    .. __: http://www.python.org

也可以简化为::

    See `the web site of my favorite programming language`__.

    __ http://www.python.org

小节标题、脚注和引用参考会自动生成超链接地址，
使用小节标题、脚注或引用参考名称作为超链接名称就可以生成隐式链接。

::

    Hyperlink
    =========
    Implict references, like Hyperlink_.


Section
=======

章节（`Section`_）的标题在双上划线符号之间（或为下划线）, 并且符号的长度不能小于文本的长度。

.. _Section: https://docutils.sourceforge.io/docs/ref/rst/restructuredtext.html#sections

下面符号在标题中都可用::

    ! " # $ % & ' ( ) * + , - . / : ; < = > ? @ [ \ ] ^ _ ` { | } ~

通常没有专门的符号表示标题的等级，但 `Python's Style Guide for documenting`__ 使用风格：

- ``#`` with overline, for parts
- ``*`` with overline, for chapters
- ``=`` for sections
- ``-`` for subsections
- ``^`` for subsubsections
- ``"`` for paragraphs

__ https://devguide.python.org/documenting/#style-guide

::

    =================
    This is a heading
    =================

    This is a section
    =================

    This is a subsection
    --------------------

.. note::

    注意输出格式（HTML，LaTeX）所支持的层次深度。


Explicit Markup
===============

显式标记（`Explicit markup`_）用在那些需做特殊处理的 reST 结构中，
如尾注、突出段落、评论、通用指令。

显式标记以 ``..`` 开始，后跟空白符，结束于下一个同级缩进的段落.

.. _Explicit markup: https://docutils.sourceforge.io/docs/ref/rst/restructuredtext.html#explicit-markup-blocks


Directive
=========

指令（`Directive`_）是显式标记最常用的模块。也是 reST 的扩展规则。在 Sphinx 经常被用到。

.. _Directive: https://docutils.sourceforge.io/docs/ref/rst/restructuredtext.html#directives

Sphinx 扩展的指令参见 `Directives <http://www.sphinx-doc.org/en/master/usage/restructuredtext/directives.html>`_ 。


指令通常由名字、参数、选项及内容组成：

::

    .. function:: foo(x)
                  foo(y, z)
       :module: some.module.name

       Return a line of text input from the user.

上面例子中，``function`` 是指令名称，包含两个参数 ``foo`` 和一个选项 ``module``
（选项在参数后给出，由冒号引出）。指令的内容在隔开一个空行后，与指令有一样缩进。


Image
=====

图片（`Image`_）支持相对路径和绝对路径（``conf.py`` 所在目录）。

.. _Image: https://docutils.sourceforge.io/docs/ref/rst/directives.html#image

::

    .. image:: _images/sphinx.jpg
       :height: 106
       :width: 160
       :scale: 70
       :alt: sphinx profile
       :align: center


Sphinx 扩展 doctuils 允许对后缀名使用通配符。每个生成器则会选择最合适的图像。

::

    .. image:: gnu.*

例如在源文件目录里文件名 ``gnu.*`` 会含有两个文件 ``gnu.pdf`` 和 ``gnu.png``，
LaTeX 生成器会选择前者，而HTML 生成器则匹配后者。



.. note::

    图片文件名不能包含空格。


Footnote
========

脚注（`Footnote`_）使用 ``[#name]_`` 标记位置，内容则在文档底部。

.. _Footnote: https://docutils.sourceforge.io/docs/ref/rst/restructuredtext.html#footnotes

::

    Lorem ipsum [#f1]_ dolor sit amet ... [#f2]_

    .. rubric:: Footnotes

    .. [#f1] Text of the first footnote.
    .. [#f2] Text of the second footnote.

脚注可以使用手工序号（``[1]_``）或不带名称的自动序号（``[#]_``）。


Citation
========

引用（`Citation`_）类似于脚注，不过没有数字标签或以 ``#`` 开始。
Sphinx 将 reST 引用扩展为全局（所有参考引用不受所在文件的限制）。

.. _Citation: https://docutils.sourceforge.io/docs/ref/rst/restructuredtext.html#citations

::

    Lorem ipsum [Ref]_ dolor sit amet.

    .. [Ref] Book or article reference, URL or whatever.


Substitution
============

替换引用（`Substitution`_）将 ``|name|`` 内文本或标记替换为文字或图片。

.. _Substitution: docutils.sourceforge.net/docs/ref/rst/restructuredtext.html#substitution-definitions

::

    The |biohazard| symbol must be used on containers
    used to dispose of |name|.

    .. |biohazard| image:: biohazard.png
    .. |name| replace:: medical waste

如果需要对所有文档进行替换，可将替换内容写入 `rst_prolog <http://www.sphinx-doc.org/en/master/usage/configuration.html#confval-rst_prolog>`_ 或 `rst_epilog <http://www.sphinx-doc.org/en/master/usage/configuration.html#confval-rst_epilog>`_ 或其他单独文件，并通过 `include` 指令在使用它们的文档文件里包含这个文件。


Sphinx 默认预定义了以下替换引用，并放置在 ``conf.py``：

- ``|release|``
- ``|version|``
- ``|today|``

Comment
=======

注释以 ``..`` 开头，后面接注释内容即可，可以是多行内容。

::

    .. This is a comment.

可以通过缩进产生多行评论。

::

    ..
       This whole indented block
       is a comment.

       Still in the comment.


HTML Metadata
=============

`meta`_ 指令运行指定 Sphinx HTML 文档元数据。

.. _meta: docutils.sourceforge.net/docs/ref/rst/directives.html#meta

::

    .. meta::
       :description: The Sphinx documentation builder
       :keywords: Sphinx, documentation, builder

将生成以下 HTML：

.. code:: html

    <meta name="description" content="The Sphinx documentation builder">
    <meta name="keywords" content="Sphinx, documentation, builder">

