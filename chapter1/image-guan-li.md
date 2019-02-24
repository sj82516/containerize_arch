當再本地端開發好 Image，執行 Container 確認沒有問題後，可以將 Image 發佈到 Repository 上，類似於 git push/pull 的概念

Docker 有提供 [Docker Hub](https://hub.docker.com/)，有許多公開的 Image 可以下載，例如 Nodejs / MySQL / Redis / MongoDB 等官方團隊都有管理各自的 Image，用起來也很安心；  
另一方面也有很多熱心的開發者，自己製作出 Image 分享在上面。

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



