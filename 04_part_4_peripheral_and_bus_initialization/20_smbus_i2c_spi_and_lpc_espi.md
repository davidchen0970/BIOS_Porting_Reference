# SMBus、I2C、SPI 與 LPC／eSPI

> 狀態：Draft
> 文件用途：本檔為章節撰寫大綱，內容、規格版本與平台資料仍需由章節負責人補充及驗證。

## 1. 文件目的
- 說明「SMBus、I2C、SPI 與 LPC／eSPI」在 周邊裝置與匯流排初始化 中的角色。
- 定義本章涵蓋範圍、非涵蓋範圍與預期讀者。
- 提供設計、移植、驗證及問題排查時可重複使用的參考架構。

## 2. 建議先備知識
- 說明裝置探索、資源配置與驅動繫結流程
- 整理各匯流排初始化的必要條件及常見相依性
- 建立裝置缺失、資源衝突及開機失敗的排查方法

## 3. 建議內容大綱

### 3.1 各匯流排拓樸、控制器與裝置位址配置

### 3.2 控制器初始化、Clock、BAR 與 Decode Window

### 3.3 SMBus／I2C Transaction、Timeout、Retry 與 Bus Recovery

### 3.4 SPI Flash Cycle、Descriptor、Region 與寫入保護

### 3.5 LPC／eSPI Channel、I/O Decode、Virtual Wire 與事件

### 3.6 Ownership、仲裁、SMM 存取與安全邊界

### 3.7 Bus Scan、Protocol Analyzer 與錯誤隔離

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
