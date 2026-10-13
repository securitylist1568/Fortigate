# Ncloud Top-N 每日 00:00 歸零重置機制異常分析與交接報告

## 📌 1. 報告摘要 (Executive Summary)

* **調閱檔案**：[`ncloud_block_src_topn.txt`](file:///d:/NTNU-DevOps/Gitbash/Fortigate/ncloud_block_src_topn.txt) 及相關獨立 Job 輸出檔（`ncloud_block_src_ext_ssh_in.txt` 等）
* **回報問題**：今日（2026-10-13）截至 08:10，`ncloud_block_src_topn.txt` 累積筆數達 **22,790 行（有效 IP 數 22,782 筆）**，經查核確認**並未於今日 00:00 確實歸零重置**。
* **核心問題根因**：
  1. 系統自 **2026-10-08 成功歸零（重置為 77 筆）** 之後，已連續 5 天（10/09 ~ 10/13）於跨夜時段未觸發歸零重置，導致名單跨日持續滾動累加。
  2. 今日（10/13）的 22,782 筆中，**有 22,232 筆是昨日（10/12 23:50）遺留的歷史基底，今日 00:00~08:10 的真實當日增量僅約 550 筆**。
  3. 腳本內部以「跨夜 00:00:00 當下是否成功 Commit & Push」作為清空檔案的唯一時機。若 00:00 排程因網路波動、API 錯誤或無異動而跳過，當天後續排程（如 02:50）將直接載入昨日實體檔案（`seed_set_from_file`），使當日永久喪失歸零機會。

---

## 📊 2. 現象描述與 Git 歷史實證 (Git Evidence)

### 2.1 今日（2026-10-13）各 Commit 累積歷程
今日自 02:50 起至 08:10 止，Git Commit 歷史顯示名單是從 **22,237 筆** 開始微幅累加，絕非從 0 筆開始：

| 排程時間 (台北時間) | Commit Hash | 檔案總行數 | 有效 IP 數 | 當批淨增量 | 狀態說明 |
| :---: | :---: | :---: | :---: | :---: | :--- |
| **昨晚 10/12 23:50:13** | `a68cb7a56` | 22,240 | 22,232 | - | 昨晚最後一筆 Commit |
| **今日 10/13 00:00 ~ 02:40** | *(無)* | - | - | - | **跨夜時段完全無 Commit 產生** |
| **今日 10/13 02:50:10** | `c97653e32` | 22,245 | 22,237 | **+5 筆** | 今日首筆 Commit（繼承昨晚 22,232 筆底數） |
| **04:10:14** | `f3975397d` | 22,376 | 22,368 | +131 筆 | 穩定增量累加 |
| **07:00:10** | `45739528b` | 22,450 | 22,442 | +74 筆 | 穩定增量累加 |
| **07:10:13** | `438ca6edd` | 22,497 | 22,489 | +47 筆 | 穩定增量累加 |
| **07:20:09** | `52b8d2941` | 22,549 | 22,541 | +52 筆 | 穩定增量累加 |
| **07:30:13** | `bb867a0f1` | 22,599 | 22,591 | +50 筆 | 穩定增量累加 |
| **07:40:16** | `45dba65c8` | 22,646 | 22,638 | +47 筆 | 穩定增量累加 |
| **07:50:09** | `1959fac00` | 22,697 | 22,689 | +51 筆 | 穩定增量累加 |
| **08:00:13** | `11deab17a` | 22,744 | 22,736 | +47 筆 | 穩定增量累加 |
| **08:10:13** | `cce651396` | **22,790** | **22,782** | +46 筆 | 當前最新狀態 |

> 📌 **驗證小結**：今日純新增量為 `22,782 - 22,232 = 550 筆`。其餘 22,232 筆均為 10/12 以前之歷史殘存資料。

---

### 2.2 跨日連續未歸零的規律鐵證 (10/08 ~ 10/13)

經調閱過去數週每日「首筆 Commit」與「末筆 Commit」有效 IP 筆數，規律極度明確：

```
2026-10-08: 首筆 00:00:06 (7c60cafe6) -> 77 筆  [成功重置！刪除 4,817 筆] -> 末筆 23:50 達 6,337 筆
2026-10-09: 首筆 00:20:07 (906e5cf81) -> 6,368 筆  [未重置！延續昨日] -> 末筆 23:50 達 10,045 筆
2026-10-10: 首筆 03:10:14 (d732ab5f2) -> 10,144 筆 [未重置！延續昨日] -> 末筆 23:50 達 13,920 筆
2026-10-11: 首筆 05:10:16 (c97b2a395) -> 13,989 筆 [未重置！延續昨日] -> 末筆 23:50 達 18,298 筆
2026-10-12: 首筆 02:00:09 (5ad1969b0) -> 18,350 筆 [未重置！延續昨日] -> 末筆 23:50 達 22,232 筆
2026-10-13: 首筆 02:50:10 (c97653e32) -> 22,237 筆 [未重置！延續昨日] -> 截至 08:10 達 22,782 筆
```

* **結論規律**：
  * **凡是首筆 Commit 落在 `00:00:xx`**（如 10/08 00:00:06、10/07 00:00:07、10/04 00:00:07、09/30 00:00:14），筆數均瞬間降至 **30 ~ 77 筆**，歸零成功。
  * **凡是 00:00:xx 沒有 Commit**（首筆落在 00:20、02:00、03:10、05:10），當日便**完全不會歸零**，直接沿用昨日最終筆數繼續往上疊加。

---

## 🔍 3. 程式碼邏輯成因深入分析 (`Fortigate-Dev`)

請至 `Fortigate-Dev` 儲存庫檢查 [`ncloud_topn_feed_push_v4_3.py`](file:///d:/NTNU-DevOps/Gitbash/Fortigate-Dev/ncloud_topn_feed_push_v4_3.py)，核心缺陷位於以下區段：

### 缺陷一：跨夜時間窗邊界問題 (`window_today_to_now`)
* **程式碼位置**：`ncloud_topn_feed_push_v4_3.py` 約第 130 ~ 134 行
  ```python
  def window_today_to_now() -> Tuple[str, str]:
      now = datetime.now(TZ).replace(second=0, microsecond=0)
      start = now.replace(hour=0, minute=0, second=0, microsecond=0)
      return start.strftime("%Y-%m-%d %H:%M:%S"), now.strftime("%Y-%m-%d %H:%M:%S")
  ```
* **問題說明**：
  當 Crontab 於每日準時 `00:00:00` 執行時，`now` 與 `start` 皆為 `00:00:00`，產生的查詢參數為：
  `stime="2026-10-13 00:00:00"` 且 `etime="2026-10-13 00:00:00"`。
  此時呼叫 Ncloud API，API 常會回傳：
  `"result": "Error. Report does not exist. Please check the parameters and try again"`
  導致當次 `rows = []`，如果當下發生例外或無資料，該次排程即無法順利產生有效輸出。

---

### 缺陷二：狀態檔與實體檔備援（Seed）邏輯的脫節
* **程式碼位置**：`ncloud_topn_feed_push_v4_3.py` 約第 330 ~ 335 行
  ```python
  if sep_name and sep_name not in per_file_sets:
      per_file_sets[sep_name] = (
          set()
          if state.get(sep_name) != today_key
          else seed_set_from_file(CFG.repo_dir / sep_name)
      )
  ```
* **問題說明**：
  1. `state` 儲存於本機 JSON 檔（`state/ncloud_topn_state_v4_3.json`），其鍵值記錄檔案最後更新的日期 `today_key`（如 `"2026-10-12"`）。
  2. 若 `00:00` 執行失敗（例如因上述 API 錯誤或連線逾時），`save_state(state)` 未被呼叫，`state` 中仍保留舊日期 `"2026-10-12"`。
  3. 但到了下一輪（例如 `00:10` 或 `02:50`），如果 `atomic_write` 在寫入前從 Repo 執行了 `git.sync_hard()`：
     - 本地檔案被還原為遠端 `origin/main`（裡面是昨日的 22,232 筆）。
     - 若中途有任何邏輯先將 `state[sep_name]` 設為 `today_key`（見第 376 行 `state[fname] = today_key`），或者 `atomic_write` 比對失敗，則在隨後的重試或執行中，`state.get(sep_name) == today_key` 成立！
     - 一旦條件成立，腳本便呼叫 `seed_set_from_file(CFG.repo_dir / sep_name)`，**把昨天 2 萬多筆全數讀回記憶體**，導致當天徹底失去清空的機會！

---

### 缺陷三：`git.sync_hard()` 的還原副作用
* **程式碼位置**：`ncloud_topn_feed_push_v4_3.py` 約第 317 行與第 419 行
  ```python
  git.sync_hard()
  ```
* **問題說明**：
  `git.sync_hard()` 會執行 `git fetch` 與 `git reset --hard origin/main`。
  在跨夜首度嘗試重置時，本機試圖將檔案從 22,000 筆縮減至 50 筆。若在 `git push` 期間遭遇 GitHub 網路抖動或認證逾時，`except` 區塊會立刻觸發 `git.sync_hard()`，**將本地工作目錄強制刷回遠端包含 22,000 筆的狀態**。如果重試 3 次未果而拋出例外中斷，本地狀態便永久停留在昨日狀態。

---

## 🛠️ 4. 交接處理與查修建議事項 (Action Items for Fortigate-Dev)

前往 `Fortigate-Dev` 查閱原始碼時，建議針對以下 4 點進行架構重構或補強：

1. **強制跨日重置判定（Explicit Date Shift Reset）**：
   - 不應僅依賴 `00:00:00` 的瞬間觸發。
   - 改為**顯式比對當前日期與前次成功重置日期**：若 `today != last_reset_date`，無論當前是 `00:00`、`00:10` 還是 `02:50`，**強制無視任何現有檔案，將所有 `per_file_sets` 強制清空為 `set()`**，並記錄 `last_reset_date = today`。
2. **避免從舊檔案自動載入種子（Remove Dangerous Fallback Seed）**：
   - `seed_set_from_file` 在每日初次啟動時極具危險性。建議建立每日獨立快照或目錄，若確認為新的一天，嚴禁從昨天的 `origin/main` 檔案載入 `seed_set`。
3. **`window_today_to_now` 邊界防禦**：
   - 當 `now` 處於 `00:00 ~ 00:05` 時，`stime == etime` 會導致 API 回報找不到報表。應特別處理邊界值（例如 `etime` 至少加 1 秒，或在無資料時預設回傳空清單而不報錯）。
4. **Kali Linux 排程環境確認**：
   - 請檢查 Kali Linux 上的 Crontab 與 `/home/kali/bin/ncloud/logs/ncloud_topn_feed_push_v4_3.log`：
   - 查閱 10/09 ~ 10/13 每日 00:00:00 ~ 02:40 之間是否有紀錄 Python 例外錯誤或 Git Push 失敗日誌。
