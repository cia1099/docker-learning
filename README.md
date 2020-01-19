# docker-learning
Docker tutorial
-----
* 基本操作：

|docker|指令|容器參數|輸入參數|鏡像參數|
|---|---|---|---|---|
|docker|run|--name db|--env MYSQL_ROOT_PASSWOED=example|-d mariadb|
|||表示創建的容器名為"db"||'-d'表示將啟動的鏡像至後台執行，如果沒有該參數進程就會在前台執行|



### Container manage
```
docker ps #查詢運行中容器ID簡略形式
docker ps -a #查詢包含已停止的容器
docker ps --no-trunc #顯示完整ID
docker [start/stop] [容器ID或創建容器名稱] #[運行/停止] 對應的容器ID或名稱，在創建容器時可以同 --name 參數給容器起一個別名
docker inspect [容器ID] #可以查詢容器所有基本信息，包含運行情況、存儲位置、配置參數、網路設置等
docker stats #實時察看容器所佔用的系統資源
docker rm 容器名ID #刪除容器
```
* 容器內部命令
docker提供了原生的方式支持登入容器docker exec
```
docker exec 容器名 容器內執行的命令
e.g.
docker exec MyWordPress ps aux
docker exec -it MyWordPress /bin/bash #加上"-it"參數相當於以root身分登入容器內
```
* 多容器管理
docker提供一個容器編排工具"Docker Compose"，他允許用戶在一個模板(YAML格式)中定義一組相關聯的應用容器，這組容器會根據配置模板中的"--link"等參數，對啟動的優先級自動排序，簡單執行一條"docker-compose up"，就可以把同一個服務中的多個容器依次創建啟動
```
#安裝docker-compose
sudo curl -L https://github.com/docker/compose/release/download/1.7.2/docker-compose-`uname -s` - `uname -m` > /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```
vim ~/wordpress/docker-compose.yml #創建一個docker-compose.yml文件，編輯如下內容：
```
wordpress:
    image: wordpress
    link:
        - db:mysql
    ports:
        - 8080:80
db:
    image: mariadb
    environment: MYSQL_ROOT_PASSWORD: example
```
cd ~/wordpress && docker-compose up -d #按順序創建啟動容器，"-d"表示在後台執行
docker-compose start/stop #按順序啟動/停止容器
# 
創建**gitlab**(版本控制管理)容器：
```
vim ~/gitlab/docker-compose.yml
gitlab:
    image: sameersbn/gitlab:8.4.4
    links:
    - redis:redisio
    - postgresql:postgresql
    ports:
    - "10080:80"
    - "10022:22"
    environment:
    - GITLAB_PORT=10080
    - GITLAB_SSH_PORT=10022
    - GITLAB_SECRETS_DB_KEY_BASE=long-and-random-alphanumeric-string
```
docker rm -f gitlab gitlab-redis gitlab-postgresql #刪除舊容器

cd ~/gitlab && docker-compose up -d #在後台創建

docker-compose -f ~/gitlab/docker-compose.yml ps #查詢gitlab項目的所有容器狀態

docker-compose -f ~/gitlab/docker-compose.yml stop #停止

docker-compose -f ~/gitlab/docker-compose.yml start #啟動

docker-compose -f ~/gitlab/docker-compose.yml down #刪除項目[p.73]

通過http://192.168.10.103:10080就可以訪問，默認的用戶名：root，密碼：5iveL!fe[p.42]
#
創建**redmine**(項目流程管理)容器：
```
docker rm -f redmine postgresql-redmine #刪除舊容器
vim ~/redmine/docker-compose.yml
postgresql:
    image: sameersbn/postgresql:9.4-12
    environment:
    - DB_NAME=redmine_production
    - DB_USER=redmine
    - DB_PASS=password

redmine:
    image: sameersbn/redmine:3.2.0-4
    links:
    - postgresql:postgresql
    ports:
    - "10083:80"
    environment:
    - REDMINE_PORT=10083
```
cd ~/redmine && docker-compose up -d #創建
最後，通過http://192.168.10.103:10083就可以訪問網站，可輸入系統默認用戶(用戶名：admin，密碼：admin)進行深入體驗[p.44]

### Image Manage
```
docker images -a #查詢本機已有的所有鏡像
docker push [IMAGENAME] #把鏡像推送到官方倉庫
```
* Dockerfile
語法規則：每一行都以一個關鍵字(大寫字母)為行首，如果一行內容過長，使用`＼`把多行連接到一起
FROM: 從哪個最底層鏡像開始創建
MAITAINER: 指定該鏡像創建者
ENV: 設置環境變量
RUN: 運行shell命令，如果有多條命令可以用`&&`連接
COPY: 將編譯機本地文件拷貝到鏡像文件系統中
EXPOSE: 指定監聽的端口
ENTRYPOINT: 欲執行行命令，在創建鏡像時不執行，要等到使用該鏡像創建容器，容器啟動後才執行的命令
Example:
```dockerfile
# Redis Dockerfile
# https://github.com/sameersbn/docker-redis
#Pull base image
FROM ubuntu:bionic-20190612

LABEL maintainer="sameer@damagehead.com"

ENV REDIS_VERSION=4.0.9 \
    REDIS_USER=redis \
    REDIS_DATA_DIR=/var/lib/redis \
    REDIS_LOG_DIR=/var/log/redis
# Install Redis
RUN apt-get update \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y redis-server=5:${REDIS_VERSION}* \
 && sed 's/^bind /# bind /' -i /etc/redis/redis.conf \
 && sed 's/^logfile /# logfile /' -i /etc/redis/redis.conf \
 && sed 's/^daemonize yes/daemonize no/' -i /etc/redis/redis.conf \
 && sed 's/^protected-mode yes/protected-mode no/' -i /etc/redis/redis.conf \
 && sed 's/^# unixsocket /unixsocket /' -i /etc/redis/redis.conf \
 && sed 's/^# unixsocketperm 700/unixsocketperm 777/' -i /etc/redis/redis.conf \
 && rm -rf /var/lib/apt/lists/*

COPY entrypoint.sh /sbin/entrypoint.sh
RUN chmod 755 /sbin/entrypoint.sh
# Expose ports
EXPOSE 6379/tcp
# Define mountable directories
VOLUME ["${REDIS_DATA_DIR}"]
ENTRYPOINT ["/sbin/entrypoint.sh"]
```
編譯生成鏡像：
創建一個資料夾，在該資料夾目錄下放入`Dockerfile`與用到的`entrypoint.sh`。然後用`docker build`命令編譯Dockerfile，通過`-t`選項給鏡像起一個名字(:代版本號）。
e.g.
```
$docker build -t image_redis:v1.0
$docker images #查看新建的image
```
使用debootstrap工具，可以定製所需要的最小化的Linux基礎鏡像[p.84]
```
sudo apt-get install debootstrap
sudo debootstrap --arch amd64 trusty ubuntu-trusty http://mirrors.163.com/ubuntu/
cd ubuntu-trusty
#修改系統時區改為東八區
sudo cp usr/share/zoneinfo/Asia/Shanghai ect/localtime
＃提交生成基礎鏡像，名字為ubuntu1404-baseimage:1.0
cd ubuntu-trusty
sudo tar -c .|docker import - ubuntu1404-baseimage:1.0
#新創一個容器，查看Ubuntu的系統版本和時區修改是否成功
$docker run -t -i ubuntu1404-baseimage:1.0 /bin/bash
root@c09ba1a54004:/# cat /etc/issue
```
### Respository Manage
```
docker search pytorch #搜索pytorch鏡像
docker pull tensorflow #下載tensorflow鏡像
docker run -p 5000:5000 registry #從Docker Hub拉取docker-register的鏡像，然後啟動docker-register服務，docker-register默認監聽5000端口[p.89]
```
### NetWork and Memory Manage
很多時候，我們會將一些相關的容器部屬在同一個Host上，並希望這些容器可以共享數據，通過`--volumes-from`在其他容器掛載某個容器的數據
e.g.
```
docker run -d -v /dbdata --name dbdata training/postgres echo Data-only
docker run -d --volumes-from dbdata --name db1 training/postgres #db1能看見容器dbdata所有數據的內容。[p.105]
#備份、恢復和遷移數據[p.107]
docker run --volumes-from dbdata -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata #這裡我們創建一個新容器，將Host本地目錄掛載到/backup，然後將數據卷容器dbdata的數據(/dbdata)打包到/backup/backup.tar。然後在Host的當前目錄下就可以得到backup.tar
```