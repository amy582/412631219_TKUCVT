# W01｜虛擬化概論、環境建置與 Snapshot 機制

## 環境資訊
Host OS：Windows 11
VM 名稱：vct-w01-412631219-new
Ubuntu 版本：Ubuntu 24.04.4 LTS
Docker 版本：Docker version 28.2.2
Docker Compose 版本：docker-compose version 1.29.2

## VM 資源配置驗證

| 項目 | VMware 設定值 | VM 內命令 | VM 內輸出 |
| CPU | 2 vCPU | lscpu | grep "^CPU(s)"| CPU(s): 2 |
| 記憶體 | 4 GB | `free -h \| grep Mem` | 3.8Gi|
| 磁碟 | 40 GB | `df -h /` | 40G |
| Hypervisor | VMware | `lscpu \| grep Hypervisor` | KVM |

## 四層驗收證據
- [x] ① Repository：`cat /etc/apt/sources.list.d/docker.list` 輸出
- [x] ② Engine：`dpkg -l | grep docker-ce` 輸出
- [x] ③ Daemon：`sudo systemctl status docker` 顯示 active
- [x] ④ 端到端：`sudo docker run hello-world` 成功輸出
- [x] Compose：`docker compose version` 可執行

## 容器操作紀錄
- [x] nginx：`sudo docker run -d -p 8080:80 nginx` + `curl localhost:8080` 輸出
- [x] alpine：`sudo docker run -it --rm alpine /bin/sh` 內部命令與輸出
- [x] 映像列表：`sudo docker images` 輸出

## Snapshot 清單

| 名稱 | 建立時機 | 用途說明 | 建立前驗證 |
|---|---|---|---|
| clean-baseline |未建立| 跳過，直接進行環境建置 | N/A|
| docker-ready | 2026-03-26 22:35 | Docker 安裝完成且測試通過 | 執行 hello-world 成功|

## 故障演練三階段對照

| 項目 | 故障前（基線） | 故障中（注入後） | 回復後 |
|---|---|---|---|
| docker.list 存在 | 是 | 否 | 是 |
| apt-cache policy 有候選版本 | 是 | 否 | 是 |
| docker 重裝可行 | 是 | 否 | 是 |
| hello-world 成功 | 是 | N/A | 是 |
| nginx curl 成功 | 是 | N/A | 是 |

## 手動修復 vs Snapshot 回復

| 面向 | 手動修復 | Snapshot 回復 |
|---|---|---|
| 所需時間 | 約 5 分鐘 | 約 30 秒 |
| 適用情境 | 知道哪裡改壞時 |完全不知道哪裡壞掉時 |
| 風險 | 可能修錯地方，導致環境更混亂| 會遺失從上次快照到現在之間的所有新資料 |

## Snapshot 保留策略
- 新增條件：每次安裝新軟體（如 Docker）或大改設定且驗證通過後
- 保留上限：最多保留 3 個節點，避免影響硬碟效能與過度佔用空間
- 刪除條件：新快照確認穩定後，刪除超過兩週或已過期的舊基線

## 最小可重現命令鏈
# 1. 驗證環境
sudo docker run hello-world
# 2. 模擬故障 (刪除 docker 執行檔)
sudo rm /usr/bin/docker
# 3. 驗證故障 (此時 docker 指令會失效)
docker --version
# 4. 執行回復 (在 VirtualBox 點選還原 Snapshot)
# 5. 再次驗證
sudo docker run hello-world

## 排錯紀錄
- 症狀：VirtualBox 無人值守安裝導致無法輸入密碼登入
- 診斷：確認為自動安裝預設密碼不一致或鍵盤驅動未載入
- 修正：重新建立 VM 並勾選「Skip Unattended Installation」，改用手動安裝
- 驗證：成功進入系統並設定自訂密碼 1234

## 設計決策
本週選擇使用 Oracle VirtualBox 替代 VMware Workstation，理由是 VirtualBox 安裝程序較精簡，且在個人電腦上無需複雜的 Broadcom 帳號審核即可快速部署實驗環境
