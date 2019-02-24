# 前言

先到 docker hub 下載對應平台的 [Docker CE \(Comunity Edition\)](https://hub.docker.com/search?q=docker&type=edition&offering=community)  
Docker 目前有 CE / EE 兩種版本，EE 需要付費主要提供企業用，看起來是差在版本維護的時間與後續的客戶服務。

docker 有幾個常用的網域名

1. [docker.com](https://www.docker.com/)：
   官網，有一些消息發布
2. hub.docker.co[https://hub.docker.com/](https://hub.docker.com/)m：  
   docker images 發布的地方

3. [https://docs.docker.com](https://docs.docker.com)  
   教學文件

安裝好後，在 Terminal 執行

```
$ docker --version

Docker version 18.09.2, build 6247962
```

接著執行

```
$ docker run hello-world
```

確認docker 安裝正常，接著條列目前本機上 docker 下載的 image 清單

```
$ docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              fce289e99eb9        7 weeks ago         1.84kB
```

查看運行中的 container，因為 hello-world 打印完就會退出，加上參數 -a 條列所有包括暫停的 container

```
$ docker ps

$ docker ps -a

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
d134a8629dce        hello-world         "/hello"            2 minutes ago       Exited (0) 2 minutes ago                       stoic_hypatia
```

# Docker Images 與 Container

用比喻來說， Image 就像是食譜，而 Container 是一塊蛋糕。

我們會寫一份文件 Dockerfile 定義 Image ，例如說這樣

```
FROM ubuntu:15.04
COPY . /app
RUN make /app
CMD python /app/app.py
```

這份文件表達

* 用 ubuntu 當做基底
* 將本機當前的目錄複製到 Image 中的 /app 路徑下
* 接著執行指定的指令 make /app
* 如果要啟動這份 Image，預設將會執行 python /app/app.py

透過 `$ docker build`可以製作出 Image

而 Container 則是指定某一個 Image 執行如 `$docker run hello-world`

Dockerfile 與相關指令這後續會再提，先看一下 Image 到底是如何構成的？

## Image 與 Container 實質上的差異

更細部了解 Image 的儲存方式，可以幫助我們製作出更正確的使用 Docker

先回想一下在部署程式碼到雲端主機上時，步驟通常會是

1. 選定 OS，開啟一台 VM
2. ssh 登入 VM後，安裝 Nodejs LTS最新版
3. 安裝 git 拉原始碼，或是用 scp 打包上傳原始碼
4. npm install 需要的模組
5. 開始執行 Nodejs 應用程式

仔細思考一下，每一個步驟就好像一層一層的 Layer，我們不斷的在基底 OS上增加需要的工具，通常這些步驟也不太會去更動，除了一些設定檔以外。

Docker Image 也是運用同樣的概念，由一層一層的Layer 組成，每一層 Layer 會有一個對應的 Hash碼，這些 Layer 是唯讀不可更動，由唯讀的 Layer 組成 Image 本身也是唯讀的；

![](/assets/container-layers.jpg)

當透過 Image 創建 Container時，會在原有的 Image Layers 上多加一層可讀寫的 Layer 層，所有 Container 的更動都會被保存在這一層。

這樣設計的好處是 如果同一個 Layer 被多個 Image 所共用，又或是同一個 Image 被多個 Container 所使用，**全部在 Disk 只要儲存一份**! 對於空間上有相當大的幫助，同時也不用每次都要重新下載重複的資料。

## The copy-on-write \(CoW\) strategy {#the-copy-on-write-cow-strategy}

[copy-on-write](https://zh.wikipedia.org/wiki/寫入時複製) 是指當有多個呼叫者想要讀取同一份檔案，系統會直接回傳該文件的指標，除非當有一位呼叫者要修改文件時，系統才會真正複製一份給該呼叫者，其餘呼叫者維持原文件的指標。  
如果應用在大量讀取的場景，這樣的做法可以大量降低 File I/O 並提升效能。

先前提到 Docker Image 中的 Layer 都是唯讀，所以共用 Image的多個 Container 都是拿到同樣的指標，但例如說 Container 在運作中想要修改 Nginx Config 檔案，就會個別寫入在 Container 的 R/W layer，後續讀取也是讀到 R/W layer 修改過後的結果，而原本的 Nginx Layer 中的檔案維持不變。

# 本章參考資料

1. [About images, containers, and storage drivers](https://docs.docker.com/v17.09/engine/userguide/storagedriver/imagesandcontainers/)
2. [Digging into Docker layers](https://medium.com/@jessgreb01/digging-into-docker-layers-c22f948ed612)

###### 



