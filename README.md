# Fortigate 資安防護清單與威脅情資自動化更新系統

## 📘 簡介

此儲存庫透過部署於 **Kali Linux** 上的 Python 自動化腳本，定期經由 **Crontab 排程** 執行，自動產生並更新各類威脅情資清單，再 Git Push 同步至本儲存庫。

自動化 Python 腳本原始碼與開發測試環境獨立託管於 [**`Fortigate-Dev` 儲存庫**](https://github.com/securitylist1568/Fortigate-Dev)。本儲存庫（`Fortigate`）專責作為 Fortigate 防火牆 **External Dynamic List (EDL / Threat Feeds)** 讀取之純文字情資發布端。

本儲存庫匯整用於 Fortigate 防火牆的所有 IP 封鎖清單、FQDN 封鎖清單、設備存取控制清單，以及 DNS 封鎖警告訊息頁面。

---

## 🔗 雙儲存庫架構對照 (Repository Architecture)

| 儲存庫名稱 | 角色與定位 | 主要內容 | 對應 GitHub 位址 |
|---|---|---|---|
| **`Fortigate`** *(本儲存庫)* | FortiGate EDL 實體 Feed 發布庫 | 產出之純文字封鎖清單 (`ncloud_block_*.txt` 等)、白名單 (`ncloud_allowlist.txt`)、人工維護檔 | [`securitylist1568/Fortigate`](https://github.com/securitylist1568/Fortigate) |
| **`Fortigate-Dev`** *(開發原始碼庫)* | 程式碼與自動化開發庫 | Python 主程式 (`v4.3`)、共用模組 (`ncloud_feed_utils.py`)、Job 設定檔、驗證腳本 (`verify_v4_3.sh`) | [`securitylist1568/Fortigate-Dev`](https://github.com/securitylist1568/Fortigate-Dev) |

---

## 🏗️ 系統架構

```
Kali Linux (自動化腳本 repo: Fortigate-Dev)
        │
        ├─ EmailSys_fortigate_block_v2_1.py    ← 郵件系統封鎖 IP 產生器 (v2.1)
        ├─ ncloud_fortigate_ntech_block_v2_1.py ← 封鎖清單產生器 (v2.1)
        └─ ncloud_topn_feed_push_v4_3.py        ← TopN 威脅情資推送器 (v4.3)
                │
                │  (Crontab 定期執行 + Git Auto Push)
                ▼
        本儲存庫：Fortigate (自動更新純文字 Feed)
                │
                ▼
        Fortigate 防火牆 (讀取 Raw URL 並套用封鎖規則)
```

---

## 📁 檔案清單說明

### 🔴 外部威脅 IP 封鎖清單

| 檔案名稱 | 說明 | 來源 | 筆數（約） |
|---|---|---|---|
| `g-BLACK_IP_Manual-v5.txt` | 手動維護之黑名單 IP，彙整各類惡意來源 | 人工維護 | ~4,370 |
| `ncloud_block_src_topn.txt` | 流量分析產生之每日 Top-N 威脅來源 IP | `ncloud_topn_feed_push_v4_3.py` 自動生成 | ~3,437 |
| `ncloud_block_src_trojan_irc.txt` | 連線至 Trojan / IRC C2 伺服器之惡意來源 IP | `ncloud_topn_feed_push_v4_3.py` 自動生成 | ~1,561 |
| `ncloud_block_src_cert_remote.txt` | 使用可疑憑證進行遠端連線之來源 IP | `ncloud_topn_feed_push_v4_3.py` 自動生成 | ~1,099 |
| `ncloud_block_src_ext_ssh_in.txt` | 外部主機對內 SSH 入侵嘗試之來源 IP | `ncloud_topn_feed_push_v4_3.py` 自動生成 | ~801 |
| `ncloud_block_src_dns_ptr_anom.txt` | DNS PTR 記錄異常之可疑來源 IP | `ncloud_topn_feed_push_v4_3.py` 自動生成 | ~180 |
| `ncloud_block_dst_topn.txt` | 流量分析產生之 Top-N 威脅目標 IP | `ncloud_topn_feed_push_v4_3.py` 自動生成 | 動態（視威脅情況） |
| `ntech_blocklist.txt` | 分析產生之惡意 IP 封鎖清單 | `ncloud_fortigate_ntech_block_v2_1.py` 自動生成 | ~391 |
| `Fortianalyzer-abnormaly_140_122_ips.txt` | FortiAnalyzer 偵測之內部異常行為主機（140.122.x.x） | FortiAnalyzer 分析結果 | ~743 |
| `g-NTNU-Lib-ThreatSeed-v1` | 圖書館端點異常連線之威脅情資種子清單 | 人工維護 | ~294 |

### 🌐 FQDN / 網域封鎖清單

| 檔案名稱 | 說明 | 來源 | 筆數（約） |
|---|---|---|---|
| `DNS-FQDN-Blocklist-v6.txt` | 主要 DNS FQDN 封鎖清單，含惡意網域、成人內容、釣魚/木馬 C2 網域，並包含萬用字元（`*.domain`）格式 | 人工維護 + 威脅情資 | ~1,880 |
| `ntech_blocklist_fqdn.txt` | 分析產生之惡意 FQDN 封鎖清單，主要包含掃描器主機 PTR 記錄 | `ncloud_fortigate_ntech_block_v2_1.py` 自動生成 | ~229 |

### 📧 郵件系統封鎖清單

| 檔案名稱 | 說明 | 來源 | 筆數（約） |
|---|---|---|---|
| `emailsys_block_ipset.txt` | 郵件系統封鎖 IP/CIDR 集合（以 CIDR 格式為主），涵蓋高風險 IP 段 | `EmailSys_fortigate_block_v2_1.py` 自動生成 | ~613 |

### 🖥️ 內部設備控管清單

| 檔案名稱 | 說明 |
|---|---|
| `g-IOT_Block-v1.txt` | 內部 IoT 設備（140.122.x.x）限制對外存取之 IP 清單 |
| `g-NetworkDevices_Block-v1.txt` | 內部網路設備（交換器、印表機等）對外 Web 服務封鎖清單，含英語學系設備、光電所印表機 |
| `g-ServerFarm-sshblocklist-v1.txt` | 伺服器農場（140.122.59.x / 63.x / 64.x / 65.x）SSH 對外封鎖清單 |

### ✅ 內部服務允許清單（Allowlist / Whitelist）

| 檔案名稱 | 說明 |
|---|---|
| `ncloud_allowlist.txt` | 全域允許清單，包含 DNS、GoogleBot CIDRs、校內重要 IP 與維護廠商 IP |
| `NTNU-FormalWebsites-v2.txt` | 正式對外網路服務伺服器 IP 清單（含 VPN、圖書館、各行政單位、學術網站，以 `#URL` 批註對應服務） |
| `NTNU-AcademicWebsites-v1.txt` | 學術網站伺服器 IP 清單，含各系所學術平台（心測中心、地球科學系、圖書館、資工系、電資學院等） |
| `NTNU-AcademicWebsites-CSIE-1.txt` | 資工系（CSIE）專屬學術網站伺服器 IP 清單 |
| `Security-UCP-maintenance-v1.txt` | 資安暨網路管理平台（UCP）維護清單，標示暫時封鎖之服務 IP |

### 🖼️ 其他工具檔案

| 檔案名稱 | 說明 |
|---|---|
| `DNS_block_Message-v2.html` | DNS 封鎖警告頁面（繁體中文），當使用者存取被封鎖網域時由 Fortigate 顯示 |

---

## ⚙️ 自動化腳本運作方式

> **執行環境**：Kali Linux  
> **排程方式**：Crontab（`Asia/Taipei` 時區）

### 腳本功能對照表

| 腳本名稱 | 主要功能 | 更新的清單 |
|---|---|---|
| `ncloud_topn_feed_push_v4_3.py` | 每日 00:00:00 自動歸零重算 + 每 10 分鐘增量更新；全面套用 `ncloud_allowlist.txt` 與 CIDR 過濾，分析流量威脅 Job（`ext_to_int_threat_unblocked`、`int_to_ext_flow_ioc`、`trojan_irc_top`、`cert_remote_top`、`dns_ptr_anomaly`、`ext_to_int_ssh_inflow`） | `ncloud_block_src_topn.txt`、`ncloud_block_src_trojan_irc.txt`、`ncloud_block_src_cert_remote.txt`、`ncloud_block_src_ext_ssh_in.txt`、`ncloud_block_src_dns_ptr_anom.txt`、`ncloud_block_dst_topn.txt` |
| `ncloud_fortigate_ntech_block_v2_1.py` | 分析產生封鎖清單（IP + FQDN） | `ntech_blocklist.txt`、`ntech_blocklist_fqdn.txt` |
| `EmailSys_fortigate_block_v2_1.py` | 分析郵件系統封鎖統計，產生 CIDR 格式封鎖清單 | `emailsys_block_ipset.txt` |

### 自動更新流程

```bash
# 腳本執行完成後自動 Git Push
git -C /path/to/Fortigate add -A
git -C /path/to/Fortigate commit -m "feed(py v4.3): union(daily topn + 10m incremental) allowlist-file+env Asia/Taipei"
git -C /path/to/Fortigate push origin main
```

---

## 📋 `ncloud_block_src_topn.txt` 產生規則

檔案標頭記錄完整的產生參數，例如：

```
# ncloud_block_src_topn.txt
# Generated by ncloud_topn_feed_push_v4_3.py
# Mode: daily recompute (00:00~now) UNION 10min incremental per-job union (10m)
# TZ: Asia/Taipei
# Jobs: ext_to_int_threat_unblocked, int_to_ext_flow_ioc, trojan_irc_top,
#        cert_remote_top, dns_ptr_anomaly, ext_to_int_ssh_inflow
# Note: main src = daily_topn_src(00:00~now) UNION incremental_union
# Rules: public IPv4 only; exclude_cidrs=140.122.0.0/16;
#        allowlist_ips=1.0.0.1,1.1.1.1,149.112.112.112,208.67.220.220,
#                     208.67.222.222,8.8.4.4,8.8.8.8,9.9.9.9,111.125.135.25,211.75.221.136...;
#        allowlist_cidrs=64.233.160.0/19,74.125.0.0/16...;
#        allowlist_file=/home/kali/bin/ncloud/ncloud_allowlist.txt
```

---

## 🆕 v4.3 版本重點更新說明 (Python Script Upgrade)

1. **00:00:00 每日重置與狀態同步解耦（State Sync Fix）**：
   - 修正 v4.2 狀態寫入早於 `git push` 導致網路波動時錯失 00:00 重置的 Bug。
   - 在 v4.3 中，`save_state(state)` 調整至 Git 推送成功後才執行。確保每日 00:00:00 準時重置歸零 `ncloud_block_src_topn.txt`，並在後續每 10 分鐘穩定進行增量累計。
2. **`separate_output` 獨立 Job 全面 Allowlist 過濾**：
   - 所有獨立 Job 清單檔案（如 `ncloud_block_src_trojan_irc.txt`、`ncloud_block_src_cert_remote.txt`、`ncloud_block_src_ext_ssh_in.txt`、`ncloud_block_src_dns_ptr_anom.txt`）均全面導入 `is_allowed_ipv4` 白名單校驗。
3. **實體動態白名單檔 `ncloud_allowlist.txt` 支援**：
   - 腳本支援透過 `ncloud_allowlist.txt` 檔案及環境變數動態解析 Allowlist IP 與 CIDR 網段（已包含 GoogleBot 全球 CIDRs、校內與維護廠商特定 IP）。
4. **`verify_v4_3.sh` 自動化全套驗證**：
   - 提供 0~5 步驟全自動校驗腳本，涵蓋白名單不誤擋檢查、跨日重置歸零與 10 分鐘增量累計驗證。

---

## 🚀 快速入門

### ✅ 需求條件

1. 支援 DNS 過濾功能的 Fortigate 設備
2. 防火牆管理員權限
3. Fortigate 可存取 GitHub Raw 內容 URL（或透過內部 Proxy）

### 🔧 使用方式

**1. 在 Fortigate 設定外部威脅情資（External Threat Feed）**

於 Fortigate 管理介面 → **Security Fabric** → **External Connectors** 建立 Threat Feed，貼入對應清單的 GitHub Raw URL。

**2. 設定 Fortigate DNS Filter（DNS FQDN 封鎖）**

將 `DNS-FQDN-Blocklist-v6.txt`（含萬用字元格式）匯入 DNS Filter 設定，搭配 `DNS_block_Message-v2.html` 作為封鎖回應頁面。

**3. 設定防火牆政策（Firewall Policy）**

建立防火牆政策，將各 IP 封鎖清單（`ncloud_block_src_*.txt`、`g-BLACK_IP_Manual-v5.txt` 等）套用為來源 / 目標位址群組，執行拒絕（Deny）動作。

---

## 🔒 封鎖類別說明

| 類別 | 封鎖項目 |
|---|---|
| **惡意程式 / C2** | Trojan IRC、Cert Remote 隧道、惡意網域 |
| **入侵嘗試** | SSH 暴力破解、外部掃描器 |
| **DNS 威脅** | 釣魚網域、惡意重定向、C2 FQDN |
| **成人 / 違規內容** | 色情、博弈相關網域（DNS FQDN 清單） |
| **內部異常設備** | IoT 設備、網路設備、FortiAnalyzer 偵測之異常主機 |
| **郵件垃圾 / 詐騙** | 全球高風險 IP 段（emailsys_block_ipset） |

---

## 📌 注意事項

- **自動更新**：本儲存庫由 Kali Linux Crontab 自動更新，**請勿手動修改自動產生的清單**（如 `ncloud_block_src_*.txt`、`ntech_blocklist.txt`、`emailsys_block_ipset.txt`），否則下次排程執行時將被覆蓋。
- **手動維護清單**：`g-BLACK_IP_Manual-v5.txt`、`DNS-FQDN-Blocklist-v6.txt`、`g-IOT_Block-v1.txt` 等為人工維護清單，可按需調整。
- **內部 CIDR 與白名單過濾**：自動化腳本已排除內部 IP 段（如 `140.122.0.0/16`）及白名單檔案 (`ncloud_allowlist.txt`) / 環境變數中指定的 IP 與 CIDR，避免誤封重要服務。
- **Cron 執行環境**：腳本使用 `Asia/Taipei` 時區，所有日誌與時間戳記均以台北時間記錄。