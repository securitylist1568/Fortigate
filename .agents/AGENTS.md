# Agent Behavioral Rules & Standard Operating Procedures

## 🔄 雙儲存庫說明文件同步規範 (Dual Repository Documentation Sync Rule)

1. **雙庫說明同步強制要求 (Mandatory Dual-Repo Sync)**：
   每次在 `Fortigate-Dev`（開發原始碼庫）進行版號變更、程式邏輯調整、輸出檔案增減或參數異動時，**必須同時更新兩邊儲存庫的 README.md 說明文件**：
   - `d:\NTNU-DevOps\Gitbash\Fortigate-Dev\README.md`
   - `d:\NTNU-DevOps\Gitbash\Fortigate\README.md`

2. **內容真實性與一致性檢核 (Consistency Verification)**：
   - `Fortigate/README.md` 必須 100% 正確對應 `Fortigate-Dev` 最新版腳本（如 `v4.3`）產出之清單名稱、實態有效筆數、標頭規範與運作邏輯。
   - 嚴禁在發布庫 `Fortigate` 留下過時的腳本版本引用（如舊版 `v4.2`）或舊筆數快照。

3. **Git Push 遠端同步關卡 (Git Push Verification)**：
   - 說明文件更新後，必須執行 Git Push 並確認 `securitylist1568/Fortigate-Dev` 與 `securitylist1568/Fortigate` 的 GitHub 遠端 `origin/main` 均推送成功。
   - 必須執行 `git status` 確保顯示 `Your branch is up to date with 'origin/main'`。
