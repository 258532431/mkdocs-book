FROM nginx

# 指定时区
RUN rm -f /etc/localtime \
&& ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
&& echo "Asia/Shanghai" > /etc/timezone

# 在容器内创建/app工作目录
WORKDIR /app

COPY site /app

RUN rm -rf /etc/nginx/nginx.conf
# 提前写好nginx配置文件，然后将准备好的配置文件拷贝至镜像中
COPY ./nginx.conf /etc/nginx/nginx.conf

# 声明服务运行的端口
EXPOSE 80