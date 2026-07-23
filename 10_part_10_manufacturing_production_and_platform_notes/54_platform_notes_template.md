# Platform Notes Template

> 狀態：Draft
> 文件用途：本檔為章節撰寫大綱，內容、規格版本與平台資料仍需由章節負責人補充及驗證。

## 1. 文件目的
- 說明「Platform Notes Template」在 製造、量產與平台特定筆記 中的角色。
- 定義本章涵蓋範圍、非涵蓋範圍與預期讀者。
- 提供設計、移植、驗證及問題排查時可重複使用的參考架構。

## 2. 建議先備知識
- 說明研發映像檔與量產映像檔的差異
- 整理板號、SKU、序號及 Provisioning 資料流
- 建立工廠、RMA 與 Field Service 的交接規則

## 3. 建議內容大綱

### 3.1 平台基本資料與版本矩陣

### 3.2 硬體方塊圖、Power／Reset／Clock 拓樸

### 3.3 CPU、Memory、PCH／SoC 與周邊配置

### 3.4 Flash Layout、Firmware Package 與建置方式

### 3.5 GPIO、Setup、PCD 與 SKU 差異

### 3.6 平台功能狀態矩陣與已知硬體限制

- 記錄產品生命週期中的靜態平台事實，包括 Supported／Unsupported／Limited 功能狀態
- 記錄 Hardware Errata、Silicon／Board Limitation、固定設計限制及平台專屬 Workaround
- 每筆限制應包含適用 SKU／Revision、影響範圍、偵測方式、處置方式與參考文件
- 不記錄專案階段的測試 Pass／Fail、完成率、Blocker 或簽核狀態
- 測試覆蓋範圍與測試結果由第 49 章及其測試報告管理
- Bring-up Gate、Acceptance 與 Release Sign-off 分別由 A04、A05 管理

### 3.7 Owner、參考文件與更新規則

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
