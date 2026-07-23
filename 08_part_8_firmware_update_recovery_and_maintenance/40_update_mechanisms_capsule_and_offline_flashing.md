# 更新機制、Capsule 與離線燒錄

> 狀態：Draft
> 文件用途：本檔為章節撰寫大綱，內容、規格版本與平台資料仍需由章節負責人補充及驗證。

## 1. 文件目的
- 說明「更新機制、Capsule 與離線燒錄」在 韌體更新、復原與維護 中的角色。
- 定義本章涵蓋範圍、非涵蓋範圍與預期讀者。
- 提供設計、移植、驗證及問題排查時可重複使用的參考架構。

## 2. 建議先備知識
- 說明映像檔生命週期及不同更新路徑
- 整理斷電、降版、設定遷移與復原情境
- 建立量產及現場維護所需的保護與稽核機制

## 3. 建議內容大綱

### 3.1 Capsule、BMC 帶外更新、Shell 與 Programmer 等更新路徑

- OS Runtime Capsule、Capsule on Disk 與離線 Capsule 路徑
- BMC 帶外更新、Host Online／Offline 狀態與所有權協調
- UEFI Shell、Recovery Media、Factory Tool 與外部 Programmer
- 各路徑的簽章、版本、權限、Audit 與 Recovery 能力差異

### 3.2 映像檔前置驗證、版本及平台相容性檢查

### 3.3 Flash Unlock、Erase、Program、Verify 與 Lock 流程

### 3.4 更新期間電源、Watchdog、Reset 與進度回報

- `UpdateCapsule()` 後的 Reset Policy 與 Cold／Warm Reset 適用條件
- Capsule 處理、FMP `CheckImage()`／`SetImage()` 與 Flash 寫入階段
- Watchdog 暫停／延長、Progress Callback 與 BMC／OS 狀態同步
- Last Attempt Status、Capsule Result Variable 與 ESRT 更新
- 更新結束後的 Reset、設定遷移與 Boot Success 判定

### 3.5 設定保留、區域選擇與完整／差分更新

### 3.6 斷電、工具中止、寫入失敗與復原行為

### 3.7 更新 Log、稽核資料與產線／現場權限

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
