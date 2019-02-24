本章將介紹如何製作 Image、如何運行 Container 與相關的 docker 指令。

# 製作 Image

參考 Nodejs 官方文件 [Dockerizing a Node.js web app](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/)

首先我們先創建一個 Nodejs 專案，並寫一個簡單的 Express server

```js
const express = require('express');

// Constants
const PORT = 8080;
const HOST = '0.0.0.0';

// App
const app = express();
app.get('/', (req, res) => {
  res.send('Hello world\n');
});

app.listen(PORT, HOST);
console.log(`Running on http://${HOST}:${PORT}`);
```

同目錄下創建 Dockerfile

```js
FROM node:8

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm ci --only=production

# Bundle app source
COPY . .

EXPOSE 8080
CMD [ "npm", "start" ]
```

一般來說Dockerfile 有幾個常用的參數

1. FROM  
   指定用哪一個 Image 當做基底，此欄位**必填且必須為Dockerfile 第一個指令**

2. WORKDIR  
   設定運作的當前目錄，影響的指令有 RUN / CMD / ENREYPOINT / COPY / ADD，如果指定的 WORKDIR 不存在會自動創建

3. COPY \[from\] \[to\]  
   將當前宿主環境的資料複製到 Docker Image 中

4. RUN  
   執行 shell command，執行過一次後會被 cached 住，可以在執行時關閉  
   有兩種格式

   1. shell form： RUN &lt;command&gt;

   2. exec form：RUN \["executable", "param1", "param2"\]  
      兩者差異在於 shell form 每次執行都是新的 Command Shell，也就是兩個 shell form 之間如果有變數設定是互相看不到，而 exec form 並不會觸發 **Command Shell**

5. EXPOSE  
   當 Container 運行時所 listen的 port number，預設為 TCP 但也可以指定為 UDP EXPOSE 80/UDP

6. CMD  
   提供 Container 運行時的預設值，有三種宣告方式，各有不同的用途

   1. CMD \["executable", "param1", "param2"\]  
      exec form，同樣是不觸發 Command Shell的指令，可用於指定不同 shell 執行

   2. CMD \["param1", "param2"\]  
      提供 ENTRYPOINT 的預設參數

   3. CMD command1 param1 param2  
      shell form，雷同

      但如果是要在 Container 中啟用服務，建議不要用 CMD，因為只要用戶輸入參數 CMD 就不會執行。

7. ENTRYPOINT  
   如果要啟用服務，官方建議使用 ENTRYPOINT，一樣是有 shell form / exec form 兩種格式；  
   使用 ENTRYPOINT 方便的地方在於一定會被執行到，且可以從 CMD 讀取參數或是在docker run 運行的時候指定

.dockerignore，類似於 .gitignore， COPY 檔案時會自動忽略

```
node_modules
npm-debug.log
```

接著在同目錄下，就可以 build 第一個 Image

```
$ docker build -f ./Dockerfile -t yuanchieh/server:1.0.0 .
```

docker build 必須指定當前路徑，這個路徑會影響如 COPY 的宿主環境路徑，可以是 PATH 或是 GIT URL；  
-f 指定 Dockerfile ，如果沒有則是當前路徑的 Dockerfile；  
-t 是為 image 標上 tag，通常命名習慣是 `[組織名]/[應用程式名]:[版本號碼]`  
如果沒有指定，則會變成 &lt;none&gt;，後續會非常不好管理

如果是第一次執行，Docker 會依照 Dockerfile 的一行一行指令依序執行，一行指令便是一層 Layer；  
第一次執行過後就會自動 Cache，如果下次再次執行沒有修改的會就會沿用；  
這裡有個小 trick 要注意，Dockerfile 中是先 COPY package.json 並安裝完成後才 COPY . . 剩下的部分，原因就在於如果是先 COPY . . 再 npm install，只要目錄下稍有變動即使 package.json 沒有更動整個 Cache 都報廢，在設計上要特別注意。

# 運行 Container

有了Image，可以實際啟用 Container

## 前景執行

```
$ docker run -p 8080:8080 yuanchieh/server
```

-p 參數是指定 \[宿主機器port\] 映射至 \[Container port\]，並需要指定 Container 才能收到外部 connection 傳送的資料

如果希望退出直接 Ctrl+C 即可，但 Container 因為是在前景執行所以退出後會自動停止。

如果是希望藉由 terminal 登入 Container 查看，可以改用

```
$ docker run -it yuanchieh/server /bin/bash
```

-it 表示已交互模式登入，並分配一個假的輸入終端口，執行 Container時用 /bin/bash

所有的 container 可以用查看

```
$ docker ps -a
```

如果已經停止了，可以用 $ docker restart \[container id\] 重新執行；  
如果要重新用前景執行，可以用 $ docker attach \[container id\]。

## 後景執行

```
$ docker run -d -p 8080:8080 yuanchieh/server 
```

指定 -d 讓 Container 於後景執行

如果 Container 運行一陣子發現有什麼問題，有 log 的話可以將 Container 製作成新的 Image，重新起一個 Container 連線進去 Debug

```
$ docker commit [container id] [image name]
```

---

# 永久化保存資料 Docker Volume

先前提到，Container 在運行時其實是在 Image 上多一層可讀寫的 R/W layer，但是這些資料只會保存在 Container 當中，如果希望這些資料永久被保存下來，就必須透過 volume

volume 可以應用在以下場景

1. 適合用於備份
2. 安全的在 Container 間共享
3. 透過不同 Driver，可以儲存在本機端或是雲端等
4. 可以先放資料再掛載到 Container 上

其餘像資料庫的資料、Log、靜態資源檔案，都非常適用放在 Volume中，盡量降低 Container 的尺寸。

```
$ docker volume create hello
```

掛載到 Container 的 /world 的路徑下

```
$ docker run -d -p 8080:8080 -v hello:/world yuanchieh/server 
```

在使用 volume時，如果希望指定宿主環境的某一個資料夾，目前必須安裝 Docker Plugin  [local-persist](https://github.com/CWSpear/local-persist)





