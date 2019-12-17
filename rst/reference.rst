.. _reference:

=========
Reference
=========

:reference: `Quick reStructuredText <https://docutils.sourceforge.io/docs/user/rst/quickref.html>`_
:updated: 2019-12-16 17:16:39


Section Title
=============

.. _section:

下面符号在标题中都可用::

    ! " # $ % & ' ( ) * + , - . / : ; < = > ? @ [ \ ] ^ _ ` { | } ~

.. Note::

    推荐使用: ``= - ` : . ' " ~ ^ _ * + #``


标题最多分六级，可以自由组合使用，reStructuredText 按照出现的顺序标记标题级别。


示例::

    =============
    Section Title
    =============

    -------------
    Section Title
    -------------

    Section Title
    =============

    Section Title
    -------------

    Section Title
    `````````````

    Section Title
    ~~~~~~~~~~~~~

    Section Title
    +++++++++++++

    Section Title
    ^^^^^^^^^^^^^

.. note::

    - 下线（也可以有上线）要至少等于标准文本的长度。

    - 文章标题推荐同时使用上线和下线。


Transition
==========

分隔符使用至少 4 字符标记，使用符号参见 section_。

示例::

    A transition marker is a horizontal line
    of 4 or more repeated punctuation
    characters.

    ------------

    A transition should not begin or end a
    section or document, nor should two
    transitions be immediately adjacent.

效果如下:

A transition marker is a horizontal line
of 4 or more repeated punctuation
characters.

------------

A transition should not begin or end a
section or document, nor should two
transitions be immediately adjacent.




Paragraph
=========

段落是被空行分割的文字片段，左侧必须对齐（没有空格，或者有相同多的空格）。

缩进的段落被视为引用（quote_）。

示例::

    This is a paragraph.

    Paragraphs line up at their left
    edges, and are normally separated
    by blank lines.

效果如下：

This is a paragraph.

Paragraphs line up at their left
edges, and are normally separated
by blank lines.


Inline Markup
=============

Emphasis
--------

使用 ``*`` 标记文本，通常显示为斜体，如::

    This is *emphasized text*.

显示为：

This is *emphasized text*.

Strong Emphasis
---------------

使用 ``**`` 标记文本，通常显示为粗体，如::

    This is **strong text**.

显示为：

This is **strong text**.

Interpreted Text
----------------

使用 ````` 标记文本，通常用于程序标识符等，如::

    This is `interpreted text`.

显示为：

This is `interpreted text`.

Inline Literal
--------------

使用 `````` 标记文本，通常显示为等宽，用于行内代码，如::

    The regular expression ``[+-]?(\d+(\.\d*)?|\.\d+)`` matches
    floating-point numbers (without exponents).

显示为：

The regular expression ``[+-]?(\d+(\.\d*)?|\.\d+)`` matches
floating-point numbers (without exponents).



List
====

Bullet List
-----------
无序列表可以使用 ``- * +`` 来表示。

.. note::

    列表开始和结尾需要空行，同级列表项之间空行可选，但上下级列表前后需要空行。

::

    - This is the first bullet list item. The blank line above the
      first list item is required; blank lines between list items
      (such as below this paragraph) are optional.

    - This is the first paragraph in the second item in the list.

      This is the second paragraph in the second item in the list.
      The blank line above this paragraph is required. The left edge
      of this paragraph lines up with the paragraph above, both
      indented relative to the bullet.

      - This is a sublist.  The bullet lines up with the left edge of
        the text blocks above. A sublist is a new list so requires a
        blank line above and below.

    - This is the third item of the main list.

    This paragraph is not part of the list.

    - The following line appears to be a new sublist, but it is not:
      - This is a paragraph continuation, not a sublist (since there's
        no blank line). This line is also incorrectly indented.

显示为：

- This is the first bullet list item. The blank line above the
  first list item is required; blank lines between list items
  (such as below this paragraph) are optional.

- This is the first paragraph in the second item in the list.

  This is the second paragraph in the second item in the list.
  The blank line above this paragraph is required. The left edge
  of this paragraph lines up with the paragraph above, both
  indented relative to the bullet.

  - This is a sublist. The bullet lines up with the left edge of
    the text blocks above. A sublist is a new list so requires a
    blank line above and below.

- This is the third item of the main list.

This paragraph is not part of the list.

- The following line appears to be a new sublist, but it is not:
  - This is a paragraph continuation, not a sublist (since there's no blank line). This line is also incorrectly indented.

Enumerated List
---------------

有序列表支持符号：

- 数字：1, 2, 3, ...
- 大写字母：A, B, C, ..., Z
- 小写字母：a, b, c, ..., z
- 大写罗马数字：I, II, III, IV, ..., MMMMCMXCIX(4999)
- 小写罗马数字：i, ii, iii, iv, ..., mmmmcmxcix

后跟符号：

- 点："1.", "A.", "a.", "I.", "i."
- 括号："(1)", "(A)", "(a)", "(I)", "(i)"
- 右括号："1)", "A)", "a)", "I)", "i)"

示例::

    1. Item 1 initial text.

       a) Item 1a.
       b) Item 1b.

    2. a) Item 2a.
       b) Item 2b.

显示为：

1. Item 1 initial text.

   a) Item 1a.
   b) Item 1b.

2. a) Item 2a.
   b) Item 2b.

.. note::

    使用 ``\`` 转义列表，如::

        \A. Einstein was a really smart dude.

    将显示为：

        \A. Einstein was a really smart dude.

.. note::

    有序列表可以结合 ``#`` 自动生成序号。

Definition List
---------------
定义列表用于名词解释。

条目占一行，解释文本要有缩进；多层可根据缩进实现。

::

    Definition lists:

    what
        Definition lists associate a term with
        a definition.

    how
        The term is a one-line phrase, and the
        definition is one or more paragraphs or
        body elements, indented relative to the
        term. Blank lines are not allowed
        between term and definition.

效果为：

Definition lists:

what
    Definition lists associate a term with
    a definition.

how
    The term is a one-line phrase, and the
    definition is one or more paragraphs or
    body elements, indented relative to the
    term. Blank lines are not allowed
    between term and definition.


Field List
----------
字段列表用于字段显示::

    :Authors:
        Tony J. (Tibs) Ibbs,
        David Goodger

        (and sundry other good-natured folks)

    :Version: 1.0 of 2001/08/08
    :Dedication: To my father.

显示为：

:Authors:
    Tony J. (Tibs) Ibbs,
    David Goodger

    (and sundry other good-natured folks)

:Version: 1.0 of 2001/08/08
:Dedication: To my father.


Option List
-----------
选项列表是一个类似两列的表格，左边是参数，右边是描述信息。当参数选项过长时，参数选项和描述信息各占一行。

选项与参数之间有一个空格，参数选项与描述信息之间至少有两个空格。

::

    -a            command-line option "a"
    -b file       options can have arguments
                  and long descriptions
    --long        options can be long also
    --input=file  long options can also have
                  arguments
    /V            DOS/VMS-style options too

显示如：

-a            command-line option "a"
-b file       options can have arguments
              and long descriptions
--long        options can be long also
--input=file  long options can also have
              arguments
/V            DOS/VMS-style options too


Block
=====

Literal Block
-------------
文字块就是一段文字信息，在需要插入文本块的段落后面加上 ``::``，接着一个空行，文字块不缩进，使用字符参见 section_。

::

    John Doe wrote::

    >> Great idea!
    >
    > Why didn't I think of that?

    You just did!  ;-)

显示为：

John Doe wrote::

>> Great idea!
>
> Why didn't I think of that?

You just did!  ;-)




Line Block
----------
行块对于地址、诗句以及无装饰列表是非常有用的。行块是以 ``|`` 开头，每一个行块可以是多段文本。

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

效果如：

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


.. _quote:

Block Quote
-----------
块引用是通过缩进来实现的，引用块要在前面的段落基础上缩进。

块引用可以使用空的注释 ``..`` 分隔上下的块引用（注释前后都要有空行）。

::

    This is an ordinary paragraph, introducing a block quote.

        "It is my business to know things.  That is my trade."

        -- Sherlock Holmes

    ..

        “真的猛士，敢于直面惨淡的人生，敢于正视淋漓的鲜血。”

        --- 鲁迅

显示如：

This is an ordinary paragraph, introducing a block quote.

    "It is my business to know things.  That is my trade."

    -- Sherlock Holmes

..

    “真的猛士，敢于直面惨淡的人生，敢于正视淋漓的鲜血。”

    --- 鲁迅


Doctest Block
-------------
文档测试块是交互式的Python会话，以 ``>>>`` 开始，一个空行结束。

::

    This is an ordinary paragraph.

    >>> print('this is a Doctest block')
    this is a Doctest block

    The following is a literal block::

        >>> This is not recognized as a doctest block by
        reStructuredText.  It *will* be recognized by the doctest
        module, though!

效果如：

This is an ordinary paragraph.

>>> print('this is a Doctest block')
this is a Doctest block

The following is a literal block::

    >>> This is not recognized as a doctest block by
    reStructuredText.  It *will* be recognized by the doctest
    module, though!


Table
=====

Grid Table
----------
使用 ``-`` 用来分隔行， ``=`` 用来分隔表头和表体行，``|`` 用来分隔列，``+`` 用来表示行和列相交的节点。

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

显示为：

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


Simple Table
------------

简单表相对于网格表，少了 ``|`` 和 ``+`` 两个符号，只用 ``-`` 和 ``=`` 表示。

::

    =====  =====  =======
    A      B    A and B
    =====  =====  =======
    False  False  False
    True   False  False
    False  True   False
    True   True   True
    =====  =====  =======

显示为：

=====  =====  =======
  A      B    A and B
=====  =====  =======
False  False  False
True   False  False
False  True   False
True   True   True
=====  =====  =======



Footnote
========

脚注有这几个方式：手动序号、自动序号、自动符号。


手动序号::

    Footnote references, like [5]_.
    Note that footnotes may get
    rearranged, e.g., to the bottom of
    the "page".

    .. [5] A numerical footnote.

效果如：

Footnote references, like [5]_.
Note that footnotes may get
rearranged, e.g., to the bottom of
the "page".

.. [5] A numerical footnote.


自动序号::

    Autonumbered footnotes are
    possible, like using [#]_ and [#]_.
    They may be assigned 'autonumber
    labels' - for instance, [#third]_.

    .. [#] This is the first one.
    .. [#] This is the second one.
    .. [#third] a.k.a. third_

效果如：

Autonumbered footnotes are
possible, like using [#]_ and [#]_.
They may be assigned 'autonumber
labels' - for instance, [#third]_.

.. [#] This is the first one.
.. [#] This is the second one.
.. [#third] a.k.a. third_

.. note::

    - ``#`` 表示的方法可以在后面加上一个名称生成链接。
    - 手工序号可以和自动序号结合使用，会自动延续手工的序号。


自动符号::

    Auto-symbol footnotes are also
    possible, like this: [*]_ and [*]_.

    .. [*] This is the first one.
    .. [*] This is the second one.

显示为：

Auto-symbol footnotes are also
possible, like this: [*]_ and [*]_.

.. [*] This is the first one.
.. [*] This is the second one.


Citation
========

::

    Citation labels contain alphanumerics,
    underlines, hyphens and fullstops.
    Case is not significant.

    Given a citation like [CIT2002]_, one
    can also refer to it like CIT2002_.

    .. [CIT2002] A citation here.

显示为：

Citation labels contain alphanumerics,
underlines, hyphens and fullstops.
Case is not significant.

Given a citation like [CIT2002]_, one
can also refer to it like CIT2002_.

.. [CIT2002] A citation here.

引用参考的内容通常放在页面结尾处。

Hyperlink
=========

reStructuredText 会自动将 URI 生成超链接，如::

    https://docutils.sourceforge.io/rst.html

显示为：

https://docutils.sourceforge.io/rst.html

External Hyperlink
------------------

最简单的外链::

    External hyperlinks, like Python_.

    .. _Python: http://www.python.org/

显示如：

External hyperlinks, like Python_.

.. _Python: http://www.python.org/

如果标记文字包含空格或符号，需要使用 ````` 标记，如::

    See the `Python home page`_ for info.

    .. _Python home page: http://www.python.org

效果如：

See the `Python home page`_ for info.

.. _Python home page: http://www.python.org

也可以简单的在 ``<>`` 内放入链接::

    External hyperlinks, like `Python
    <http://www.python.org/>`_.

显示为：

External hyperlinks, like `Python
<http://www.python.org/>`_.


Internal Hyperlink
------------------

内链通过在文内设置锚点以支持跳转，如::

    Internal crossreferences, like example_.

    .. _example:

    This is an example crossreference target.

效果如：

Internal crossreferences, like example_.

.. _example:

This is an example crossreference target.


Anonymous Hyperlink
-------------------

匿名超链接不通过标记引用，而按顺序引用，使用 ``__`` 标记，如::

    See `the web site of my favorite programming language`__.

    .. __: http://www.python.org

也可以简化为::

    See `the web site of my favorite programming language`__.

    __ http://www.python.org

显示为：

See `the web site of my favorite programming language`__.

.. __: http://www.python.org


Indirect Hyperlink
------------------

间接超链接基于匿名链接，将匿名链接地址换成了外部引用名。

::

    Python_ is `my favourite
    programming language`__.

    .. _Python: http://www.python.org/

    __ Python_

显示为：

Python_ is `my favourite
programming language`__.

.. _Python: http://www.python.org/

__ Python_


Implicit Hyperlink
------------------

小节标题、脚注和引用参考会自动生成超链接地址，使用小节标题、脚注或引用参考名称作为超链接名称就可以生成隐式链接。

::

    Hyperlink
    =========
    Implict references, like Hyperlink_.

效果如：

Implict references, like Hyperlink_.


Substitution Reference
======================

替换引用就是用定义的指令替换对应的文字或图片，使用 ``|`` 标记文本，如::

    The |biohazard| symbol must be used on containers
    used to dispose of |name|.

    .. |biohazard| image:: _images/biohazard.png
    .. |name| replace:: medical waste

显示为：

The |biohazard| symbol must be used on containers
used to dispose of |name|.

.. |biohazard| image:: _images/biohazard.png
.. |name| replace:: medical waste


Comment
=======

注释以 ``..`` 开头，后面接注释内容即可，可以是多行内容，如::

    .. This text will not be shown
       (but, for instance, in HTML might be
       rendered as an HTML comment)

注释内容不会在页面显示。

Directive
=========

参考 `reStructuredText Directives`_。

.. _reStructuredText Directives: https://docutils.sourceforge.io/docs/ref/rst/directives.html
