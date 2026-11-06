# Agent Behavioral Rules & Standard Operating Procedures

## 🔄 雙儲存庫說明文件同步規範 (Dual Repository Documentation Sync Rule)

1. **雙庫說明同步強制要求 (Mandatory Dual-Repo Sync)**：
   每次在 `Fortigate-Dev`（開發原始碼庫）進行版號變更、程式邏輯調整、輸出檔案增減或參數異動時，**必須同時更新兩邊儲存庫的 README.md 說明文件**：
   - `d:\NTNU-DevOps\Gitbash\Fortigate-Dev\README.md`
   - `d:\NTNU-DevOps\Gitbash\Fortigate\README.md`

2. **內容真實性與一致性檢核 (Consistency Verification)**：
   - `Fortigate/README.md` 必須 100% 正確對應 `Fortigate-Dev` 最新版腳本（如 `v4.5` 與 `LongTerm v1.0.1`）產出之清單名稱、實態有效筆數、標頭規範與運作邏輯。
   - 嚴禁在發布庫 `Fortigate` 留下過時的腳本版本引用（如舊版 `v4.2`、`v4.3` 或舊 `TTL=90d`）或舊筆數快照。

3. **Git Push 遠端同步關卡 (Git Push Verification)**：
   - 說明文件更新後，必須執行 Git Push 並確認 `securitylist1568/Fortigate-Dev` 與 `securitylist1568/Fortigate` 的 GitHub 遠端 `origin/main` 均推送成功。
   - 必須執行 `git status` 確保顯示 `Your branch is up to date with 'origin/main'`。

4. **所有 *.md 說明文件與 Ground Rules 同步審查 (All Markdown Documentation & Ground Rules Sync)**：
   - 每次有任何維護清單主機異動（如 Security-UCP-maintenance-v1.txt 新增 IP）、清單筆數累積跨入新數量級（如 g-BLACK_IP_AutoLongTerm-v1.txt 破 5 萬筆）、或排程腳本輸出規則微調時，**必須即時檢閱並更新相關 *.md 說明文件**（包括 README.md、walkthrough.md 等），確保說明文件中所列之清單筆數快照、主機說明與範例標頭完全符合真實環境現況。
   - 所有 AI Agent 在執行任務時，必須將「檢視並同步雙庫 *.md 文件與 Ground Rules」列為常態標準作業流程（SOP），嚴禁僅更新資料檔案而遺漏技術與管理說明文件。
