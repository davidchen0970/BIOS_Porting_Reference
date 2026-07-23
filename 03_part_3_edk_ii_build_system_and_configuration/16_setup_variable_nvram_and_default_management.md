# Setup Variable、NVRAM 與預設值管理

> 狀態：Draft
> 文件用途：本檔為章節撰寫大綱，內容、規格版本與平台資料仍需由章節負責人補充及驗證。

## 1. 文件目的
- 說明「Setup Variable、NVRAM 與預設值管理」在 EDK II、建置系統與設定資料 中的角色。
- 定義本章涵蓋範圍、非涵蓋範圍與預期讀者。
- 提供設計、移植、驗證及問題排查時可重複使用的參考架構。

## 2. 建議先備知識
- 說明 EDK II 來源樹、描述檔與建置產物的關係
- 整理設定資料的來源、覆寫優先順序與維護方式
- 建立建置失敗及設定未生效時的排查路徑

## 3. 建議內容大綱

### 3.1 UEFI Variable Service、Attribute 與存取權限

### 3.2 NV Storage、Variable Store、FTW Working／Spare 區域佈局

### 3.3 Standard、Manufacturing 與平台預設值來源

### 3.4 首次開機、Load Default、CMOS Clear 與預設值載入時機

### 3.5 Setup Browser、HII、VarStore 與 Callback 資料流

### 3.6 Variable Store 損壞、空間不足與遺失還原

### 3.7 更新後 Migration、Reset Requirement 與相容性驗證

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
