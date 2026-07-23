# Firmware Trust Chain、Boot Guard 與 Hardware Root of Trust

> 狀態：Draft
> 文件用途：本檔為章節撰寫大綱，內容、規格版本與平台資料仍需由章節負責人補充及驗證。

## 1. 文件目的
- 說明「Firmware Trust Chain、Boot Guard 與 Hardware Root of Trust」在 安全性與信任鏈 中的角色。
- 定義本章涵蓋範圍、非涵蓋範圍與預期讀者。
- 提供設計、移植、驗證及問題排查時可重複使用的參考架構。

## 2. 建議先備知識
- 說明信任根、驗證鏈與量測鏈的關係
- 整理金鑰、Policy、更新與復原的安全邊界
- 建立安全功能啟用、驗證及失敗處理方式

## 3. 建議內容大綱

### 3.1 Hardware Root of Trust 與啟動信任邊界

### 3.2 Boot Guard Profile、Key Manifest 與 Boot Policy Manifest

### 3.3 IBB／OBB 區域與驗證／量測流程

### 3.4 Fuse、Key Hash、Provisioning 與生命週期狀態

### 3.5 失敗政策、Recovery、Dead-loop 與磚化風險

### 3.6 韌體更新、金鑰輪替及 Anti-rollback 相依性

### 3.7 驗證工具、事件紀錄與負向測試

### 3.8 PCIe／CXL DOE 與 SPDM 元件認證（選用）

- DOE Capability Discovery、Mailbox、Data Object Type 與交易狀態
- SPDM Version／Capability／Algorithm Negotiation
- Certificate Chain、Digest、Challenge 與裝置身分驗證
- Measurement、Measurement Summary Hash 與韌體版本證據
- Session Establishment、Key Exchange、Heartbeat 與 Session Teardown
- 裝置未支援、憑證失效、量測不符、逾時與 Retry Policy
- 認證失敗時的裝置封鎖、降級啟動、事件記錄與管理介面回報
- 僅適用於支援 DOE／SPDM 的 PCIe、CXL 或加速器裝置

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
