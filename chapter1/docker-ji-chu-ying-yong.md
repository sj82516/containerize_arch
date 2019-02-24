本章將介紹如何製作 Image、如何運行 Container 與相關的 docker 指令。

製作 Image

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



