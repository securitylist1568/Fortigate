Fortigate DNS FQDN 阻擋清單與網絡安全配置
簡介
此儲存庫為 Fortigate 防火牆 提供一系列針對 DNS 和網絡安全的工具與配置文件，重點在於加強惡意網域的阻擋能力，並管理物聯網（IoT）與網絡設備的訪問控制，打造安全穩定的網絡環境。

功能特色
DNS FQDN 阻擋清單：多版本阻擋清單可根據需求靈活應用。
網絡黑名單管理：包含手動維護的 IP 阻擋清單。
物聯網與網絡設備限制：專用阻擋清單管理 IoT 與網絡設備的訪問規則。
可持續更新：阻擋清單可以根據威脅動態輕鬆擴展。
快速入門
需求條件
支援 DNS 過濾的 Fortigate 設備。
防火牆的管理員權限。
使用方式
克隆儲存庫：

git clone https://github.com/securitylist1568/Fortigate.git
cd Fortigate
檢視與自定義阻擋清單：

根據需求修改以下清單文件：
DNS-FQDN-Blocklist-v3.txt：最新的域名阻擋清單。
g-BLACK_IP_Manual-v1.txt：手動維護的 IP 黑名單。
g-IOT_Block-v1.txt：針對物聯網設備的阻擋清單。
g-NetworkDevices_Block-v1.txt：針對網絡設備的阻擋清單。
匯入清單至 Fortigate：

根據 Fortinet 官方文檔 的指引匯入。
套用防火牆策略：

根據阻擋需求將清單應用於相應的策略。
檔案結構
DNS-FQDN-Blocklist-v3.txt：包含最新的 DNS FQDN 阻擋清單。
g-BLACK_IP_Manual-v1.txt：手動維護的 IP 黑名單，用於阻擋高風險或惡意 IP。
g-IOT_Block-v1.txt：阻擋物聯網設備的訪問規則。
g-NetworkDevices_Block-v1.txt：限制網絡設備訪問的清單。
貢獻指南
歡迎社群成員的貢獻！可以提交問題（Issues）或提出合併請求（Pull Requests），幫助改進清單或增加功能。
