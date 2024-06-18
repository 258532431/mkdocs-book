[material]: #修改颜色

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

如果你想丰富主题，可以借助 [material] 主题添加额外的一些配置。

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

## 2. Material扩展

### 修改颜色

颜色用于标题、侧边栏、文本链接和其他几个组件。为了改变颜色，需要在mkdocs中设置以下值。将Yml转换为有效的颜色名称:

=== "mkdocs.yml"

```yaml
theme:
  palette:
    primary: indigo
```

点击一个按钮来改变颜色:

<div class="mdx-switch">
  <button data-md-color-primary="red"><code>red</code></button>
  <button data-md-color-primary="pink"><code>pink</code></button>
  <button data-md-color-primary="purple"><code>purple</code></button>
  <button data-md-color-primary="deep-purple"><code>deep purple</code></button>
  <button data-md-color-primary="indigo"><code>indigo</code></button>
  <button data-md-color-primary="blue"><code>blue</code></button>
  <button data-md-color-primary="light-blue"><code>light blue</code></button>
  <button data-md-color-primary="cyan"><code>cyan</code></button>
  <button data-md-color-primary="teal"><code>teal</code></button>
  <button data-md-color-primary="green"><code>green</code></button>
  <button data-md-color-primary="light-green"><code>light green</code></button>
  <button data-md-color-primary="lime"><code>lime</code></button>
  <button data-md-color-primary="yellow"><code>yellow</code></button>
  <button data-md-color-primary="amber"><code>amber</code></button>
  <button data-md-color-primary="orange"><code>orange</code></button>
  <button data-md-color-primary="deep-orange"><code>deep orange</code></button>
  <button data-md-color-primary="brown"><code>brown</code></button>
  <button data-md-color-primary="grey"><code>grey</code></button>
  <button data-md-color-primary="blue-grey"><code>blue grey</code></button>
  <button data-md-color-primary="black"><code>black</code></button>
  <button data-md-color-primary="white"><code>white</code></button>
</div>

<script>
  var buttons = document.querySelectorAll("button[data-md-color-primary]")
  buttons.forEach(function(button) {
    button.addEventListener("click", function() {
      var attr = this.getAttribute("data-md-color-primary")
      document.body.setAttribute("data-md-color-primary", attr)
      var name = document.querySelector("#__code_1 code span.l")
      name.textContent = attr.replace("-", " ")
    })
  })
</script>