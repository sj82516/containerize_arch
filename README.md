# 容器化服務架構

因應服務的擴大，需要管理的應用程式與機器越來越複雜，從部署時間、版本回滾、安全設置、機器效能監控等都日益麻煩；

如果以現在的角度思考服務架構，我會希望是個

1. 有清楚 Dashboard 掌握所有機器的狀態
2. 當我寫完程式碼可以有快速且安全的方式部署到測試環境、正式環境
3. 將不同的 Service 部署到不同規格的機器上
4. 當流量暴增可以自動 scaling，流量下降自動 scale down
5. 歡迎補充

將需求具體化與目標化  
希望透過 Containerize 應用程式，解決

1. 開發與部屬環境的差異
2. 降低部署時間
3. 版本回滾更迅速

透過 Container Orchestration 解決

1. 多個 Container 間的協作關係
2. Container Scale out / Scale up 管理
3. Zero Downtime 升級版本、版本回滾管理
4. 統一防火牆與安全配置

以公司目前架構為主，依序章節逐步往容器化服務架構邁進，階段為

1. Docker 
   1. 基礎介紹
   2. 安裝與常用指令操作
   3. 網路
   4. 效能比對與進階研究
2. Dockerize Nodejs App
   1. 製作 Nodejs App Image
   2. 程式碼版本控制與 Image 版本控制
   3. Benchmark
3. Kubernetes\(簡稱 k8s\)
   1. 基礎介紹
   2. 操作介面說明
   3. 網路結構
4. k8s 結合 Docker
   1. 實際部署
   2. 監控
   3. Zero Downtime 部署與版本回滾
   4. 加入 Jenkins 自動部署
5. k8s 結合 GCP
6. case study：k8s 實戰經驗



