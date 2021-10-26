# learning_docker_note

Docker是一種容器化技術

Docker跟傳統VM的比較([圖片來源](https://www.docker.com/resources/what-container#/package_software))

實務應用([圖片來源](https://larrylu.blog/step-by-step-dockerize-your-app-ecd8940696f4))

快速部屬, 可跨平台部屬

## Docker安裝

[windows install Docker desktop](https://docs.docker.com/desktop/windows/install/)

[mac install Docker desktop](https://docs.docker.com/desktop/mac/install/)

### 使用流程
write dockerfile-> create image-> build-> run

#### dockerfile
for example:
```
FROM node:6.2.2          #這行會載入程式需要的執行環境，會根據不同的需求下載不同的映像檔，這裡是指 node v6.2.2
WORKDIR /app             #在這個 Docker 中的 Linux 即將會建立一個目錄 /app
ADD . /app               #代表會將本機端與 Dockerfile 同一層的所有檔案加到 Linux 的 /app 目錄底下
RUN npm install          #運行 npm install，npm install 會下載 nodejs 相依的 libraries
EXPOSE 300               #是指 container 對外的埠號，再與外界溝通時使用
CMD npm start            #最後透過 npm start 會運行 Nodejs App
```

## 相關指令
[Dockerfile指令教學, 含範例解說](https://www.jinnsblog.com/2018/12/docker-dockerfile-guide.html)

```bash=
docker -v                         #查看版本
docker build -t getting-started . #建立image檔案
docker images                     #列出所有image
docker ps                         #列出所有container
docker rmi imageID                #刪除image
docker image inspect --format='{{.RepoTags}} {{.Id}} {{.Parent}}' $(docker image ls -q --filter since=imageID) # 找出父相關的image
# 假定回傳sha256: bbc8XXXXXXXXXXXXXX, 則以docker rmi bbc8刪除相關的image

docker run -itd -p 3000:3000 --name getting-started getting-started # 執行image，形成container
# -i , --interactive : 讓 Container 的標準輸入保持打開
# -t , --tty : 讓Docker分配一個虛擬終端（pseudo-tty）並綁定到 Container 的標準輸入上
# -d , --detach : 讓 Container 處於背景執行狀態並印出 Container ID
# --name : 指定 Container 名稱
# -p , --publish : 將 Container 發布到指定的port號

docker start getting-started     #開啟container
docker restart  getting-started  #重啟container
docker stop getting-started      #關閉container

docker rm containerID            #刪除container
```
## 修改image

```bash=
docker build -t {image_name} .          #建立image檔案
docker images                           #列出所有image
docker ps                               #列出所有container
docker ps -a                            #列出所有container 包含停止的
docker run -d -p 3000:3000 {image_name} #執行image，形成container
docker stop {container_name}            #停止container
docker rm {container_name}              #移除container
```

## 使用Docker的GUI操作
* 可以用Docker Desktop
* 可以使用Vscode的套件

## volume
Volume 是一種Docker 的元件，它提供 container 保存資料或共享資料的機制。

因為每次移除container就會讓資料消失，所以需要 volume 儲存container裡面的資料

```bash=
docker volume ls                    #檢視所有volume
docker volume create todo-db        #創建一個新的volume
docker run -dp 3000:3000 -v todo-db:/etc/todos --name getting-started-volume getting-started #run with volume
# todo-db:/etc/todos => /etc/todos是container的位置
docker volume ls                    #列出所有volume
docker volume inspect todo-db       #觀察todo-db這個volume
docker volume pruns                 #移除所有未使用的volumes
docker voume rm                     #移除指定的volumes
docker volume inspect {volume_name} #查詢volume的存放路徑
```
## network
network就是container的網路模式

預設的network模式，Bridge、Host、None
```
docker network connect             #將容器連接到容器網路
docker network create              #建立容器網路
docker network disconnect          #從容器網路移除指定容器
docker network inspect             #列出所有容器網路
docker network ls                  #列出所有容器網路
docker network prune               #移除所有未使用的容器網路
docker network rm                  #移除指定的容器網路
```

### Bridge模式

同個network的container可以互通

```bash=
docker run -d -p 3000:3000 --network bridge getting-started
```

### Host模式

會使container的隔離性消失，等同於host有什麼網路資源，container都會有

```bash=
docker run -d --network host getting-started
```

### None模式

獨立的模式，無法連外，也無法被連

```bash=
docker run -d -p 3000:3000 --network none getting-started
```

相關指令

```bash
dokcer network ls                    #列出所有network
docker network create todo-app       #建立新的network 

docker inspect todo-app              #查看特定的network
docker ps                            #列出所有container
docker exec -it ${contianer} /bin/sh #進入container
```

## docker compose
建立mysql的資料庫([參考](https://hub.docker.com/_/mysql/))

```bash
docker run -dp 3306:3306
     --network todo-app --network-alias mysql 
     -v todo-mysql-data:/var/lib/mysql 
     -e MYSQL_ROOT_PASSWORD=secret 
     -e MYSQL_DATABASE=todos 
     mysql:5.7
```

run app 並連線到資料庫

```bash
docker run -dp 3000:3000 
   --network todo-app 
   -e MYSQL_HOST=mysql 
   -e MYSQL_USER=root 
   -e MYSQL_PASSWORD=secret 
   -e MYSQL_DB=todos 
   get-start
```

將上面的docker run轉換成docker-compose.yml

```docker
version: "3.7"

services:
  app:
    image: get-start
    ports:
      - 3000:3000
    build:
      context: .
      dockerfile: Dockerfile
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

  mysql:
    image: mysql:5.7
		ports:
      - 3306:3306
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

相關指令

```bash
docker-compose up -d          #在背景執行docker
docker-compose logs -f        #查看logs
docker-compose logs -f app    #查看特定服務的logs
docker-compose down           #關掉服務
docker-compose down --volumes #關掉服務並刪掉volumes
```

工具

```bash
docker run -it --network todo-app nicolaka/netshoot #別人寫的工具
dig mysql                                           #找出mysql ip
```

[YAML語法教學](http://www.wl-chuang.com/blog/2011/11/06/yaml-tutorial/)
