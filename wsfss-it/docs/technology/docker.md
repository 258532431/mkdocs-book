---
comments: true
---

`Docker` 是一个开源的容器平台，可以轻松创建、运行、共享和部署应用。

阅读本文，您将获得 `Docker` 的实践经验，学习如何运行您的第一个容器，了解容器化的基础知识及其好处。本文还将指导您构建第一个 `Docker` 镜像，如何在 `Docker Hub` 上发布您的镜像，通过镜像运行您的应用。

## 能学到什么

* 如何运行第一个容器

* 如何使用 Dockerfile 构建镜像

* 如何在 Docker Hub 上发布镜像

* 如何部署运行您的镜像

## 安装 Docker

!!! note "注意"
    在规模较大的企业(员工超过250人或年收入超过1000万美元)中，Docker Desktop 的商业使用需要付费订阅。

官方提供了三种安装方式：

<div class="grid cards" markdown>

-   :simple-apple:{ .lg .middle } Docker Desktop for Mac

    ---

    [:octicons-arrow-right-24: Download (Apple Silicon)](https://desktop.docker.com/mac/main/arm64/Docker.dmg?utm_source=docker&utm_medium=webreferral&utm_campaign=docs-driven-download-mac-arm64&_gl=1*xzml4r*_ga*MTI2MjgyNjgwMy4xNzIwNjgzMzc2*_ga_XJWPQMJYHQ*MTcyMDY4MzM3Ni4xLjEuMTcyMDY4NzcyNy42MC4wLjA.)

    [:octicons-arrow-right-24: Download (Intel)](https://desktop.docker.com/mac/main/amd64/Docker.dmg?utm_source=docker&utm_medium=webreferral&utm_campaign=docs-driven-download-mac-amd64&_gl=1*12sny4l*_ga*MTI2MjgyNjgwMy4xNzIwNjgzMzc2*_ga_XJWPQMJYHQ*MTcyMDY4MzM3Ni4xLjEuMTcyMDY4NzcyNy42MC4wLjA.)

    [:octicons-arrow-right-24: Install instructions](https://docs.docker.com/desktop/install/mac-install)

-   :fontawesome-brands-windows:{ .lg .middle } Docker Desktop for Windows

    ---

    [:octicons-arrow-right-24: Download](https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe?utm_source=docker&utm_medium=webreferral&utm_campaign=docs-driven-download-windows&_gl=1*1uket0f*_ga*MTI2MjgyNjgwMy4xNzIwNjgzMzc2*_ga_XJWPQMJYHQ*MTcyMDY4MzM3Ni4xLjEuMTcyMDY4NzcyNy42MC4wLjA.)

-   :material-linux:{ .lg .middle } Docker Desktop for Linux

    ---

    [:octicons-arrow-right-24: Install instructions](https://docs.docker.com/desktop/install/linux-install/)

</div>

## 使用容器进行开发

官方提供一个简单的App样例 [getting-started-todo-app](https://github.com/docker/getting-started-todo-app)，我们可以使用它来制作镜像。

1. 下载样例项目到本地，使用 `Docker Compose` 启动开发环境。
```
git clone https://github.com/docker/getting-started-todo-app.git

cd getting-started-todo-app

docker compose watch
```

2. 打开浏览器访问：[http://localhost](http://localhost)，您将看到如下页面：
![示例](https://docs.docker.com/guides/getting-started/images/develop-getting-started-app-first-launch.webp){ align=enter }

## 开始你的第一个镜像

我们需要注册一个 [Docker Hub](https://hub.docker.com/?_gl=1*jcs0vz*_ga*MTI2MjgyNjgwMy4xNzIwNjgzMzc2*_ga_XJWPQMJYHQ*MTcyMDY4MzM3Ni4xLjEuMTcyMDY4ODk5MS40Ny4wLjA.) 账户，然后创建一个镜像仓库。

### 编写 Dockerfile

我们需要编写一个 `Dockerfile`，相关的参考命令可以在 [这里](https://docs.docker.com/reference/dockerfile/) 查阅。

在需要构建镜像的项目根目录创建一个 `Dockerfile` 文件，以 `spring-boot` 项目为例，内容如下：
```dockerfile title="Dockerfile"
# 该镜像需要依赖的基础镜像
FROM openjdk:8-jdk-alpine

# 在容器内创建/app工作目录
WORKDIR /app

# 将当前目录下的jar包复制到docker容器的/目录下 指定新的jar名称
ADD target/website_api-1.0.0-SNAPSHOT.jar website_api.jar

# 声明服务运行的端口
EXPOSE 9200

# 指定docker容器启动时运行jar包
ENTRYPOINT ["java", "-jar","/app/website_api.jar"]

# 指定维护者的名字
MAINTAINER fss
```

### 构建镜像

在 `Dockerfile` 所在目录，执行镜像构建命令：
```bash
# -t 表示指定<镜像仓库用户名>/<镜像名称> .表示使用当前目录下的 Dockerfile，注意镜像名称不要有短横杠，貌似无法识别
docker build -t yang258532/website_api .

# 删除镜像 表示指定<镜像仓库用户名>/<镜像名称>:<tag名称>
docker rmi yang258532/website_api:1.0.0

# 查看刚刚构建的镜像
docker image ls

# 给镜像打上版本号tag 指定<镜像仓库用户名>/<镜像名>:<tag名称> <镜像仓库用户名>/<镜像名>:<要打的tag名称>
docker tag yang258532/website_api:latest yang258532/website_api:1.0.0

# 如果需要的话，可以上传镜像到镜像仓库 指定<镜像仓库用户名>/<镜像名>:<tag名称>
docker push yang258532/website_api:1.0.0
```

如果 `push` 镜像报错：```denied: requested access to the resource is denied```，说明没有权限，我们登录一下就可以了

```bash
# 登录授权
docker login
```

### 运行镜像

使用刚刚构建好的镜像，部署在 `Docker` 容器中运行

```bash
# -d 后台运行返回容器ID -p 端口映射<主机(宿主)端口>:<容器端口> --name 为容器指定一个名称 <镜像名称>
docker run -d -p 9200:9200 --name website_api yang258532/website_api:1.0.0
```

### 验证

使用 `postman` 或者 `curl` 命令调用App的接口进行测试

```
curl -X GET "http://localhost:9200/site/getPv" -H "accept: application/json"
```

## 扩展

