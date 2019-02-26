一個正常的應用程式往往會切成多個 Container 負責不同的服務，此時 Container 之間就會需要彼此間通信、跨 Docker Host、跨實體主機等，透過 Docker Network 設定來完成這些事。

# Network 種類介紹

1. none  
   Container 禁用任何網路

2. host  
   直接使用宿主環境的網路設定，這會有點危險因為 Container 的修改會直接影響宿主環境

3. bridge  
   預設的網路設定方式

4. overlay  
   可以跨 Docker Host 通信

5. Macvlan  
   直接指定Container 實體的 MAC 地址，讓 Container 在 Network 中以實體裝置顯示，因為是直接接到實體網路而不是透過 Docker Host Network，適合用於要求低延遲的需求。

以下僅介紹 Bridge Network

## Bridge Network

```
|-------------------------------------|
|                                     |
| |------------|      |------------|  |
| | Container1 <----->| Container2 |  |
| |------------|      |------------|  |
|                                     |
|-------------------------------------|
Docker Host
```

Bridge Network 適用於同一個 Docker Host 下的 Container 通信，Bridge 設定主要儲存於 Docker Host 當中，所以如果需要跨 Docker Host 通信就需要使用 Overlay Network\(基於 Host OS Level 的網路設定\)

當安裝完 Docker 後，會有預設的 Bridge Network，用戶也可以後續自訂 Bridge Network；  
Docker 官方建議使用自定義的 Bridge，有以下優點

1. 用戶自訂義的 Bridge 有更好的隔離性  
   當 Container 連上同一個自定義的 Bridge 後，Container 的 **port 對內是完全開放，對外完全封閉**，這使得內部 Container 間通信非常簡單，也不用擔心外部的惡意 access；  
   例如說有一個前後端分離的應用程式，希望後端的 Container 對外開放 80 port，但是資料庫的 port 希望只有內部能夠使用，用戶自訂義的 Bridge 可以做到這樣的設定；  
   但如果是用預設的 Bridge，Container 間是用 -p 對外開放 port，就必須用額外的手段篩選外部對資料庫的連線。

2. 自動 DNS 解析  
   如果使用預設的 Bridge，Container 間溝通只能用 ip，除非使用 --link 舊的寫法；  
   自定義的Bridge 可以直接透過 Container 名稱，DNS會自動解析；  
   當然也可以透過自己管理 /etc/hosts 做到一樣的事情，但就是會比較麻煩

3. 隨時解除Network  
   用戶可以隨時在 Container 運行過程中移除或加入新的 Network；  
   但如果是用預設的 Bridge就必須要先停止 Container

4. 只有預設 Network 可以用 --link 共享環境變數，自定義 Bridge 不行；  
   但可以透過 volume 共享設定檔、docker-composer 定義在設定檔中或是用 docker-swarm 等其他做法

# 實作

網路相關的指令都是定義在 `$ docker network`

### 條列目前所有的網路

```
$ docker network ls
最基本一定有三組，bridge + host + none
```

觀察 network 中的設定

```
$ docker network inspect bridge
```

#### 使用預設 bridge

接著啟用兩個 Alpine container，並加入預設的 bridge 當中， Alpine 是 Docker 中最迷你的 OS Image 檔

```
$ docker run -dit --name alpine1 alpine ash
$ docker run -dit --name alpine2 alpine ash

$ docker network inspect bridge
"Containers": {
    "a19dd1007ed6c3ca5a5ac49f1f7487cc4d11a145ef321e177bee4409c8a90303": {
        "Name": "alpine1",
        "EndpointID": "82c7af3dd580602e7e72536325de2221f6ebe523da52c8e1cca6484c1e967ae6",
        "MacAddress": "02:42:ac:11:00:02",
        "IPv4Address": "172.17.0.2/16",
        "IPv6Address": ""
    },
    "c238bef9287acf20a89297bc5dfe3b3473c4060080481d56f522a97aec1ea4d3": {
        "Name": "alpine2",
        "EndpointID": "0a23a34673eaa2824cf74e0b4c8597acd96d4842df36792f8e8d553ca119bd56",
        "MacAddress": "02:42:ac:11:00:03",
        "IPv4Address": "172.17.0.3/16",
        "IPv6Address": ""
    }
},
```

此時進入 alpine1，觀察 network 行為

```
$ docker attach alpine1
$ ping -c 2 google.com
// ping 得出去

$ ping -c 2 172.17.0.3
// 可以 ping 到 alpine2 的ip

$ ping -c 2 alpine2
// 失敗，ping: bad address 'alpine2'
```

#### 使用自定義 bridge

建立一個名為 alpine-net 的 bridge network

```
$ docker network create --driver bridge alpine-net
```

注意到 alpine-net的 gateway 為 172.18.0.1

```
$ docker network inspect alpine-net
```

接著創建四個 Container，並依序加入 bridge，1,2,4 加入了自定義 bridge，3,4 加入預設 bridge

```
$ docker run -dit --name alpine1 --network alpine-net alpine ash
$ docker run -dit --name alpine2 --network alpine-net alpine ash
$ docker run -dit --name alpine3 alpine ash
$ docker run -dit --name alpine4 --network alpine-net alpine ash
$ docker network connect bridge alpine4
```

同樣在進入到 alpine1 當中，直接用 container name 會被自動解析成 ip

```
$ docker attach alpine1

$ ping -c 2 alpine2
// 可以 ping 到 alpine2
```

# 參考資料

1. [Use bridge networks](https://docs.docker.com/network/bridge/)
2. [Networking with standalone containers](https://docs.docker.com/network/network-tutorial-standalone/)

#  {#title}



