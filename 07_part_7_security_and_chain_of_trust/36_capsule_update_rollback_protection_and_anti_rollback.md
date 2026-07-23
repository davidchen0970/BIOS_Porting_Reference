# Capsule Update、Rollback Protection 與 Anti-rollback

> 狀態：Draft
> 文件用途：本檔為章節撰寫大綱，內容、規格版本與平台資料仍需由章節負責人補充及驗證。

## 1. 文件目的
- 說明「Capsule Update、Rollback Protection 與 Anti-rollback」在 安全性與信任鏈 中的角色。
- 定義本章涵蓋範圍、非涵蓋範圍與預期讀者。
- 提供設計、移植、驗證及問題排查時可重複使用的參考架構。

## 2. 建議先備知識
- 說明信任根、驗證鏈與量測鏈的關係
- 整理金鑰、Policy、更新與復原的安全邊界
- 建立安全功能啟用、驗證及失敗處理方式

## 3. 建議內容大綱

### 3.1 FMP Image Descriptor、ESRT 與 Capsule 格式

### 3.2 Capsule 提交、跨重開機保存與處理生命週期

- OS 透過 `QueryCapsuleCapabilities()` 與 `UpdateCapsule()` 提交更新
- `EFI_CAPSULE_HEADER`、Capsule Flags、Scatter Gather List 與 Block Descriptor
- Reset Type、平台 Reset Policy 與更新所需重開機條件
- Capsule on Disk、記憶體保存型 Capsule 與平台支援差異
- PEI／DXE Capsule 處理模組的搜尋、重組、驗證與分派
- 跨重開機資料完整性、容量限制與錯誤清除
- 更新成功、失敗或中斷後的下一次開機行為

### 3.3 FMP `CheckImage()`／`SetImage()` 與映像驗證

- `EFI_FIRMWARE_MANAGEMENT_PROTOCOL` Image Descriptor 與 Image Index
- `GetImageInfo()`、`CheckImage()`、`SetImage()` 與 Progress Callback
- 映像格式、Hardware Instance、平台相容性及依賴條件檢查
- Authentication、映像簽章、版本與 Lowest Supported Version
- Flash Region 選擇、寫入邊界與更新後驗證
- Abort、Reset Required、Status Code 與錯誤映射

### 3.4 Monotonic Counter、Security Version 與 Anti-rollback

### 3.5 電源中斷、部分寫入與交易完整性

### 3.6 更新狀態 Variable、ESRT 與 OS 回報

- Last Attempt Version、Last Attempt Status 與目前 Firmware Version
- ESRT 狀態更新、Capsule Result Variable 與 OS 查詢
- 成功、拒絕、驗證失敗、資源不足及寫入失敗的狀態一致性
- 更新 Log、BMC Event 與 OS Update History 的事件關聯

### 3.7 升版、同版、降版、錯誤映像及復原測試

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
