# PCIe Enumeration、Resource Allocation 與 Option ROM

> 狀態：Draft
> 文件用途：本檔為章節撰寫大綱，內容、規格版本與平台資料仍需由章節負責人補充及驗證。

## 1. 文件目的
- 說明「PCIe Enumeration、Resource Allocation 與 Option ROM」在 周邊裝置與匯流排初始化 中的角色。
- 定義本章涵蓋範圍、非涵蓋範圍與預期讀者。
- 提供設計、移植、驗證及問題排查時可重複使用的參考架構。

## 2. 建議先備知識
- 說明裝置探索、資源配置與驅動繫結流程
- 整理各匯流排初始化的必要條件及常見相依性
- 建立裝置缺失、資源衝突及開機失敗的排查方法

## 3. 建議內容大綱
### 3.1 Root Complex、Root Port、Bridge 與 Endpoint
### 3.2 Bus Enumeration 與 Configuration Space
### 3.3 BAR Sizing、MMIO/IO、Prefetchable Resource
### 3.4 Above 4G Decoding、ReBAR、SR-IOV 與 CXL 資源配置

- CXL.io 裝置的 PCIe Enumeration 與能力探索
- CXL Capability、DVSEC 與裝置類型辨識
- CXL Type 1／Type 2／Type 3 裝置的適用範圍
- Host-managed Device Memory 與 HDM Decoder 設定
- System Memory Map、Interleave、NUMA 與 ACPI Table 銜接
- CXL QoS、頻寬／延遲能力、Telemetry 與平台 Policy
- CDAT、ACPI HMAT 與 SRAT 的記憶體距離、頻寬及延遲屬性配置
- CXL.cache／CXL.mem 流量需求與 Host Bridge／Memory Controller 資源協調
- CXL Hot-plug、資源預留及裝置移除後的錯誤處理
- Flex Bus、Link Mode 與平台韌體 Policy
- 僅適用於支援 CXL 的 Server／Workstation 平台
### 3.5 Link Training、Speed、Width 與 Equalization
### 3.6 Option ROM、UEFI Driver 與 Legacy 相容性
### 3.7 AER、資源不足與裝置缺失

### 3.8 PCIe Hot-plug 控制器、資源預留、事件處理與 OS 交接

- Slot Capability、Slot Control／Status 與 Hot-plug Controller 初始化
- Presence Detect、Power Controller、Attention Button／Indicator 與 Command Completed Event
- Bus Number、MMIO、Prefetchable MMIO 及 I/O Window 預留策略
- ACPI `_OSC` 能力協商與 Native PCIe Hot-plug 控制權交接
- SCI／GPE、PME、Hot-plug Wake 與第 25 章電源管理流程銜接
- Surprise Removal、Orderly Removal、Link Down 與錯誤事件處理
- CXL／PCIe Hot-plug 的平台適用條件、OS 相容性與壓力測試

### 3.9 PCIe NTB 資源配置與多主機通訊（選用）

- NTB Topology、Primary／Secondary Interface 與多主機責任邊界
- Inbound／Outbound Translation Window、BAR 與 Base／Limit 設定
- Doorbell、Scratchpad、Message Register 與 Peer Memory Window
- Bus Number、MMIO Aperture、IOMMU／DMA Remapping 與資源隔離
- Link Initialization、Peer Discovery、Heartbeat 與 Failover Policy
- ACPI／Device Tree 或 OS Driver 所需的裝置描述與能力交接
- Peer Reset、Link Loss、Split-brain、Surprise Removal 與復原測試
- 僅適用於具備 NTB 的 Multi-host、HA 或 HCI 平台

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
