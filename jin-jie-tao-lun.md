# 是否該將 Database 也 Dockerize ?

參考此篇文章 [Should You Run Your Database in Docker?](https://vsupalov.com/database-in-docker/)，作者提到 Docker 適合用來跑 Stateless 的 App，尤其是搭配其他的調度機制如 Kubernetes，因為 container 隨時都可能被加入與移除，所以如果是 stateful APP可能會有狀態不一致的問題；

很顯然 Database 就是 Stateful APP，他必須永久保存資料，如果有 Replica Set 設定檔也是個問題，所以作者不建議在正式環境將 Database 放在 Docker 中執行，畢竟資料是最重要的，不要增加不必要的運營風險。



