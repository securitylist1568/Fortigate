# Fortigate 資安防護清單與威脅情資自動化更新系統

## 📘 簡介

此儲存庫透過部署於 **Kali Linux** 上的 Python 自動化腳本，定期經由 **Crontab 排程** 執行，自動產生並更新各類威脅情資清單，再 Git Push 同步至本儲存庫。

自動化 Python 腳本原始碼與開發測試環境獨立託管於 [**`Fortigate-Dev` 儲存庫**](https://github.com/securitylist1568/Fortigate-Dev)。本儲存庫（`Fortigate`）專責作為 Fortigate 防火牆 **External Dynamic List (EDL / Threat Feeds)** 讀取之純文字情資發布端。

本儲存庫彙整用於 Fortigate 防火牆的所有 IP 封鎖清單、FQDN 封鎖清單、設備存取控制清單，以及 DNS 封鎖警告訊息頁面。

---

## 🔗 雙儲存庫架構對照 (Repository Architecture)

| 儲存庫名稱 | 角色與定位 | 主要內容 | 對應 GitHub 位址 |
|---|---|---|---|
| **`Fortigate`** *(本儲存庫)* | FortiGate EDL 實體 Feed 發布庫 | 產出之純文字封鎖清單 (`ncloud_block_*.txt` 等)、白名單 (`ncloud_allowlist.txt`)、人工維護檔 | [`securitylist1568/Fortigate`](https://github.com/securitylist1568/Fortigate) |
| **`Fortigate-Dev`** *(開發原始碼庫)* | 程式碼與自動化開發庫 | Python 主程式 (`v4.5`)、共用模組 (`ncloud_feed_utils.py`)、Job 設定檔、驗證腳本 (`verify_v4_5.sh`) | [`securitylist1568/Fortigate-Dev`](https://github.com/securitylist1568/Fortigate-Dev) |

---

## 🏗️ 系統架構

```
Kali Linux (自動化腳本 repo: Fortigate-Dev)
        │
        ├─ EmailSys_fortigate_block_v2_1.py    ← 郵件系統封鎖 IP 產生器 (v2.1)
        ├─ ncloud_fortigate_ntech_block_v2_1.py ← 封鎖清單產生器 (v2.1)
        └─ ncloud_topn_feed_push_v4_5.py        ← TopN 威脅情資推送器 (v4.5)
                │
                │  (Crontab 定期執行 + Git Auto Push)
                ▼
        本儲存庫：Fortigate (自動更新純文字 Feed)
                │
                ▼
        Fortigate 防火牆 (讀取 Raw URL 並套用封鎖規則)
```

---

## 📁 檔案清單說明與實時筆數 (Data Feeds)

### 🔴 外部威脅 IP 封鎖清單

| 檔案名稱 | 說明 | 來源腳本 | 現行有效筆數（約） |
|---|---|---|---|
| `g-BLACK_IP_Manual-v5.txt` | 手動維護之黑名單 IP，彙整各類惡意來源 | 人工維護 | ~3,830 |
| `ncloud_block_src_topn.txt` | 流量分析產生之每日 Top-N 威脅來源 IP（日歸零 + 10 分鐘增量累計） | `ncloud_topn_feed_push_v4_5.py` | ~39,100 |
| `ncloud_block_src_trojan_irc.txt` | 連線至 Trojan / IRC C2 伺服器之惡意來源 IP | `ncloud_topn_feed_push_v4_5.py` | ~5,300 |
| `ncloud_block_src_cert_remote.txt` | 使用可疑憑證進行遠端連線之來源 IP | `ncloud_topn_feed_push_v4_5.py` | ~4,100 |
| `ncloud_block_src_ext_ssh_in.txt` | 外部主機對內 SSH 入侵嘗試之來源 IP | `ncloud_topn_feed_push_v4_5.py` | ~29,600 |
| `ncloud_block_src_dns_ptr_anom.txt` | DNS PTR 記錄異常之可疑來源 IP | `ncloud_topn_feed_push_v4_5.py` | ~630 |
| `ncloud_block_dst_topn.txt` | 流量分析產生之 Top-N 威脅目標 IP | `ncloud_topn_feed_push_v4_5.py` | 動態（無威脅時 0 筆） |
| `ntech_blocklist.txt` | 分析產生之惡意 IP 封鎖清單 | `ncloud_fortigate_ntech_block_v2_1.py` | ~160 |
| `Fortianalyzer-abnormaly_140_122_ips.txt` | FortiAnalyzer 偵測之內部異常行為主機（140.122.x.x） | FortiAnalyzer 分析結果 | ~740 |
| `g-NTNU-Lib-ThreatSeed-v1` | 圖書館端點異常連線之威脅情資種子清單 | 人工維護 | ~290 |

### 🌐 FQDN / 網域封鎖清單

| 檔案名稱 | 說明 | 來源腳本 | 現行有效筆數（約） |
|---|---|---|---|
| `DNS-FQDN-Blocklist-v6.txt` | 主要 DNS FQDN 封鎖清單，含惡意網域、成人內容、釣魚/木馬 C2 網域，並包含萬用字元（`*.domain`）格式 | 人工維護 + 威脅情資 | ~1,890 |
| `ntech_blocklist_fqdn.txt` | 分析產生之惡意 FQDN 封鎖清單，主要包含掃描器主機 PTR 記錄 | `ncloud_fortigate_ntech_block_v2_1.py` | ~70 |

### 📧 郵件系統封鎖清單

| 檔案名稱 | 說明 | 來源腳本 | 現行有效筆數（約） |
|---|---|---|---|
| `emailsys_block_ipset.txt` | 郵件系統封鎖 IP/CIDR 集合（以 CIDR 格式為主），涵蓋高風險 IP 段 | `EmailSys_fortigate_block_v2_1.py` | ~2,360 |

### 🖥️ 內部設備控管清單

| 檔案名稱 | 說明 | 現行有效筆數（約） |
|---|---|---|
| `g-IOT_Block-v1.txt` | 內部 IoT 設備（140.122.x.x）限制對外存取之 IP 清單 | ~705 |
| `g-NetworkDevices_Block-v1.txt` | 內部網路設備（交換器、印表機等）對外 Web 服務封鎖清單 | ~107 |
| `g-ServerFarm-sshblocklist-v1.txt` | 伺服器農場（140.122.59.x / 63.x / 64.x / 65.x）SSH 對外封鎖清單 | ~44 |

### ✅ 允許清單與服務維護（Allowlist / Whitelist）

| 檔案名稱 | 說明 |
|---|---|
| `ncloud_allowlist.txt` | 全域實體允許清單，包含公用 DNS、GoogleBot CIDRs、校內重要 IP 與維護廠商 IP |
| `NTNU-FormalWebsites-v2.txt` | 正式對外網路服務伺服器 IP 清單（含 VPN、圖書館、行政單位、學術網站） |
| `NTNU-AcademicWebsites-v1.txt` | 學術網站伺服器 IP 清單，含各系所學術平台 |
| `NTNU-AcademicWebsites-CSIE-1.txt` | 資工系（CSIE）專屬學術網站伺服器 IP 清單 |
| `Security-UCP-maintenance-v1.txt` | 資安暨網路管理平台（UCP）維護清單 |

---

## ⚙️ 自動化腳本運作機制 (Python v4.5)

> **執行環境**：Kali Linux  
> **排程方式**：Crontab（`Asia/Taipei` 時區）

### 腳本功能對照表

| 腳本名稱 | 主要功能 | 更新的清單 |
|---|---|---|
| `ncloud_topn_feed_push_v4_5.py` | 每日 00:00:00 自動歸零重算 + 每 10 分鐘增量更新；全面套用 `ncloud_allowlist.txt` 與 CIDR 過濾，分析流量威脅 Job（`ext_to_int_threat_unblocked`、`int_to_ext_flow_ioc`、`trojan_irc_top`、`cert_remote_top`、`dns_ptr_anomaly`、`ext_to_int_ssh_inflow`） | `ncloud_block_src_topn.txt`、`ncloud_block_src_trojan_irc.txt`、`ncloud_block_src_cert_remote.txt`、`ncloud_block_src_ext_ssh_in.txt`、`ncloud_block_src_dns_ptr_anom.txt`、`ncloud_block_dst_topn.txt` |
| `ncloud_fortigate_ntech_block_v2_1.py` | 分析產生封鎖清單（IP + FQDN） | `ntech_blocklist.txt`、`ntech_blocklist_fqdn.txt` |
| `EmailSys_fortigate_block_v2_1.py` | 分析郵件系統封鎖統計，產生 CIDR 格式封鎖清單 | `emailsys_block_ipset.txt` |

---

## 🛠️ 工具與查詢腳本 (Utility Scripts)

本系統提供獨立之正規化 Python 工具腳本，方便維護人員隨時調閱威脅情資歷史狀態與統計：

### IP 威脅情資歷史調閱工具 (`query_ip_history.py`)

- **功能描述**：支援單一或多個 IP 位址，高速查詢其在過去 N 天（預設 7 天）內落入 Fortigate 黑名單清單（如 `ncloud_block_src_topn.txt`、`ncloud_block_src_ext_ssh_in.txt` 等）的每日封鎖次數統計、當前 HEAD 最新狀態，以及全 Git 歷史異動紀錄 (`git log -S`)。
- **使用範例**：
  ```bash
  # 查詢單一 IP 過去 7 天紀錄
  python query_ip_history.py 60.251.148.94

  # 查詢多個 IP 過去 7 天紀錄
  python query_ip_history.py 35.201.232.104 35.201.248.254 220.130.189.210 --days 7

  # 查詢指定 IP 過去 14 天紀錄
  python query_ip_history.py --ips 123.192.200.16 211.75.221.136 --days 14
  ```

---

## 📋 `ncloud_block_src_topn.txt` 產生標頭規範

產出的文字檔標頭上記錄真實的產生參數，範例如下：

```
# ncloud_block_src_topn.txt
# Generated by ncloud_topn_feed_push_v4_5.py
# Mode: daily recompute (00:00~now) UNION 10min incremental per-job union (10m)
# TZ: Asia/Taipei
# Jobs: ext_to_int_threat_unblocked, int_to_ext_flow_ioc, trojan_irc_top, cert_remote_top, dns_ptr_anomaly, ext_to_int_ssh_inflow
# Note: main src = daily_topn_src(00:00~now) UNION incremental_union
# Rules: public IPv4 only; exclude_cidrs=140.122.0.0/16; allowlist_ips=1.0.0.1,1.1.1.1,111.125.135.25,111.235.251.250,149.112.112.112,208.67.220.220,208.67.222.222,210.203.56.199,211.75.221.136,8.4.224.250,8.8.4.4,8.8.8.8,9.9.9.9; allowlist_cidrs=(none); allowlist_file=/home/kali/bin/ncloud/ncloud_allowlist.txt
#
```

---

## 🆕 v4.5 現行版本重點特性 (Python v4.5 Upgrade)

1. **跨平台換行符規範化與防禦性比對（Line Ending & Diff Normalization）**：
   - 在 `atomic_write` 與 `write_with_header` 比對實體檔案內容時強制標準化換行符（`.replace('\r\n', '\n')`），徹底根除 Windows CRLF 與 Linux LF 格式混用導致的假性檔案異動（False Changed Trigger）。
2. **精確 Git 暫存與推送診斷（Accurate Git Delta & Push Diagnostics）**：
   - 在 `GitOps.commit_push` 明確偵測無實質暫存變更（`git diff --cached --quiet` 回傳 0）時，精準輸出日誌 `No git delta staged; skip commit and push.`。
   - 主流程嚴格區分真實推送（`pushed=True`）與無變更略過狀態，杜絕未有 Git Push 卻誤報「State successfully committed」之假成功現象。
3. **錯開每 10 分鐘排程避免並行競爭（Staggered Crontab Execution）**：
   - 建議 Crontab 排程設定為 `2-59/10 * * * *`（如每小時 02, 12, 22 分），與 `ncloud_fortigate_ntech_block_v2_1.py` 的整點 00, 10, 20 分錯開 2 分鐘，徹底防杜雙程式同時競爭 Git 倉庫鎖與 `git.sync_hard()` 強制重置干擾。
4. **跨日絕對歸零重置機制（Explicit Date Shift Reset）**：
   - 引入 `last_reset_date` 顯式日期狀態追蹤。腳本每次啟動時比對目前台北日期與前次成功重置日期；一旦偵測到跨日（`last_reset_date != today_key`），**強制將所有累計集合初始化為空集合，嚴禁自本機檔案或遠端快照載入歷史種子（Remove Fallback Seed）**。
5. **防禦性跨夜時間窗（Defensive Time Window Guard）**：
   - 在 `window_today_to_now()` 增加邊界防禦：當處於跨夜 `00:00:00` 當下時（`now <= start`），自動將 `etime` 微調為 `start + 1s`，徹底防範因 `stime == etime` 導致 Ncloud API 拋出 `"Report does not exist"` 錯誤。
   - 在 `window_last_minutes()` 增量查詢中，將起始時間錨定於當日 `00:00:00`，杜絕跨夜時查詢範圍回溯至前日 `23:50` 所造成的歷史舊資料滲漏。
6. **原子性狀態提交（Atomic State Commit）**：
   - 狀態檔（`state/ncloud_topn_state_v4_5.json`）之日期標記嚴格綁定於 Git Commit & Push 成功後才寫入。
7. **`verify_v4_5.sh` 自動化全套驗證**：
   - 包含 Step 0 ~ Step 5 的全自動校驗腳本，嚴格確保白名單 IP 不會誤入任何封鎖清單，並校驗狀態檔與排程相容性。

---

## 📌 注意事項

- **自動更新**：本儲存庫由 Kali Linux Crontab 自動更新，**請勿手動修改自動產生的清單**（如 `ncloud_block_src_*.txt`、`ntech_blocklist.txt`、`emailsys_block_ipset.txt`），否則下次排程執行時將被覆蓋。
- **手動維護清單**：`g-BLACK_IP_Manual-v5.txt`、`DNS-FQDN-Blocklist-v6.txt`、`g-IOT_Block-v1.txt` 等為人工維護清單，可按需調整。
- **內部 CIDR 與白名單過濾**：自動化腳本已排除內部 IP 段（如 `140.122.0.0/16`）及白名單檔案 (`ncloud_allowlist.txt`) / 環境變數中指定的 IP 與 CIDR，避免誤封重要服務。
- **Cron 執行環境**：腳本使用 `Asia/Taipei` 時區，所有日誌與時間戳記均以台北時間記錄。