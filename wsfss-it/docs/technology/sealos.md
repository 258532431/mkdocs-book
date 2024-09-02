---
comments: true
---

手动配置 kubernetes 集群十分耗时，有没有更简单的方式呢？

当然有，本文将介绍如何使用 `sealos` 在10分钟内快速部署 kubernetes 集群。

## 准备工作

如果你需要配置高可用的集群，至少需要准备三台linux系统主机(Ubuntu 22.04 LTS)，资源有限的情况下可以考虑使用单台机器。

| 节点名称          | 节点IP:material-information-outline:{ title="这里是内网IP" }                          |
| ---------------- | ------------------------------------ |
| `k8s-master01`   | 192.168.1.189  |
| `k8s-worker01`   | 192.168.4.61 |
| `k8s-worker02`   | 192.168.13.199 |

机器准备好之后需要确定已卸载 `docker`、`kubeadm` 等工具，最好重装系统。

三台机器的root密码设置为相同密码，并且允许ssh方式root用户登录。

首先修改一下主机名称
```bash
# master节点修改主机名称
hostnamectl set-hostname k8s-master01
bash

# worker节点1修改主机名称
hostnamectl set-hostname k8s-worker01
bash

# worker节点2修改主机名称
hostnamectl set-hostname k8s-worker02
bash

# 修改所有节点hosts文件内容
cat >> /etc/hosts << EOF
192.168.1.189 k8s-master01
192.168.4.61 k8s-worker01
192.168.13.199 k8s-worker02
EOF

# 验证各节点是否互通
ping k8s-master01
ping k8s-worker01
ping k8s-worker02
```

接下来配置系统参数，所有节点都要操作
```bash
# 关闭swap
swapoff -a && sed -ri 's/.*swap.*/#&/' /etc/fstab
# 关闭 SELinux
apt update && apt install selinux-utils
setenforce 0 && sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinuxconfig
# 关闭防火墙，清空防火墙规则
sudo ufw disable && ufw status
# 设置时区
timedatectl set-timezone Asia/Shanghai
# 同步系统时间
apt install -y ntpdate
ntpdate time1.aliyun.com
# 重启一下系统
reboot

# master节点生成秘钥，用于通过ssh连接worker节点
ssh-keygen
# 拷贝master秘钥公钥到worker节点
ssh-copy-id -i .ssh/id_rsa.pub root@192.168.4.61
ssh-copy-id -i .ssh/id_rsa.pub root@192.168.13.199
```

使用云服务商的机器，还需要开放相关端口

| 端口              | 说明                           |
| ---------------- | ------------------------------ |
| `6443`          | Kubernetes API server           |
| `8443`          | Kubernetes Dashboard            |
| `2379-2380`     | etcd server client API          |
| `10250`         | Kubelet API                     |
| `10251`         | kube-scheduler                  |
| `10252`         | kube-controller-manager         |
| `10255`         | Read-only Kubelet API (Heapster)|
| `30000-32767`   | NodePort Services默认端口范围     |

## 安装sealos

Sealos 是一款基于 Kubernetes 的云操作系统发行版，它旨在简化 Kubernetes 集群的部署和管理过程。Sealos 的特点在于它采用云原生的方法来构建和管理集群，使得用户能够像使用个人电脑一样简便地管理 Kubernetes 集群。[官方文档](https://sealos.run/docs/self-hosting/lifecycle-management/quick-start/deploy-kubernetes/)

官方提供了多种安装方式，这里使用二进制安装，我们只需要在master节点进行安装。
```bash
# 查看系统内核，确定安装哪个版本的 sealos，查看版本列表（https://github.com/labring/sealos/releases）
uname -a

# 下载最新版sealos，在版本下面的安装包右键复制地址
wget https://githubfast.com/labring/sealos/releases/download/v5.0.0/sealos_5.0.0_linux_amd64.tar.gz  && \
    tar -zxvf sealos_5.0.0_linux_amd64.tar.gz sealos && chmod +x sealos && mv sealos /usr/bin

# 验证安装是否成功
sealos help
```

## Sealos镜像仓库

云服务器网络原因无法使用Docker Hub镜像，我们可以使用Sealos私有仓库。

首先将本地的docker镜像导出为tar包：website-api.tar

```bash
# 在master节点登录 sealos 私有仓库
sealos login -u admin -p passw0rd sealos.hub:5000
# 上传本地镜像到服务器，用 sealos 加载
sealos load -i website-api.tar
sealos push sealos.hub:5000/website_api:latest
```

## 创建集群

Sealos 提供了阿里云镜像仓库服务，我们替换为阿里云镜像仓库，以加速镜像拉取。

[部署集群命令参考](https://github.com/labring/sealos/tree/release-v4.1)。对应的镜像版本可以在dockerhub或aliyuncs[https://explore.ggcr.dev](https://explore.ggcr.dev)上搜索查看，比如搜索：labring/kubernetes，后面的小版本越高越稳定

```bash
# 生成配置文件 Clusterfile，注意：labring/helm 应当在 labring/calico 之前
sealos gen registry.cn-shanghai.aliyuncs.com/labring/kubernetes:v1.25.0 registry.cn-shanghai.aliyuncs.com/labring/helm:v3.14.1-amd64 registry.cn-shanghai.aliyuncs.com/labring/calico:v3.24.1 \
     --masters 192.168.1.189 \
     --nodes 192.168.4.61,192.168.13.199 \
     --passwd 你的主机root密码 -o Clusterfile

# 启动集群，中间可能会卡住一段时间比较慢，看网络，快的话10分钟左右
sealos apply -f Clusterfile

# 后续增加主节点
sealos add --masters ip1,ip2,ip3
# 后续增加工作节点
sealos add --nodes ip1,ip2,ip3
```

集群运行成功后会把 Clusterfile 保存到 .sealos/default/Clusterfile 文件中，可以修改其中字段来重新 apply 对集群进行变更

安装过程中如果终端断连导致日志不输出，可以执行
```bash
tail -f ~/.sealos/logs/sealos.log
```

如果安装失败，需要清理安装文件进行重装，需要执行reset命令：
```bash
sealos reset
# 清理文件
rm /root/.sealos -rf
# 清理安装过程的缓存文件
sealos unmount --all && sealos rm --all
```

## 集群验证

部署完成后，可以查看集群状态
```bash
# 查看集群状态
kubectl get nodes
kubectl get pods -o wide -n kube-system
```

集群部署完成后，我们还需要进行一些配置，方便我们快速使用集群。
```bash
# 使用阿里云镜像源
cat >> /etc/containers/registries.conf << EOF
[[registry.mirror]]
prefix = "docker.io/labring"
location = "registry.cn-shanghai.aliyuncs.com/labring"
EOF

# 需要在master节点外的机器上控制集群
kubectl --kubeconfig /etc/kubernetes/admin.conf get nodes

# 允许master节点上创建pod
kubectl taint nodes k8s-master01 node-role.kubernetes.io/control-plane:NoSchedule-
```

查看集群证书
```bash
# 查看集群证书剩余时间
kubeadm certs check-expiration

# 证书作用
admin.conf 用于管理员与Kubernetes API服务器进行交互的身份验证。这个配置文件包含了访问API服务器所需的证书和密钥。
apiserver 安全通信：API服务器使用此证书与其他集群组件（如kubelet、控制器管理器等）进行TLS加密通信。确保只有授权的客户端可以访问API服务器。
apiserver-etcd-client API服务器用于与etcd集群通信的客户端证书，确保API服务器与etcd之间的通信是加密和安全的。
apiserver-kubelet-client API服务器用于与kubelet通信的客户端证书，确保API服务器可以安全地与集群中的节点进行通信。
controller-manager.conf 控制器管理器用于与API服务器交互的身份验证证书，确保控制器管理器可以访问和操作集群资源。
etcd-healthcheck-client 用于etcd健康检查的客户端证书，确保可以安全地检查etcd集群的健康状态。
etcd-peer etcd集群中节点之间通信的证书，确保etcd集群内部的通信是加密和安全的。
etcd-server etcd服务器用于与客户端通信的证书，确保etcd服务器可以安全地与集群中的其他组件进行通信。
front-proxy-client 前端代理客户端证书，用于与API服务器进行身份验证，通常与某些代理或负载均衡器相关。
scheduler.conf

# 根证书
ca（Certificate Authority） 根证书颁发机构，用于签发和验证集群中其他所有证书的有效性。这是整个证书信任链的起点。
etcd-ca etcd集群的根证书颁发机构，专门用于签发和验证etcd相关的证书。
front-proxy-ca 前端代理的根证书颁发机构，用于签发和验证与前端代理相关的证书。
```

## 安装dashboard

下载依赖文件：[recommended.yaml](https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml)，重命名文件为 `dashboard.yaml` 。

修改一下 `dashboard.yaml` 配置，将类型改为 NodePort ，并设置 ```spec.ports.nodePort: 30043```
```yaml title="dashboard.yaml"
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
```bash
kubectl apply -f dashboard.yaml

# 查看状态
kubectl get pods -o wide -n kubernetes-dashboard
```

配置一下访问权限，首先在命名空间kubernetes-dashboard创建名为admin-user的服务帐户。
```yaml title="dashboard-adminuser.yaml"
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```

现在我们需要找到可用于登录的令牌。执行以下命令：
```bash
# 创建admin-user
kubectl apply -f dashboard-adminuser.yaml

# 获取token
kubectl -n kubernetes-dashboard create token admin-user
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
```bash
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

我们已经配置好了，访问dashboard：[https://121.37.19.93:30043](https://121.37.19.93:30043)。

但是使用token方式登录比较麻烦，token也会过期，我们配置一下kubeconfig文件验证登录。

需要的证书内容在配置文件中，位于 `~/.kube/config`，查看内容：cat ~/.kube/config
```bash
# 查看证书文件内容
cat ~/.kube/config | grep certificate-authority-data | awk -F ":" '{print $2}' | tr -d " " | base64 -d > ca.crt
```

获取Kubernetes API Server 的信息。执行以下命令：
```bash
cat ~/.kube/config
```

输出：https://apiserver.cluster.local:6443，根据config内容cluster: kubernetes知道，集群名称为 kubernetes，用户名是 kubernetes-admin。

使用 `kubectl` 工具，生成 kubeconfig 登录认证文件。
```bash
# 创建集群
kubectl config set-cluster kubernetes \
    --certificate-authority=./ca.crt \
    --embed-certs=true \
    --server=https://apiserver.cluster.local:6443 \
    --kubeconfig=kubernetes-admin.conf

# 获取token
SECRET=$(kubectl get secret -n kubernetes-dashboard | grep dashboard-admin | awk '{print $1}')
kubectl describe secret -n kubernetes-dashboard $SECRET

# 创建credentials
kubectl config set-credentials kubernetes-admin \
    --token=上一步获取的token \
    --kubeconfig=kubernetes-admin.conf

# 创建context
kubectl config set-context kubernetes-admin@kubernetes \
    --cluster=kubernetes \
    --user=kubernetes-admin \
    --kubeconfig=kubernetes-admin.conf

# 切换context的current-context
kubectl config use-context kubernetes-admin@kubernetes --kubeconfig=kubernetes-admin.conf
```

登录的时候选择刚刚生成的 kubernetes-admin.conf ，可以保存到本地电脑备用

## MetalLB 负载均衡

由于我们购买的是特价云服务器，没有负载均衡功能，这里使用免费的 MetalLB 用作负载均衡。[参考文档](https://metallb.universe.tf/installation/)

需要修改一下 kube 配置文件：
```bash
kubectl edit configmap -n kube-system kube-proxy

# 修改如下内容，
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true

# 修改完更新一下pod
kubectl get pod -n kube-system |grep kube-proxy | awk '{system("kubectl delete pod "$1" -n kube-system")}'
```

准备 metallb 配置文件[下载地址](https://raw.githubusercontent.com/metallb/metallb/v0.14.0/config/manifests/metallb-native.yaml)。
```yaml title="metallb-native.yaml"
# 下载后保存到本地
wget https://raw.githubusercontent.com/metallb/metallb/v0.14.0/config/manifests/metallb-native.yaml
```

地址池，这里配置的公网IP地址

```yaml title="metallb-ippool.yaml"
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: metallb-ippool
  namespace: metallb-system
spec:
  addresses:
  - 公网IP1/32
  - 公网IP2/32
  - 公网IP3/32
  autoAssign: true
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: metallb-l2
  namespace: metallb-system
spec:
  ipAddressPools:
  - metallb-ippool
```

部署命令：
```bash
kubectl apply -f metallb-native.yaml
kubectl apply -f metallb-ippool.yaml

# 检查状态
kubectl get all -n metallb-system
kubectl get ipaddresspool -n metallb-system
# 排查日志
kubectl logs deployment/controller -n metallb-system
# 查看哪些ip地址被分配出去了
kubectl get po --all-namespaces -o wide
```

部署过程报错秘钥不存在，可以手动创建一下：
```bash
# 查看创建的密钥
kubectl -n metallb-system get secrets
## 不存在就创建密钥
kubectl create secret generic -n metallb-system  memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```

## Ingress 控制器

我们想再集群外部访问集群内的服务，需要一个暴露服务的组件，Ingress 控制器就是用来实现这个功能的。

可以通过 Sealos 快速部署一个 Nginx Ingress Controller：
```bash
sealos run registry.cn-shanghai.aliyuncs.com/labring/ingress-nginx:v1.9.4-amd64

# 查看 Ingress Controller 服务的状态
kubectl get svc -n ingress-nginx

# 检查类型是否为LoadBalancer（TYPE=LoadBalancer）
kubectl --namespace ingress-nginx get services -o wide -w ingress-nginx-controller
# 查看ingress的外部ip地址（当我们部署metallb之后，会自动分配），用于域名解析
kubectl get svc ingress-nginx-controller -n ingress-nginx
```

## 域名访问

我们虽然使用了 ingress-nginx 来实现域名解析，但是 ingress-nginx 并没有开放80端口和443端口给外部访问，我们通过nginx来进行一层转发

物理机安装nginx，用来支持域名访问不加端口（这里ingress的外部ip地址对应的机器都要安装）

```bash
# 安装nginx
apt install nginx
# 修改nginx配置
vim /etc/nginx/nginx.conf

# 反向代理到ingress-nginx的控制器外部IP地址的80端口和443端口
server {
    listen 80;
    server_name *.top;

    location / {
        proxy_pass http://121.37.19.93;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 443;
    server_name *.top;

    location / {
        proxy_pass http://121.37.19.93:443;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# 启动
sudo systemctl start nginx
# 停止
sudo systemctl stop nginx
# 设置开机启动
sudo systemctl enable nginx
# 查看状态
sudo systemctl status nginx
# 重载
sudo nginx -s reload
```

## 扩展

1. 修改镜像站，加速镜像拉取（我试了貌似不起作用）[参考文章](https://zhuanlan.zhihu.com/p/702497587?utm_id=0)

2. traefik 反向代理，可以替换 ingress-nginx [参考文章](https://www.lvbibir.cn/posts/tech/kubernetes-traefik-1-deploy/)









