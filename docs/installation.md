# 1、Install

## 1.1 docker install and configure

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

# Install necessary dependencies
sudo dnf install -y dnf-plugins-core
 
# Install Docker core components
sudo dnf install -y docker-ce docker-ce-cli containerd.io
 
# Install Docker Compose plugin (recommended method)
sudo dnf install -y docker-compose-plugin

#  Configure image accelerator
sudo tee /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": [
    "https://<your-mirror-id>.mirror.aliyuncs.com",   
    "https://docker.mirrors.ustc.edu.cn"              
  ]
}
EOF


# Start Docker and set it to boot up automatically
sudo systemctl enable docker --now
 
# Add the current user to the Docker group (avoid frequent use of sudo)
sudo usermod -aG docker $USER
Newgrp Docker # Effective immediately or login again

# Check Docker version and service status
Docker -- version # should output something similar to Docker version 26.1.3
Docker Compose version # Should output Docker Compose version v2.27.0
Systemctl status Docker # Confirm status as' active (running) '
 
# test
docker run --rm hello-world
```



1.2 模型配置

```markdown
# Cloud/API based model
	anthropic/claude-sonnet-4-20250514
	openai/o4-mini
	gemini/gemini-2.5-pro
	deepseek/deepseek  
	

# The following settings can be made in the OpenHands UI:

LLM Provider
LLM Model
API Key
Base URL（通过Advanced设置）
```



## 2.1 Start OpenHands

```markdown
# Mirror Pull
docker pull docker.all-hands.dev/all-hands-ai/runtime:0.48-nikolaik

# Connect to local file system
export SANDBOX_VOLUMES=/opt/openhands_code:/workspace:rw

docker run # ...
    -e SANDBOX_USER_ID=$(id -u) \
    -e SANDBOX_VOLUMES=$SANDBOX_VOLUMES \
    # ...



# Start image
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

