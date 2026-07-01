# 基於 AWS 雲端環境之 FPS 遊戲伺服器防禦、資安職責分離與惡意行為監控自動化系統

本專案以經典多人連線遊戲 Counter-Strike 1.6 (CS 1.6) 為實驗場景，建置了一套雲端原生 SecOps 自動化監控與防禦架構。針對公開伺服器面臨的拒絕服務攻擊 (DoS/DDoS) 痛點，達成秒級觸發自動化網路層剔除，並導入企業級 IAM 職責分離與 FinOps 財務維運治理。

## 專題核心痛點與解決方案
- **痛點**：惡意攻擊者透過工具發送密集 UDP 流量（Port 27015），導致遊戲伺服器高延遲（Lag）甚至斷線（Crash）。
- **解法**：部署 AWS 雲端原生監控，配合 CloudWatch Alarms 與本機自動化防禦腳本，在指標衝破閾值時，**1 秒內自動生成 iptables 剛性防禦規則**，實現秒級阻斷。

## 系統架構拓撲

![AWS Architecture](Network Architecture.png)

## 核心功能展示流程 (Demo Steps)

1. **環境初始化與監控準備**：
   - 監控端啟動 `watch -n 1 iptables -L -n -v` 即時觀看內核防火牆變化。
   - 監控控制台開啟 **AWS CloudWatch Alarms** 監控 27015 埠口流量。

2. **發動密集連線攻擊**：
   - 模擬黑客利用地端工具對 EC2 遊戲伺服器發送異常密集連線。
   - 伺服器端流量指標瞬間飆升。

3. **秒級自動化網路層剔除**：
   - CloudWatch Alarms 燈號精準跳轉成紅色 **🔴 In alarm**。
   - 觸發 SOAR 自動化防禦機制，Linux 內核在 1 秒內自動生成防禦規則：
     ```bash
     1 REJECT udp -- <攻擊者IP> 0.0.0.0/0 udp dpt:27015 reject-with icmp-port-unreachable
     ```
   - 攻擊流量戛然而止，成功保障遊戲伺服器連線品質。

4. **資安審計與數位鑑識**：
   - 即使本機日誌遭破壞，雲端完好保留不可篡改的攻擊者 IP 與高精度時間線（Timeline），提供完善的事後審計。

## 企業級 IAM 資安職責分離
本系統嚴格遵循「最小權限原則」，將運維、資安與財務權限切分：
- **Game-Server-Admin (遊戲維運主管)**：僅具備管理 EC2 與遊戲伺服器權限。
- **Security-Analyst (資安分析師)**：僅具備讀取 CloudWatch Logs 與資安審計權限，無權限變動基礎設施。
- **FinOps-Manager (財務運維主管)**：僅具備 AWS Budgets 預算管控與帳務盤查權限。

## 前瞻 FinOps 治理 (MCP 整合)
專案前瞻性地整合 **Model Context Protocol (MCP)** 架構，高階主管不需登入複雜的 AWS 帳務後台，即可透過圖形化 AI 運維大腦，以**自然語言（如：幫我盤查當月雲端帳務與預算）**即時獲取雲端實驗環境的帳務預測與超額告警。
