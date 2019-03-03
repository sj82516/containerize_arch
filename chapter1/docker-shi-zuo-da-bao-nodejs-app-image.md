經過了幾個章節，介紹了 Docker 的源起，Image 的構成與 Container 的概念，Volume 的使用與 Network 通信的方式。

接著就要開始擴展 Container 的使用，一般來說應用程式會由多個服務構成，每個服務通常代表一個 Image，依照需求執行一到多個 Container，如果每個都要手動管理那會累得半死。

這一章要介紹 Docker-Composer，透過設定檔一次性啟用整個應用程式於同一個 Docker Host上，常見的用途有

1. 開發環境
2. CI/CD 建置環境
3. 單主機上的服務

以下實作 Nodejs Api Server 連線 MongoDB 與 Redis。

先修改 server.js 檔案，當 docker-compose 啟用時，會把所有的 container 都連接到自定義的 bridge 當中，在程式碼中可以直接指定 container 名稱，會自動解析成對應的 private ip。

```js
const express = require('express');
const MongoClient = require('mongodb').MongoClient;
const Redis = require('ioredis');
var redis = new Redis({
  port: 6379,          // Redis port
  host: 'redis',   // Redis host
  db: 0
});

// Constants
const PORT = 8080;
const HOST = '0.0.0.0';

// Connect using MongoClient
MongoClient.connect("mongodb://mongodb:27017", function(err, client) {
  if(err){
    return console.error(err, client);
  }
  // Use the admin database for the operation
  const logCollection = client.db("test").collection("log");
  // List all the available databases

  // App
  const app = express();
  app.get('/', async (req, res) => {
    const logList =  await logCollection.find().toArray();
    const view = await redis.get("view");
    console.log(logList.length);
    res.send(`Hello world, this page is viewed ${view} times.\n`);
    await redis.incr("view");
    await logCollection.insertOne({
      log: view
    });
  });

  app.listen(PORT, HOST);
  console.log(`Running on version3 http://${HOST}:${PORT}`);
});
```

## 定義 docker-compose.yml

```yml
version: '3.7'
services:
  redis:
    image: "redis:alpine"
    volumes:
      - type: volume
        source: redisdata
        target: /data
    networks: 
      - custom-network
    configs:
      - appendonly: 'yes'
  mongodb:
    image: "mongo:4.0.6-xenial"
    volumes:
      - "dbdata:/data/db"
    networks: 
      - custom-network
  web:
    container_name: my-web
    build: .
    ports:
      - "8080:8080"
    depends_on:
      - redis
      - mongodb
    networks: 
      - custom-network
networks: 
  custom-network:
    external: true
volumes:
  dbdata: 
    external: true
  redisdata:
    external: true
```

常見的參數有

1. version:   
   指定版本號

2. services：  
   指定服務的內容，預設key 為 container 名稱

   1. build：  
      從指定的 Dockerfile 建立 Image 並啟用 Container

   2. volumes：  
      指定volumes，記得要在 service 之外定義 volumes 來源，如果要用預先自定義好的必須加 external: true，否則 docker-compose 會自動幫你創建新的

   3. networks  
      雷同於上者

   4. prots：  
      指定對外開放與連接 Host 的 port mapping

   5. depends_on：  
      docker-compose 預設並行啟用 container，_如果有服務前後相依就必須設定\_ depends\_\_on  
      例如web service 相依於 mongodb 與 redis

   6. container\_name：  
      指定 container 名稱

# docker-compose 指令

必須再有 docker-compose.yml 的路徑下才可以執行

```
// 如果有 image 使用 build，每次更改後要主動 build，不然 docker-compose 會沿用舊的
$ docker-compose build

// 啟用服務
$ docker-compose up

// 前景啟用服務
$ docker-compose up

// 背景啟用服務
$ docker-compose up -d

// 條列所有服務
$ docker-compose ps

// 刪除舊的服務
$ docker-compose down
```

# 參考資料

1. [Compose file version 3 reference](https://docs.docker.com/compose/compose-file/)
2. [Overview of Docker Compose](https://docs.docker.com/compose/overview/)



