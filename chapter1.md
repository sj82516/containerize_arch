以下介紹與圖片，出自於 [The whole story - Docker, The linux container engine](/h ttp://scm.zoomquiet.top/data/20131004215734/index.html#)

# 什麼是Docker

Docker 主要是基於 Linux LXC 技術開發，提供一個虛擬化的環境運行 Containers，Container \(譯作容器、或是貨櫃\)，顧名思義即是將內容物封裝的一種方式，封裝成容器後隔離了跟實體主機\(Host Machine\)耦合，不論是運行在個人筆電上、雲端伺服器上、虛擬機上等等，都會得到相同的運作機制。

Docker 常用於

1. 自動化打包與部屬應用程式
2. 打造私人、輕量的 PAAS\(Platform as Service\)環境
3. CI/CD
4. 部署並可擴展的服務

# 背景知識

在十幾年前，應用程式幾乎都是全部放在同一台實體主機上，又稱為 Monolithic 整體式架構；  
而今日，開發者開始應用程式拆解，將不同可獨立運行的元件部署在不同的機器上，例如 Api Server、資料庫、快取資料庫被分散在不同的機器上。

分散在不同的機器上的好處是

1. 不同的應用程式有各自適合的環境，例如 Api Server 可能需要較高的 CPU運算能力、資料庫需要較多的記憶體跟硬碟空間等
2. 可以針對效能貧頸個別優化，Api Server 回應太慢可以增加多台 Servre 回應等

但多台機器，帶來管理上的困擾，同一個應用程式運行在不同的硬體上，可能會有許多不預期的錯誤發生，例如說每一位新人進來就必須要在本地端部署一份開發環境，安裝 MongoDB、Nodejs、Redis等，如果裝到不同的版本行為上可能就會不一樣，更別說如果是不同 OS 不同硬體，會有更多潛在的問題

將不同的服務 x 不同的機器會產生許多場景要去處理與測試，會遇到 matrix hell

![](/assets/matrix.jpg)

## 如何解決

回到 1960年代，當時的貨運業也遇到相同的困境，當貨物在工廠生產完畢，會需要先用貨車運送至火車站，接著火車一路到港口上船，接著在運行到世界各地；  
貨運業從農產品如大豆，到奢侈品如跑車，要處理的貨物各式各樣，有大有小、有貴有便宜，所以該如何運送是個大麻煩

![](/assets/standard_container.jpg)

幸運的最後找到了解法 **貨櫃**，不論是哪個類型的商品，一律裝載到標準尺寸的貨櫃當中，接著各個運輸工具就可以很方便的運載貨櫃，同時貨櫃的好處是可以被堆疊、有效率的移動、方便搬移等，因為這樣的改革也間接使世界貿易更加盛行。

他山之石可以攻錯，資訊界也運用相同的概念，將應用程式封裝到 Container\(慣例譯成 容器\)，往後就不太需要理會運行的宿主環境，專注於應用程式的開發，部署就全由容器化技術去煩惱。

> 對開發者來說，Build once run anywhere；對Devops來說，Configure once run anything.

# Docker 內部技術的使用

剛剛是用概念性的方式介紹Docker 技術與嘗試解決的問題，接著看一下 Docker 內部技術的使用。

## 虛擬化

[虛擬化](https://zh.wikipedia.org/wiki/虛擬化)是電腦科學的一項資源管理技術，主要是將實體資源\(如 CPU / Network/ Memory\)予以抽象、轉換後呈現出來並可供分割、組合為一個或多個電腦組態環境。

而 Docker，則是位於**作業系統層虛擬化 \(**Operating system–level virtualization**\)，**亦稱**容器化\(**Containerization\)，這項技術源起於 \*unix 的 chroot機制，[chroot](http://man.linuxde.net/chroot) 可以改變運行指令的根目錄，系統讀取的目錄與文件就會是新目錄底下，帶來的好處有

1. 增加系統安全性
2. 與原系統目錄結構隔離
3. 切換系統根目錄，引導 Linux 系統啟動與急救系統

基於 chroot 機制，Linux 在 2006年加入了 [LXC\(Linux Containers\)](https://zh.wikipedia.org/wiki/LXC) 作業系統層級的虛擬化，將應用軟體系統打包成一個軟體容器（Container），內含應用軟體本身的程式碼，以及所需要的作業系統核心和函式庫。透過統一的命名空間和共用API來分配不同軟體容器的可用硬體資源，創造出應用程式的獨立Sandbox\(沙箱\)執行環境。

虛擬化的另一個好處是 Isolation\(隔離\)，限制每個 Container 運作時只能存取自己的資源，比較安全外同時一個 Container 掛掉也不會影響到其他 Container 的運作；  
Container 是屬於 Process Level 的 Isolation。

\(以上內容擷取自 Wiki參考資料\)

Docker 便是基於 LXC開發，所以只能運行在 Linux 環境上，至於 Mac 透過 Virtualbox / Windows 透過 Hyper-V 先創建Linux VM，在VM中才又運行 Docker。

延伸閱讀：[Understanding the Docker Internals](https://medium.com/@nagarwal/understanding-the-docker-internals-7ccb052ce9fe)

## Container vs VM

內容參考自 [What’s the Diff: VMs vs Containers](https://www.backblaze.com/blog/vm-vs-containers/)

![](/assets/container_vs_vm.jpeg)

最大的差別在於虛擬化的層級不同，VM是位於硬體層虛擬化，而Docker 位於作業系統層虛擬化；

這樣帶來的實質差異是

VM會吃掉相當多的資源，如 Windows OS Image 就十幾GB，但好處是要應用程式可以使用OS全部的資源、修改OS層級系統設定、安全\( OS level isolation\)相關的控管會比較方便；

而 Container 好處在於輕量、啟動快速等。

# 進階 - LXC 與 Docker 如何實作

先前提到 Docker 是基於  Linux LXC 容器化技術開發，在 Linux Kernel 中，並不存在所謂的 LXC，所謂的 LXC 其實是工具組的統稱

1. cgroup  
   限制 container 能做到哪些事，在系統中將物理資源做分配，並以虛擬化的層級形式顯示；  
   有趣的是系統預設就是在一個沒有做任何資源限制的 container 當中，所以不需要在懷疑 container 是否會對性能有多大的影響，因為你就在 container 當中。

   1. Memory

   2. CPU

   3. Blkio  
      IO 相關限制，例如 Disk 讀寫速度等

2. namespace  
   限制 process 能看到系統的哪些資源

   1. Mount\(mnt\)  
      只能讀寫特定的資料夾

   2. PID  
      只能看到某些 process

   3. IPC  
      不能使用 pipe , share memory 當作 process間通信機制

   4. UTS  
      限制 hostname / domain name

   5. Users  
      獨立於 host 的用戶管理，在 container 中的 root 不會是 host 的 root

參考資料：

[This Is How Docker Works, The Fun Way!](https://www.youtube.com/watch?v=-NzfOhSAZpA&t=72s)  
講者使用 Golang 實作簡單的 Docker，主要先起一個 parent process，接著接收用戶輸入的指令，並在 fork child process 時設定 namespace，達到資源限制的設定。

[Cgroups, namespaces, and beyond: what are containers made from?](https://www.youtube.com/watch?v=sK5i-N34im8)  
Docker 內部工程師的分享，對 cgroup 與 namespace 有更深入的講解，並在最後 Demo 實作一個 Container。



