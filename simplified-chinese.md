% [Pandoc's Markdown]
% by John MacFarlane; Translated by [PPType](http://github.com/pptype) based on the Traditional Chinese Translation by [Tzeng Yuxio](http://pages.tzengyuxio.me/pandoc)

前言
--------
本文档是 [Pandoc][] 版 Markdown 语法的简体中文翻译。Pandoc 本身是由 [John MacFarlane][] 所开发的文档转换工具，可以在 HTML, Markdown, PDF, TeX 等格式之间进行转换。有许多喜欢纯文本编辑的人，利用 Pandoc 进行论文撰写或投影片制作。除了转换功能外，Pandoc 所定义的 Markdown 扩展语法也是这套工具的一大亮点，在 Pandoc 的官方文档中，仅针对 Markdown 格式的扩展说明就占了一半左右的篇幅。

本文档翻译自 [Pandoc - Pandoc User’s Guide][userguide] 中的 "Pandoc's Markdown" 一节。你可以在 GitHub 上查看这份文档的[源文件][source]。

[John MacFarlane]: http://johnmacfarlane.net/
[pandoc]: http://pandoc.org
[userguide]: http://pandoc.org/MANUAL.html
[source]: https://github.com/pptype/pandoc-markdown/simplified-chinese.md

以下翻译开始。

----

Pandoc's Markdown
=================

与 John Gruber 的 原始 [Markdown] 相比，Pandoc 版本的 Markdown 在语法上进行了扩展和少许修正。本文档解释了这些语法，并指出其与标准 Markdown 的差异。除非特别提到，这些差异均可借由使用 `markdown_strict` 而非 `markdown` 的格式来消除。一项扩展可通过 `+EXTENSION` 或 `-EXTENSION` 的方式来开启或关闭。例如，`markdown_strict+footnotes` 表示加上脚注扩展的严格 Markdown，而 `markdown-footnotes-pipe_tables` 则是去掉了脚注与管线表格扩展的 pandoc Markdown。

哲学
----------

Markdown 的设计目标是易于书写，并且更重要的是易于阅读：

> 一份 Markdown 格式的文档应该能原封不动地以纯文本形式发表，并且看上去不像是被标签或格式指令标记过。
> <small>[John Gruber](http://daringfireball.net/projects/markdown/syntax#philosophy)</small>

Pandoc 在制订表格、脚注以及其他扩展的语法时都遵循了这项原则。

不过，pandoc 是针对多种输出格式设计的，而 Markdown 最初主要考虑的是生成 HTML，两者在这方面目标不同。因此虽然 pandoc 允许直接嵌入原始 HTML，但并不鼓励这样做，而是提供了许多非 HTML 风格的方式，让用户输入定义清单、表格、数学公式以及脚注等重要文档元素。

段落
----------

一个段落指的是一行及以上的文本，后面紧跟着一个或多个空行。换行字符会被当作空格，因此你可以依自己喜好排列段落文本。如果你需要插入一个强制换行符，在行尾放两个或以上的空格即可。

**Extension: `escaped_line_breaks`**

一个反斜线后面加一个换行符，同样构成一个强制换行符。
注意：在多行内容和表格的单元格中，这是创造强制换行符的唯一方法，因为单元格中，行尾的空格字符会被忽略。

标题
-------

有两种标题：Setext 和 ATX。

### Setext 风格标题 ###

Setext 风格的标题上面是一行文本，下面接着一行 `=` 符号（用于一阶标题）或 `-` 符号（用于二阶标题）：

    A level-one header
    ==================

    A level-two header
    ------------------

标题文本可以包含行内格式，例如强调（见下方 [行内格式] 一节）。


### ATX 风格标题 ###

ATX 风格的标题由一到六个 `#` 符号及一行文本组成，文本后面也可以选择性地加上任意数量的 `#` 符号。行首的 `#` 符号数量决定了标题的层级：

    ## A level-two header

    ### A level-three header ###

如同 setext，ATX 风格标题的文本也可以包含行内格式：

    # A level-one header with a [link](/url) and *emphasis*

**Extension: `blank_before_header`**

标准 Markdown 语法并不要求在标题之上留一个空行，而 pandoc Markdown 的标题上面则必须加上空行（除非标题位于文档第一行）。这是因为 `#`` 符号很可能完全意外地出现在行首（也许是因为自动断行）。考虑下面的例子：

    I like several of their flavors of ice cream:
    #22, for example, and #5.


### HTML, LaTeX 与 ConTeXt 的标题识别符 ###

**Extension: `header_attributes`**

在标题文本所在行的行尾，可以使用以下语法为标题加上属性：

    {#identifier .class .class key=value key=value}

虽然这个语法也包含加入类别 (class) 以及键／值形式的属性 (attribute)，但目前只有识别符 (identifier/ID) 在输出时有实际作用（且只在部分格式的输出，包括：HTML, LaTeX, ConTeXt, Textile, AsciiDoc）。举例来说，下面是将标题加上 `foo` 识别符的几种方法：

    # My header {#foo}

    ## My header ##    {#foo}

    My other header   {#foo}
    ---------------

（此语法与 [PHP Markdown Extra] 兼容。）

具有 `unnumbered` 类别的标题将不会被编号，即使 `--number-sections` 的选项是开启的。单一连字符号 (`-`) 等同于 `.unnumbered`，且更适用于非英文文档中。因此，

    # My header {-}

与下面这行是等价的

    # My header {.unnumbered}

**Extension: `auto_identifiers`**

没有明确指定 ID（识别符）的标题将会依据其标题文本，自动指派一个独一无二的 ID。由标题文本推导 ID 的规则如下：

  - 移除所有格式，链接等。
  - 移除所有标点符号，除了底线、连字符号与句号。
  - 以连字符号取代所有空白与换行字符。
  - 将所有英文本母转为小写。
  - 移除第一个字符前的所有内容（ID 不能以数字或标点符号开头）。
  - 如果剩下为空字符串，则使用 `section` 作为 ID。

以下是一些范例，

  Header                            Identifier
  -------------------------------   ----------------------------
  Header identifiers in HTML        `header-identifiers-in-html`
  *Dogs*?--in *my* house?           `dogs--in-my-house`
  [HTML], [S5], or [RTF]?           `html-s5-or-rtf`
  3. Applications                   `applications`
  33                                `section`

在大多数情况下，这些规则应该让人能够直接从标题文本推导出 ID。唯一的例外是当有多个标题具有同样文本的情况；在这情况下，第一个标题的 ID 仍旧是透过以上规则推导而得；第二个则是在同样 ID 后加上 `-1`；第三个加上 `-2`；以此类推。

在开启 `--toc|--table-of-contents` 的选项时，这些 ID 是用来产生目录 (Table of Contents) 所需的页面链接。此外，这些 ID 也提供了一个简便的方式来输入跳到指定章节的链接。一个以 ID 产生的链接，其使用的语法看起来就像下面的例子：

    See the section on
    [header identifiers](#header-identifiers-in-html-latex-and-context).

然而要注意的一点是，只有在以 HTML、LaTeX 与 ConTeXt 格式输出时，才能以这种方式产生对应的章节链接。

如果指定了 `--section-divs` 选项，则每一个小节都会以 `div` 标签包住（或是 `section` 标签，如果有指定 `--html5` 选项的话），并且 ID 会被附加在用来包住小节的 `<div>`（或是 `<section>`）标签，而非附加在标题上。这使得整个小节都可以透过 javascript 来操作，或是采用不同的 CSS 设置。

**Extension: `implicit_header_references`**

Pandoc 假设每个标题都定义了其参考链接，因此，相较于以下的链接语法

    [header identifiers](#header-identifiers-in-html)

你也可以单纯只写

    [header identifiers]

或

    [header identifiers][]

或

    [the section on header identifiers][header identifiers]

如果有多个标题具有同样文本，对应的参考只会链接到第一个符合的标题，这时若要链接到其他符合的标题，就必须以先前提到的方式，明确指定链接到该标题的 ID。

与其他一般参考链接不同的是，这些参考链接是大小写有别的。

注意：如果你有明确定义了任何一个标题的标示符，那么选项 `implicit_header_references` 就没有作用。

区块引言
----------------

Markdown 使用 email 的习惯来创建引言区块。一个引言区块可以由一或多个段落或其他的区块元素（如清单或标题）组成，并且其行首均是由一个 `>` 符号加上一个空白作为开头。（`>` 符号不一定要位在该行最左边，但也不能缩进超过三个空白）。

    > This is a block quote. This
    > paragraph has two lines.
    >
    > 1. This is a list inside a block quote.
    > 2. Second item.

有一个「偷懒」的形式：你只需要在引言区块的第一行行首输入 `>` 即可，后面的行首可以省略符号：

    > This is a block quote. This
    paragraph has two lines.

    > 1. This is a list inside a block quote.
    2. Second item.

由于区块引言可包含其他区块元素，而区块引言本身也是区块元素，所以，引言是可以嵌套入其他引言的。

    > This is a block quote.
    >
    > > A block quote within a block quote.

**Extension: `blank_before_blockquote`**

原始 markdown 语法在区块引言之前并不需要预留空白行。Pandoc 则需要（除非区块引言位于文档最开始的地方）。这是因为以 `>` 符号开头的情况在一般文本段落中相当常见（也许由于断行所致），这会导致非预期的格式。因此，除非是指定为 `markdown_strict` 格式，不然以下的语法在 pandoc 中将不会产生出嵌套区块引言：

    > This is a block quote.
    >> Nested.


字面（代码）区块
----------------------

### 缩进代码区块 ###

一段以四个空白（或一个 tab）缩进的文本区块会被视为字面区块 (Verbatim Block)：换句话说，特殊字符并不会转换为任何格式，单纯只以字面形式呈现，而所有的空白与换行也都会被保留。例如，

        if (a > 3) {
          moveShip(5 * gravity, DOWN);
        }

位于行首的缩进（四个空白或一个 tab）并不会被视为字面区块的一部分，因此在输出时会被移除掉。

注意：在字面文本之间的空白行并不需要也以四个空白字符做开头。


### 围栏代码区块 ###

**Extension: `fenced_code_blocks`**

除了标准的缩进代码区块外，Pandoc 也支持了**围栏** (*fenced*) 代码区块的语法。这区块需以包含三个以上波浪线 (`~`) 或反引号 (`` ` ``) 的一行作为开始，并以同样符号且至少同样长度的一行作为结束。所有介于开始与结束之间的文本行都会视为代码。不需要额外的缩进：

    ~~~~~~~
    if (a > 3) {
      moveShip(5 * gravity, DOWN);
    }
    ~~~~~~~

如同一般的代码区块，围栏代码区块与其前后的文本之间必须以空白行作间隔。

如果代码本身也包含了一整行的波浪线或反引号，那么只要在区块首尾处使用更长的波浪线或反引号即可：

    ~~~~~~~~~~~~~~~~
    ~~~~~~~~~~
    code including tildes
    ~~~~~~~~~~
    ~~~~~~~~~~~~~~~~

你也可以选择性地使用以下语法附加属性到代码区块上：

    ~~~~ {#mycode .haskell .numberLines startFrom="100"}
    qsort []     = []
    qsort (x:xs) = qsort (filter (< x) xs) ++ [x] ++
                   qsort (filter (>= x) xs)
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

这里的 `mycode` 为 ID，`haskell` 与 `numberLines` 是类别，而 `startsFrom` 则是值为 `100` 的属性。有些输出格式可以利用这些信息来作语法高亮。目前有使用到这些信息的输出格式仅有 HTML 与 LaTeX。如果指定的输出格式及语言类别有支持语法高亮，那么上面那段代码区块将会以高亮并带有行号的方式呈现。（要查找支持的编程语言清单，可在命令行输入 `pandoc --version`。）反之若无支持，则上面那段代码区块则会以下面的形式呈现：

    <pre id="mycode" class="haskell numberLines" startFrom="100">
      <code>
      ...
      </code>
    </pre>

下面这个是针对代码区块只有指定编程语言属性的简便形式：

    ```haskell
    qsort [] = []
    ```

这与下面这行的效果是相同的：

    ``` {.haskell}
    qsort [] = []
    ```

要取消所有语法高亮，使用 `--no-highlight` 选项。要设置语法高亮的配色，则使用 `--highlight-style`。

行区块
-----------

**Extension: `line_blocks`**

行区块是一连串以竖线 (`|`) 加上一个空格所构成的连续行。行与行间的区隔在输出时将会以原样保留，行首的空白字符数目也一样会被保留；反之，这些行将会以 markdown 的格式处理。这个语法在输入诗句或地址时很有帮助。

    | The limerick packs laughs anatomical
    | In space that is quite economical.
    |    But the good ones I've seen
    |    So seldom are clean
    | And the clean ones so seldom are comical

    | 200 Main St.
    | Berkeley, CA 94718

如果有需要的话，书写时也可以将完整一行拆成多行，但后续行必须以空白作为开始。下面范例的前两行在输出时会被视为一整行：

    | The Right Honorable Most Venerable and Righteous Samuel L.
      Constable, Jr.
    | 200 Main St.
    | Berkeley, CA 94718

这是从 [reStructuredText] 借来的语法。

清单
-----

### 无序清单 ###

无序清单是以项目符号作枚举的清单。每条项目都以项目符号 (`*`, `+` 或 `-`) 作开头。下面是个简单的例子：

    * one
    * two
    * three

这会产生一个「紧凑」清单。如果你想要一个「宽松」清单，也就是说以段落格式处理每个项目内的文本内容，那么只要在每个项目间加上空白行即可：

    * one

    * two

    * three

项目符号不能直接从行首最左边处输入，而必须以一至三个空白字符作缩进。项目符号后必须跟着一个空白字符。

清单项目中的接续行，若与该项目的第一行文本对齐（在项目符号之后），看上去会较为美观：

    * here is my first
      list item.
    * and my second.

但 markdown 也允许以下「偷懒」的格式：

    * here is my first
    list item.
    * and my second.

### 四个空白规则 ###

一个清单项目可以包含多个段落以及其他区块等级的内容。然而，后续的段落必须接在空白行之后，并且以四个空白或一个 tab 作缩进。因此，如果项目里第一个段落与后面段落对齐的话（也就是项目符号前置入两个空白），看上去会比较整齐美观：

      * First paragraph.

        Continued.

      * Second paragraph. With a code block, which must be indented
        eight spaces:

            { code }

清单项目也可以包含其他清单。在这情况下前置的空白行是可有可无的。嵌套清单必须以四个空白或一个 tab 作缩进：

    * fruits
        + apples
            - macintosh
            - red delicious
        + pears
        + peaches
    * vegetables
        + brocolli
        + chard

上一节提到，markdown 允许你以「偷懒」的方式书写，项目的接续行可以不和第一行对齐。不过，如果一个清单项目中包含了多个段落或是其他区块元素，那么每个元素的第一行都必须缩进对齐。

    + A lazy, lazy, list
    item.

    + Another one; this looks
    bad but is legal.

        Second paragraph of second
    list item.

**注意：**尽管针对接续段落的「四个空白规则」是出自于官方的 [markdown syntax guide]，但是作为对应参考用的 `Markdown.pl` 实作版本中并未遵循此一规则。所以当输入时若接续段落的缩进少于四个空白时，pandoc 所输出的结果会与 `Markdown.pl` 的输出有所出入。

在 [markdown syntax guide] 中并未明确表示「四个空白规则」是否一体适用于 **所有** 位于清单项目里的区块元素上；规范文档中只提及了段落与代码区块。但文档暗示了此规则适用于所有区块等级的内容（包含嵌套清单），并且 pandoc 以此方向进行解读与实作。

  [markdown syntax guide]:
    http://daringfireball.net/projects/markdown/syntax#list

### 有序清单 ###

有序清单与无序清单相类似，唯一的差别在于清单项目是以枚举编号作开头，而不是项目符号。

在原始 markdown 中，枚举编号是阿拉伯数字后面接着一个句点与空白。数字本身代表的数值会被忽略，因此下面两个清单并无差别：

    1.  one
    2.  two
    3.  three

上下两个清单的输出是相同的。

    5.  one
    7.  two
    1.  three

**Extension: `fancy_lists`**

与原始 markdown 不同的是，Pandoc 除了使用阿拉伯数字作为有序清单的编号外，也可以使用大写或小写的英文本母，以及罗马数字。清单标记可以用括号包住，也可以单独一个右括号，抑或是句号。如果清单标记是大写字母接着一个句号，句号后请使用至少两个空白字符。[^2]

[^2]:  之所以有这条规则，主要是要避免以人名头文本缩写作为开头的段落所带来的混淆，像是

        B. Russell was an English philosopher.

    这样就不会被当作清单项目了。

    这条规则并不会避免以下

        (C) 2007 Joe Smith

    这样的叙述被解释成清单项目。在这情形下，可以使用反斜线：

        (C\) 2007 Joe Smith

**Extension: `startnum`**

除了清单标记外，Pandoc 也能判读清单的起始编号，这两项信息都会保留于输出格式中。举例来说，下面的输入可以产生一个从编号 9 开始，以单括号为编号标记的清单，底下还跟着一个小写罗马数字的子清单：

     9)  Ninth
    10)  Tenth
    11)  Eleventh
           i. subone
          ii. subtwo
         iii. subthree

当遇到不同形式的清单标记时，Pandoc 会重新开始一个新的清单。所以，以下的输入会产生三份清单：

    (2) Two
    (5) Three
    1.  Four
    *   Five

如果需要缺省的有序清单标记符号，可以使用 `#.`：

    #.  one
    #.  two
    #.  three


### 定义清单 ###

**Extension: `definition_lists`**

Pandoc 支持定义清单，其语法的灵感来自于 [PHP Markdown Extra] 以及 [reStructuredText]：[^3]

    Term 1

    :   Definition 1

    Term 2 with *inline markup*

    :   Definition 2

            { some code, part of Definition 2 }

        Third paragraph of definition 2.

每个专有名词 (term) 都必须单独存在于一行，后面可以接着一个空白行，也可以省略，但一定要接上一或多笔定义内容。一笔定义需由一个冒号或波浪线作开头，可以接上一或两个空白作为缩进。定义本身的内容主体（包括接在冒号或波浪线后的第一行）应该以四个空白缩进。一个专有名词可以有多个定义，而每个定义可以包含一或多个区块元素（段落、代码区块、清单等），每个区块元素都要缩进四个空白或一个 tab。

如果你在定义内容后面留下空白行（如同上面的范例），那么该段定义会被当作段落处理。在某些输出格式中，这意谓着成对的专有名词与定义内容间会有较大的空白间距。在定义与定义之间，以及定义与下个专有名词间不要留空白行，即可产生一个比较紧凑的定义清单：

    Term 1
      ~ Definition 1
    Term 2
      ~ Definition 2a
      ~ Definition 2b

[^3]:  [David Wheeler](http://www.justatheory.com/computers/markup/modest-markdown-proposal.html) 对于 markdown 的建议也同时影响了我。

[PHP Markdown Extra]: http://www.michelf.com/projects/php-markdown/extra/


### 编号范例清单 ###

**Extension: `example_lists`**

这个特别的清单标记 `@` 可以用来产生连续编号的范例清单。清单中第一个以 `@` 标记的项目会被编号为 '1'，接着编号为 '2'，依此类推，直到文档结束。范实例目的编号不会局限於单一清单中，而是文档中所有以 `@` 为标记的项目均会次序递增其编号，直到最后一个。举例如下：

    (@)  My first example will be numbered (1).
    (@)  My second example will be numbered (2).

    Explanation of examples.

    (@)  My third example will be numbered (3).

编号范例可以加上标签，并且在文档的其他地方作参照：

    (@good)  This is a good example.

    As (@good) illustrates, ...

标签可以是由任何英文本母、底线或是连字符号所组成的字符串。


### 紧凑与宽松清单 ###

在与清单相关的「边界处理」上，Pandoc 与 `Markdown.pl` 有着不同的处理结果。考虑如下代码：

    +   First
    +   Second:
        -   Fee
    	-   Fie
    	-   Foe

    +   Third

Pandoc 会将以上清单转换为「紧凑清单」（在 "First", "Second" 或 "Third" 之中没有 `<p>` 标签），而 markdown 则会在 "Second" 与 "Third" （但不包含 "First"）里面置入 `<p>` 标签，这是因为 "Third" 之前的空白行而造成的结果。Pandoc 依循着一个简单规则：如果文本后面跟着空白行，那么就会被视为段落。既然 "Second" 后面是跟着一个清单，而非空白行，那么就不会被视为段落了。至于子清单的后面是不是跟着空白行，那就无关紧要了。（注意：即使是设置为 `markdown_strict` 格式，Pandoc 仍是依以上方式处理清单项目是否为段落的判定。这个处理方式与 markdown 官方语法规范里的描述一致，然而却与 `Markdown.pl` 的处理不同。）


### 结束一个清单 ###

如果你在清单之后放入一个缩进的代码区块，会有什么结果？

    -   item one
    -   item two

        { my code block }

问题大了！这边 pandoc（其他的 markdown 实作也是如此）会将 `{ my code block }` 视为 `item two` 这个清单项目的第二个段落来处理，而不会将其视为一个代码区块。

要在 `item two` 之后「切断」清单，你可以插入一些没有缩进、输出时也不可见的内容，例如 HTML 的注解：

    -   item one
    -   item two

    <!-- end of list -->

        { my code block }

当你想要两个各自独立的清单，而非一个大且连续的清单时，也可以运用同样的技巧：

    1.  one
    2.  two
    3.  three

    <!-- -->

    1.  uno
    2.  dos
    3.  tres

分隔线
----------------

一行中若包含三个以上的 `*`, `-` 或 `_` 符号（中间可以以空白字符分隔），则会产生一条分隔线：

    *  *  *  *

    ---------------


表格
------

有四种表格的形式可以使用。前三种适用于等宽字体的编辑环境，例如 Courier。第四种则不需要直行的对齐，因此可以在比例字体的环境下使用。

### 简单表格

**Extension: `simple_tables`, `table_captions`**

简单表格看起来像这样子：

      Right     Left     Center     Default
    -------     ------ ----------   -------
         12     12        12            12
        123     123       123          123
          1     1          1             1

    Table:  Demonstration of simple table syntax.

表头与数据列分别以一行为单位。直行的对齐则依照表头的文本和其底下虚线的相对位置来决定：[^4]

  - 如果虚线与表头文本的右侧有切齐，而左侧比表头文本还长，则该直行为靠右对齐。
  - 如果虚线与表头文本的左侧有切齐，而右侧比表头文本还长，则该直行为靠左对齐。
  - 如果虚线的两侧都比表头文本长，则该直行为置中对齐。
  - 如果虚线与表头文本的两侧都有切齐，则会套用缺省的对齐方式（在大多数情况下，这将会是靠左对齐）。

[^4]:  这个方案是由 Michel Fortin 在 [Markdown discussion list](http://six.pairlist.net/pipermail/markdown-discuss/2005-March/001097.html) 的讨论中所提出。

表格底下必须接着一个空白行，或是一行虚线后再一个空白行。表格标题为可选的（上面的范例中有出现）。标题需是一个以 `Table:` （或单纯只有 `:`）开头作为前缀的段落，输出时前缀的这部份会被去除掉。表格标题可以放在表格之前或之后。

表头也可以省略，在省略表头的情况下，表格下方必须加上一行虚线以清楚标明表格的范围。例如：

    -------     ------ ----------   -------
         12     12        12             12
        123     123       123           123
          1     1          1              1
    -------     ------ ----------   -------

当省略表头时，直行的对齐会以表格内容的第一行数据列决定。所以，以上面的表格为例，各直行的对齐依序会是靠右、靠左、置中以及靠右对齐。

### 多行表格

**Extension: `multiline_tables`, `table_captions`**

多行表格允许表头与表格数据格的文本能以复数行呈现（但不支持横跨多栏或纵跨多列的数据格）。以下为范例：

    -------------------------------------------------------------
     Centered   Default           Right Left
      Header    Aligned         Aligned Aligned
    ----------- ------- --------------- -------------------------
       First    row                12.0 Example of a row that
                                        spans multiple lines.

      Second    row                 5.0 Here's another one. Note
                                        the blank line between
                                        rows.
    -------------------------------------------------------------

    Table: Here's the caption. It, too, may span
    multiple lines.

看起来很像简单表格，但两者间有以下差别：

  - 在表头文本之前，必须以一列虚线作为开头（除非有省略表头）。
  - 必须以一列虚线作为表格结尾，之后接一个空白行。
  - 数据列与数据列之间以空白行隔开。

在多行表格中，表格分析器会计算各直行的栏宽，并在输出时尽可能维持各直行在原始文档中的相对比例。因此，要是你觉得某些字段在输出时不够宽，你可以在 markdown 的源文件中加宽一点。

和简单表格一样，表头在多行表格中也是可以省略的：

    ----------- ------- --------------- -------------------------
       First    row                12.0 Example of a row that
                                        spans multiple lines.

      Second    row                 5.0 Here's another one. Note
                                        the blank line between
                                        rows.
    ----------- ------- --------------- -------------------------

    : Here's a multiline table without headers.

多行表格中可以单只包含一个数据列，但该数据列之后必须接着一个空白行（然后才是标示表格结尾的一行虚线）。如果没有此空白行，此表格将会被解读成简单表格。

### 格框表格

**Extension: `grid_tables`, `table_captions`**

格框表格看起来像这样：

    : Sample grid table.

    +---------------+---------------+--------------------+
    | Fruit         | Price         | Advantages         |
    +===============+===============+====================+
    | Bananas       | $1.34         | - built-in wrapper |
    |               |               | - bright color     |
    +---------------+---------------+--------------------+
    | Oranges       | $2.10         | - cures scurvy     |
    |               |               | - tasty            |
    +---------------+---------------+--------------------+

以 `=` 串成的一行区分了表头与表格本体，这在没有表头的表格中也是可以省略的。在格框表格中的数据格可以包含任意的区块元素（复数段落、代码区块、清单等等）。不支持对齐，也不支持横跨多栏或纵跨多列的数据格。格框表格可以在 [Emacs table mode] 下轻松创建。

  [Emacs table mode]: http://table.sourceforge.net/

### 管线表格

**Extension: `pipe_tables`, `table_captions`**

管线表格看起来像这样：

    | Right | Left | Default | Center |
    |------:|:-----|---------|:------:|
    |   12  |  12  |    12   |    12  |
    |  123  |  123 |   123   |   123  |
    |    1  |    1 |     1   |     1  |

      : Demonstration of simple table syntax.

这个语法与 [PHP markdown extra 中的表格语法][the same as in PHP markdown extra] 相同。开始与结尾的管线字符是可选的，但各直行间则必须以管线区隔。上面范例中的冒号表明了对齐方式。表头可以省略，但表头下的水平虚线必须保留，因为虚在线定义了数据栏的对齐方式。

因为管线界定了各栏之间的边界，表格的原代码并不需要像上面例子中各栏之间保持直行对齐。所以，底下一样是个完全合法（虽然丑陋）的管线表格：

    fruit| price
    -----|-----:
    apple|2.05
    pear|1.37
    orange|3.09

管线表格的数据格不能包含如段落、清单之类的区块元素，也不能包含复数行文本。

  [the same as in PHP markdown extra]:
    http://michelf.ca/projects/php-markdown/extra/#table

注意：Pandoc 也可以看得懂以下形式的管线表格，这是由 Emacs 的 orgtbl-mod 所绘制：

    | One | Two   |
    |-----+-------|
    | my  | table |
    | is  | nice  |

主要的差别在于以 `+` 取代了部分的 `|`。其他的 orgtbl 功能并未支持。如果要指定非缺省的直行对齐形式，你仍然需要在上面的表格中自行加入冒号。

文档标题区块
-----------

（译注：本节中提到的「标题」均指 Title，而非 Headers）

**Extension: `pandoc_title_block`**

如果文件以文档标题（Title）区块开头

    % title
    % author(s) (separated by semicolons)
    % date

这部份将不会作为一般文本处理，而会以书目信息的方式解析。（这可用在像是单一 LaTeX 或是 HTML 输出文档的书名上。）这个区块仅能包含标题，或是标题与作者，或是标题、作者与日期。如果你只想包含作者却不想包含标题，或是只有标题与日期而没有作者，你得利用空白行：

    %
    % Author

    % My title
    %
    % June 15, 2006

标题可以包含多行文本，但接续行必须以空白字符开头，像是：

    % My title
      on multiple lines

如果文档有多个作者，作者也可以分列在不同行并以空白字符作开头，或是以分号间隔，或是两者并行。所以，下列各种写法得到的结果都是相同的：

    % Author One
      Author Two

    % Author One; Author Two

    % Author One;
      Author Two

日期就只能写在一行之内。

所有这三个 metadata 字段都可以包含标准的行内格式（斜体、链接、脚注等等）。

文档标题区块一定会被分析处理，但只有在 `--standaline` (`-s`) 选项被设置时才会影响输出内容。在输出 HTML 时，文档标题会出现的地方有两个：一个是在文档的 `<head>` 区块里－－这会显示在浏览器的窗口标题上－－另外一个是文档的 `<body>` 区块最前面。位于 `<head>` 里的文档标题可以选择性地加上前缀文本（透过 `--title-prefix` 或 `-T` 选项）。而在 `<body>` 里的文档标题会以 H1 元素呈现，并附带 "title" 类别 (class)，这样就能借由 CSS 来隐藏显示或重新定义格式。如果以 `-T` 选项指定了标题前缀文本，却没有设置文档标题区块里的标题，那么前缀文本本身就会被当作是 HTML 的文档标题。

而 man page 的输出器会分析文档标题区块的标题行，以解出标题、man page section number，以及其他页眉 (header) 页脚 (footer) 所需要的信息。一般会假设标题行的第一个单字为标题，标题后也许会紧接着一个以括号包住的单一数字，代表 section number（标题与括号之间没有空白）。在此之后的其他文本则为页脚与页眉文本。页脚与页眉文本之间是以单独的一个管线符号 (`|`) 作为区隔。所以，

    % PANDOC(1)

将会产生一份标题为 `PANDOC` 且 section 为 1 的 man page。

    % PANDOC(1) Pandoc User Manuals

产生的 man page 会再加上 "Pandoc User Manuals" 在页脚处。

    % PANDOC(1) Pandoc User Manuals | Version 4.0

产生的 man page 会再加上 "Version 4.0" 在页眉处。


反斜线跳脱字符
-----------------

**Extension: `all_symbols_escapable`**

除了在代码区块或行内代码之外，任何标点符号或空白字符前面只要加上一个反斜线，都能使其保留字面原义，而不会进行格式的转义解读。因此，举例来说，下面的写法

    *\*hello\**

输出后会得到

    <em>*hello*</em>

而不是

    <strong>hello</strong>

这条规则比原始的 markdown 规则来得好记许多，原始规则中，只有以下字符才有支持反斜线跳脱，不作进一步转义：

    \`*_{}[]()>#+-.!

（然而，如果使用了 `markdown_strict` 格式，那么就会采用原始的 markdown 规则）

一个反斜线之后的空白字符会被解释为不断行的空白 (nonbreaking space)。这在 TeX 的输出中会显示为 `~`，而在 HTML 与 XML 则是显示为 `\&#160;` 或 `\&nbsp;`。

一个反斜线之后的换行字符（例如反斜线符号出现在一行的最尾端）则会被解释为强制换行。这在 TeX 的输出中会显示为 `\\`，而在 HTML 里则是 `<br />`。相对于原始 markdown 是以在行尾加上两个空白字符这种「看不见」的方式进行强制换行，反斜线接换行字符会是比较好的替代方案。

反斜线跳脱字符在代码上下文中不起任何作用。

智能型标点符号
-----------------

**Extension**

如果指定了 `--smart` 选项，pandoc 将会输出正式印刷用的标点符号，像是将 straight quotes 转换为 curly quotes[^T1]、`---` 转为破折号 (em-dashes)，`--` 转为连接号 (en-dashes)，以及将 `...` 转为删节号。不断行空格 (Nonbreaking spaces) 将会插入某些缩写词之后，例如 "Mr."。

注意：如果你的 LaTeX template 使用了 `csquotes` 套件，pandoc 会自动侦测并且使用 `\enquote{...}` 在引言文本上。

[^T1]: 译注：straight quotes 指的是左右两侧都长得一样的引号，例如我们直接在键盘上打出来的单引号或双引号；curly quotes 则是左右两侧不同，有从两侧向内包夹视觉效果的引号。

行内格式
-----------------

### 强调 ###

要 *强调* 某些文本，只要以 `*` 或 `_` 符号前后包住即可，像这样：

    This text is _emphasized with underscores_, and this
    is *emphasized with asterisks*.

重复两个 `*` 或 `_` 符号以产生 **更强烈的强调**：

    This is **strong emphasis** and __with underscores__.

一个前后以空白字符包住，或是前面加上反斜线的 `*` 或 `_` 符号，都不会转换为强调格式：

    This is * not emphasized *, and \*neither is this\*.

**Extension: `intraword_underscores`**

因为 `_` 字符有时会使用在单字或是 ID 之中，所以 pandoc 不会把被字母包住的 `_` 解读为强调标记。如果有需要特别强调单字中的一部分，就用 `*`：

    feas*ible*, not feas*able*.


### 删除线 ###

**Extension:  `strikeout`**

要将一段文本加上水平线作为删除效果，将该段文本前后以 `~~` 包住即可。例如，

    This ~~is deleted text.~~


### 上标与下标 ###

**Extension: `superscript`, `subscript`**

要输入上标可以用 `^` 字符将要上标的文本包起来；要输入下标可以用 `~` 字符将要下标的文本包起来。直接看范例，

    H~2~O is a liquid.  2^10^ is 1024.

如果要上标或下标的文本中包含了空白，那么这个空白字符之前必须加上反斜线。（这是为了避免一般使用下的 `~` 和 `^` 在非预期的情况下产生出意外的上标或下标。）所以，如果你想要让字母 P 后面跟着下标文本 'a cat'，那么就要输入 `P~a\ cat~`，而不是 `P~a cat~`。


### 字面文本 ###

要让一小段文本直接以其字面形式呈现，可以用反引号将其包住：

    What is the difference between `>>=` and `>>`?

如果字面文本中也包含了反引号，那就使用双重反引号包住：

    Here is a literal backtick `` ` ``.

（在起始反引号后的空白以及结束反引号前的空白都会被忽略。）

一般性的规则如下，字面文本区段是以连续的反引号字符作为开始（反引号后的空白字符为可选），一直到同样数目的反引号字符出现才结束（反引号前的空白字符也为可选）。

要注意的是，反斜线跳脱字符（以及其他 markdown 结构）在字面文本的上下文中是没有效果的：

    This is a backslash followed by an asterisk: `\*`.

**Extension: `inline_code_attributes`**

与[围栏代码区块]一样，字面文本也可以附加属性：

    `<$>`{.haskell}

数学
----

**Extension: `tex_math_dollars`**

所有介于两个 `$` 字符之间的内容将会被视为 TeX 数学公式处理。开头的 `$` 右侧必须立刻接上任意文本，而结尾 `$` 的左侧同样也必须紧挨着文本。这样一来，`$20,000 and $30,000` 就不会被当作数学公式处理了。如果基于某些原因，有必须使用 `$` 符号将其他文本括住的需求时，那么可以在 `$` 前使用反斜线跳脱字符，这样 `$` 就不会被当作数学公式的分隔符。

TeX 数学公式会在所有输出格式中印出。至于会以什么方式演算编排 (render) 则取决于输出的格式：

Markdown, LaTeX, Org-Mode, ConTeXt
  ~ 公式会以字面文本呈现在两个 `$` 符号之间。

reStructuredText
  ~ 公式会使用 [此处](http://www.american.edu/econ/itex2mml/mathhack.rst) 所描述的 `:math:` 这个 "interpreted text role" 来进行演算编排。

AsciiDoc
  ~ 公式会以 `latexmath:[...]` 演算编排。

Texinfo
  ~ 公式会在 `@math` 指令中演算编排。

groff man
  ~ 公式会以去掉 `$` 后的字面文本演算编排。

MediaWiki
  ~ 公式会在 `<math>` 标签中演算编排。

Textile
  ~ 公式会在 `<span class="math">` 标签中演算编排。

RTF, OpenDocument, ODT
  ~ 如果可以的话，公式会以 unicode 字符演算编排，不然就直接使用字面字符。

Docbook
  ~ 如果使用了 `--mathml` 旗标，公式就会在 `inlineequation` 或 `informalequation` 标签中使用 mathml 演算编排。否则就会尽可能使用 unicode 字符演算编排。

Docx
  ~ 公式会以 OMML 数学标记的方式演算编排。

FictionBook2
  ~ 如果有使用 `--webtex` 选项，公式会以 Google Charts 或其他兼容的网络服务演算编排为图片，并下载嵌入于电子书中。否则就会以字面文本显示。

HTML, Slidy, DZSlides, S5, EPUB
  ~ 公式会依照以下命令行选项的设置，以不同的方法演算编排为 HTML 代码。

    1.  缺省方式是将 TeX 数学公式尽可能地以 unicode 字符演算编排，如同 RTF、DocBook 以及 OpenDocument 的输出。公式会被放在附有属性 `class="math"` 的 `span` 标签内，所以可以在需要时给予不同的样式，使其突出于周遭的文本内容。

    2.  如果使用了 `--latexmathml` 选项，TeX 数学公式会被显示于 `$` 或 `$$` 字符中，并放在附带 `LaTeX` 类别的 `<span>` 标签里。这段内容会用 [LaTeXMathML] script 演算编排为数学公式。（这个方法无法适用于所有浏览器，但在 Firefox 中是有效的。在不支持 LaTeXMathML 的浏览器中，TeX 数学公式会单纯的以两个 `$` 字符间的字面文本呈现。）

    3.  如果使用了 `--jsmath` 选项，TeX数学公式会放在 `<span>` 标签（用于行内数学公式）或 `<div>` 标签（用于区块数学公式）中，并附带类别属性 `math`。这段内容会使用 [jsMath] script 来演算编排。

    4.  如果使用了 `--mimetex` 选项，[mimeTeX] CGI script 会被调用来产生每个 TeX 数学公式的图片。这适用于所有浏览器。`--mimetex` 选项有一个可选的 URL 参数。如果没有指定 URL，它会假设 mimeTeX CGI script 的位置在 `/cgi-bin/mimetex.cig`。

    5.  如果使用了 `--gladtex` 选项，TeX 数学公式在 HTML 的输出中会被 `<eq>` 标签包住。产生的 `htex` 文件之后可以透过 [gladTeX] 处理，这会针对每个数学公式生成图片，并于最后生成一个包含这些图片链接的 `html` 文件。所以，整个处理流程如下：

            pandoc -s --gladtex myfile.txt -o myfile.htex
            gladtex -d myfile-images myfile.htex
            # produces myfile.html and images in myfile-images

    6.  如果使用了 `--webtex` 选项，TeX 数学公式会被转换为 `<img>` 标签并链接到一个用以转换公式为图片的外部 script。公式将会编码为 URL 可接受格式并且与指定的 URL 参数串接。如果没有指定 URL，那么将会使用 Google Chart API (`http://chart.apis.google.com/chart?cht=tx&chl=`)。

    7.  如果使用了 `--mathjax` 选项，TeX 数学公式将会被包在 `\(...\)`（用于行内数学公式）或 `\[...\]`（用于区块数学公式）之间显示，并且放在附带类别 `math` 的 `<span>` 标签之中。这段内容会使用 [MathJax] script 演算编排为页面上的数学公式。

[LaTeXMathML]: http://math.etsu.edu/LaTeXMathML/
[jsMath]:  http://www.math.union.edu/~dpvc/jsmath/
[mimeTeX]: http://www.forkosh.com/mimetex.html
[gladTeX]:  http://ans.hsh.no/home/mgg/gladtex/
[MathJax]: http://www.mathjax.org/

Raw HTML
--------

**Extension: `raw_html`**

Markdown 允许你在文档中的任何地方插入原始 HTML（或 DocBook）指令（除了在字面文本上下文处，此时的 `<`, `>` 和 `&` 都会按其字面意义显示）。（技术上而言这不算扩展功能，因为原始 markdown 本身就有提供此功能，但做成扩展形式便可以在有特殊需要的时候关闭此功能。）

输出 HTML, S5, Slidy, Slideous, DZSlides, EPUB, Markdown 以及 Textile 等格式时，原始 HTML 代码会不作修改地保留至输出文件中；而其他格式的输出内容则会将原始 HTML 代码去除掉。

**Extension: `markdown_in_html_blocks`**

原始 markdown 允许你插入 HTML「区块」：所谓的 HTML 区块是指，上下各由一个空白行所隔开，开始与结尾均由所在行最左侧开始的一连串对称均衡的 HTML 标签。在这个区块中，任何内容都会当作是 HTML 来分析，而不再视为 markdown；所以（举例来说），`*` 符号就不再代表强调。

当指定格式为 `markdown_strict` 时，Pandoc 会以上述方式处理；但缺省情况下，Pandoc 能够以 markdown 语法解读 HTML 区块标签中的内容。举例说明，Pandoc 能够将底下这段

    <table>
    	<tr>
    		<td>*one*</td>
    		<td>[a link](http://google.com)</td>
    	</tr>
    </table>

转换为

    <table>
    	<tr>
    		<td><em>one</em></td>
    		<td><a href="http://google.com">a link</a></td>
    	</tr>
    </table>

而 `Markdown.pl` 则是保留该段原样。

这个规则只有一个例外：那就是介于 `<script>` 与 `<style>` 之间的文本都不会被拿来当作 markdown 解读。

这边与原始 markdown 的分歧，主要是为了让 markdown 能够更便利地混入 HTML 区块元素。比方说，一段 markdown 文本可以用 `<div>` 标签将其前后包住来进行样式指定，而不用担心里面的 markdown 不会被解译到。

Raw TeX
-------

**Extension: `raw_tex`**

除了 HTML 之外，pandoc 也接受文档中嵌入原始 LaTeX, TeX 以及 ConTeXt 代码。行内 TeX 指令会被保留并不作修改地输出至 LaTeX 与 ConTeXt 格式中。所以，举例来说，你可以使用 LaTeX 来导入 BibTeX 的引用文献：

    This result was proved in \cite{jones.1967}.

请注意在 LaTeX 环境下时，像是底下

    \begin{tabular}{|l|l|}\hline
    Age & Frequency \\ \hline
    18--25  & 15 \\
    26--35  & 33 \\
    36--45  & 22 \\ \hline
    \end{tabular}

位在 `begin` 与 `end` 标签之间的内容，都会被当作是原始 LaTeX 数据解读，而不会视为 markdown。

行内 LaTeX 在输出至 Markdown, LaTeX 及 ConTeXt 之外的格式时会被忽略掉。

LaTeX 宏
------------

**Extension: `latex_macros`**

当输出格式不是 LaTeX 时，pandoc 会分析 LaTeX 的 `\newcommand` 和 `\renewcommand` 定义，并套用其产生的宏到所有 LaTeX 数学公式中。所以，举例来说，下列指令对于所有的输出格式均有作用，而非仅仅作用于 LaTeX 格式：

    \newcommand{\tuple}[1]{\langle #1 \rangle}

    $\tuple{a, b, c}$

在 LaTeX 的输出中，`\newcommand` 定义会单纯不作修改地保留至输出结果。


链接
-----

Markdown 接受以下数种指定链接的方式。

### 自动链接 ###

如果你用角括号将一段 URL 或是 email 地址包起来，它会自动转换成链接：

    <http://google.com>
    <sam@green.eggs.ham>

### 行内链接 ###

一个行内链接包含了位在方括号中的链接文本，以及方括号后以圆括号包起来的 URL。（你可以选择性地在 URL 后面加入链接标题，标题文本要放在引号之中。）

    This is an [inline link](/url), and here's [one with
    a title](http://fsf.org "click here for a good time!").

方括号与圆括号之间不能有空白。链接文本可以包含格式（例如强调），但链接标题则否。


### 参考链接 ###

一个 **明确** 的参考链接包含两个部分，链接本身以及链接定义，其中链接定义可以放在文档的任何地方（不论是放在链接所在处之前或之后）。

链接本身是由两组方括号所组成，第一组方括号中为链接文本，第二组为链接标签。（在两个方括号间可以有空白。）链接定义则是以方括号框住的链接标签作开头，后面跟着一个冒号一个空白，再接着一个 URL，最后可以选择性地（在一个空白之后）加入由引号或是圆括号包住的链接标题。

以下是一些范例：

    [my label 1]: /foo/bar.html  "My title, optional"
    [my label 2]: /foo
    [my label 3]: http://fsf.org (The free software foundation)
    [my label 4]: /bar#special  'A title in single quotes'

链接的 URL 也可以选择性地以角括号包住：

    [my label 5]: <http://foo.bar.baz>

链接标题可以放在第二行：

    [my label 3]: http://fsf.org
      "The free software foundation"

需注意链接标签并不区分大小写。所以下面的例子会创建合法的链接：

    Here is [my link][FOO]

    [Foo]: /bar/baz

在一个 **隐性** 参考链接中，第二组方括号的内容是空的，甚至可以完全地略去：

    See [my website][], or [my website].

    [my website]: http://foo.bar.baz

注意：在 `Markdown.pl` 以及大多数其他 markdown 实作中，参考链接的定义不能存在于嵌套结构中，例如清单项目或是区块引言。Pandoc lifts this arbitrary seeming restriction。所以虽然下面的语法在几乎所有其他实作中都是错误的，但在 pandoc 中可以正确处理：

    > My block [quote].
    >
    > [quote]: /foo

### 内部链接

要链接到同一份文档的其他章节，可使用自动产生的 ID（参见 [HTML, LaTeX 与 ConTeXt 的标题识别符] 一节后半）。例如：

    See the [Introduction](#introduction).

或是

    See the [Introduction].

    [Introduction]: #introduction

内部链接目前支持的格式有 HTML（包括 HTML slide shows 与 EPUB）、LaTeX 以及 ConTeXt。

图片
------

在链接语法的前面加上一个 `!` 就是图片的语法了。链接文本将会作为图片的替代文本（alt text）：

    ![la lune](lalune.jpg "Voyage to the moon")

    ![movie reel]

    [movie reel]: movie.gif

### 附上说明的图片 ###

**Extension: `implicit_figures`**

一个图片若自身单独存在一个段落中，那么将会以附上图片说明 (caption) 的图表 (figure) 形式呈现。[^5]（在 LaTeX 中，会使用图表环境；在 HTML 中，图片会被放在具有 `figure` 类别的 `div` 元素中，并会附上一个具有 `caption` 类别的 `p` 元素。）图片的替代文本同时也会用来作为图片说明。

    ![This is the caption](/url/of/image.png)

[^5]: 这项功能尚未在 RTF, OpenDocument 或 ODT 格式上实现。在这些格式中，你会得到一个在段落中只包含自己的图片，而无图片说明。

如果你只是想要个一般的行内图片，那么只要让图片不是段落里唯一的元素即可。一个简单的方法是在图片后面插入一个不断行空格：

    ![This image won't be a figure](/url/of/image.png)\


脚注
---------

**Extension: `footnotes`**

Pandoc's markdown 支持脚注功能，使用以下的语法：

    Here is a footnote reference,[^1] and another.[^longnote]

    [^1]: Here is the footnote.

    [^longnote]: Here's one with multiple blocks.

        Subsequent paragraphs are indented to show that they
    belong to the previous footnote.

            { some.code }

        The whole paragraph can be indented, or just the first
        line.  In this way, multi-paragraph footnotes work like
        multi-paragraph list items.

    This paragraph won't be part of the note, because it
    isn't indented.

脚注参考用的 ID 不得包含空白、tabs 或换行字符。这些 ID 只会用来创建脚注位置与脚注文本的对应关连；在输出时，脚注将会依序递增编号。

脚注本身不需要放在文档的最后面。它们可以放在文档里的任何地方，但不能被放入区块元素（清单、区块引言、表格等）之中。

**Extension: `inline_notes`**

Pandoc 也支持了行内脚注（尽管，与一般脚注不同，行内脚注不能包含多个段落）。其语法如下：

    Here is an inline note.^[Inlines notes are easier to write, since
    you don't have to pick an identifier and move down to type the
    note.]

行内与一般脚注可以自由交错使用。


引用
---------

**Extension: `citations`**

Pandoc 能够以数种形式自动产生引用与参考书目（使用 Andrea Rossato 的 `hs-citeproc`）。为了使用这项功能，你需要一个下列其中一种格式的参考书目数据库：

  Format            File extension
  ------------      --------------
  MODS              .mods
  BibLaTeX          .bib
  BibTeX            .bibtex
  RIS               .ris
  EndNote           .enl
  EndNote XML       .xml
  ISI               .wos
  MEDLINE           .medline
  Copac             .copac
  JSON citeproc     .json

需注意的是扩展名 `.bib` 一般而言同时适用于 BibTeX 与 BibLaTeX 的文件，不过你可以使用 `.bibtex` 来强制指定 BibTeX。

你需要使用命令行选项 `--bibliography` 来指定参考书目文件（如果有多个书目档就得反复指定）。

缺省情况下，pandoc 会在引用文献与参考书目中使用芝加哥「作者－日期」格式。要使用其他的格式，你需要用 `--csl` 选项来指定一个 [CSL] 1.0 格式的文件。关于创建与修改 CSL 格式的入门可以在 <http://citationstyles.org/downloads/primer.html> 这边找到。<https://github.com/citation-style-language/styles> 是 CSL 格式的文件库。也可以在 <http://zotero.org/styles> 以简单的方式浏览。

[CSL]: http://CitationStyles.org

引用信息放在方括号中，以分号区隔。每一条引用都会有个 key，由 `@` 加上数据库中的引用 ID 组成，并且可以选择性地包含前缀、定位以及后缀。以下是一些范例：

    Blah blah [see @doe99, pp. 33-35; also @smith04, ch. 1].

    Blah blah [@doe99, pp. 33-35, 38-39 and *passim*].

    Blah blah [@smith04; @doe99].

在 `@` 前面的减号 (`-`) 将会避免作者名字在引用中出现。这可以用在已经提及作者的文章场合中：

    Smith says blah [-@smith04].

你也可以在文本中直接插入引用信息，方式如下：

    @smith04 says blah.

    @smith04 [p. 33] says blah.

如果引用格式档需要产生一份引用作品的清单，这份清单会被放在文档的最后面。一般而言，你需要以一个适当的标题结束你的文档：

    last paragraph...

    # References

如此一来参考书目就会被放在这个标题后面了。

[markdown]: http://daringfireball.net/projects/markdown/
[reStructuredText]: http://docutils.sourceforge.net/docs/ref/rst/introduction.html


本简体中文译本是基于曾于修([Tzeng Yuxio](http://tzengyuxio.me)) 的繁体字译本:
[home]: http://pages.tzengyuxio.me/pandoc/
