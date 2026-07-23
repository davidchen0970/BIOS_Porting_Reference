# BIOS General Test Matrix

> 狀態：Draft
> 文件用途：本檔為章節撰寫大綱，內容、規格版本與平台資料仍需由章節負責人補充及驗證。

## 1. 文件目的
- 說明「BIOS General Test Matrix」在 Debug、效能與測試 中的角色。
- 定義本章涵蓋範圍、非涵蓋範圍與預期讀者。
- 提供設計、移植、驗證及問題排查時可重複使用的參考架構。

## 2. 建議先備知識
- 建立可重現、可量測、可比較的除錯流程
- 整理 Log、Trace、POST Code 與測試證據
- 定義通過條件、回歸範圍與效能基準

## 3. 建議內容大綱

### 3.1 測試矩陣的 Scope、平台組合與不涵蓋項目

### 3.2 CPU、Memory、SKU、Board Revision 與裝置組合

### 3.3 Cold／Warm Boot、Reset、AC Cycle、Sleep 與長循環

### 3.4 Setup、Variable、周邊、Boot Media、Network 與 OS 覆蓋

### 3.5 安全、更新、降版、斷電與 Recovery 測試

- 無效、遭竄改或使用不受信任金鑰簽署的映像
- 遭撤銷或過期的金鑰，以及 db／dbx 更新
- Secure Boot Setup Mode／User Mode／Custom Mode 轉換
- TPM Clear、PCR 不一致與 Event Log 異常
- TPM Dictionary Attack Lockout 與復原
- Capsule 版本檢查、Lowest Supported Version 與 Anti-rollback
- 更新途中斷電、簽章錯誤與 Recovery Image 驗證
- OS Runtime Variable 的 `GetVariable()`、`SetVariable()`、`GetNextVariableName()` 壓力與邊界測試
- SMM／MM Communication Buffer 的大小、對齊、權限、重入與惡意輸入測試
- S3 Resume 前後的 Variable Service、SMRAM／MMRAM Lock 與 Runtime 位址轉換驗證
- S4 Resume、Kernel Runtime 呼叫與長時間反覆存取的相容性測試

### 3.6 自動化／手動測試分類、執行頻率與證據格式

### 3.7 覆蓋率、Pass／Fail、Blocker 與回歸選擇；簽核流程另見 A04／A05

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
