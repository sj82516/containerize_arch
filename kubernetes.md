雖然 Docker Composer 可以一次性將 Application 啟動，但是目前只能部署在同一台機器上，如果希望能將 Container 分散到不同機器上、即時監控、版本控制等更進階的正式環境管理，則需要 Container Orchestration 容器調度工具的協助；

在管理 Docker Container 集群上，Docker 有出 Docker-Swarm 與 Docker-Machine 來管理，但是看起來業界比較盛行 Kubernetes \(俗稱 k8s\)，後續探討 k8s 的應用。

