###CentOS 安装 docker

##centos8.2以下的如果部署不成功，先重置服务器，再升级一下内核就可以了

#升级内核命令
```
sudo yum update
```
##安装依赖
```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

wget -O /etc/yum.repos.d/docker-ce.repo https://download.docker.com/linux/centos/docker-ce.repo

sudo sed -i 's+download.docker.com+mirrors.tuna.tsinghua.edu.cn/docker-ce+' /etc/yum.repos.d/docker-ce.repo
```
##安装
```
sudo yum makecache fast

sudo yum install docker-ce
```
##启动并加入开机启动
```
sudo systemctl start docker

sudo systemctl enable docker
```
##换源

#腾讯云用腾讯云的
```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://mirror.ccs.tencentyun.com"]
}
EOF
```
#阿里云服务器 用网易的加速器
```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
EOF
```
#重启docker
```
sudo service docker restart
```
#卸载Docker
```
sudo yum remove docker docker-common docker-selinux docker-engine
```
##Docker-Portainer汉化面板
```
sudo docker pull portainer/portainer
```
#下载文件，解压到桌面，然后上传文件到根目录

#下载链接：https://afw.lanzous.com/iM5CZl722cj
```
docker run -d \
   -p 9000:9000 \
   --restart=always  \
   -v /var/run/docker.sock:/var/run/docker.sock \
   -v portainer_data:/data \
   -v /public:/public \
   --name prtainer-test  \
   portainer/portainer
```
##如果已部署面板，需要将之前的容器删除。密码设置成功，选择本地就可以了，也就是第一个

##Docker 部署京东脚本

#e大v4部署
```
docker run -dit \
   -v /jd/config:/jd/config \
   -v /jd/log:/jd/log \
   -v /jd/scripts:/jd/scripts \
   -v /jd/own:/jd/own \
   -p 5678:5678 \
   -e ENABLE_HANGUP=true \
   -e ENABLE_WEB_PANEL=true \
   -e ENABLE_WEB_TTYD=true \
   --name jd \
   --hostname jd \
   --restart always \
   nevinee/jd:v4
```
命令
```
docker exec jd jtask   # 运行scripts脚本
docker exec jd otask   # 运行own脚本
docker exec jd mtask   # 运行你自己的脚本，如果某些own脚本识别不出来cron，你也可以自行添加mtask任务
docker exec jd jlog    # 删除旧日志
docker exec jd jup     # 更新所有脚本，up=update，如果在jup后增加一个参数"shell", 或"scripts", 或"own"则可以控制只更新该类脚本
docker exec jd jcode   # 导出所有互助码
docker exec jd jcsv    # 记录豆豆变化情况
docker exec jd hangup  # 运行挂机程序
```

#青龙部署
```
docker run -dit \
   -v /qinglong/config:/ql/config \
   -v /qinglong/log:/ql/log \
   -v /qinglong/scripts:/ql/scripts \
   -v /qinglong/db:/ql/db \
   -p 5700:5700 \
   -e ENABLE_HANGUP=true \
   -e ENABLE_WEB_PANEL=true \
   --name qinglong \
   --hostname qinglong \
   --restart always \
   whyour/qinglong:latest
```

##多容器配置

#要想换库直接改最后一行
```
docker run -dit \
    -v /你想保存的目录/jd1/config:/jd/config `# 配置保存目录，冒号左边请修改为你想存放的路径`\
    -v /你想保存的目录/jd1/log:/jd/log `# 日志保存目录，冒号左边请修改为你想存放的路径` \
    -v /你想保存的目录/jd1/scripts:/jd/scripts  `# 脚本文件目录，映射脚本文件到安装路径` \
    -v /jd/own:/jd/own \
    -p 5679:5678 \
    -e ENABLE_HANGUP=true \
    -e ENABLE_WEB_PANEL=true \
    -e ENABLE_WEB_TTYD=true \
    --name jd1 \
    --hostname jd1 \
    --restart always \
    nevinee/jd:v4
```

##自动更新Docker容器（也就是更新京东文件）
```
docker run -d \
        --name watchtower \
        -v /var/run/docker.sock:/var/run/docker.sock \
        containrrr/watchtower
```
##手动更新

#青龙更新命令
```
docker exec -it qinglong bash git_pull
```
#v4更新命令
```
docker exec -it jd bash jup
```
##安装v4面板

#开启DIY每次重启会重启面板

#先进入容器

```
docker exec -it jd bash
```
```
wget -q https://ghproxy.com/https://raw.githubusercontent.com/afwfv/dd/main/v4mb.sh -O v4mb.sh && chmod +x v4mb.sh && ./v4mb.sh
```
#重启手动运行面板

#先进入容器
```
cd panel
npm i
pm2 start server.js
```
