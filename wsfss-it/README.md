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

### 生产部署

1. 构建镜像
```
docker build --platform linux/amd64 -t yang258532/mkdocs:latest . 
```

2. 导出镜像
```
docker save -o /Users/ycl/Desktop/soft/docker/sealos-k8s/mkdocs.tar yang258532/mkdocs:latest
```

3. 上传镜像和mkdocs-deployment.yaml到云服务器

4. 加载镜像
```
# 登录私有镜像仓库
sealos login -u admin -p passw0rd sealos.hub:5000
# 加载镜像
sealos load -i mkdocs.tar
# 打tag
sealos tag yang258532/mkdocs:latest sealos.hub:5000/mkdocs:latest
# 推送镜像到仓库
sealos push sealos.hub:5000/mkdocs:latest
```

5. 部署
```
kubectl apply -f mkdocs-deployment.yaml
```