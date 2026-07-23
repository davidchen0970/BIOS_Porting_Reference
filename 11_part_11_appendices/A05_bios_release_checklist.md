# BIOS Release Checklist

> 狀態：Draft
> 文件用途：本檔為章節撰寫大綱，內容、規格版本與平台資料仍需由章節負責人補充及驗證。

## 1. 文件目的
- 說明「BIOS Release Checklist」在 附錄 中的角色。
- 定義本章涵蓋範圍、非涵蓋範圍與預期讀者。
- 提供設計、移植、驗證及問題排查時可重複使用的參考架構。

## 2. 建議先備知識
- 提供跨章節可重複引用的速查資料
- 統一術語、工具輸出、收集格式與檢查表
- 降低文件維護與跨平台複製成本

## 3. 建議內容大綱

### 3.1 Release 範圍、候選版本與變更凍結

- 定義 Release Type、目標產品／SKU、版本、分支、Tag、基準設定與變更範圍
- 設定 Code Freeze、Content Freeze、Release Candidate 與最終核准時間點
- 未納入本次發佈的變更須有明確延後原因與追蹤項目

### 3.2 Release Gate 欄位與簽核責任

- 建議欄位：Gate、Required Artifact、Acceptance Threshold、Evidence、Owner、Approver、Status
- 指定 Engineering、Validation、Security、Manufacturing、Service、Program 與客戶介面的簽核責任
- Release Gate 引用第 49 章測試結果，不重複建立技術測試案例

### 3.3 Build、映像與供應鏈完整性 Gate

- Build Reproducibility、來源版本、Binary Provenance、映像 Hash 與簽章已確認
- SBOM、弱點掃描、License、金鑰權限與發佈環境稽核符合門檻
- Release Candidate 與核准映像之間不得存在未審閱差異

### 3.4 Validation 與 Security Completion Gate

- 定義必要測試完成率、通過率、重跑政策及允許的未執行項目
- 不得存在未核准的 Blocker、Critical Issue 或安全紅線
- 功能、回歸、相容性、效能、安全、更新及 Recovery 結果須有核准摘要
- 所有 Waiver 須包含風險、補償措施、適用版本、期限與核准者

### 3.5 Release Readiness 與營運準備 Gate

- Release Note、Known Issue、Upgrade／Downgrade Path、Recovery Guide 已審閱
- BMC／CPLD／EC／OS／Driver 相依版本及部署順序已確認
- 工具、設定檔、簽章檔、更新套件、Rollback 與撤回機制已備妥
- Manufacturing、Support、RMA／Field Service 與必要客戶通知已完成

### 3.6 最終核准、發佈與證據封存

- 核准者確認所有 Gate、例外、交付物與通知均已完成
- 封存映像、Hash、簽章、來源 Tag、Build Log、測試摘要及簽核紀錄
- 正式發佈位置、存取權限、保留期限與稽核資訊須可追溯

### 3.7 撤回、Rollback、Hotfix 與發佈後監控

- 定義 Stop-ship、撤回、Rollback、Hotfix 與緊急發佈的觸發條件
- 建立發佈後 Telemetry、現場事件、回歸問題與客戶回饋的監控窗口
- 明確指定事件判定、決策、溝通與後續修正版 Owner

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
