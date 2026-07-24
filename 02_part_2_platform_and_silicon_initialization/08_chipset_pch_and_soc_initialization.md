# Ch8 Chipset、PCH 與 SoC 初始化

狀態：Draft  
文件用途：本章說明 BIOS 在 Chipset、PCH 與 SoC 初始化期間的主要責任、資料來源、執行順序、驗證方式與問題排查方向。平台專屬的暫存器位址、Policy 欄位、Silicon Stepping、Errata 與工具版本，仍須由章節負責人依實際專案補充及驗證。

修訂說明：Rev2 強化 PI 階段對照、Bring-up 最小可行初始化、Policy 衝突處理、Silicon Init 失敗資料收集，以及與 Chapter 4 HOB／PPI／Protocol 機制的銜接。

Rev3 進一步補強 Boot Mode 路徑分流、初始化後健康狀態、Policy 動態更新窗口、Silicon Package 相容性檢查，以及 Silicon Trace 與 BIOS Log 的時間軸整合。

Rev4 明確定義 Silicon 軟體介面邊界，補充 ACPI BERT／WHEA／APEI 錯誤交接、動態 PCIe 資源預留、初始化路徑分流矩陣，以及 Lockdown 與 Protection 的層級差異。

## 8.1 文件目的

Chipset、PCH 與 SoC 初始化是平台由 Reset 狀態進入可執行後續韌體、裝置列舉及作業系統啟動環境的關鍵程序。

本章所稱的 **Silicon**，並非單指實體晶片或所有硬體暫存器，而是涵蓋 CPU／Silicon Vendor 提供的韌體元件、初始化程式庫或 Binary Package，例如 FSP、AGESA 或同類元件，以及其公開介面、Policy 結構、程式設計指南與適用 Errata。BIOS 通常負責收集平台資訊、建立並驗證 Policy、準備前置條件、呼叫供應商介面，以及保存初始化結果；Silicon Package 則依介面契約執行其負責的硬體初始化。

因此，BIOS 與 Silicon 的互動原則上應透過供應商定義且與版本相容的軟體介面進行，不應將所有初始化工作理解為 BIOS 直接寫入暫存器。只有在供應商文件明確要求、介面未涵蓋，或平台整合責任確實屬於 BIOS 時，才應由平台程式直接設定對應暫存器，並記錄其版本、Stepping、時序與鎖定相依性。

本章目的如下：

- 說明 Chipset、PCH 與 SoC 初始化在 BIOS 啟動流程中的角色。
- 釐清 Silicon、Board、Platform 與 BIOS 各層的責任邊界。
- 說明初始化所需的 Strap、Fuse、Policy、Setup Variable 與板級設定來源。
- 整理 Host Bridge、DMI／Fabric、內部匯流排、Interrupt、DMA、IOMMU 與資源配置的相依關係。
- 建立可重複使用的 Bring-up、驗證與問題排查方法。
- 說明 End-of-POST 前後的設定鎖定、安全性與相容性考量。

本章著重於初始化原則與工程方法，不取代 Silicon Vendor Programming Guide、Datasheet、BIOS Writer's Guide、Errata 或專案內部設計文件。當本章內容與平台供應商正式規格不一致時，應以該平台適用且經確認的正式規格為準。

## 8.2 適用範圍與責任邊界

### 8.2.1 涵蓋範圍

本章涵蓋下列項目：

- Reset 後 PCH／SoC 的初始狀態確認。
- Pre-Memory 與 Post-Memory 階段的 Silicon 初始化。
- Silicon Policy 建立、更新、傳遞與套用。
- Strap、Fuse、SKU、Stepping 與 Board ID 的判讀。
- Host Bridge、DMI／Fabric 與內部匯流排設定。
- PCH 內建周邊的 Enable／Disable Policy。
- Interrupt、DMA、IOMMU 與 MMIO／I/O／Bus Number 資源配置。
- End-of-POST、設定鎖定與安全相關控制。
- Cold Boot、Warm Reset、S3 Resume 與其他平台支援路徑的差異。
- Stepping 差異、Errata、驗證及問題排查。

### 8.2.2 非涵蓋範圍

除非與 PCH／SoC 初始化有直接相依，本章不深入說明：

- CPU 核心與記憶體控制器的完整訓練流程。
- 單一外接 PCIe 裝置的驅動程式設計。
- 作業系統內部的裝置驅動與電源管理策略。
- BMC 韌體內部架構及管理協定細節。
- 各 Silicon Vendor 未公開之內部設計。

### 8.2.3 建議讀者

- BIOS／UEFI 韌體工程師
- Silicon 與 Platform 整合工程師
- Board Bring-up 與驗證工程師
- 系統測試、問題分析及維護人員

## 8.3 建議先備知識

閱讀本章前，建議先具備下列基礎：

- UEFI PI 啟動階段，包括 SEC、PEI、DXE 與 BDS 的基本概念。
- PCI／PCIe Configuration Space、BAR、Bus／Device／Function 與資源配置概念。
- MMIO、I/O Port、Interrupt、DMA 與 IOMMU 的基本用途。
- Reset 類型及其對暫存器、電源域與裝置狀態的影響。
- 平台原理圖、Board ID、Strap 與 GPIO 的基本判讀方式。
- PCD、HOB、Protocol、Setup Variable 與 Silicon Policy 的資料傳遞方式。

## 8.4 初始化流程總覽

PCH／SoC 初始化通常跨越多個 BIOS 階段。實際分工會因平台架構與 Silicon Package 而異，但可概略區分如下。

### 8.4.1 Reset 與早期環境建立（SEC／PEI Pre-Memory）

系統離開 Reset 後，CPU 從 Reset Vector 開始執行。此時可使用的記憶體與服務有限，早期程式應避免依賴尚未初始化的 DRAM、PCIe 裝置或完整 UEFI Service。

此階段的重點包括：

1. 建立最基本的執行環境。
2. 判斷 Boot Mode 與 Reset Cause。
3. 讀取可在早期取得的 Strap、Fuse、SKU 或固定平台設定。
4. 建立 Pre-Memory Silicon Policy。
5. 執行與記憶體初始化具有前置相依的 PCH／SoC 設定。
6. 保留足以分析早期失敗的 POST Code、Serial Log 或 Trace Hub 資訊。

### 8.4.2 Pre-Memory 初始化（PEI）

Pre-Memory 階段只應設定記憶體初始化前必須完成的項目。常見內容包括：

- 基礎 Clock、Reset、Power Well 或 Fabric 前置設定。
- 與 Boot Device、SPI、eSPI、LPC、UART 或 Debug Port 有關的早期設定。
- 記憶體初始化需要參考的 Board、SKU 與 Policy 資訊。
- Silicon Vendor 要求在特定時間點寫入的暫存器。

設計時應明確記錄每一項 Pre-Memory 設定的必要性。若某設定可延後至 Post-Memory，通常應避免放在早期階段，以降低早期流程複雜度與除錯成本。

### 8.4.3 Post-Memory 初始化（PEI Post-Memory／DXE Early）

DRAM 可用後，BIOS 可建立較完整的資料結構並執行後續初始化，例如：

- 建立或更新 Post-Memory Silicon Policy。
- 初始化 Host Bridge、DMI／Fabric 與內部匯流排。
- 設定 PCH 內建周邊與其電源管理策略。
- 建立 PCIe Root Port、USB、SATA、Audio、Network 與其他整合裝置的初始狀態。
- 準備 Interrupt、DMA、IOMMU 與資源配置。
- 將 Silicon 初始化結果透過 HOB 或其他介面傳遞至 DXE。

### 8.4.4 DXE、BDS 與 End-of-POST（DXE／BDS）

DXE 階段通常負責裝置列舉、資源配置、ACPI／SMBIOS 資料建立及平台服務安裝。接近 End-of-POST 時，BIOS 應完成需要在交付作業系統前執行的鎖定與安全設定。

須特別注意：設定鎖定的時間點不可僅依「越早越安全」判斷。若鎖定過早，後續模組可能無法完成必要設定；若鎖定過晚，則可能增加設定遭非預期修改的風險。適當時點應依 Silicon 規格、平台資料流及安全設計共同決定。

### 8.4.5 Boot Mode 路徑分流原則

`BootMode`、Reset Cause 與更新狀態應在建立 Silicon Policy 前完成判斷，並成為初始化流程的第一層分流條件。各路徑不宜先執行共同的完整初始化，再於後段嘗試修正差異。

#### Cold Boot／AC Cycle

- 執行平台要求的完整 Pre-Memory 與 Post-Memory Silicon Init。
- 建立新的記憶體、Host Bridge、DMI／Fabric、PCIe 與周邊狀態。
- 重新解析 Board ID、Strap、Fuse 與需要在本次啟動生效的 Setup Variable。
- 完成資源配置、錯誤狀態基線建立及 End-of-POST 鎖定。

#### Warm Reset／Global Reset

- 不應直接假設所有 Silicon 狀態均已清除，須依各暫存器的 Reset Domain 判斷保留範圍。
- 若 Silicon Vendor 指定完整重新初始化，應依指定流程執行；若部分 Power Well、Link 或 Fabric 狀態保留，則應避免重複寫入 Write Once 或已鎖定欄位。
- 保存並記錄 Reset Cause，以便區分正常重啟、Watchdog、錯誤復原與更新程序觸發的 Reset。

#### S3 Resume

- 優先從 ACPI NVS、S3 Boot Script 或 Silicon Vendor 定義的 Resume Context 還原必要設定。
- 僅執行 Resume 所需的 PCH／SoC 恢復程序，例如必要的 Clock／Power 恢復、輕量 Link Retraining 與周邊 Context Restore。
- 原則上跳過完整記憶體訓練、Host Bridge 資源重建、PCI Bus 重新編號與完整 DMI／Fabric 初始化，除非適用規格明確要求。
- 不應重新解析會改變硬體拓樸的 Setup Variable，避免 Resume 前後的資源配置不一致。
- 若偵測到 NVS 不完整、Policy Revision 不相容或 Silicon 狀態不可信，應依平台定義轉入安全的完整重啟路徑，而非持續使用可能失效的 Resume Context。

#### Firmware Update 後重啟

- 設定明確的 Update Pending／Update Complete 狀態，並在重啟早期驗證映像版本、完整性與更新結果。
- 重新建立 SPI／Flash Region、Write Protection、Protected Range 與相關鎖定，不應假設更新前的保護暫存器狀態仍有效。
- 若更新改變 Silicon Package、Policy Revision、Setup Schema 或 NVRAM 格式，應執行相容性檢查與必要的設定遷移。
- 更新成功、失敗復原與斷電後首次啟動應分別定義 Boot Path、Log 與 Pass／Fail 條件。

#### 建議的路徑決策 Log

```text
BOOT_PATH: Mode=S3Resume ResetCause=WakeFromS3
PolicyRevision=0x03 ResumeContext=Valid
FullSiliconInit=Skip DmiRestore=Required ResourceReallocation=Skip
```

文件與程式審查時，建議為每個初始化模組標示 `Run`、`Build`、`Restore`、`Skip`、`Conditional` 或 `Reinitialize`，並對 Cold Boot、Warm Reset、S3 Resume、Recovery 與 Update 後重啟建立路徑矩陣。

下表為概念模板，實際模組名稱、動作與 Reset Domain 應依專案及 Silicon Vendor 流程調整。

| 初始化模組 | Cold Boot | Warm Reset | S3 Resume | Update 後重啟 |
|---|---|---|---|---|
| Early Debug UART | Run | Run | Skip／Restore | Run |
| Pre-Memory Silicon Policy | Build | Build／Reuse | Skip／Restore Context | Build／Migrate |
| Memory Initialization | Run | Platform-defined | Restore／Skip Full Training | Run |
| DMI／Fabric Link Training | Run | Run／Conditional | Restore／Light Retrain | Run |
| PCIe Resource Allocation | Run | Run／Reuse | Skip | Run |
| PCH Integrated Devices | Run | Reinitialize／Restore | Restore | Run |
| Silicon Health Snapshot | Run | Run | Conditional | Run |
| End-of-POST Protection | Run | Run | Conditional／Verify | Run／Reapply |

Code Review 時，矩陣中的每一格應能對應至明確的 Boot Mode 判斷、函式入口、資料來源與測試項目。若同一模組在不同路徑採取不同動作，應同時記錄其狀態保存位置、失效條件及降級路徑。

## 8.5 PCH／SoC 初始狀態與初始化階段

### 8.5.1 初始狀態盤點

開始移植或 Bring-up 前，應建立平台初始狀態清單，至少包含：

- Reset 類型與 Reset Cause。
- 各 Power Well、Clock Domain 與 Reset Domain 的預期狀態。
- Strap、Fuse、SKU、Stepping 與 Security State。
- Boot Device、Recovery Path 與 Debug Interface 狀態。
- 已由硬體、Boot ROM、Microcode 或前一階段完成的設定。
- 暫存器屬性，例如 Read Only、Write Once、Lockable、Reset Type 與 Side Effect。

此清單可協助區分「硬體預設值」、「前置元件已設定」與「BIOS 必須設定」三類責任，避免重複寫入或遺漏必要步驟。

### 8.5.2 Boot Path 差異

不同 Boot Path 不一定可共用完全相同的初始化程序。設計與測試時至少應區分：

- Cold Boot
- Warm Reset
- Global Reset
- AC Power Cycle
- S3 Resume，若平台支援
- Firmware Update 後首次開機
- Recovery／Crisis Mode
- Watchdog 或錯誤復原路徑

對於每一種路徑，應確認哪些設定會保留、哪些狀態會被清除，以及是否需要重新初始化或避免重複初始化。

### 8.5.3 Bring-up 最小可行初始化檢查表

新板首次上電時，目標不是立即完成所有周邊與安全設定，而是先建立一條可觀察、可重複且可逐步擴充的最小啟動路徑。下列項目可作為第一階段檢查表，實際順序仍應依 Silicon Vendor 的最小初始化序列及平台電源設計調整。

- [ ] 確認電源時序、主要 Clock 與 Reset 訊號符合平台規格。
- [ ] 確認 Reset Vector 可執行，並可由 SPI Flash、Boot ROM 或指定啟動媒體取得第一段指令。
- [ ] 確認 Boot Block／SEC Entry 可到達，並配置可辨識的 POST Code 或最早期觀測點。
- [ ] 設定 Early Debug UART 所需的 Clock、Power、GPIO／Pin Mux、Base Address 與 Baud Rate。
- [ ] 輸出第一筆固定格式 Serial Log，內容至少包含建置版本與目前執行階段。
- [ ] 讀取並驗證 Board ID、Fab ID、Strap、Fuse、SKU 與 Silicon Stepping。
- [ ] 初始化 PCH／SoC 必要的基礎 Clock、Power Well、Reset Domain 與 Fabric 前置條件。
- [ ] 啟用 SPI、eSPI 或 LPC 等啟動媒體所需介面，若平台架構需要。
- [ ] 確認至少一個 Boot Device 可穩定讀取，並驗證映像檔範圍與基本完整性。
- [ ] 建立最小 Pre-Memory Silicon Policy，只開啟記憶體初始化及 Debug Path 所必需的功能。
- [ ] 進入記憶體初始化前，保存 Policy 摘要、Reset Cause、Board／Silicon 識別資訊及最後檢查點。

建議將第一個 Bring-up 里程碑定義為：「每次上電皆可穩定到達相同檢查點，並輸出可辨識的 Serial Log」，而不是一次開啟所有控制器。若尚無 Serial 輸出，應以 POST Code、GPIO Toggle、Trace Port 或邏輯分析儀建立替代觀測點。

## 8.6 Silicon Policy、Strap 與 Fuse 的資料來源

### 8.6.1 Policy 資料來源

Silicon Policy 可能由多個來源組成：

- Silicon Vendor 提供的預設值。
- 平台固定設定或 PCD。
- Board ID、Fab ID、BOM ID 與 GPIO Strap。
- Silicon Fuse、SKU 與 Stepping。
- BIOS Setup Variable。
- Manufacturing Mode 或工廠設定。
- 安全策略、產品需求及功能授權狀態。
- BMC 或其他管理控制器提供的平台資訊，若平台設計有此相依。

Policy 在 PEI 階段通常透過 PPI 提供服務，並以 GUID HOB 傳遞已解析的平台資訊或 Silicon 初始化結果；進入 DXE 後，則可依模組責任轉換為 Protocol，或由適當的 Dynamic PCD 提供設定。各專案應明確定義資料的擁有者、建立時間與有效生命週期。

為維持單一事實來源，DXE 模組原則上不應各自重新解析 Strap、Fuse 或 Board ID。若這些資訊已由 PEI 建立並放入 HOB，DXE 應使用該 HOB 或由其衍生的 Protocol。如此可降低 Cold Boot、Warm Reset、S3 Resume 與 Recovery 路徑採用不同 Policy 的風險。HOB、PPI 與 Protocol 的責任、生命週期及使用方式，請參閱 Chapter 4。

### 8.6.2 優先順序與衝突處理

專案應定義 Policy 的合併順序。例如：

1. 載入 Silicon 預設值。
2. 套用平台共用設定。
3. 套用 Board／SKU 差異。
4. 套用經允許的 Setup 選項。
5. 依安全限制及硬體能力修正最終值。
6. 在送入 Silicon Init 前進行一致性檢查。

若不同資料來源產生衝突，不應僅以最後寫入值作為規則。應確認硬體能力、安全限制與產品需求，並在 Log 中記錄最終值及其來源。

#### Policy 動態更新窗口

Policy 合併應在平台定義的提交點前完成。多數平台可將 PEI Post-Memory 初期、DMI／Fabric 初始化及 PCIe 資源配置之前，視為會影響拓樸之 Policy 的最後更新窗口；實際時間點仍須依 Silicon Vendor 流程確認。

下列欄位一旦進入 Silicon Init、Host Bridge Window 建立或 PCIe 資源配置後，原則上不得於同一次 Boot 動態變更：

- Controller／Root Port Enable Mask
- Lane Ownership、Bifurcation 與 Link Width
- BAR Size、Resizable BAR、Above 4G Decoding 與 MMIO Window
- Interrupt Routing、DMA Remapping 與 Reserved Memory
- 會改變 ACPI Namespace 或 PCI 拓樸的功能選項
- 需要重新訓練 Link 或重新配置 Clock／Power／Reset 的選項

若使用者於 Setup 修改上述欄位，應採用下列其中一種策略：

1. 將新值保存為 `Pending Policy`，在下一次符合條件的 Cold Boot 套用。
2. 顯示「重新啟動後生效」，並由平台觸發受控重啟以重新建立資源。
3. 若變更需要 AC Cycle、Global Reset 或清除特定 Context，應明確提示並由平台流程處理。
4. 若目前 Boot Path 為 S3 Resume 或 Firmware Update 中間狀態，延後套用並保留原因紀錄。

建議區分三種值：

```text
RequestedValue = 使用者或管理介面要求的值
CommittedValue = 本次 Boot 提交給 Silicon Init 的值
EffectiveValue = 經硬體限制與初始化結果修正後的實際值
```

在 Policy 提交後收到新的設定要求時，不應修改已提交的記憶體副本。應建立 Pending 狀態、要求適當 Reset，並於下一個允許窗口重新合併。如此可避免 Setup 顯示值、Silicon 實際狀態與 ACPI／PCI 資源描述彼此不一致。

#### 實例：PCIe Root Port Enable 衝突處理

假設 Board ID 或平台連接表顯示 PCIe Root Port 4 未接線，或 Silicon Fuse 表示該 Port 不可使用，但 Setup Variable 因使用者設定、版本遷移或 NVRAM 資料異常而要求啟用。建議依下列方式處理：

1. 先確認 Fuse、SKU 與 Silicon 能力。硬體不支援的功能不得由 Setup 覆寫。
2. 再確認 Board Routing、Lane Ownership、Clock、Reset、Power 與 Pin Mux。板級未連接或資源已分配給其他功能時，應強制停用。
3. 僅在硬體與板級條件均允許時，才接受產品預設值或 Setup Variable 的啟用要求。
4. 將 Requested Value、Effective Value、限制來源及判斷理由寫入 Debug Log。
5. Setup 畫面應呈現實際有效狀態，例如「Disabled by hardware configuration」，避免選單顯示 Enabled，但硬體實際未啟用。
6. 若偵測到 NVRAM 欄位超出範圍或版本不相容，應採安全預設值並留下可追蹤紀錄，不應將未驗證資料直接寫入 Silicon Policy。

建議 Log 格式：

```text
POLICY_CONFLICT: RP4 Requested=Enabled Effective=Disabled
Source=Setup Constraint=BoardRouting BoardId=0x03 PolicyRevision=0x02
```

此判斷方式同樣適用於 USB Port、SATA Controller、Integrated LAN、Debug Interface 與其他會受 Fuse、Board Routing 或安全策略限制的功能。

### 8.6.3 Policy 驗證

建議對關鍵 Policy 執行下列檢查：

- 欄位值是否在規格允許範圍內。
- 相依功能是否同時開啟或關閉。
- 不同 Root Port、Lane 或 Controller 是否發生資源衝突。
- 設定是否符合實際 Board Routing 與 BOM。
- Stepping／SKU 是否支援該功能。
- 安全相關功能是否會被一般 Setup 選項降低。

## 8.7 Host Bridge、DMI／Fabric 與內部匯流排設定

### 8.7.1 Host Bridge

Host Bridge 連結 CPU、記憶體與下游 I/O 資源。初始化時通常須考量：

- PCI Configuration Space 存取方式。
- MMIO、Prefetchable MMIO 與 I/O Window。
- DRAM Top、保留記憶體與 Memory-Mapped Device 區域。
- Above 4G Decoding 與大型 BAR 支援。
- System Agent／Host Bridge 相關鎖定。

資源配置應與記憶體地圖共同檢視。若只觀察單一裝置 BAR，可能無法發現與保留區、Graphics Aperture、IOMMU 或其他 MMIO Window 的重疊。

### 8.7.2 DMI／Fabric

DMI 或 SoC Fabric 負責 CPU、PCH、內部 Agent 與周邊之間的資料傳輸。初始化重點可包括：

- Link Training 與 Link Width／Speed。
- Lane／Port 配置。
- Error Reporting 與錯誤狀態清除原則。
- Power Management 與低功耗狀態。
- QoS、Traffic Class 或平台專屬的 Fabric Policy。
- Link 降速、重訓與錯誤復原行為。

若 Link 未達預期速度，排查時應同時比對硬體訊號、Strap／Fuse、Policy、Training Log、Error Counter 與 Stepping Errata，避免只依單一狀態位元判斷。

### 8.7.3 內部匯流排

SoC 內部可能存在多種私有或標準匯流排。BIOS 應透過供應商定義的介面進行設定，並遵守存取時序與鎖定要求。對未公開或不建議直接存取的介面，不應以硬編碼位址取代正式 Library／API。

## 8.8 PCH 內建周邊的 Enable／Disable Policy

### 8.8.1 設定原則

內建周邊的啟用狀態應由下列因素共同決定：

- Board 是否有實際連接該功能。
- SKU 與 Fuse 是否支援。
- 產品需求是否允許使用。
- 所需 Clock、Reset、Power Rail 與 GPIO 是否可用。
- 是否與其他裝置共用 Lane、Pin Mux 或資源。
- 安全策略是否要求停用未使用介面。

### 8.8.2 常見裝置類型

依平台不同，可能包含：

- PCIe Root Port
- USB Host／Device Controller
- SATA／UFS／eMMC／SD Controller
- HD Audio／SoundWire
- Integrated Network Controller
- SPI、eSPI、LPC
- I2C、I3C、SPI、UART、GPIO
- Sensor Hub、CNVi 或其他平台專屬裝置

### 8.8.3 停用層級

「Disable」可能代表不同層級：

- 不向 PCI 或 ACPI 呈現裝置。
- 關閉功能控制位元。
- Gate Clock 或進入低功耗狀態。
- Assert Reset。
- 關閉 Power Well。
- 隱藏 Configuration Space。

文件與 Log 應明確說明採用哪一層級，避免 BIOS 顯示為停用，但硬體仍維持供電或可被其他路徑存取。

## 8.9 Interrupt、DMA、IOMMU 與資源配置

### 8.9.1 Interrupt

Interrupt 設定應涵蓋：

- Legacy INTx Routing，若平台仍使用。
- MSI／MSI-X 能力與 OS 交接。
- IOAPIC、Local APIC 與 GSI 配置。
- GPIO Interrupt Routing 與觸發型態。
- ACPI Table 中 Interrupt Resource 的一致性。

發生 Interrupt 問題時，可比對裝置 Configuration Space、IOAPIC／GPIO Routing、ACPI Resource、OS Log 與實際 Interrupt Counter。

### 8.9.2 DMA 與 IOMMU

IOMMU 用於限制 DMA 裝置可存取的記憶體範圍。BIOS 的主要責任通常包括：

- 依平台需求啟用 IOMMU。
- 配置必要的 Reserved Memory Region。
- 建立並發布作業系統所需的 ACPI 資訊。
- 處理開機早期 DMA 裝置與安全策略的相依。
- 確認裝置隔離群組與硬體拓樸相符。

若平台支援 Pre-Boot DMA Protection，應確認 Thunderbolt、USB4、PCIe Hot Plug 或其他外部 DMA 路徑在 OS 接管前的限制方式。

### 8.9.3 資源配置

資源配置至少包含：

- PCI Bus Number
- MMIO 與 Prefetchable MMIO
- I/O Space
- Interrupt Resource
- Reserved Memory
- ACPI Operation Region

對支援大型 BAR、Above 4G Decoding、SR-IOV 或多層 PCIe Switch 的平台，應預留足夠資源並測試最大裝置組合。

對支援 PCIe Hot Plug、Thunderbolt、USB4 Dock 或其他可在 OS 執行期間改變拓樸的連接埠，BIOS 應在啟動時為下游裝置預留額外的 Bus Number、MMIO、Prefetchable MMIO 及必要的 I/O Space。預留量應依平台允許的最大裝置數、PCIe Switch 層數、Bridge Window、BAR 尺寸、Resizable BAR 與 SR-IOV VF 需求規劃，不宜只依開機當下已連接的裝置計算。

預留策略可由 Board ID、Port Capability、產品類型或受控 Setup 選項決定，但變更後通常需要重新啟動並重建 PCI 資源。ACPI 對 Hot Plug Port 所描述的資源範圍也應與實際保留量一致。驗證時至少應包含「開機時未連接外部裝置，進入 OS 後再熱插拔最大支援拓樸」以及「多層 Switch 加入大型 BAR／SR-IOV 裝置」等情境，並確認不發生 Bus Number 耗盡、Bridge Window 不足或資源重疊。

### 8.9.4 初始化後的 Silicon 健康狀態檢查

主要初始化完成後、End-of-POST 鎖定與 OS 交接前，建議由單一模組聚合 Silicon 與高速 I/O 的健康狀態。目的不是將所有錯誤一律清除，而是先建立可追蹤的 POST 錯誤基線，再依嚴重度、所有權及規格決定保留、回報、清除或停止啟動。

建議觀測項目包括：

- PCIe Root Port AER Correctable、Uncorrectable 與 Root Error Status
- DMI／Fabric／Internal Bus 的 Link、Protocol、Replay、Timeout 或 Overflow 狀態
- Host Bridge、IOMMU、DMA Remapping 與相關 Fault Status
- 內建控制器的初始化失敗、Parity／ECC 或 Firmware Status
- Link Training Retry、降速、降寬與 Recovery 次數
- 平台供應商指定的 Machine Check、WHEA 預備資料或錯誤摘要

#### 建議處理順序

1. **快照**：在任何清除動作前，讀取原始狀態、Error Source、Requester ID、Severity、First Error Pointer 與必要的 Header Log。
2. **關聯**：將錯誤與 Boot Path、Port、BDF、初始化階段、Link Training 結果及時間戳記關聯。
3. **分類**：區分預期的訓練暫態、Silicon Errata、可糾正錯誤、不可糾正非致命錯誤與致命錯誤。
4. **決策**：依平台策略選擇繼續、降速／停用裝置、重新訓練、觸發 Recovery，或停止 Boot。
5. **保存**：將摘要寫入可供後續 DXE、BERT／WHEA、BMC SEL 或維修工具取得的資料結構，避免只留在易失性 Serial Log。
6. **清除**：僅對規格允許且已保存的 W1C／RW1C 狀態進行清除，並再次讀回確認。對 Clear-on-Read、Sticky、Fatal 或由 OS／SMM 擁有的狀態不可套用通用清除程序。

#### 錯誤聚合器輸出

建議以 GUID HOB 或平台定義的資料結構傳遞健康摘要，DXE 再依需求轉換為 Protocol、ACPI Error Record 或維修紀錄。至少應包含：

```text
BootPath, ResetCause, Timestamp
ErrorSource, Segment:Bus:Device.Function
Severity, StatusBits, FirstErrorPointer
LinkSpeed, LinkWidth, TrainingRetryCount
ActionTaken, ClearResult, ErrataReference
SiliconStepping, PackageVersion, BIOSBuildId
```

健康狀態摘要除了供 BIOS 內部模組、BMC 或維修工具使用，也應規劃與平台的標準錯誤交接機制對接。對於 POST 階段已發生、但需要由 OS 得知的嚴重硬體錯誤，平台可依適用規格將記錄轉換為 ACPI BERT（Boot Error Record Table）所指向的 Boot Error Region，使 Windows WHEA 或 Linux APEI 等機制在接管後取得啟動階段錯誤。

錯誤聚合器的內部結構不必直接等同 BERT 格式，但應保留可轉換所需的 Error Severity、Error Source、Section Type、FRU／裝置識別、時間資訊、原始狀態與處置結果。是否發布 BERT、哪些錯誤可進入 Boot Error Region，以及記錄的生命週期與清除時點，應依 ACPI 規格、OS 相容性、平台錯誤架構及隱私要求共同定義。一般 Debug Log 或可糾正的訓練暫態不應未經分類即寫入 BERT。

#### 清除與 OS 交接邊界

POST 階段的暫態錯誤若未清除，可能在 OS 接管後被視為新錯誤；但若未先保存證據就清除，也會失去板級訊號、Link Training 或 Silicon 問題的線索。因此應由平台定義每一類 Status Register 的擁有者、快照時間、清除時點及 OS 交接方式。已知 Errata 也應以明確的條件判斷處理，不宜使用「看到特定位元就忽略」的通用規則。

## 8.10 Lockdown、End-of-POST 與設定鎖定

### 8.10.1 鎖定目標

本章所稱的 **Lockdown**，泛指將設定轉入最終受控狀態，使其不再接受非預期或未授權變更；它不必然等同單一 Write Once Lock Bit。實際機制可能包含 Write Once／Lockable 暫存器、權限層級切換，例如僅允許 SMM 或受信任執行環境修改、SPI Flash Protected Range／Block Protect、存取控制、介面隱藏，或在特定生命週期狀態下停用 Debug 能力。

因此，文件應分別標示「設定鎖定」與「存取保護」：前者著重設定值在指定 Reset 前不可再變更；後者著重只有授權執行環境、身分或流程可修改。每一項機制的 Owner、可逆性、Reset 條件、更新例外與驗證方法，均應依 Silicon Vendor 規格及平台 Threat Model 定義。

常見鎖定或保護對象包括：

- BIOS Write Protection 與 SPI Descriptor 相關控制。
- SMI、TCO Watchdog 與特定 SMM 控制。
- GPIO Pad Configuration Lock。
- Flash Region、Protected Range 與 Boot Guard 相關狀態。
- Host Bridge、PCH Configuration 與安全暫存器。
- Debug Interface 或製造模式介面。

實際項目應依平台 Threat Model 與 Silicon 規格確認。

### 8.10.2 鎖定時序

每個鎖定項目應記錄：

- 設定者與擁有者。
- 最後允許修改的階段。
- 鎖定位元的 Reset 條件。
- 是否為 Write Once。
- S3 Resume 或 Firmware Update 是否需要特殊處理。
- 驗證鎖定生效的方法。

### 8.10.3 End-of-POST 檢查

在 End-of-POST 前，建議執行一致性檢查：

- 必要裝置是否完成初始化。
- ACPI／SMBIOS／PCI 拓樸是否一致。
- 安全鎖定是否完成。
- Debug／Manufacturing 功能是否依產品狀態關閉。
- 錯誤狀態是否已記錄，而非直接清除後失去證據。
- OS 接管所需資料是否完整。

## 8.11 Stepping 差異、Errata 與版本管理

### 8.11.1 Stepping 判斷

Stepping 差異可能影響暫存器定義、初始化順序、功能支援與 Workaround。程式應使用經維護的識別介面，不宜在多處散布 CPUID、Device ID 或 Revision ID 的直接判斷。

### 8.11.2 Errata 管理

每一項 Workaround 建議記錄：

- Errata 編號與適用 Stepping。
- 觸發條件與外部症狀。
- 套用階段與設定內容。
- 是否影響效能、功耗或功能。
- 可移除條件。
- 驗證方式及回歸範圍。

### 8.11.3 版本可追溯性

測試報告與問題單至少應包含：

- BIOS 版本與建置識別。
- Silicon Package／FSP／AGESA 或同類元件版本。
- Microcode 版本。
- Board Revision、BOM、SKU 與 Silicon Stepping。
- BMC／EC／CPLD 版本，若與平台初始化有相依。
- 使用的規格與 Errata 版本。

### 8.11.4 Silicon Package 相容性檢查

更新 FSP、AGESA、Silicon Library、Binary Package 或其同類元件時，不應只確認能否編譯與進入 OS。更新流程應包含介面、Policy、預設值、執行路徑與輸出資料的相容性檢查。

#### 更新前靜態檢查

- 比對新舊版本的 Policy Header、Revision、`sizeof()`、欄位偏移、Alignment 與 Reserved Field。
- 比對 Enumeration 數值、Bit Mask、Default Value、Deprecated Item 與新增欄位。
- 確認專案內所有 Policy Override 仍指向相同語意，沒有因欄位改名或重新排列而寫入錯誤位置。
- 檢查 Library Class、PPI、Protocol、HOB GUID、API Prototype 與回傳狀態是否變更。
- 閱讀 Release Notes 中的 Breaking Changes、Integration Notes、Known Issues、New Defaults 與適用 Stepping。

只有 `sizeof()` 相同不足以證明相容，因為欄位語意、列舉值或預設值仍可能改變；同樣地，結構變長也不代表既有欄位必然不相容。應同時驗證版本標頭、欄位定義與實際 Dump。

#### 進入 Silicon Init 前的防護

建議加入只在開發或驗證版本啟用的 `ValidateSiliconPolicy()` 類型檢查，至少確認：

- Policy Revision 與 Silicon Package 支援版本一致。
- 結構長度、Buffer Boundary 與必要子結構存在。
- PCIe Enable Mask、Lane Configuration、BAR Size、MMIO Window 與 IOMMU 設定在允許範圍。
- Board／SKU 不支援的功能未被啟用。
- 相依欄位沒有互斥或缺漏。

檢查失敗時，開發版本可停止在明確檢查點並輸出差異；量產版本則應依風險採安全預設值、停用受影響功能或進入 Recovery，且必須留下可追蹤紀錄。

#### 更新後動態驗證

- 在相同硬體與 Setup 條件下保存新舊版本的最終 Policy Dump，執行欄位級差異比較。
- 比對 PCI 拓樸、ACPI Table、Memory Map、Link Speed／Width、啟動時間與健康狀態摘要。
- 覆蓋 Cold Boot、Warm Reset、S3 Resume、Firmware Update 後首次啟動及 Recovery。
- 對 Release Notes 宣告的新預設值建立顯式測試，避免無聲改變產品行為。
- 若版本回退受支援，驗證新版本寫入的 NVRAM／Context 是否可由舊版本安全解析；否則應定義拒絕降版或清除／遷移策略。

## 8.12 驗證與測試重點

### 8.12.1 測試環境

測試前應記錄硬體、韌體、工具與外接裝置組合，並保存可重現問題所需的 BIOS 設定。若使用客製設定檔，應納入版本管理。

### 8.12.2 基本驗證矩陣

建議至少覆蓋：

- Cold Boot、Warm Reset 與 AC Cycle。
- 各主要 SKU、Board Revision 與 Silicon Stepping。
- 預設設定、最佳化預設值與支援的使用者設定。
- Firmware Update 前後及設定保留／重置行為。
- 裝置最大配置、最小配置與缺件配置。
- 正常啟動、Recovery 與 Watchdog 路徑。
- 支援的低功耗與 Resume 路徑。
- IOMMU 開啟／關閉及相關外接裝置情境。

### 8.12.3 Pass／Fail 判定

測試項目應定義可觀察且可重複的判定依據，例如：

- POST Code 是否到達預期階段。
- 關鍵 API 是否回傳成功。
- 指定暫存器欄位是否符合預期。
- PCI／ACPI 裝置是否正確呈現。
- Link Width／Speed 與 Error Counter 是否符合門檻。
- OS 是否無新增的裝置、IOMMU、ACPI 或 WHEA 錯誤。
- 鎖定後是否無法從非授權路徑修改設定。

「可以進入作業系統」不應作為唯一通過條件。

### 8.12.4 建議保留的證據

- 完整 Serial Log 與時間戳記。
- POST Code 或 Trace 資料。
- 關鍵 Policy Dump。
- PCI Configuration Space、ACPI Table 與 Memory Map。
- 關鍵暫存器 Before／After Dump。
- OS Event Log、Kernel Log 或裝置管理資訊。
- 失敗與正常樣本的差異比較。

### 8.12.5 與 Silicon Debug 工具的協同

當 Serial Log 只能顯示結果，無法呈現 Link Training、Fabric State Machine、Power Transition 或內部錯誤來源時，應使用 Silicon Vendor 支援且適用於該平台的 Trace 機制，例如 Trace Hub、UART over eSPI、Vendor Training Log、SMU／PMU Debug、JTAG 或平台專屬 Trace Port。工具名稱、可用性與存取權限依供應商、SKU 及產品安全狀態而異。

#### Trace 啟用原則

- 在開發 BIOS 保留由 Build Flag、受控 Setup 選項或簽章 Debug Policy 啟用 Trace 的機制。
- 量產版本預設關閉，並確認 Debug Interface、Trace Buffer 與解鎖路徑符合產品 Threat Model。
- 啟用 Trace 時記錄 Buffer Size、輸出介面、事件遮罩、觸發條件及是否影響時序。
- 先以最小事件集合重現問題，再逐步增加 Trace 類別，避免大量資料改變問題時序或造成關鍵事件被覆寫。

#### 建立共同時間軸

Silicon Trace、POST Code 與 Serial Log 應具有可對齊的同步點。可在呼叫重要 Silicon API 前後寫入相同的 Marker ID，並同時輸出至 Serial、POST Code 與 Trace Channel。

```text
T+000123456 us POST=0x4A TRACE_MARKER=0x104A API=DmiInit Phase=Enter
T+000126902 us POST=0x4B TRACE_MARKER=0x104B API=DmiInit Phase=Exit Status=EFI_SUCCESS
```

若不同資料源使用不同 Clock Domain，應記錄各自的頻率、起始點、Wrap-around 條件與同步 Marker。分析時應先校正時間軸，再判斷事件的先後關係。

#### Raw Data 的可重現性

每份 Trace 應一併保存：

- 原始 Binary／Raw Data，不只保存解碼後文字。
- Decoder／Analysis Tool 名稱與版本。
- Symbol、Register Dictionary、配置檔與事件定義版本。
- Silicon Package、Microcode、Stepping、Board 與 BIOS Build ID。
- 擷取起訖條件、Buffer 模式、事件遮罩與觸發方式。
- 與該次擷取對應的完整 Serial Log、POST Code 與 Policy Dump。

#### 工具侵入性檢查

Trace、JTAG Halt、額外 UART 輸出或高層級 Debug Log 可能改變 Boot Timing、Watchdog 行為、Power State 或 Link Training 結果。若問題只在 Trace 開啟或關閉時出現，應將此差異視為觀測結果的一部分，並使用較低侵入性的事件遮罩、較大 Buffer 或硬體 Trigger 重新驗證。

## 8.13 常見問題與排查方向

### 8.13.1 系統停在 Pre-Memory

優先確認：

1. Reset Cause、POST Code 與最後一筆有效 Log。
2. 早期 Debug Port、Clock、Reset 與 Boot Device 是否可用。
3. Board ID、Strap、Fuse 與 Pre-Memory Policy 是否合理。
4. Silicon Init 回傳狀態與 Assert 資訊。
5. 是否套用錯誤的 SKU、Stepping 或 Board 設定。
6. 最近變更是否涉及記憶體前置條件或 Write Once 暫存器。

#### 8.13.1.1 Silicon Init 失敗的除錯路徑與資料收集

當 Silicon Init API 回傳失敗或系統停在 API 內部時，建議依「先保存證據、再縮小差異、最後驗證假設」的順序處理。

1. **固定失敗位置**
   - 記錄呼叫的 API 名稱、進入與離開檢查點。
   - 保存完整 `EFI_STATUS`、Vendor Status、Assert 資訊與 `DEBUG_ERROR`／`DEBUG_WARN` 輸出。
   - 確認錯誤發生於 API 呼叫前、API 內部，或 API 返回後的資料處理。

2. **建立可重現條件**
   - 記錄 Cold Boot、Warm Reset、AC Cycle 或 Resume 路徑。
   - 固定 BIOS、Silicon Package、Microcode、Board、SKU、Stepping 與 Setup 設定。
   - 至少重複測試三次，確認檢查點與錯誤碼是否一致。

3. **保存最終 Policy Dump**
   - Dump 送入 Silicon Init 前的最終結構，而非只保存 Default Policy。
   - 標示 Policy Revision、結構大小、來源與每一層 Override。
   - 過濾可能包含金鑰、認證資料或其他敏感資訊的欄位。

4. **與已知可開機版本進行差異比較**
   - 比對 Default、Board、Setup 合併後的最終 Policy。
   - 比對 Silicon Package、Microcode、PCD、Setup Default 與 Board Detection 結果。
   - 將差異依 Clock／Power、Lane／Port、Memory Dependency、Security 與 Debug 類別分組，避免一次回退大量無關變更。

5. **檢查初始化時序與暫存器屬性**
   - 確認是否有 Write Once、Lockable 或具有 Side Effect 的暫存器被提前或重複寫入。
   - 確認呼叫順序、前置 Clock／Power／Reset 條件及必要 PPI 是否符合 Silicon 規格。
   - 比對失敗前後的關鍵暫存器，但避免在文件未允許的時間點讀取具有 Clear-on-Read 等副作用的欄位。

6. **核對供應商文件與版本資訊**
   - 查閱目前 Silicon Package 的 Release Note、Integration Guide、錯誤碼定義及適用 Errata。
   - 確認 Policy 結構版本與 Binary／Library 版本相容。
   - 不應直接以其他 Stepping 或不同 Package 版本的錯誤碼說明進行判斷。

7. **保存 Vendor Trace**
   - 若平台支援 Training Log、Trace Hub、Status Code 或供應商專用 Trace，應保存原始輸出及擷取條件。
   - 提交給 Silicon Vendor／FAE 時，應一併提供最小重現步驟、完整版本矩陣、Policy Dump、Serial Log、Trace 與正常／異常版本的差異摘要。

建議保存下列最小資料包：

```text
Failure stage and API name
EFI_STATUS and vendor status
Boot path and reset cause
BIOS build ID and commit ID
Silicon package and microcode version
Board ID, SKU, stepping and BOM revision
Final merged policy dump
Full serial log and POST code history
Training log or Trace Hub output
Last-known-good comparison summary
Reproduction rate and exact steps
```

若 API 沒有返回，仍應區分「無輸出但 CPU 持續執行」、「發生 Reset」、「觸發 Watchdog」、「進入 Dead Loop」與「硬體停止回應」等現象，因為各自需要的觀測工具與後續排查方向不同。

### 8.13.2 裝置未出現在 PCI／ACPI

建議依序比對：

1. 硬體是否裝載且供電、Clock、Reset 正常。
2. Fuse／SKU 是否支援。
3. Pin Mux、Lane Ownership 與 Controller Policy。
4. 裝置是否被 Disable、Hide 或 Power Gate。
5. PCI Enumeration 與資源配置是否成功。
6. ACPI Table 是否依實際裝置狀態建立。

### 8.13.3 Link 降速或不穩定

可收集：

- 預期與實際 Link Width／Speed。
- Training State、Correctable／Uncorrectable Error Counter。
- ASPM、L1 Substates 與 Clock Request 設定。
- 不同線材、轉接板、插槽與裝置的交叉測試結果。
- Silicon／Board Stepping、原理圖及相關 Errata。

### 8.13.4 IOMMU 或 DMA 相關錯誤

可檢查：

- IOMMU 是否在硬體、BIOS Policy 與 ACPI Table 三處保持一致。
- Reserved Memory Region 是否涵蓋必要裝置。
- 裝置 Requester ID、PCI 拓樸與隔離群組是否正確。
- Firmware 與 OS Log 中的 Fault Address、Source ID 與 Reason Code。
- 問題是否只在特定 Boot Path 或 Hot Plug 情境發生。

### 8.13.5 鎖定後功能異常

可建立鎖定前後的 A／B 測試，確認：

- 哪一個 Lock Bit 與異常首次出現具有時間關聯。
- 是否有模組在鎖定後仍嘗試修改設定。
- S3 Resume、Firmware Update 或 Recovery 是否需要例外路徑。
- 鎖定順序是否符合 Silicon 規格。

## 8.14 安全性與相容性注意事項

### 8.14.1 安全性

- 未使用的外部介面應依產品需求停用或限制。
- Debug、Manufacturing 與 Service Mode 應有明確進入條件及量產關閉方式。
- 安全相關 Policy 不應因一般 Setup Variable 損毀而回復至較寬鬆狀態。
- Flash、防寫、SMM、DMA 與 GPIO Lock 應納入整體 Threat Model。
- Log 不應輸出金鑰、認證資料或其他敏感內容。

### 8.14.2 相容性

- 更新 Silicon Package 或工具鏈後，應比較 Policy 結構、預設值與 API 行為。
- 新舊 BIOS 版本間應確認 NVRAM 相容性與預設值遷移。
- 不同 OS 版本可能使用不同 ACPI、IOMMU 或電源管理路徑。
- 降版、更新中斷與設定資料損毀時，應有可測試的復原方式。
- 保留既有產品行為時，應同時評估是否延續已知限制或安全風險。

## 8.15 建議圖表與資料

正式版本建議補充下列內容：

1. **初始化階段圖**：Reset、SEC、PEI Pre-Memory、PEI Post-Memory、DXE、BDS 與 End-of-POST。
2. **Policy 資料流圖**：Default、PCD、Board ID、Fuse、Setup Variable 至 Silicon Init 的合併過程。
3. **模組關係圖**：CPU、Host Bridge、DMI／Fabric、PCH、Root Port 與內建周邊。
4. **資源配置圖**：DRAM、Reserved Memory、MMIO、Above 4G MMIO 與 PCI Resource Window。
5. **鎖定時序表**：設定項目、設定者、Lock Owner、鎖定階段與 Reset 條件。
6. **平台差異表**：SKU、Board Revision、Stepping、功能支援與 Errata。
7. **正常／異常 Log 對照**：以相同檢查點標示差異，並避免只截取錯誤前後少量資訊。

## 8.16 章節維護檢查清單

章節發布前，負責人應確認：

- [ ] 平台名稱、SKU、Board Revision 與 Silicon Stepping 已更新。
- [ ] 所有暫存器名稱、位元定義與 Reset 條件已依適用版本規格確認。
- [ ] Policy 欄位與程式中的實際結構一致。
- [ ] 初始化排序與模組責任已由程式流程或 Log 驗證。
- [ ] Cold Boot、Warm Reset、AC Cycle、Update 與支援的 Resume 路徑已覆蓋。
- [ ] Errata 與 Workaround 有版本、適用範圍及移除條件。
- [ ] 安全鎖定的時點與驗證方法已記錄。
- [ ] 圖表、Log 與測試結果不含敏感資料。
- [ ] 參考文件的版本與內部連結仍有效。

## 8.17 參考資料

正式版本可依平台適用性補充：

- UEFI Specification
- UEFI Platform Initialization Specification
- EDK II 文件與來源碼位置
- ACPI Specification
- PCI Express Base Specification
- TCG 相關規格
- SMBIOS Specification
- Silicon Vendor Datasheet、Programming Guide、BIOS Writer's Guide 與 Errata
- 專案內部架構文件、Issue、變更紀錄與測試報告

> 注意：引用供應商或標準組織文件時，應標示文件名稱、版本、發布日期及適用平台，避免不同版本的欄位定義混用。
