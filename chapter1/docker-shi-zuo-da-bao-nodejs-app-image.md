經過了幾個章節，介紹了 Docker 的源起，Image 的構成與 Container 的概念，Volume 的使用與 Network 通信的方式。

接著就要開始擴展 Container 的使用，一般來說應用程式會由多個服務構成，每個服務通常代表一個 Image，依照需求執行一到多個 Container，如果每個都要手動管理那會累得半死。

這一章要介紹 Docker-Composer，透過設定檔一次性啟用整個應用程式於同一個 Docker Host上。

以下實作 Nodejs Api Server 連線 MongoDB 與 Redis。



//TBD

