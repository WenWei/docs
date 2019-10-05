OpenVAS image for Docker
========================

以下關於 mikesplain/openvas 用到的指令請參考 `https://github.com/mikesplain/openvas-docker`

## 從 Docker Hub 下載安裝

https://hub.docker.com/r/mikesplain/openvas

1. 使用 docker run 指令執行，會自動下載 image 後啟動 container

```
docker run -d -p 443:443 -p 9390:9390 --name openvas mikesplain/openvas
```

如果是從另一台主機透過瀏覽器連到安裝服務的主機則需要額外設定名稱，`-e PUBLIC_HOSTNAME=example.com`

```
docker run -d -p 443:443 -p 9390:9390 -e PUBLIC_HOSTNAME=example.com --name openvas mikesplain/openvas
```

## 更新漏洞資訊

建議在每次掃描前先更新最新的漏洞資料庫

```
docker exec -it openvas bash

## inside container
# 同步 openvas 最新的漏洞資訊，資料存放在 /var/lib/openvas/plugins
greenbone-nvt-sync
# 重建 openvas 資料庫
openvasmd --rebuild --progress
# 同步 SCAP、CERT 到本機(同步時,一個 IP 只能一條 session,如果中斷再重連,有可能被鎖)
greenbone-certdata-sync
greenbone-scapdata-sync
openvasmd --update --verbose --progress

/etc/init.d/openvas-manager restart
/etc/init.d/openvas-scanner restart
```

## 開啟管理介面

https://127.0.0.1:443/

預設帳號和密碼
帳號：admin
密碼: admin

> 建議在啟動時就修改admin 帳號的預設密碼，透過 OV_PASSWORD 的環境變數更改，例如：
> `docker run -d -p 443:443 -e OV_PASSWORD=securepassword41 --name openvas mikesplain/openvas`

![登入頁](imgs/openvas_login_page.png "登入頁")


## 掃描單一目標 IP

主選單 Scans/Tasks

![工作頁](imgs/openvas_menu_scans_tasks.png "工作頁")

滑鼠移至「紫色小圖仙女棒」在展開的選項中選取「Task Wizard」

![工作精靈](imgs/openvas_tasks_page_taskwazard.png "工作精靈")

設定目標 IP 或主機名稱

![設定 IP 或 hostname](imgs/openvas_tasks_page_taskwazard_set_target.png "設定 IP 或 hostname")

啟動掃描 Start Scan

![啟動掃描](imgs/openvas_tasks_page_taskwazard_set_target_start.png)

![工作項清單](imgs/openvas_menu_scans_tasks_list.png)


## 掃描多個目標 IP

主選單 Configuration/Targets

![設定目標](imgs/openvas_menu_config_targets.png)

新增目標設定，滑鼠單擊左上角星號按鈕

![目標設定按鈕](imgs/openvas_targets_page_new_target.png)

開啟的目標設定對話方塊

![目標設定對話方塊](imgs/openvas_targets_page_new_target_dialog.png)

填入要掃描的相關資料

![目標設定輸入範圍](imgs/openvas_targets_page_new_target_dialog_input_target.png)

填寫完成按下「Create」

會在清單中看到剛才新建立的目標項目

![清單中的新目標](imgs/openvas_targets_page_new_target_add.png)

回到 Scans/Tasks

![Scan/Task](imgs/openvas_targets_page_new_target_back_tasks.png)

在左上星號圖示，選擇 New Task

![New Task](imgs/openvas_tasks_page_create_task.png)

開啟新工作對話方塊

![New Task Dialog](imgs/openvas_tasks_page_create_task_dialog.png)

填入新工作的資訊，這裡 Scan Targets 要選擇剛才建立的 `LAN Range` 這個目標

![input New Task Dialog](imgs/openvas_tasks_page_create_task_dialog_input.png)

填完後按下「Create」，工作清單中就能看到新增的工作項目

![Create Task](imgs/openvas_tasks_page_create_task_dialog_create.png)

剛建立好的工作(Task)項目狀態是 New，這時候要滑鼠點撃「Start」圖示啟動掃描

![Start Scan](imgs/openvas_tasks_page_create_task_start_scan.png)

## 查看掃描報告(Reports)

在工作項清單可以看到各個工作的狀態，在 Reports Total 欄位可以查看報告。

滑鼠點擊數字

![Report](imgs/openvas_report.png)

會開啟至報告頁 Scans/Reports

![Reports Page](imgs/openvas_report_page.png)

在下方的清單中選取掃描日期的報告

![By date](imgs/openvas_report_page_report_by_date.png)

就能夠看到掃出來的已知漏洞，再依各別項目查看詳細內容

![Vulnerability](imgs/openvas_report_page_date_vulnerability.png)


## OpenVAS Master-Slave

因為沒有最新版本，所以尚未測試

[openvas-8-docker](https://github.com/wcollani/openvas-8-docker)

參考：[Docker-based OpenVAS Scanning Cluster to Improve Scope Scalability](https://www.nopsec.com/docker-based-openvas-scanning-cluster-to-improve-scope-scalability/)