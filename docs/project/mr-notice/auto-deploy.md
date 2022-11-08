# 项目部署启动
## 1. 服务器配置(centos)
### 1. docker
```shell
# 添加安装包
sudo yum install -y yum-utils
# 设置镜像仓库
sudo yum-config-manager 
--add-repo 
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 安装前先更新yum软件包索引
yum makecache fast
# 安装docker-ce（社区版-免费的）
sudo yum install docker-ce docker-ce-cli containerd.io
# 启动docker
sudo systemctl start docker
# 验证
docker version
```



### 2. git
```shell
# 更新yum的git源
wget http://opensource.wandisco.com/centos/7/git/x86_64/wandisco-git-release-7-1.noarch.rpm && rpm -ivh wandisco-git-release-7-1.noarch.rpm
# 安装
yum install git -y
# 验证
git --version

```


### 3. node
安装包(如果安装包版本过高可能有问题)
```shell
wget https://nodejs.org/dist/v16.14.0/node-v16.14.0-linux-x64.tar.xz
# 解压
tar xvf node-v16.14.0-linux-x64.tar.xz
# 改名便于后续操作
mv node-v16.14.0-linux-x64.tar.xz nodejs
# 软链接(以便可以在任意目录下使用 node 和 npm 命令, 类似在windows上配置全局环境变量)
ln -s /root/nodejs/bin/node /usr/local/bin/node
ln -s /root/nodejs/bin/npm /usr/local/bin/npm
ln -s /root/nodejs/bin/npx /usr/bin/npx
# 检查是否安装完成
node -v
npm -v

```


### 4. pm2
```shell
# 安装
npm i -g pm2
# 软链
sudo ln -s /download/nodejs/bin/npx /usr/bin/npx
# 验证
pm2 -v
```


## 2. 启动各服务
### 1. mysql
```shell
docker pull mysql:5.7
docker run -d --name=mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7
docker exec -it mysql bash
# (如果登录以及CREATE步骤报错Operation CREATE USER failed for 'root'@'%', 那么使用这个命令)
# drop user 'root';
bash-4.2# mysql -u root -p 123456
# 授权
mysql>CREATE USER 'root'@'%' IDENTIFIED BY 'root';
mysql>GRANT ALL ON *.* TO 'root'@'%';
# 刷新权限
mysql> flush privileges;
# 修改root用户密码
mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
# 刷新权限
mysql> flush privileges;
```



### 2. 启动后端服务
```shell
# 使用pm2启动
pm2 start main.js
# 查看是否启动成功
pm2 status
```

自动部署(配合github action)
```yaml
name: Node.js Package

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  docker-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: connect serve and deploy
        uses: appleboy/ssh-action@v0.1.5
        with:
          username: root
          host: ${{ secrets.REMOTE_HOST }}
          password: ${{ secrets.REMOTE_PASSWORD }}
          script: ${{ secrets.REMOTE_DEPLOY_SHELL }}
```
主要原理是连接宿主机, 调用宿主机脚本更新pm2进程
上述的变量均保存在github上


宿主机(服务器)上的脚本`refresh-backend.sh`
```shell
#!/bin/bash
cd /root/zk/project/mr-notice-backend && git fetch --all && git reset --hard origin/master && git pull
if [[ -n $(pm2 show mr-notice-backend) ]]; then
  echo "restart app mr-notice-backend..."
  pm2 restart mr-notice-backend
else
  echo "start app 'mr-notice-backend'..."
  pm2 start --name mr-notice-backend
fi;
```




### 3. 启动nginx(挂载文件)
具体启动项目自定义nginx的配置在下一条
```shell
# 启动标准nginx容器
docker pull nginx
docker run -itd --name=nginx -p 80:80 nginx
# 复制挂载文件模板到宿主机中
mkdir nginx
docker cp [容器id]:/etc/nginx/nginx.conf ./
docker cp [容器id]:/etc/nginx/conf.d/default.conf ./conf/
# 删除标准容器
docker rm -f [容器id]
# 配置nginx(自己配置即可)
vim ./conf/default.conf
```



### 4. 前端自动部署(github action)
```yaml
# This is a basic workflow to help you get started with Actions
name: Auto deploy
# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the main branch
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  docker-build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      - name: Login to DockerHub
        uses: docker/login-action@v2.1.0
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push Docker images
        uses: docker/build-push-action@v3.2.0
        with:
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/mr-notice-frontend:latest

  docker-deploy:
    needs: docker-build
    runs-on: ubuntu-latest
    steps:
      - name: connect serve and deploy
        uses: appleboy/ssh-action@v0.1.5
        with:
          username: root
          host: ${{ secrets.REMOTE_HOST }}
          password: ${{ secrets.REMOTE_PASSWORD }}
          script: ${{ secrets.REMOTE_DEPLOY_SHELL }}
```
首先登录并打包最新项目为docker镜像, 推送到docker仓库,
然后连接宿主机, 调用宿主机脚本更新前端容器
上述的变量均保存在github上


宿主机(服务器)上的脚本`refresh-front.sh`
```shell
refresh-front.sh
#!/bin/bash
if [[ -n $(docker ps -q -f "name=^mr-notice-frontend$") ]];then
 docker rm -f mr-notice-frontend
 docker pull 413365742/mr-notice-frontend:latest
fi
docker run -itd --name mr-notice-frontend -p 80:80 -v /root/zk/deploy-dist/nginx-config/conf:/etc/nginx/conf.d 413365742/mr-notice-frontend:latest
docker image prune -a -f
echo 'success'
```

docker的多阶段打包文件在项目里面



