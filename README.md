# Fortigate DNS FQDN 阻擋清單與網絡安全配置

## 📘 簡介

此儲存庫提供 Fortigate 防火牆用的 DNS 與網絡安全工具與配置檔案。  
重點功能為加強惡意網域的阻擋能力、管理物聯網（IoT）與網絡設備的訪問控制，打造安全穩定的網絡環境。

---

## ✨ 功能特色

- **DNS FQDN 阻擋清單**：多版本阻擋清單可依需求靈活應用。 (DNS-FQDN-Blocklist-vXX)
- **網絡黑名單管理**：包含手動維護的 IP 阻擋清單。 (g-BLACK_IP_Manual-vXX)
- **IoT 與網絡設備限制**：專用清單限制不必要的設備存取。(g-IOT_Block-vXX & g-NetworkDevices_Block-vXX)
- **可持續更新**：根據威脅情資可彈性擴充阻擋清單。

---

## 🚀 快速入門

### ✅ 需求條件

1. 支援 DNS 過濾功能的 Fortigate 設備  
2. 防火牆管理員權限

### 🔧 使用方式

1. **Clone 儲存庫**
   ```bash
   git clone https://github.com/securitylist1568/Fortigate.git
   cd Fortigate