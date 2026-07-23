# Bring-up 與 Acceptance Checklist

> 狀態：Draft
> 文件用途：本檔為章節撰寫大綱，內容、規格版本與平台資料仍需由章節負責人補充及驗證。

## 1. 文件目的
- 說明「Bring-up 與 Acceptance Checklist」在 附錄 中的角色。
- 定義本章涵蓋範圍、非涵蓋範圍與預期讀者。
- 提供設計、移植、驗證及問題排查時可重複使用的參考架構。

## 2. 建議先備知識
- 提供跨章節可重複引用的速查資料
- 統一術語、工具輸出、收集格式與檢查表
- 降低文件維護與跨平台複製成本

## 3. 建議內容大綱

### 3.1 Bring-up 階段、Gate 與治理責任

- 定義 Power-on、Minimal Boot、Feature Enablement、OS Boot、System Integration 與 Acceptance 等 Gate
- 指定各 Gate 的 Owner、Approver、Consulted Party 與 Escalation Owner
- 技術測試項目與覆蓋範圍引用第 49 章，不在本附錄重複維護

### 3.2 Gate 檢查表欄位與證據規則

- 建議欄位：Gate、Entry Criteria、Exit Criteria、Required Evidence、Owner、Due Date、Status、Approver
- 證據應連結至測試報告、Log Package、Issue、版本資訊及風險紀錄
- Gate 狀態僅表示流程是否可前進，不取代個別測試案例的 Pass／Fail

### 3.3 進入 Bring-up 的前置條件

- 硬體版本、原理圖、BOM、電源時序與 Board Readiness 已確認
- Debug Tool、燒錄工具、量測設備、測試治具與存取權限已備妥
- BIOS、BMC、CPLD／EC、Silicon Package 與工具鏈版本已固定
- Known-good Image、Recovery Path、Log 收集方式與 Issue Tracker 已可用

### 3.4 各階段退出標準與問題升級規則

- 每一 Gate 應定義 Minimum Exit Criteria、必要證據與不可接受的 Blocker
- 明確區分 Blocker、Critical、Major、Minor 與 Observation
- 定義跨 BIOS、BMC、Hardware、Validation、Security 團隊的 Escalation 路徑
- 未達 Exit Criteria 時，須記錄停留、回退、條件式前進或暫停專案的決策

### 3.5 Acceptance 簽核層級、風險接受與交付物

- 定義 Engineering、Validation、Security、Manufacturing、Service 與 Program 層級簽核
- 風險接受應包含影響、發生機率、補償措施、到期日與核准者
- 必要交付物包括版本矩陣、測試摘要、Open Issue、Known Limitation、Recovery 指引及平台筆記
- Acceptance 只確認 Gate 與交付物達標，不重新定義第 49 章的技術覆蓋範圍

### 3.6 例外、Waiver 與條件式 Acceptance

- 定義例外申請、審閱、期限、復查及關閉流程
- 禁止以未追蹤的口頭共識取代正式風險紀錄

### 3.7 Acceptance Sign-off、基準封存與交接

- 封存核准版本、Hash、設定、證據索引、決策紀錄與簽核結果
- 完成對 Release、Manufacturing、RMA／Field Service 與維護團隊的交接

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
