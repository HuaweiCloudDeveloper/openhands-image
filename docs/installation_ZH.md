# 1、安装

## 1.1 docker安装和配置

```markdown
# 安装docker
apt  install docker.io  # ubuntu 系统
yum install docker  # 欧拉系统安装

# 配置镜像加速器
vi /etc/docker/daemon.json

{
    "registry-mirrors": [ "https://51262a7168c84cfd97b7c142adbd05ca.mirror.swr.myhuaweicloud.com" ]
}

# 重启容器引擎
systemctl restart docker

执行docker info，当Registry Mirrors字段的地址为加速器的地址时，说明加速器已经配置成功。

#卸载docker 

sudo dnf remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-engine
    
 # 添加 Docker CE 仓库并修改为国内镜像
sudo dnf config-manager --add-repo=https://repo.huaweicloud.com/docker-ce/linux/centos/docker-ce.repo
sudo sed -i 's/download.docker.com/repo.huaweicloud.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo
sudo sed -i 's/\$releasever/8/g' /etc/yum.repos.d/docker-ce.repo  # openEuler 22.03 对应 CentOS 8

# 安装必要依赖
sudo dnf install -y dnf-plugins-core
 
# 安装 Docker 核心组件
sudo dnf install -y docker-ce docker-ce-cli containerd.io
 
# 安装 Docker Compose 插件（推荐方式）
sudo dnf install -y docker-compose-plugin

#  配置镜像加速器​
sudo tee /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": [
    "https://<your-mirror-id>.mirror.aliyuncs.com",  # 阿里云加速器（需替换为个人ID）
    "https://docker.mirrors.ustc.edu.cn"             # 中科大镜像源
  ]
}
EOF

# 启动 Docker 并设置开机自启
sudo systemctl enable docker --now
 
# 将当前用户加入 docker 组（避免频繁使用 sudo）
sudo usermod -aG docker $USER
newgrp docker  # 立即生效或重新登录

# 检查 Docker 版本及服务状态
docker --version        # 应输出类似 Docker version 26.1.3
docker compose version  # 应输出 Docker Compose version v2.27.0
systemctl status docker # 确认状态为 "active (running)"
 
# 测试镜像拉取与运行
docker run --rm hello-world
```



1.2 模型配置

```markdown
# 基于云/API 的模型
	anthropic/claude-sonnet-4-20250514（推荐）
	openai/o4-mini
	gemini/gemini-2.5-pro
	deepseek/deepseek 聊天
	

# 可以通过设置在 OpenHands UI 中进行以下设置：

LLM Provider
LLM Model
API Key
Base URL（通过Advanced设置）
```



## 1.2 启动 OpenHands

```markdown
# 镜像拉取
docker pull docker.all-hands.dev/all-hands-ai/runtime:0.48-nikolaik

# 连接本地文件系统
export SANDBOX_VOLUMES=/opt/openhands_code:/workspace:rw

docker run # ...
    -e SANDBOX_USER_ID=$(id -u) \
    -e SANDBOX_VOLUMES=$SANDBOX_VOLUMES \
    # ...



# 启动镜像
docker run -d \
  --restart=always \
  --name openhands-app \
  -e SANDBOX_RUNTIME_CONTAINER_IMAGE=docker.all-hands.dev/all-hands-ai/runtime:0.48-nikolaik \
  -e LOG_ALL_EVENTS=true \
  -e SANDBOX_VOLUMES=$SANDBOX_VOLUMES \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /opt/.openhands:/.openhands \
  -p 3000:3000 \
  --add-host host.docker.internal:host-gateway \
  docker.all-hands.dev/all-hands-ai/openhands:0.48
```

