當再本地端開發好 Image，執行 Container 確認沒有問題後，可以將 Image 發佈到 Repository 上，類似於 git push/pull 的概念

Docker 有提供 [Docker Hub](https://hub.docker.com/)，有許多公開的 Image 可以下載，例如 Nodejs / MySQL / Redis / MongoDB 等官方團隊都有管理各自的 Image。

搜尋時可以用網頁，也可以在 Terminal 直接搜尋

```
$ docker search --limit 10 --filter "stars=10" mysql

NAME                            DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mysql                           MySQL is a widely used, open-source relation…   7837                [OK]
mariadb                         MariaDB is a community-developed fork of MyS…   2596                [OK]
mysql/mysql-server              Optimized MySQL Server Docker images. Create…   591                                     [OK]
```

search 預設會依照星星數由高至低排序，有兩個參數我個人比較常用

* limit：限制搜尋返回的結果
* filter：增加搜尋條件，可以指定 stars / is-automated / is-official 

# Registry

Registry，用來保存 Image 登錄檔， 除了 Docker Hub 外，其他雲端服務如 AWS 有提供 [ECR](https://aws.amazon.com/tw/ecr/)，又或是可以自己透過 Registry，Docker 官方提供用來架設[私有 Registry 的 Image](https://hub.docker.com/_/registry)

用 Docker Image 架設 Docker Registry 來管理 Docker Image 有點拗口，但這也是 Docker 或是說 Container 技術有趣的地方，大多數的應用都可以被製作成 Container 使用，包含 Docker 內還可以在執行 Docker。

# Push / Pull Image

```
$ docker login
$ docker push [image name: tag]
```

要 pull Image 有幾種方式

1. docker run 直接執行，如果本地端沒有該 Image 會從 DockerHub 或是指定的 Registry 拉
2. Dockerfile 中定義，同樣是本地端沒有就會去遠端拉
3. `$ docker pull [image name: tag]`



