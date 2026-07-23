# Memory Reference Code 與 DRAM Training

> 狀態：Draft
> 文件用途：本檔為章節撰寫大綱，內容、規格版本與平台資料仍需由章節負責人補充及驗證。

## 1. 文件目的
- 說明「Memory Reference Code 與 DRAM Training」在 平台與矽晶片初始化 中的角色。
- 定義本章涵蓋範圍、非涵蓋範圍與預期讀者。
- 提供設計、移植、驗證及問題排查時可重複使用的參考架構。

## 2. 建議先備知識
- 釐清平台、Silicon 與 Board 層的責任邊界
- 整理硬體初始化相依順序與必要前置條件
- 建立可重複使用的 Bring-up 驗證方式

## 3. 建議內容大綱
### 3.1 SPD、Topology 與記憶體參數來源
### 3.2 MRC/FSP-M 介面與輸入 Policy
### 3.3 JEDEC 初始化與 Training 階段
### 3.4 Training Data Cache 與 Fast Boot
### 3.5 ECC、RAS、Memory Map 與保留區

- ECC Mode、Error Threshold、Scrub Policy 與錯誤回報路徑
- Memory Map、Reserved Range、NUMA 與 OS 可見記憶體

### 3.6 Memory Sparing、Mirroring 與 Address Range RAS（選用）

- Rank／Bank／Channel Sparing 的能力偵測與 Policy
- Memory Mirroring 的通道配對、容量折減與 Address Mapping
- Spare Activation、Failover、Degraded Mode 與事件回報
- Patrol Scrub、Demand Scrub、Memory Poison 與 Page Offlining 的分工
- SMBIOS、ACPI SRAT／HMAT、APEI 與 BMC Inventory／SEL 的資訊一致性
- 錯誤注入、Sparing 切換、Mirroring Failover 與重開機持續性驗證
- 僅適用於具備對應 Memory Controller 能力的高階伺服器平台

### 3.7 冷開機、暖開機與不同 Reset Path

### 3.8 Training 失敗資訊、邊界條件與降速策略

## 4. 建議圖表與資料
- 架構圖或模組關係圖
- 初始化時序圖、狀態圖或資料流圖
- 關鍵設定、PCD、Variable、Register 或 Table 欄位表
- 正常與異常 Log 對照
- 平台差異或版本差異比較

## 5. 驗證與測試重點
- 定義測試環境、韌體版本、硬體版本及必要工具。
- 列出正常流程、邊界條件、錯誤注入及復原測試。
- 記錄可判定 Pass／Fail 的 Log、狀態、暫存器或輸出。
- 說明 Cold Boot、Warm Reset、AC Cycle、更新前後及不同 SKU 是否需要覆蓋。

## 6. 常見問題與排查方向
- 症狀與發生階段
- 首要觀測點與資料收集方式
- 可能原因與驗證順序
- 暫時規避方式、正式修正方向與回歸範圍

## 7. 安全性與相容性注意事項
- 權限、信任邊界、敏感資料與金鑰處理。
- 規格版本、工具鏈、OS、Silicon Stepping 及既有產品相容性。
- 更新、降版、斷電及失敗復原時的資料完整性。

## 8. 參考資料
- [ ] UEFI Specification
- [ ] UEFI Platform Initialization Specification
- [ ] EDK II 文件與來源碼位置
- [ ] TCG、ACPI、SMBIOS、PCI-SIG 或平台供應商規格（依本章適用性補充）
- [ ] 專案內部設計文件、Issue 與測試報告
