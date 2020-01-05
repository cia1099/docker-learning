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