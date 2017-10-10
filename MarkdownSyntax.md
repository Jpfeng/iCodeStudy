# Markdown语法(Syntax)

Markdown 是一种轻量级标记语言，这个语言的目的是希望大家使用“易于阅读、易于撰写的纯文本格式，并选择性的转换成有效的 XHTML (或是 HTML)”。其中最重要的设计是可读性，也就是说这个语言应该要能直接在字面上的被阅读，而不用被一些格式化指令标记 (像是 RTF 与 HTML)。许多网站都使用 Markdown 或是其变种，例如：GitHub, reddit, Stack Exchange 与 SourceForge 让用户更利于讨论。

## 目录(Index)

- [段落(Paragraphs)](#para)
- [换行(Line Breaks)](#lb)
- [标题(Headers)](#header)
- [列表(Lists)](#list)
- [引用(Quotes)](#quote)
- [强调(Emphasis)](#emphasis)
- [代码(Code)](#code)
- [分隔线(Horizontal Rules)](#hr)
- [链接(Links)](#link)
- [图片(Images)](#image)

<h2 id="para">段落(Paragraphs)</h2>

一个 Markdown 段落是由一个或多个连续的文本行组成，它的前后要有一个以上的空行。普通段落不该用空格或制表符来缩进。

    这是一个段落。
    段落之间要有空行。

    这是另外一个段落。

**效果：**

这是一个段落。
段落之间要有空行。

这是另外一个段落。

<h2 id="lb">换行(Line Breaks)</h2>

Markdown 允许段落内的强制换行（插入换行符），在需要插入处先按入两个以上的空格然后回车。

    在这里  
    强制  
    换行。

**效果：**

在这里  
强制  
换行。

<h2 id="header">标题(Headers)</h2>

Markdown 支持两种标题语法。第一种，使用 `=` 和 `-` 作为底线，代表一级和二级标题。

    一级标题
    =======

    二级标题
    -------

**效果：**

一级标题
=======

二级标题
-------

第二种，在行首插入 1 到 6 个 `#` ，对应到标题 1 到 6 级标题，标题字号会相应降低。`#` 与标题文本间需要一个空格。

    # 一级标题
    ## 二级标题
    ### 三级标题
    #### 四级标题
    ##### 五级标题
    ###### 六级标题

**效果：**

# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题

<h2 id="list">列表(Lists)</h2>

Markdown 支持有序列表和无序列表。无序列表使用 `-` 作为标记, 使用 `+` 或者 `*` 同样可以。与标题相似，标记与文本之间需要一个空格。

    - 项目
    - 项目

    + 项目
    + 项目

    * 项目
    * 项目

**效果：**

- 项目
- 项目

+ 项目
+ 项目

* 项目
* 项目

有序列表则使用英文的阿拉伯数字和句点作为标记。

    1. 项目1
    2. 项目2
    3. 项目3

**效果：**

1. 项目1
2. 项目2
3. 项目3

<h2 id="quote">引用(Quotes)</h2>

在需要引用他人的文本时，你只需要在你希望引用的文本前面加上 `>` 标记。为了整洁，标记与文本之间可以添加一个空格。

    > Lorem ipsum dolor sit amet, consectetur adipiscing elit

**效果：**

> Lorem ipsum dolor sit amet, consectetur adipiscing elit

引用的区块中同样支持 Markdown 语法：如标题，列表等。

    > ### 标题3
    >
    > - 项目1
    > - 项目2
    >
    > 引用可以嵌套：
    >
    > > Lorem ipsum dolor sit amet

**效果：**

> ### 标题3
>
> - 项目1
> - 项目2
>
> 引用可以嵌套：
>
> > Lorem ipsum dolor sit amet

<h2 id="emphasis">强调(Emphasis)</h2>

Markdown 使用 `*` 或 `_` 作为标记强调文本的符号。被一个 `*` 或 `_` 包围的文本会显示为斜体，被两个 `*` 或 `_` 包围的文本会显示为粗体。需要注意，用什么符号开启标签，就要用什么符号结束。并且`*` 或 `_` 两边都有空格的话，它们就只会被当成普通符号。使用 `~~` 包围文本则会在文本上显示一条删除线。

    *斜体1*

    _斜体2_

    **粗体1**

    __粗体2__

    ***加粗斜体1***

    ___加粗斜体2___

    ~~删除线~~

**效果：**

*斜体1*

_斜体2_

**粗体1**

__粗体2__

***加粗斜体1***

___加粗斜体2___

~~删除线~~

<h2 id="code">代码(Code)</h2>

如果要标记一小段行内代码，你可以使用 `` ` `` 包围代码。

    `Hello World`

**效果：**

`Hello World`

如果要标记多行代码，可以将 `` ``` `` 置于代码的首行和末行，并在可以首行标记后声明代码类型。

    ```java
    TextView tv = (TextView) findViewById(R.id.tv_msg);
    if (show) {
        tv.setText("Hello World");
    }
    Log.d("TAG", "finish");
    ```

**效果：**

```java
TextView tv = (TextView) findViewById(R.id.tv_msg);
if (show) {
    tv.setText("Hello World");
}
Log.d("TAG", "finish");
```

<h2 id="hr">分隔线(Horizontal Rules)</h2>

可以在一行中用三个以上的 `*` ， `-` 或者 `_` 来建立一个分隔线，行内不能有其他文本。在 `*` 或者 `-` 中间可以插入空格。

    ***

    * * *

    ---

    ___

    ----------

**分隔线效果：**

----------

<h2 id="link">链接(Links)</h2>

Markdown 支持两种形式的链接语法： **行内式**和**参考式**两种形式。

### 行内式(inline)

`[]` 内是链接文字， `()` 内是链接地址。你还可以使用 `""` 指定title，鼠标悬停在链接文字时弹出的说明文字。链接地址与title之间需要有一个空格。

    行内式链接： [Bing](http://www.bing.com/ "微软 Bing 搜索")

**效果：**

行内式链接： [Bing](http://www.bing.com/ "微软 Bing 搜索")

### 参考式(reference)

参考式链接分为两部分。在需要添加链接的地方插入两个 `[]` ，第一个方括号中填写链接文字，第二个方括号中填写链接标记。然后在文本的任意位置用包裹链接标记的方括号，冒号，一个空格，链接地址以及可选的title文字来定义链接地址。

    参考式链接 [Bing][1]

    [1]: http://www.bing.com "微软 Bing 搜索"

**效果：**

参考式链接 [Bing][1]

[1]: http://www.bing.com "微软 Bing 搜索"

如果链接文字本身可以做为链接标记，你也可以在添加链接时省略链接标记，在定义链接地址时使用链接文字作为链接标记。

    简化的参考式链接 [Bing][]

    [Bing]: http://www.bing.com "微软 Bing 搜索"

**效果：**

简化的参考式链接 [Bing][]

[Bing]: http://www.bing.com "微软 Bing 搜索"

### 自动链接(automatic links)

Markdown 支持以比较简短的自动链接形式来处理网址和电子邮件信箱，只要是用 `<>` 包起来， Markdown 就会自动把它转成链接。一般网址的链接文字就和链接地址一样。

    <http://www.bing.com>

    <fengjup@live.com>

**效果：**

<http://www.bing.com>

<fengjup@live.com>

<h2 id="image">图片(Images)</h2>

插入图片的方法与插入链接类似。在链接的语法前加一个 `!` 就可以插入图片。插入图片同样支持行内式和参考式两种形式。

    ![Wechat](https://open.weixin.qq.com/zh_CN/htmledition/res/assets/res-design-download/icon64_wx_logo.png "Wechat logo")

    ![Weibo][logo]

    [logo]: https://www.sinaimg.cn/blog/developer/wiki/LOGO_64x64.png "Weibo logo"

**效果：**

![Wechat](https://open.weixin.qq.com/zh_CN/htmledition/res/assets/res-design-download/icon64_wx_logo.png "Wechat logo")

![Weibo][logo]

[logo]: https://www.sinaimg.cn/blog/developer/wiki/LOGO_64x64.png "Weibo logo"
