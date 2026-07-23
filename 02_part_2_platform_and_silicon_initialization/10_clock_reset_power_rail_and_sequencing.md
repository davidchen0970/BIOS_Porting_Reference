# Clock、Reset、Power Rail 與電源時序

> 狀態：Draft
> 文件用途：本檔為章節撰寫大綱，內容、規格版本與平台資料仍需由章節負責人補充及驗證。

## 1. 文件目的
- 說明「Clock、Reset、Power Rail 與電源時序」在 平台與矽晶片初始化 中的角色。
- 定義本章涵蓋範圍、非涵蓋範圍與預期讀者。
- 提供設計、移植、驗證及問題排查時可重複使用的參考架構。

## 2. 建議先備知識
- 釐清平台、Silicon 與 Board 層的責任邊界
- 整理硬體初始化相依順序與必要前置條件
- 建立可重複使用的 Bring-up 驗證方式

## 3. 建議內容大綱

### 3.1 Clock Generator、BCLK、Reference Clock 與 PLL 初始化

### 3.2 Reset Domain 與 PLTRST#、SYS_RST#、RSMRST# 時序

### 3.3 Power Rail、Power Good、SLP_Sx# 與電源狀態轉換

### 3.4 PCH／SoC Power State Machine 與韌體介入點

### 3.5 Cold Boot、Warm Reset、Global Reset 與 AC Cycle 差異

### 3.6 Clock／Power／Reset 故障偵測及 Timeout Policy

### 3.7 示波器、邏輯分析儀、GPIO 與暫存器驗證方法

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
