---
title: docker部署rails app示例
date: 2019-05-01 20:49:08
tags: docker
---

### 创建用户，并授予`sudo`权限
```
useradd -d /home/robot/ -G wheel robot
passwd robot #
```

### 安装docker
```
su - robot
sudo curl -sS https://get.docker.com/ | sh # 安装的是docker-ce
# 安装完后会提示将当前用户加入到`docker`用户组
sudo usermod -aG docker robot

#将用户加入`docker`用户组将取得运行容器的权限，运行容器需要root权限，因此这意味着该用户可以获得root权限。

#允许docker开机启动
sudo systemctl enable docker
```

### 安装docker-compose
docker-compose 让你可以用`.yml`/`.yaml`文件来部署docker容器和应用，当前最新的版本是`1.2.0`，你可以通过[这里](https://github.com/docker/compose/releases)来查看最新的版本
```
sudo curl -L https://github.com/docker/compose/releases/download/1.24.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### 部署Rails app

#### 创建rails 

```
# 创建rails 应用。可以直接在自己的机器上用rails new创建
rails new docker_demo --api --skip-bundle

# 如果没有安装ruby环境，也可以用docker的ruby镜像跑一个docker容器，在容器里面创建rails app
docker pull ruby:2.5
docker run --rm -it -v "$PWD:/app" ruby:2.5 bash   # --rm 退出后删除容器

# 在docker容器中运行
gem install rails # 安装rails
cd /app && rails new docker_demo --api --skip-bundle # 新建rails app

```
#### dockerfile & docker-compose.yml

```
# dockerfile

FROM ruby:2.5

WORKDIR /var/www/app
COPY Gemfile* ./
RUN gem install bundler && bundle install
COPY . .

EXPOSE 3000

CMD rails s -b 0.0.0.0

# docker-compose.yml
version: '1'

services:
  docker_demo:
      build: .
      ports:
        - "3000:3000"
```

#### 启动服务
```
docker-compose up -d docker-demo
```
### Troubleshooting

#### 启动docker container 导致主机ssh中断,并且只能通过vultr提供的网页console ssh上去.
`serverSpeeder`服务造成的, 卸载'serverSpeeder', 安装`bbr`
