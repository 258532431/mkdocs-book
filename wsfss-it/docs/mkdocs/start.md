---
comments: true
---

## 1. 介绍

Material for MkDocs是MkDocs的一个主题，MkDocs是一个用于生成静态页面的程序。我们使用 `pip` 这个Python的包管理器来安装Material for MkDocs。

## 2. 安装

### 安装 python 虚拟环境

安装好 `python` 后（这里使用 python 3.12.2 版本），在命令行执行以下命令：

=== "创建虚拟环境"

    ```python
    python3 -m venv path/to/venv
    ```

=== "激活虚拟环境"

    ```python
    source path/to/venv/bin/activate
    ```

### 安装 mkdocs

要安装MkDocs，在命令行中运行以下命令：

=== "Mkdocs"

```python
pip3 install mkdocs
```

???+ 提示

    查询到 Mkdocs 版本信息，表示安装成功

    ```
    $ mkdocs --version
    mkdocs, version 1.6.0 from /Users/ycl/Desktop/myspace/mkdocs/path/to/venv/lib/python3.12/site-packages/mkdocs (Python 3.12)
    ```

### 自定义主题

MkDocs支持多种主题，这里使用 `pip3` 命令，安装 material 主题依赖，详情请查看官方文档 [^^mkdocs-material^^](https://squidfunk.github.io/mkdocs-material/)。

=== "Material for MkDocs"

```python
pip3 install mkdocs-material
```

## 3. 创建项目

要创建一个新项目，在命令行中运行以下命令：

```
$ mkdocs new project-name
$ cd project-name
```

## 4. 运行

在 `mkdocs.yml` 同级目录下，执行命令：

```
# 构建
$ mkdocs build

# 启动服务
$ mkdocs serve
```

在浏览器中打开 [http://127.0.0.1:8000/](http://127.0.0.1:8000/) ，您将看到显示默认主页，请放心食用。

## 5. 发布

将网站托管在git库中的最大好处是能够在推送新更改时自动部署它，MkDocs使得这一操作更加简单，当然你也可以选择手动部署。

### 手动部署

`mkdocs build` 构建成功后，会生成 `site/` 文件夹，将此文件夹上传至任何服务器即可，这里使用宝塔面板进行静态网站部署。

???+ 提示

    如果使用宝塔面板部署多个静态网站，不要设置默认站点，否则无法访问新添加的网站。

首先，新建一个网站站点，PHP版本选择纯静态，其他配置保持默认。

上传 `site/` 文件夹到站点目录下，然后在站点配置文件中指定网站根目录为 `站点目录/site`。

```xml
server {
    root /www/wwwroot/book.wsfss.top/site;
}
```

:smile: 恭喜你，部署成功！

### GitHub Pages

使用GitHub Actions可以自动部署网站。在库的根目录下新建一个GitHub Actions workflow，比如：`.github/workflows/ci.yml`，并粘贴入以下内容：

=== "ci.yml"

```yaml
name: ci
on:
  push:
    branches:
      - master
      - main
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-python@v2
        with:
          python-version: 3.x
      - run: pip install mkdocs-material
      - run: mkdocs gh-deploy --force
```

此时，当一个新的提交推送到master或main时，我们的静态网站的内容将自动生成并完成部署。可以尝试推送一个提交来查看GitHub Actions的工作状况。

如果是倾向于手动部署网站，请在包含 `mkdocs.yml` 文件的目录中运行以下命令：

```
mkdocs gh-deploy --force
```


