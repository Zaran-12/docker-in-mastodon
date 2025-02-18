


# Docker 部署 Mastodon - 一个去中心化的社交平台

## 开始之前

准备一个域名和证书

- 域名：`yourDomain.com`
- 证书：`yourDomain.key`、`yourDomain.pem`

如果只是想本地跑一下，也行

- 修改hosts：`127.0.0.1 yourDomain.com`
- web、streaming、sidekiq 这3个服务增加`extra_hosts`，如下：

```
extra_hosts: 
  - "yourDomain:192.168.0.109"

#192.168.0.109 为宿主机ip
#extra_hosts作用是 往容器内/etc/hosts文件中添加记录，注意格式是相反的
```
- 修改nginx/conf.d/zaranmm.xin.conf
   修改如何地方为你自己的域名和ssl证书实际地方
```
server_name yourDomain.com;
ssl_certificate   ssl/yourDomain.com.pem;
ssl_certificate_key  ssl/yourDomain.com.key;
```
- 对于ssl证书，可以在nginx目录下新建ssl目录，将证书放在里面
## 快速开始

- 更新系统依赖
```
sudo apt update && sudo apt upgrade -y
sudo apt install -y \
  git curl build-essential libssl-dev libreadline-dev \
  zlib1g-dev libidn11-dev libicu-dev libpq-dev redis-server \
  imagemagick ffmpeg libprotobuf-dev protobuf-compiler
sudo apt install nginx
```

- 下载docker
```
bash <(curl -L https://get.docker.com/)
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

- 如何配置docker
```
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER 
```

- 安装Node.js和yarn
```
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt install -y nodejs
npm install -g yarn
```

### 初始化
```
docker compose -f docker-compose.yml run --rm web bundle exec rake mastodon:setup
```

上一步执行成功，会启动`db`和`redis`两个容器，同时会提示你输入域名（先别输），先进到`db`容器
```
sudo docker exec -it mastodon-db-1 psql -U postgres
```
创建一个给`mastodon`用的数据库，如下创建一个用户和数据库，名称都是`mastodon`，密码为空
```
psql -U postgres
CREATE USER mastodon CREATEDB;
create database mastodon owner mastodon encoding UTF8;
```

接着，按照提示，一步步来，以下是我的可做参考

```
LOCAL_DOMAIN=zaranmm.xin
```
```
SINGLE_USER_MODE=false
```
```
SECRET_KEY_BASE=ee9095affc96ac922059b93728da143acfc4d4af16721c92ed11b2166583e31454f3e19b5ae8f66f3183127f0ed8ee07aaa08006d5e00c5515fe86d8aa8e032
```
```
OTP_SECRET=b182f5070d0a3c3c1f52ade3ef5ffd95ecfebdc1291a9ae602c2a382d686bde1eea3cdea2a9710e96e1dea3875f6307adc20ad4c735459ce4ca2a0cc3ea27bb3
```
```
VAPID_PRIVATE_KEY=Nnh--Pw5lyIsZQCm9cTuW4f6epX4cC4H7QlxgkXK0Sk=
```
```
VAPID_PUBLIC_KEY=BHExfrW4_fJqrlqTAV-orMIX85Axyc9n_Jcsu4VCYzZcThxZOjZezY6r0G5zxXWlerBkk0nybvoa5OdnvTJB0dY=
```
```
DB_HOST=db
DB_PORT=5432
DB_NAME=mastodon
DB_USER=mastodon
DB_PASS=
```
```
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=
```
```
SMTP_SERVER=smtp.mailgun.org
SMTP_PORT=587
SMTP_LOGIN=
SMTP_PASSWORD=
SMTP_AUTH_METHOD=plain
SMTP_OPENSSL_VERIFY_MODE=none
SMTP_ENABLE_STARTTLS=auto
SMTP_FROM_ADDRESS=Mastodon <notifications@zaranmm.xin>
```
```
UPDATE_CHECK_URL=
```
接下来，生成一份配置，需要手动复制到`.env.production`文件
也就是我上面给你的那些

最后是导入数据，和创建管理员用户
管理员密码记一下


### 启动服务
初始化完成，就能启动服务了
```
docker compose up -d
```

## 访问（关掉代理）
http://yourDomain.com
也可以使用以下语句看看是否运行成功
```
curl -H "Host:zaranmm.xin" http://127.0.0.1/
```


## 其他
`.env.production` 从何而来？

下载官方代码
```
git clone git@github.com:mastodon/mastodon.git
```
根目录有个`.env.production.sample`文件，改名为 `.env.production`，(必须的)

如果是初次运行，记得把里面`LOCAL_DOMAIN`, `PostgreSQL`，`redis`这些你知道的都配好（不配也可以，只是最后一步创建管理员账号会失败）

端口占用解决办法
```
sudo lsof -i:80
sudo kill -9 <pid>
```

结束运行
```
docker compose stop
```

删除镜像
```
sudo docker stop $(sudo docker ps -q)
sudo docker rm$(sudo docker ps -aq)
```
