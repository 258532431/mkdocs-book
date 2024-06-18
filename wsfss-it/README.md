### 运行环境

1. python3.12.2

2. mkdocs 1.6.0

### 运行项目

1. 进入python虚拟环境

```
    # 创建虚拟环境
    python3 -m venv path/to/venv

    # 激活虚拟环境
    source path/to/venv/bin/activate

    # 首次运行需安装material主题
    pip3 install mkdocs-material
```

2. 在 mkdocs.yml 同级目录下，执行命令

```
    # 构建
    mkdocs build

    # 启动服务
    mkdocs serve
```

3. 访问地址：http://127.0.0.1:8000/