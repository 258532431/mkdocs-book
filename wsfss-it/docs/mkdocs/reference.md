---
comments: true
---

## 1. Mkdocs

### 网站设置

设置网站基本信息。

```yaml
site_name: mysite #网站名字
site_url: https://www.example.com #网站地址
site_author: wsfss #作者名
site_description: >- #网站描述
    Hello World! 
```

### 主题

只需要简单的添加以下几行内容到 `mkdocs.yml` 即可启用主题。请注意，由于有几种不同的安装方法，因此配置可能会略有不同。这里的案例是使用 `pip` 方式安装的。如果想要快速食用，可以使用最小配置：

=== "pip方式安装"

  ```yaml
  theme:
    name: material
  ```

如果你想丰富主题，可以借助 [material](#Material扩展) 主题添加额外的一些配置。

=== "mkdocs.yml"

```yaml
theme:
  name: material
  logo: assets/logo.png # 导航栏logo
  palette:
    - scheme: default # 日间模式
      primary: white #导航背景
      accent: amber #强调色 如悬停、按钮
      toggle:
        icon: material/weather-night # 图标
        name: 切换至夜间模式 # 鼠标悬浮提示
    - scheme: slate # 夜间模式
      primary: black #导航背景
      accent: amber #强调色 如悬停、按钮
      toggle:
        icon: material/weather-sunny
        name: 切换至日间模式
  language: zh # 语言
```

### 导航

通过添加 `nav` 设置来顺序添加有关导航中的每个页面，使用缩进添加子导航，配置如下:

=== "mkdocs.yml"

  ```yaml
  nav:
    - Home: index.md
    - About: about.md
    - Guide:
      - Overview: guide/overview.md
      - Getting Started: guide/getting-started.md
  ```

## 2. Material扩展<a name="Material扩展"></a>

### 修改颜色

MkDocs支持两种配色方案，浅色模式 `default` 和 深色模式 `slate` ，配色方案可以通过 `mkdocs.yml` 设置:

=== "mkdocs.yml"

  ``` yaml
  theme:
    palette:
      scheme: default
  ```

点击按钮切换配色模式:

<div class="mdx-switch">
  <button data-md-color-scheme="default"><code>default</code></button>
  <button data-md-color-scheme="slate"><code>slate</code></button>
</div>

<script>
  var buttons = document.querySelectorAll("button[data-md-color-scheme]")
  buttons.forEach(function(button) {
    button.addEventListener("click", function() {
      document.body.setAttribute("data-md-color-switching", "")
      var attr = this.getAttribute("data-md-color-scheme")
      document.body.setAttribute("data-md-color-scheme", attr)
      var name = document.querySelector("#__code_0 code span.l")
      name.textContent = attr
      setTimeout(function() {
        document.body.removeAttribute("data-md-color-switching")
      })
    })
  })
</script>

颜色用于标题、侧边栏、文本链接和其他几个组件。为了改变颜色，需要在mkdocs中设置以下值。将Yml转换为有效的颜色名称:

=== "mkdocs.yml"

  ```yaml
  theme:
    palette:
      primary: indigo
  ```

### 缩略语

技术文档通常会使用许多缩写词，这可能需要额外的解释，特别是对于项目的新用户。对于这些问题，Material For MkDocs使用Markdown扩展的组合来启用站点范围的词汇表。

我们需要配置扩展支持：

=== "mkdocs.yml"

  ```yaml
  markdown_extensions:
    - abbr
    - pymdownx.snippets
  ```

配置完成后，我们可以使用 `abbr` 标记来定义缩写词。

=== "示例"

  ```
  The HTML specification is maintained by the W3C.

  *[HTML]: Hyper Text Markup Language
  *[W3C]: World Wide Web Consortium
  ```

显示效果：

The HTML specification is maintained by the W3C.

*[HTML]: Hyper Text Markup Language
*[W3C]: World Wide Web Consortium

### 代码块

代码块和示例是技术项目文档的重要组成部分。Material for MkDocs提供了不同的方法来设置代码块的语法高亮显示。

需要配置如下扩展：

```yaml
markdown_extensions:
  - pymdownx.superfences
  - pymdownx.highlight:
      anchor_linenums: true #显示行号
      linenums_style: pymdownx.inline #行号样式
  - pymdownx.snippets #代码片段
  - pymdownx.keys #快捷键
```

通过代码块标记语法，您可以轻松地创建提示块。属性 `title` 指定代码块名称，`linenums` 指定行号开始的行数，`hl_lines` 指定高亮行（注意高亮行的起始行号为1，与 `linenums` 指定的行号无关）。

示例：

```
  ```java title="test.java" linenums="1" hl_lines="4"
  public class Test {

    public static void main(String[] args) {
      System.out.println("Hello World!");
    }
    
  }
  ```
```

显示效果：

```java title="test.java" linenums="1" hl_lines="4"
public class Test {

  public static void main(String[] args) {
    System.out.println("Hello World!");
  }

}
```

当启用 InlineHilite 时，还可以通过使用类似 `shebang-like` 的前缀来高亮显示，例如 `#!`，后面紧接该语言的短名称。

示例：```The `#!python range()` function is used to generate a sequence of numbers.```

显示效果：The `#!python range()` function is used to generate a sequence of numbers.

快捷键扩展支持允许插入键盘键，例如 ++ctrl+alt+del++。启用Keys后，键盘键可以用简单的语法呈现。查阅 [Python Markdown Extensions](https://facelessuser.github.io/pymdown-extensions/extensions/keys/) 文档了解所有可用的关键代码。

=== "示例"

  ```
  ++ctrl+alt+del++
  ```

### 提示

MkDocs的材料提供了几种不同类型的提示效果，并允许包含和嵌套任意内容。

首先需要配置扩展支持：

```yaml
markdown_extensions:
  - admonition
  - pymdownx.details
  - pymdownx.superfences
```

简单的用法是以 `!!!` 开始，空格后跟一个关键字，该关键字用作块的类型限定符。然后，该块的内容跟随下一行，缩进四个空格。

=== "示例":

  ```
  !!! note
      Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod
      nulla. Curabitur feugiat, tortor non consequat finibus, justo purus auctor
      massa, nec semper lorem quam in massa.
  ```

显示效果：

!!! note
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod
    nulla. Curabitur feugiat, tortor non consequat finibus, justo purus auctor
    massa, nec semper lorem quam in massa.

如果不需要显示 note 标题，可以如下配置：

=== "示例":

  ```
  !!! note ""
      Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod
      nulla. Curabitur feugiat, tortor non consequat finibus, justo purus auctor
      massa, nec semper lorem quam in massa.
  ```

显示效果：

!!! note ""
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod
    nulla. Curabitur feugiat, tortor non consequat finibus, justo purus auctor
    massa, nec semper lorem quam in massa.

使用 `???+` 作为前缀，可以实现折叠效果。

=== "示例":

  ```
  ???+ tip
      Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod
      nulla. Curabitur feugiat, tortor non consequat finibus, justo purus auctor
      massa, nec semper lorem quam in massa.

      === "代码块"

      ```javascript linenums="1"
      def bubble_sort(items):
      for i in range(len(items)):
          for j in range(len(items) - 1 - i):
              if items[j] > items[j + 1]:
                  items[j], items[j + 1] = items[j + 1], items[j]
      ```
  ```

显示效果：

???+ tip
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod
    nulla. Curabitur feugiat, tortor non consequat finibus, justo purus auctor
    massa, nec semper lorem quam in massa.

    === "代码块"

    ```javascript linenums="1"
    def bubble_sort(items):
    for i in range(len(items)):
        for j in range(len(items) - 1 - i):
            if items[j] > items[j + 1]:
                items[j], items[j + 1] = items[j + 1], items[j]
    ```

### 表格

数据表可以在项目文档的任何位置使用，并且可以包含任意 Markdown，包括内联代码块，以及图标和表情符号。

示例：

```
| Method      | Description:material-information-outline:{ title="Important information" }                          |
| ----------- | ------------------------------------ |
| `GET`       | :material-check:     Fetch resource  |
| `PUT`       | :material-check-all: Update resource |
| `DELETE`    | :material-close:     Delete resource |
```

显示效果：

| Method      | Description:material-information-outline:{ title="Important information" }                          |
| ----------- | ------------------------------------ |
| `GET`       | :material-check:     Fetch resource  |
| `PUT`       | :material-check-all: Update resource |
| `DELETE`    | :material-close:     Delete resource |

### 表情

Emoji扩展是Python Markdown扩展的一部分，它增加了在*.svg文件格式中集成表情符号和图标的能力，这些文件格式在构建网站时被内联 [[^^搜索表情^^](https://squidfunk.github.io/mkdocs-material/reference/icons-emojis/)]。

需要配置如下扩展：

=== "mkdoc.yml"

  ```
  markdown_extensions:
    - pymdownx.emoji:
        emoji_index: !!python/name:material.extensions.emoji.twemoji
        emoji_generator: !!python/name:material.extensions.emoji.to_svg
  ```

示例：```:smile: 开心```

显示效果：:smile: 开心

### 列表和链接

可以通过在一行前加上 `-` 、`*` 或 `+` 列表标记来编写无序列表，所有这些标记都可以互换使用。此外，所有类型的列表都可以相互嵌套。

示例：

```
- Nulla et rhoncus turpis. Mauris ultricies elementum leo. Duis efficitur
  accumsan nibh eu mattis. Vivamus tempus velit eros, porttitor placerat nibh
  lacinia sed. Aenean in finibus diam.

    * Duis mollis est eget nibh volutpat, fermentum aliquet dui mollis.
    * Nam vulputate tincidunt fringilla.
    * [baidu.com](https://www.baidu.com).
```

显示效果：

- Nulla et rhoncus turpis. Mauris ultricies elementum leo. Duis efficitur
  accumsan nibh eu mattis. Vivamus tempus velit eros, porttitor placerat nibh
  lacinia sed. Aenean in finibus diam.

    * Duis mollis est eget nibh volutpat, fermentum aliquet dui mollis.
    * Nam vulputate tincidunt fringilla.
    * [baidu.com](https://www.baidu.com).

### 图片

属性列表扩展是标准Markdown库的一部分，它允许向Markdown元素添加HTML属性和CSS类，并且可以通过 `mkdocs.yml` 启用。当属性列表扩展启用时，图像可以通过 `align` 属性添加各自的对齐方向来对齐，即 `align=left` 或 `align=right`

需要配置扩展：

=== "mkdocs.yml"

  ```
  markdown_extensions:
    - attr_list
    - md_in_html
  ```

如果你想在你的文档中添加图像缩放功能，则需要使用 `pip` 安装安装 `glightbox` 插件:

```
pip3 install mkdocs-glightbox
```

然后添加插件支持：

=== "mkdocs.yml"

  ```
  plugins:
    - glightbox
  ```

示例：

=== "图片在左侧"

    ``` markdown title="左侧效果展示"
    ![Image title](https://dummyimage.com/600x400/eee/aaa){ align=left width=300 loading=lazy }

    <div markdown>这里是文本内容！</div>
    ```

    <div markdown>

    ![Image title](https://dummyimage.com/600x400/f5f5f5/aaaaaa?text=–%20Image%20–){ align=left width=300 loading=lazy }

    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod
    nulla. Curabitur feugiat, tortor non consequat finibus, justo purus auctor
    massa, nec semper lorem quam in massa.

    </div>

=== "图片在右侧"

    ``` markdown title="右侧效果展示"
    ![Image title](https://dummyimage.com/600x400/eee/aaa){ align=right width=300 loading=lazy }

    <div markdown>这里是文本内容！</div>
    ```

    <div markdown>

    ![Image title](https://dummyimage.com/600x400/f5f5f5/aaaaaa?text=–%20Image%20–){ align=right width=300 loading=lazy }

    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod
    nulla. Curabitur feugiat, tortor non consequat finibus, justo purus auctor
    massa, nec semper lorem quam in massa.

    </div>

### 网格

如果需要将相近含义内容排列成网格，或用于构建索引页，可以添加如下配置：

```
<div class="grid cards" markdown>

- :fontawesome-brands-html5: __HTML__ for content and structure
- :fontawesome-brands-js: __JavaScript__ for interactivity
- :fontawesome-brands-css3: __CSS__ for text running out of boxes
- :fontawesome-brands-internet-explorer: __Internet Explorer__ ... huh?

</div>
```

显示效果：

<div class="grid cards" markdown>

- :fontawesome-brands-html5: __HTML__ for content and structure
- :fontawesome-brands-js: __JavaScript__ for interactivity
- :fontawesome-brands-css3: __CSS__ for text running out of boxes
- :fontawesome-brands-internet-explorer: __Internet Explorer__ ... huh?

</div>

进阶的用法，在网格列表元素中可以包含任意 Markdown 语句，只要周围的 `div` 定义了 Markdown 属性。下面是一个更复杂的例子，其中包括图标和链接:

```
<div class="grid cards" markdown>

-   :material-clock-fast:{ .lg .middle } __Set up in 5 minutes__

    ---

    Install [`mkdocs-material`](#) with [`pip`](#) and get up
    and running in minutes

    [:octicons-arrow-right-24: Getting started](#)

-   :fontawesome-brands-markdown:{ .lg .middle } __It's just Markdown__

    ---

    Focus on your content and generate a responsive and searchable static site

    [:octicons-arrow-right-24: Reference](#)

-   :material-format-font:{ .lg .middle } __Made to measure__

    ---

    Change the colors, fonts, language, icons, logo and more with a few lines

    [:octicons-arrow-right-24: Customization](#)

-   :material-scale-balance:{ .lg .middle } __Open Source, MIT__

    ---

    Material for MkDocs is licensed under MIT and available on [GitHub]

    [:octicons-arrow-right-24: License](#)

</div>
```

显示效果：

<div class="grid cards" markdown>

-   :material-clock-fast:{ .lg .middle } __Set up in 5 minutes__

    ---

    Install [`mkdocs-material`](#) with [`pip`](#) and get up
    and running in minutes

    [:octicons-arrow-right-24: Getting started](#)

-   :fontawesome-brands-markdown:{ .lg .middle } __It's just Markdown__

    ---

    Focus on your content and generate a responsive and searchable static site

    [:octicons-arrow-right-24: Reference](#)

-   :material-format-font:{ .lg .middle } __Made to measure__

    ---

    Change the colors, fonts, language, icons, logo and more with a few lines

    [:octicons-arrow-right-24: Customization](#)

-   :material-scale-balance:{ .lg .middle } __Open Source, MIT__

    ---

    Material for MkDocs is licensed under MIT and available on [GitHub]

    [:octicons-arrow-right-24: License](#)

</div>

### 脚注

脚注内容使用 `[^1]:` 标识符声明。它可以插入到文档中的任意位置，并且总是呈现在页面的底部。此外，还会自动添加到脚注引用的反向链接。

=== "脚注引用"

  ```
  [^1]: Lorem ipsum dolor sit amet, consectetur adipiscing elit.

  [^2]:
      Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod
      nulla. Curabitur feugiat, tortor non consequat finibus, justo purus auctor
      massa, nec semper lorem quam in massa.
  ```

显示效果：

[^1]: Lorem ipsum dolor sit amet, consectetur adipiscing elit.

[^2]:
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod
    nulla. Curabitur feugiat, tortor non consequat finibus, justo purus auctor
    massa, nec semper lorem quam in massa.

