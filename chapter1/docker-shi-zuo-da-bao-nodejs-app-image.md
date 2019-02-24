基礎用法、工具都熟悉了，接著要開始組合 Services，一個完整的應用程式會由許多的子服務所構成，例如說一個網站通常會有 前端、 Api Server、資料庫等，這些就是一個一個的子服務；

通常一個子服務便是一個 Image，但 Image 間如果需要通信，必須要定義對外開放的 port、處於哪個 network、要啟用幾份 Container等，這也就是 Docker Composer 的功能。

以下實作 Nodejs Api Server 連線 MongoDB 與 Redis。

//TBD

