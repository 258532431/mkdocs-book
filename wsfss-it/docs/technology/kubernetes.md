---
comments: true
---

我们已经学会了使用 Docker Compose 来管理我们的应用程序，但是如果我们想要在 Kubernetes 上运行我们的应用程序呢？

容器化有效地保证您的应用程序在任何地方以相同的方式运行，使您能够快速轻松地利用所有这些环境。此外，当您扩展应用程序时，您需要一些工具来帮助自动维护这些应用程序，自动替换失败的容器，并管理这些容器在其生命周期内更新。

## 安装

Mac Docker Desktop 安装 Kubernetes。

1. 从Docker仪表板，导航到设置，然后选择Kubernetes选项卡。

2. 选择标有Enable Kubernetes的复选框，然后选择Apply & Restart。Docker Desktop会自动为您设置Kubernetes。当您在“设置”中看到“Kubernetes正在运行”旁边的绿灯时，您就会知道Kubernetes已成功启用。

!!! note "注意"
    如果使用最新版，请不要使用国内的镜像仓库，否则会找不到版本。使用默认的镜像仓库，请打开VPN，否则会下载失败。

3. 要确认Kubernetes已启动并运行，请创建一个名为pod.yaml的文本文件，其中包含以下内容：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo
spec:
  containers:
  - name: testpod
    image: alpine:latest
    command: ["ping", "8.8.8.8"]
```
这描述了一个带有单个容器的pod，将一个简单的ping隔离到8.8.8.8。

4. 在终端中，导航到您创建pod.yaml的位置并创建您的pod：
```bash
## 创建pod
kubectl apply -f pod.yaml

## 查看pod
kubectl get pods

## 测试
kubectl logs demo

## 删除pod
kubectl delete -f pod.yaml
```

## 启用仪表盘

下载依赖文件：[recommended.yaml](https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml)，重命名文件为 `dashboard.yaml` 。

修改一下 `dashboard.yaml` 配置，将类型改为 NodePort ，并设置 ```spec.ports.nodePort: 30043```
```yaml
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30043
  selector:
    k8s-app: kubernetes-dashboard
```

通过 `yaml` 部署：
```
kubectl apply -f dashboard.yaml
```

安装完成后，我们需要配置一下访问权限，以便我们能够访问到 Kubernetes Dashboard。使用Kubernetes的服务帐户机制创建新用户，授予此用户管理员权限，并使用与该用户绑定的无记名令牌登录仪表板。

我们正在首先在命名空间kubernetes-dashboard创建名为admin-user的服务帐户。
```yaml title="dashboard-adminuser.yaml"
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```

现在我们需要找到可用于登录的令牌。执行以下命令：
```
# 创建admin-user
kubectl apply -f dashboard-adminuser.yaml

# 获取token
kubectl -n kubernetes-dashboard create token admin-user
```

它应该打印以下内容：
```
eyJhbGciOiJSUzI1NiIsImtpZCI6IlJxMGhDcThhTm0wc0huS05keWFVWkxiaENjZ25xQWlZajVvdnd6TlM3ZkEifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzM2ODI0NjExLCJpYXQiOjE3MzY4MjEwMTEsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwianRpIjoiNjIzYWQ1ZWYtNmVmNy00MWQ2LTk2NmItNzEyZmJkMjUyY2E3Iiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJhZG1pbi11c2VyIiwidWlkIjoiOGYxYjBiMWMtZWUzMy00OTg5LTg0MjAtYjY5ZjU2MGMyMGM2In19LCJuYmYiOjE3MzY4MjEwMTEsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDphZG1pbi11c2VyIn0.uzB76SRw0UnlDw2DvyCugJz-2ktXfaSH3L2wb9QkLApM0pxEsQDyRk2G6KUWOnU74P54Lvp_vIwjLtl8i2epe_EAr5c4nHGSGA_Ar_K8I_RgV_cPbk6-fa2g3VHZWn0LpPBGshF6cUvyhS4O94avU_jvTK3C4A6oInuUicc1OseqmSm7aieMTHsWOFKuymGmSZAISJZPVOfb6FUzYg3FnBRCWq2fNfOfRxBrtnZG3-HzcdLDJ5kqncPny3bEf4QuBQk99d7KGftXpt9NHf7vuTU_-iP6wzSlEtSVyoHCaiHkF0jp6-LCwWFrgj0aprWYK41-BVWftDnMr_hcrfu7cw
```

为了避免每次登录都需要获取令牌，我们还可以创建一个带有绑定服务帐户的密钥的令牌，令牌将保存在密钥中：
```yaml title="dashboard-adminuser-secret.yaml"
apiVersion: v1
kind: Secret
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/service-account.name: "admin-user"   
type: kubernetes.io/service-account-token  
```

创建秘钥后，我们可以执行以下命令来获取保存在秘密中的令牌，方便下次取用：
```
# 创建秘钥
kubectl apply -f dashboard-adminuser-secret.yaml

# 这里获取的token多了一个'%'号，不要复制它（不知道为啥会出现，囧）
kubectl get secret admin-user -n kubernetes-dashboard -o jsonpath={".data.token"} | base64 -d
```

将admin-user权限授权给cluster-admin：
```yaml title="dashboard-adminuser-service-account.yaml"
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

执行命令，为ServiceAccount创建ClusterRoleBinding：
```
kubectl apply -f dashboard-adminuser-service-account.yaml
```

我们已经配置好了，下面可以访问仪表盘了。

现在访问仪表板，将上面获取的token复制粘贴进去：[https://localhost:30043](https://localhost:30043)
![alt text](../assets/k8s/kubernetes-dashboard.png)

配置的token默认900秒（15分钟）会过期，我们通过修改 Deployments 中的 `kubernetes-dashboard.yaml` 来延长过期时间：
```yaml
# 增加 '--token-ttl=86400' 这一行，然后点击更新
spec:
  template:
    spec:
      containers:
        - name: kubernetes-dashboard
          image: kubernetesui/dashboard:v2.7.0
          args:
            - '--auto-generate-certificates'
            - '--namespace=kubernetes-dashboard'
            - '--token-ttl=86400'
```

## 生成 kubeconfig 文件

使用token进行用户验证，每次过期都需要重新输入token，比较繁琐。

我们可以使用 kubeconfig 文件方式进行用户验证。

第一步，需要导出ca证书。Docker Destop 的证书内容在配置文件中，在Mac系统中位于 `~/.kube/config`。打开配置文件，可以找到一个名为 `certificate-authority-data` 的字段，该字段的值是经过Base64编码的ca.crt文件内容。可以将该值进行Base64解码，得到ca.crt。
```
# ca.crt
cat ~/.kube/config | grep certificate-authority-data | awk -F ":" '{print $2}' | tr -d " " | base64 -d > ca.crt
```

第二步，获取Kubernetes API Server 的信息。执行以下命令：
```
cat ~/.kube/config | grep server
```
输出：```server: https://kubernetes.docker.internal:6443```，因为我们是本地安装，所以地址对应的是：`https://localhost:6443`，匹配的集群名称是 `docker-desktop`，dashboard 匹配的用户名是 `dashboard-admin`。这里使用的 `token` 是我们进行登录验证用的 `token`。

第三步，使用 `kubectl` 工具，生成 kubeconfig 文件。
```
# 创建集群
kubectl config set-cluster docker-desktop \
    --certificate-authority=./ca.crt \
    --embed-certs=true \
    --server=https://localhost:6443 \
    --kubeconfig=dashboard-admin.conf

# 创建credentials
kubectl config set-credentials dashboard-admin \
    --token=eyJhbGciOiJSUzI1NiIsImtpZCI6InhOUXNjOGdzbjNSX2hfS21uM19WQWxDWmgxN1B5cmlvRnhvS2RoQi1LNWcifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzQyNTY5NDg3LCJpYXQiOjE3NDI1NjU4ODcsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwianRpIjoiNmZlOTYxYzItZjEwOC00MGIzLWE1NzktODE4YzRkYTAwYWYzIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJhZG1pbi11c2VyIiwidWlkIjoiN2U3NzY1MWYtMTEzMS00NzBmLWIzYWQtMDAyMTIyMWI4YmUxIn19LCJuYmYiOjE3NDI1NjU4ODcsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDphZG1pbi11c2VyIn0.WSnVztBGrg_p5xAhRkoWQdLO-1uxFOB3IYXY2oaonKOU1bFz3SuDcDqocsnlJvA9oWiAwAtlL3wtjdDfTaeM5SWPMkJcZioGT9t1zfPjqh9A6ClDvdUkvL-oP_2mAHbzGZhrjoDu94z9Hv3b_2c6Dhpt06hv8cmErpRHO5pbKvpY30b_IOxFU3aexYM96s8SsSDxYlPyY5cTGKYR01rOZyimnqGgYX_oayEU66dafeoWagz5Mgqa_jQQlMmXQ9SGQgG1vkfWXzGXn6KZHUriL0GUNZ3h2utYYXkxibwRQ-MvwK8QQX5bNVtmDPt0jRmNKMrK0_AUsk83HtvsJN1CZQ \
    --kubeconfig=dashboard-admin.conf

# 创建context
kubectl config set-context dashboard-admin@docker-desktop \
    --cluster=docker-desktop \
    --user=dashboard-admin \
    --kubeconfig=dashboard-admin.conf

# 切换context的current-context
kubectl config use-context dashboard-admin@docker-desktop --kubeconfig=dashboard-admin.conf
```

参考文献：

1. [https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)
2. [https://github.com/kubernetes/dashboard/blob/master/docs/user/accessing-dashboard/README.md](https://github.com/kubernetes/dashboard/blob/master/docs/user/accessing-dashboard/README.md)

## 集群通信凭证

k8s 使用 kubeconfig 文件进行通信验证。

第一步，需要导出ca证书。Docker Destop 的证书内容在配置文件中，在Mac系统中位于 `~/.kube/config`。打开配置文件，可以找到一个名为 `certificate-authority-data` 的字段，该字段的值是经过Base64编码的ca.crt文件内容。可以将该值进行Base64解码，得到ca.crt文件的内容，将其复制到创建的ca.crt文件中。`client-certificate-data` 字段的内容是经过Base64编码的client.crt文件内容。`client-key-data` 字段的内容对应client.key文件。
```
# ca.crt
cat config | grep certificate-authority-data | awk -F ":" '{print $2}' | tr -d " " | base64 -d > ca.crt

# client.crt
cat config | grep client-certificate-data | awk -F ":" '{print $2}' | tr -d " " | base64 -d > client.crt

# 查看证书内容
openssl x509 -in client.crt -noout -text

# client.key
cat config | grep client-key-data | awk -F ":" '{print $2}' | tr -d " " | base64 -d > client.key
```

第二步，获取Kubernetes API Server 的信息。执行以下命令：
```
cat ~/.kube/config | grep server
```
输出：```server: https://kubernetes.docker.internal:6443```，因为我们是本地安装，所以地址对应的是：`https://localhost:6443`，匹配的集群名称和用户名都是 `docker-desktop`。

第三步，使用 `kubectl` 工具，生成 kubeconfig 文件。
```
kubectl config set-cluster docker-desktop \
    --certificate-authority=./ca.crt \
    --embed-certs=true \
    --server=https://localhost:6443 \
    --kubeconfig=kubeconfig

kubectl config set-credentials docker-desktop \
    --client-certificate=./client.crt \
    --client-key=./client.key \
    --embed-certs=true \
    --kubeconfig=kubeconfig

kubectl config set-context docker-desktop \
    --cluster=docker-desktop \
    --user=docker-desktop \
    --kubeconfig=kubeconfig

kubectl config use-context docker-desktop --kubeconfig=kubeconfig
```

## 部署应用

我们现在要部署一个简单的应用，用于测试Kubernetes集群的部署功能。

### 创建命名空间

我们将创建一个开发环境的命名空间，命名空间名称指定为 `dev`，用于测试我们的应用。一般情况下，不同的命名空间其资源是隔离的。

```bash
# 创建dev命名空间
kubectl create namespace dev

# 列出所有的命名空间
kubectl get namespace
```

### 创建Pod和Service

我们需要编写一个配置文件，用于创建一个Deployment资源。Deployment资源是一个用来管理Pod的资源，它允许我们创建和删除Pod，并确保Pod的数量符合我们的需求。

除了Deployment资源，我们还需要创建一个Service资源，用于暴露Deployment的端口。Service资源是一个用来暴露Pod的资源，它允许我们通过一个固定的IP地址和端口来访问Pod。

我们继续使用介绍 Docker 时使用的 `springboot` 项目镜像来进行测试。下面是一个简单的Deployment和Service的配置文件，当配置文件中有多个对象时，需要使用 `---` 分隔开：
```yaml title="website_api-deployment.yaml"
# 解析此对象的Kubernetes API
apiVersion: apps/v1
# Pod的类型，这里指定为Deployment
kind: Deployment
metadata:
  # Deployment的名称
  name: website-api
  # Deployment所属的命名空间
  namespace: dev
spec:
  # 选择器，用于匹配Pods的标签
  selector:
    matchLabels:
      # 必须和 spec.template.metadata.labels 相同
      app: website-api
  # 副本数，即运行的Pod实例数
  replicas: 1
  # Pod准备好之前等待的最短秒数，用于健康检查，在该段时间内，更新操作会被阻塞
  minReadySeconds: 10
  # Pod模板
  template:
    metadata:
      # 标签，用于Selector选择，此处的定义会应用到 spec.template.spec下定义的所有pod副本
      labels:
        app: website-api
    spec:
      # 容器列表
      containers:
      # 容器名称
      - name: website-api
        # 容器镜像
        image: yang258532/website_api:1.0.0
        # 镜像拉取策略 
        # Always：每次启动Pod时都会尝试拉取镜像
        # IfNotPresent：如果本地不存在镜像则拉取，存在则使用本地镜像。如果不指定值，这是默认行为
        # Never：永远不会拉取镜像，只会使用本地镜像，Always：总是拉取镜像
        # imagePullPolicy: Always
        ports:
        # 容器暴露的端口，targetPort映射到containerPort
        - containerPort: 9200
---
apiVersion: v1
kind: Service
metadata:
   name: website-api
   namespace: dev
spec:
   # 服务类型，NodePort表示可以通过Node的IP和指定的端口访问服务
   type: NodePort
   # 选择器，用于选择哪些Pods属于此服
   selector:
      app: website-api
   # 服务端口配置
   ports:
      - name: website-api
        # 其他服务或Pods可以在集群内部通过端口80来访问标记为 app: website_api 的Pods
        port: 80
        # 目标端口，Pods上容器的实际监听端口
        targetPort: 9200
        # 外部访问此服务的端口，必须在30000-32767之间
        nodePort: 30000
```

执行命令：
```
# 创建Deployment和Service
kubectl apply -f website_api-deployment.yaml

# 查看， -n dev 表示指定命名空间为dev
kubectl get deployments -n dev

# 删除
kubectl delete -f website_api-deployment.yaml
```

由于我们的项目依赖redis，所以还需要创建redis的Deployment和Service。
```yaml title="redis-deployment.yaml"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: dev
spec:
  selector:
    matchLabels:
      app: redis
  replicas: 1
  minReadySeconds: 5
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        # 容器镜像
        image: yang258532/redis:latest
        ports:
        # 容器的端口号
        - containerPort: 6379
---
apiVersion: v1
kind: Service
metadata:
   name: redis
   namespace: dev
spec:
   type: NodePort
   selector:
      app: redis
   ports:
      - name: redis
        port: 6379
        targetPort: 6379
        nodePort: 30001
```

### 验证

我们现在登录 k8s dashboard，查看容器运行情况。

Pods运行状态正常。
![alt text](../assets/k8s/pods.png)

Services运行状态正常。
![alt text](../assets/k8s/services.png)

查看Service对应的外部IP地址和端口。
![alt text](../assets/k8s/service.png)

通过 `curl` 命令验证 app 接口是否可以正常调用：
```
curl --location 'http://localhost:30000/site/setPv'
```

!!! note "提示"
    由于我们是本地集群部署，所以Service对应的外部IP地址是 `localhost`，端口我们在上面配置的 `nodePort`。

## 搭建云环境K8s集群

云服务器系统选择的是 `Ubuntu 22.04 64位 UEFI版`

### 安装Docker

参考：
- [Ubuntu安装指南](https://docs.docker.com/engine/install/ubuntu/)

- [CentOS安装指南](https://docs.docker.com/engine/install/centos/)
```
# 建议用root用户登录

# 关闭swap
sudo swapoff -a
swapoff -a
#永久关闭swap分区，&符号在sed命令中代表上次匹配的结果
sed -ri 's/.*swap.*/#&/' /etc/fstab

# 修改主机名
hostnamectl set-hostname master01

# 所有节点修改hosts文件
cat >> /etc/hosts << EOF
10.0.20.16 master01
EOF

# 调整内核参数
vim /etc/sysctl.d/kubernetes.conf
# 修改内容
net.bridge.bridge-nf-call-ip6tables=1
net.bridge.bridge-nf-call-iptables=1
net.ipv6.conf.all.disable_ipv6=1
net.ipv4.ip_forward=1
#使参数生效
sysctl --system

# 关闭 SELinux
setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config

# 开放防火墙相关端口 或者 停止防火墙
sudo systemctl stop firewalld

# 卸载旧的docker
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
# 使用阿里云镜像
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 安装docker
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
# 启动docker
sudo systemctl start docker
sudo systemctl start containerd
# 设置开机启动
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
# 禁止开机启动
sudo systemctl disable docker.service
sudo systemctl disable containerd.service
# 验证
docker -v
sudo docker ps

# 提示无权限，执行命名后重新登录
sudo groupadd docker
sudo usermod -aG docker $USER

# 如有需要，删除安装包
# yum remove docker-ce
# 如有需要，删除容器、镜像、配置文件等
# rm -rf /var/lib/docker

# 设置镜像源
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 安装kubelet kubeadm
sudo yum install -y kubelet kubeadm
# 验证 版本号是v1.28.2
kubelet --version
kubeadm version

# 安装kubectl（超时多试几次）
# curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.28.2/bin/linux/x86_64/kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
# （可选）校验文件
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
# （可选）校验输出：kubectl: OK
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
# 执行安装
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl

sudo chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl


# 如果需要，卸载
# sudo yum remove kubelet kubeadm kubectl

# 启动kubelet，显示activating (auto-restart)不用管，等待kubeadm init
systemctl enable kubelet && systemctl start kubelet
systemctl status kubelet 

# 验证
kubectl version --client
kubectl version --client --output=yaml

# 初始化（需要保证docker在运行状态），为了使网络正常运行（使用calico），执行kubeadm init命令时需要增加--pod-network-cidr=192.168.0.0/16参数，这里修改了镜像源（因为k8s是谷歌开发的被和谐了）
# kubeadm init 会创建/etc/kubernetes/admin.conf、/etc/kubernetes/kubelet.conf、/var/lib/kubelet/config.yaml
# --kubernetes-version=v1.28.2 指定版本号和上面安装的一致；--apiserver-advertise-address 指定master节点的内网ip地址
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --image-repository=registry.aliyuncs.com/google_containers --token-ttl=0 --kubernetes-version=v1.28.2 --apiserver-advertise-address=10.0.20.16
# 当前机器会作为主节点自动加入k8s集群，如果没有加入k8s集群，需要执行kubeadm join命令
kubeadm join 10.0.20.16:6443 --token j3ceo1.y6swqt9qqhpg754q \
        --discovery-token-ca-cert-hash sha256:b3380c191e863741b90243ec70ed6e8a129a4ca7654cebca3bd5c45bb12c9309
# 设置集群用户
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
# 设置环境变量
cat <<EOF >> /root/.bashrc
export KUBECONFIG=/etc/kubernetes/admin.conf
EOF
source /root/.bashrc
# 查看集群信息
kubectl cluster-info
kubectl get cs
kubectl get node
kubectl get nodes -o wide

# 如果初始化失败时需要重置一下
sudo kubeadm reset
rm -rf $HOME/.kube
rm -rf /etc/cni/net.d
# 然后重新执行上面的 sudo kubeadm init 命令

# pod网络插件必要安装，以便pod可以相互通信
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
# 网络不行先下载到本地然后上传服务器
sudo scp /Users/ycl/Desktop/soft/docker/cloud_k8s/kube-flannel.yml root@106.55.31.167:/root
kubectl apply -f kube-flannel.yml
# 查看节点状态
kubectl get pods -n kube-system

# 发现coredns节点状态为pending，通过命令查看问题
kubectl describe pod coredns-66f779496c-bxg5w -n kube-system
kubectl describe node vm-20-16-centos
# 发现主节点节点被污点影响
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
                    node.kubernetes.io/not-ready:NoSchedule
# 移除污点
kubectl taint nodes vm-20-16-centos node.kubernetes.io/not-ready-

# 默认情况下，由于安全原因，集群不会将pod安排在Master服务器上。如果你希望能够将pod放到Master服务器上，例如，单机Kubernetes集群用于开发，请运行
# 查看污点
kubectl describe node vm-20-16-centos
移除污点
kubectl taint nodes vm-20-16-centos node-role.kubernetes.io/control-plane:NoSchedule-


# 如果初始化报错：[ERROR CRI]: container runtime is not running: output: time="2024-07-30T09:14:06+08:00" level=fatal msg="validate service connection: CRI v1 runtime API is not implemented for endpoint \"unix:///var/run/containerd/containerd.sock\": rpc error: code = Unimplemented desc = unknown service runtime.v1.RuntimeService"
# 编辑，找到disabled_plugins = ["cri"]，改为disabled_plugins = []
vi /etc/containerd/config.toml
# 重启containerd服务
systemctl restart containerd

# 如果初始化报错，注意这里 misconfiguration of the node
This error is likely caused by:
        - The kubelet is not running
        - The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)
# 修改/lib/systemd/system/kubelet.service，增加或修改参数ExecStart
# 修改后重启系统，重新初始化，注意先reset
ExecStart=/usr/bin/kubelet --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml
# 检查Docker 和 kubelet 的 cgroup driver 是否一致
# docker输出：Cgroup Driver: cgroupfs 或 Cgroup Driver: systemd
sudo docker info | grep Cgroup
# kubelet输出cgroupDriver: systemd， 发现不一致o>o
sudo cat /var/lib/kubelet/config.yaml | grep cgroup
# 这里修改 Docker 的 cgroup driver， 没有这个文件就创建一个
sudo touch /etc/docker/daemon.json
vim /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
  "registry-mirrors": ["https://9n08s726.mirror.aliyuncs.com"]
}
# 生成默认配置
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
# 修改SystemdCgroup=true
sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
# 查看kubeadm需要的镜像版本，输出registry.k8s.io/pause:3.9
kubeadm config images list
# 查看docker定义的版本，输出 sandbox_image = "registry.k8s.io/pause:3.6"，不一致，改为3.9
containerd config dump
sudo vim /etc/containerd/config.toml
# 修改内容
  [plugins."io.containerd.grpc.v1.cri"]
    sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.9"
sudo systemctl restart containerd
# 拉取新镜像
sudo ctr -n k8s.io images pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.9
# 重启服务，再reset init，囧
sudo systemctl restart kubelet




# 发现主节点未显示，查看pod，发现dns pod未成功
kubectl get pods -A
# 进一步查看，分析原因可能是在join主节点之前执行了 kubectl taint nodes --all node-role.kubernetes.io/master-
kubectl describe pod coredns-bc9d5cdc6-pr24r -n kube-system
# 清除一下 taint 标记
kubectl taint nodes vm-20-16-centos node.kubernetes.io/not-ready:NoSchedule-
# 重启容器，等待 coredns pod 重启完成
systemctl restart containerd

# 查看节点状态，发现存 Taints:node.kubernetes.io/not-ready:NoSchedule，状态显示 Ready false，原因为：container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized 这意味着CNI（Container Network Interface）插件没有初始化
kubectl describe node vm-20-16-centos





# 子节点加入时，创建token
kubeadm token create --print-join-command
# 加入集群机器，SSH到机器，成为root用户，几秒钟后，在master节点上运行kubectl get nodes命令，会显示所有已添加到集群中的节点主机
kubeadm join --token <token> 10.0.20.16:6443

# 授予子节点管理集群权限
scp root@<master ip>:/etc/kubernetes/admin.conf .
kubectl --kubeconfig ./admin.conf get nodes

# 安装dashborad
sudo scp /Users/ycl/Desktop/soft/docker/k8s/dashboard.yaml root@106.55.31.167:/root
sudo scp /Users/ycl/Desktop/soft/docker/k8s/dashboard-adminuser.yaml root@106.55.31.167:/root
sudo scp /Users/ycl/Desktop/soft/docker/k8s/dashboard-adminuser-secret.yaml root@106.55.31.167:/root
sudo scp /Users/ycl/Desktop/soft/docker/k8s/dashboard-adminuser-service-account.yaml root@106.55.31.167:/root
kubectl apply -f dashboard.yaml
kubectl apply -f dashboard-adminuser.yaml
kubectl apply -f dashboard-adminuser-secret.yaml
kubectl apply -f dashboard-adminuser-service-account.yaml
# 访问 https://106.55.31.167:30043

# 参考：https://www.cnblogs.com/qingfeng2010/p/10540832.html
```



1. 卸载冲突软件包
```
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```
2. 设置Docker的apt存储库
```
# 更新apt存储库
sudo apt-get update
# 安装必要的证书并允许 apt 包管理器使用以下命令通过 HTTPS 使用存储库（这个可能要多试几次，连接大概率超时）
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
# 添加 Docker 的官方 GPG 密钥（这里替换成了阿里的源，不然下载超时）
sudo curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
# 授权 apt 使用 GPG 密钥
sudo chmod a+r /etc/apt/keyrings/docker.asc

# 添加 Docker 存储库（这里替换成了阿里的源）
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] http://mirrors.aliyun.com/docker-ce/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 更新 apt 缓存
sudo apt-get update

# 安装 docker
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 验证
docker --version
sudo docker run hello-world

# 为了避免每次使用Docker命令时都需要切换到特权身份，可以将当前用户加入安装中自动创建的docker用户组
sudo usermod -aG docker docker
```

### 安装kubectl
```
# Ubuntu 支持快速安装
sudo snap install kubectl --classic

# 验证
kubectl version
```

### 安装kubeadm

1. 开放所需端口，参考：[http://docs.kubernetes.org.cn/457.html#i](http://docs.kubernetes.org.cn/457.html#i)
- Master节点
| 端口范围 | 用途 |
| ----------- | ------------------------ |
| 6443 | Kubernetes API server |
| 2379-2380 |	etcd server client API |
| 10250 |	Kubelet API |
| 10251 |	kube-scheduler |
| 10252 |	kube-controller-manager |
| 10255 |	Read-only Kubelet API (Heapster) |

- 工作节点
| 端口范围 | 用途 |
| ----------- | ------------------------ |
| 10250 |	Kubelet API |
| 10255 |	Read-only Kubelet API (Heapster) |
| 30000-32767 |	NodePort Services默认端口范围 |
| 8080 | kubectl |

### kubelet和kubeadm 安装

所有的集群机器都需要安装kubelet和kubeadm。
```
# 更新 apt，保证全部hit，没有的话则多试几次
apt-get update && apt-get install -y apt-transport-https

# 添加 apt key（这里换成了阿里云镜像，谷歌的用不了，你懂得）
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -

# 修改 apt 源
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
#deb http://apt.kubernetes.io/ kubernetes-xenial main
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF

# 更新 apt，保证全部hit，没有的话则多试几次
apt-get update
# 安装 kubelet kubeadm
apt-get install -y kubelet kubeadm

# 注意：必须使用运行setenforce 0命令来禁用SELinux，因为需要允许容器访问主机文件系统，这是配置pod网络所要求的。（直到kubelet中对SELinux支持得到改进）
apt install selinux-utils
setenforce 0
```

## 使用kubeadm创建kubernetes集群

初始化Master节点
```
kubeadm init

kubeadm reset
```