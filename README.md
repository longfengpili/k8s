# Kubernetes 集群部署指南

## 1. 安装 Docker

按照官方文档，使用以下步骤安装 Docker：

### 更新 apt 包索引
```bash
sudo apt-get update
```

### 安装必要的依赖
```bash
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
```

### 添加 Docker 的官方 GPG 密钥
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

### 设置 Docker 仓库
```bash
sudo add-apt-repository \  
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

### 再次更新 apt 包索引
```bash
sudo apt-get update
```

### 安装 Docker
```bash
sudo apt-get install docker-ce
```

### 启动并检查 Docker 服务
```bash
sudo systemctl start docker
sudo systemctl enable docker
sudo docker --version
```

### 安装 Docker Compose
```bash
sudo apt install docker-compose
```

### 将当前用户添加到 Docker 用户组
```bash
sudo usermod -aG docker $USER
```

## 2. 修改 Docker 配置

### 创建配置目录
```bash
sudo mkdir -p /etc/docker
```

### 配置镜像加速器
```bash
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://docker.longfengpili.cn"]
}
EOF
```

### 重启 Docker 服务
```bash
sudo systemctl restart docker
```

## 3. 设置主机名

### 设置主机名示例
```bash
sudo hostnamectl set-hostname master
sudo hostnamectl set-hostname node1
sudo hostnamectl set-hostname node2
```

## 4. 配置 /etc/hosts

### 在每个节点的 `/etc/hosts` 文件中添加如下内容
```plaintext
192.168.2.151 master 
192.168.2.152 node1 
192.168.2.156 node2
```

## 5. 禁用 Swap

### 禁用 Swap
```bash
sudo swapoff -a
```

### 注释掉 Swap 的相关行
```bash
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

### 验证
```bash
free -h
```

重启系统以确保修改生效。

## 6. 修改内核参数

### 载入内核模块
```bash
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

### 配置网络参数
```bash
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

## 7. 安装 containerd

### 安装依赖
```bash
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
```

### 添加 Docker 仓库
```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository \  
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

### 安装 containerd
```bash
sudo apt update
sudo apt install -y containerd.io
```

### 配置 containerd 使用 systemd 作为 cgroup
```bash
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
```

### 配置容器运行时默认沙盒镜像

#### 修改 containerd 配置
```bash
sudo vim /etc/containerd/config.toml
```
确保以下内容正确：
```toml
[plugins."io.containerd.grpc.v1.cri".containerd]
  sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.10"

[plugins."io.containerd.grpc.v1.cri".registry.mirrors]
  config_path = "/etc/containerd/certs.d"
```

#### 创建景象加速器配置文件
```bash
sudo vim /etc/containerd/certs.d/docker.io/hosts.toml
```
确保添加一下内容：
```toml
[host."https://docker.longfengpili.cn"]
  capabilities = ["pull", "resolve", "push"]

server = "https://registry-1.docker.io"
```

#### 重启 containerd 服务
```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

## 8. 安装 Kubernetes 组件

### 更新 apt 包索引并安装依赖
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

### 添加 Kubernetes 仓库
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

### 安装 Kubernetes 组件
```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## 9. 创建 Kubernetes 集群

### 拉取沙盒镜像
```bash
sudo ctr image pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.10
```

### 初始化集群
```bash
sudo kubeadm init --control-plane-endpoint=192.168.2.151 --kubernetes-version=v1.32.0 --image-repository=registry.aliyuncs.com/google_containers

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## 10. 部署网络插件

### 删除旧的插件
```bash
kubectl delete -f https://docs.projectcalico.org/manifests/calico.yaml
```

### 部署 Calico 网络插件
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

## 11. 生成和管理 Token

### 查看现有 Token
```bash
sudo kubeadm token list
```

### 创建新的 Token
```bash
sudo kubeadm token create
```

### 查看证书哈希
```bash
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | awk '{print $2}'
```

## 12. 设置代理

### 配置代理
```bash
export http_proxy=http://xxxxx:3018
export https_proxy=http://xxxxx:3018
export no_proxy=localhost,127.0.0.1,*.local
```

### 拉取镜像
```bash
ctr image pull docker.io/calico/cni:v3.24.0
```

## 13. 加入节点

### 加入控制平面节点
```bash
sudo kubeadm join 192.168.2.151:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash> --control-plane
```

### 加入工作节点
```bash
sudo kubeadm join 192.168.2.151:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

### 验证节点状态
```bash
kubectl get nodes
```


## 14. 创建nginx服务

1. 执行以下命令创建一个 Nginx 部署：
```bash
kubectl create deployment nginx --image=nginx
```

2. 删除原有的 Nginx Service（如果存在）：
   ```bash
   kubectl delete svc nginx
   ```

3. 编写 YAML 文件来定义具有指定 `NodePort` 的 Service。创建一个名为 `nginx-service.yaml` 的文件，内容如下：

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: nginx
   spec:
     type: NodePort
     selector:
       app: nginx
     ports:
       - protocol: TCP
         port: 80
         targetPort: 80
         nodePort: 30080
   ```

4. 应用 YAML 文件来创建 Service：
   ```bash
   kubectl apply -f nginx-service.yaml
   ```

5. 验证创建的 Service

运行以下命令，确认服务已创建且绑定到指定的 NodePort：

```bash
kubectl get svc
```

示例输出：
```plaintext
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
nginx        NodePort    10.109.206.194   <none>        80:30080/TCP   6s
```

6. 访问服务

使用浏览器或 `curl` 访问服务：
```
http://<节点IP>:30080
```


## 15. 删除 Nginx 部署

+ 执行以下命令删除 Nginx 的部署：

```bash
kubectl delete deployment nginx
```

---

+ 删除 Nginx 服务

执行以下命令删除 Nginx 服务：

```bash
kubectl delete service nginx
```

---

+ 验证资源已删除

运行以下命令确认 Nginx 部署和服务已成功删除：

```bash
kubectl get deployment
kubectl get svc
```

如果输出中没有 `nginx` 相关的资源，表示删除成功。