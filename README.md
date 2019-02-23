# 容器化服務架構

因應服務的擴大，需要管理的應用程式與機器越來越複雜，從部署時間、版本回滾、安全設置、機器效能監控等都日益麻煩；

希望透過 Containerize 應用程式，解決

1. 開發與部屬環境的差異
2. 降低部署時間
3. 版本回滾更迅速

並透過 Container Orchestration 解決

1. 多個 Container 間的協作關係
2. Container Scale out / Scale up 管理
3. Zero Downtime 升級版本、版本回滾管理
4. 統一防火牆與安全配置

本書會以公司目前架構為主，依序章節逐步往容器化服務架構邁進，階段為

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
5. k8s 結合 GCP
6. case study：k8s 實戰經驗



