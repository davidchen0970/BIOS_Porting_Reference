## 4. HOB、Protocol、PPI 與 UEFI Service

### 適用範圍

本章說明 UEFI Platform Initialization 架構中，HOB、PPI、Protocol 與各類 Service 的角色及彼此關係，涵蓋 PEI 到 DXE 的資料交接、模組間介面發布與查找、Handle Database、Driver Binding，以及常見生命週期與記憶體所有權問題。

本章聚焦於 BIOS 移植、平台初始化與早期開機問題排查。SEC 的 CPU 初始狀態、完整 DXE Dispatcher 演算法、BDS 開機選項政策、作業系統 Runtime 行為及特定晶片供應商私有介面，僅在與本章主題直接相關時補充。

### 適用讀者

- 負責 BIOS、UEFI、EDK II、平台初始化或韌體移植的開發與整合人員。
- 需要排查 PEI、DXE、BDS 階段模組相依、介面找不到、HOB 資料錯誤或 Driver Binding 問題的人員。
- 撰寫 PEIM、DXE Driver、UEFI Driver、Application 或平台共用 Library 的工程人員。

### 快速導覽

- [理解 PEI 到 DXE 的資料與介面模型](#41-整體架構與資料流)：HOB、PPI、Protocol、Service 的責任邊界。
- [查閱 HOB 的建立與消費方式](#42-hob-list-的建立傳遞與消費)：HOB List、PHIT、End of HOB List 與 DXE IPL 交接。
- [選擇合適的 HOB 類型](#43-常用-hob-類型與自訂-guid-hob)：資源、記憶體、Firmware Volume 與 GUID HOB。
- [理解 PPI Database 與 Notify](#44-ppi-database-與-notify-機制)：Install、Reinstall、Locate、Notify 與 PEIM 相依。
- [管理 Protocol 與 Handle](#45-protocol-installlocateopen-與-close)：Protocol Database、Protocol Notification 與使用追蹤。
- [區分 UEFI 各類 Service](#46-boot-serviceruntime-service-與-dxe-service)：可使用階段、記憶體限制與 ExitBootServices 邊界。
- [理解 Driver Binding](#47-handleagentcontroller-與-driver-binding)：Supported、Start、Stop 與控制器拓樸。
- [排查生命週期與所有權問題](#48-生命週期記憶體所有權與除錯)：常見錯誤、觀測點與驗證順序。

---

### 4.1 整體架構與資料流

HOB、PPI 與 Protocol 都可傳遞資訊，但服務的階段與使用模型不同：

| 機制 | 主要階段 | 主要用途 | 典型特性 |
|---|---|---|---|
| HOB | PEI 建立，DXE 消費 | 跨階段傳遞平台與資源資訊 | 線性資料結構，交接後通常視為唯讀 |
| PPI | PEI | PEIM 之間發布功能或狀態 | 由 GUID 識別，透過 PEI Services 管理 |
| Protocol | DXE、BDS、UEFI 執行環境 | Driver、Application 與 Controller 之間提供介面 | 安裝在 Handle 上，可追蹤開啟關係 |
| PEI Services | PEI | HOB、PPI、記憶體、FV 與 PEIM 支援服務 | 只適用 PEI 執行環境 |
| Boot Services | DXE 至 ExitBootServices 前 | 記憶體、事件、Protocol、Image 與裝置服務 | ExitBootServices 成功後不得再呼叫 |
| Runtime Services | 開機期間與 OS Runtime | Variable、Time、Reset、Capsule 等服務 | Runtime 映射後仍需遵守 Runtime 記憶體規則 |
| DXE Services | DXE Core 內部與 DXE Driver | GCD、Dispatcher、FV 等 PI 層服務 | 不等同於 UEFI Boot Services |

簡化資料流如下：

```mermaid
flowchart LR
    SEC[SEC] --> PEI[PEI Foundation / PEIM]
    PEI -->|Install / Locate / Notify| PPI[PPI Database]
    PEI -->|Build HOB| HOB[HOB List]
    HOB --> IPL[DXE IPL]
    IPL --> DXE[DXE Core / DXE Drivers]
    DXE -->|Install / Open / Locate| PROTO[Handle & Protocol Database]
    DXE --> BDS[BDS / OS Loader]
    BDS -->|ExitBootServices| OS[OS Runtime]
```

判讀問題時，可先依兩個問題分類：

1. 問題是否跨越 PEI 與 DXE？若是，先檢查 HOB 的建立內容與交接時點。
2. 問題是否為同一階段內的模組相依？PEI 優先檢查 PPI，DXE 優先檢查 Protocol、Handle 與 Driver Binding。

### 4.2 HOB List 的建立、傳遞與消費

#### 4.2.1 HOB 的角色

HOB（Hand-Off Block）是 PI 架構使用的交接資料結構。PEI 階段完成最低限度的平台初始化後，透過 HOB List 將記憶體配置、資源描述、Firmware Volume、CPU 能力、開機模式及平台自訂資料交給 DXE。

HOB 適合傳遞「DXE 必須知道，但無法或不應重新探測」的資訊，例如：

- PEI 已辨識的系統記憶體與保留區域。
- DXE Core 或模組所在的 Firmware Volume。
- SEC／PEI 使用的 Stack、Module 或特殊記憶體配置。
- 平台早期初始化結果、Silicon Policy 摘要或一次性狀態。

HOB 不適合取代 DXE Protocol。若資料需要在 DXE 期間持續更新、提供函式介面或管理多個 Controller，通常應在 DXE 建立對應 Protocol 或其他長期服務。

#### 4.2.2 HOB List 結構

HOB List 為連續排列的可變長度資料項目。每個 HOB 具有通用 Header，至少包含 HOB Type 與 HOB Length。走訪時依目前 HOB 的長度移到下一個 HOB，直到 `EFI_HOB_TYPE_END_OF_HOB_LIST`。

第一個 HOB 為 PHIT（Phase Handoff Information Table），用來描述開機模式、HOB List 邊界與可用記憶體資訊。建立或搬移 HOB List 時，需保持下列條件：

- HOB 位址與長度符合規格要求的對齊。
- `HobLength` 不得為 0，也不得超出 HOB List 邊界。
- PHIT 的 FreeMemoryTop、FreeMemoryBottom 與 EndOfHobList 必須一致。
- 最後必須存在 End of HOB List，避免消費端無限走訪或讀到無效記憶體。

#### 4.2.3 PEI 建立與 DXE 消費

EDK II 中常透過 HobLib 建立及查找 HOB。概念性流程如下：

```c
VOID *Data;

Data = BuildGuidHob (&gMyPlatformInfoGuid, sizeof (MY_PLATFORM_INFO));
if (Data == NULL) {
  return EFI_OUT_OF_RESOURCES;
}

CopyMem (Data, &PlatformInfo, sizeof (MY_PLATFORM_INFO));
```

DXE 消費端應先確認 HOB 存在，再驗證長度、版本及內容：

```c
VOID                  *GuidHob;
MY_PLATFORM_INFO      *Info;
UINTN                  DataSize;

GuidHob = GetFirstGuidHob (&gMyPlatformInfoGuid);
if (GuidHob == NULL) {
  return EFI_NOT_FOUND;
}

DataSize = GET_GUID_HOB_DATA_SIZE (GuidHob);
if (DataSize < sizeof (MY_PLATFORM_INFO)) {
  return EFI_COMPROMISED_DATA;
}

Info = GET_GUID_HOB_DATA (GuidHob);
```

對自訂 HOB，建議資料結構包含 `Revision`、`Length` 與保留欄位，使不同 BIOS 版本或 SKU 能判斷相容性。

### 4.3 常用 HOB 類型與自訂 GUID HOB

| HOB 類型 | 主要內容 | BIOS 移植時的檢查重點 |
|---|---|---|
| PHIT HOB | Boot Mode、HOB List 與記憶體邊界 | 第一個 HOB、邊界一致性、Boot Mode |
| Resource Descriptor HOB | System Memory、MMIO、I/O、保留資源 | Base／Length、重疊、屬性與 GCD 對應 |
| Memory Allocation HOB | Stack、Module、Boot／Runtime 記憶體配置 | Memory Type、Owner GUID、是否被後續覆寫 |
| Firmware Volume HOB | FV Base 與 Length | FV 是否可讀、範圍是否在有效資源內 |
| CPU HOB | 實體位址與 I/O 位址寬度 | CPU 能力是否與平台及 GCD 設定一致 |
| GUID Extension HOB | 平台或元件自訂資料 | GUID 唯一性、Revision、Length、輸入驗證 |

#### 4.3.1 Resource Descriptor 與 Memory Allocation 的差異

Resource Descriptor HOB 描述「平台有哪些資源」，Memory Allocation HOB 描述「其中哪些區域已分配給特定用途」。兩者的 Base／Length 不一定一一對應，但不得造成自相矛盾的資源視圖。

常見問題包括：

- DRAM 被描述為 System Memory，但部分保留區未另行標示，後續可能被配置器使用。
- MMIO Hole 與 System Memory 重疊，造成 PCI 資源或 DMA 問題。
- PEI Stack、DXE Core 或 Runtime 區域的 Memory Type 不正確。
- 同一範圍由多個 HOB 重複描述，且屬性不同。

#### 4.3.2 自訂 GUID HOB 設計原則

- GUID 代表資料契約，不應因產品名稱相近而重複使用。
- 結構需避免未定義 padding，必要時明確定義欄位寬度。
- 不在 HOB 中保存只對 PEI 有效的指標，例如 Temporary RAM 位址。
- 若必須保存位址，需明確說明實體位址、虛擬位址或 Offset 語意。
- 消費端不得只依 `sizeof()` 假設版本一致，應檢查實際資料長度。
- 敏感資訊、金鑰或可重放認證資料不應以一般 GUID HOB 長期暴露。

### 4.4 PPI Database 與 Notify 機制

#### 4.4.1 PPI 的定位

PPI（PEIM-to-PEIM Interface）是 PEI 階段的模組介面。PEIM 透過 GUID 發布 PPI，其他 PEIM 使用 `LocatePpi()` 取得介面。PPI 可表示函式集合、資料介面或初始化完成狀態。

常用 PEI Services：

| Service | 用途 | 注意事項 |
|---|---|---|
| `InstallPpi()` | 新增 PPI Descriptor | Descriptor 與 Interface 的生命週期需涵蓋後續使用期間 |
| `ReInstallPpi()` | 以新 Descriptor 取代既有 PPI | 舊介面持有者是否仍保存舊指標需評估 |
| `LocatePpi()` | 依 GUID 與 Instance 查找 PPI | 同 GUID 可有多個 Instance，不應固定假設只有一個 |
| `NotifyPpi()` | 註冊 PPI 出現時的通知 | Dispatch Notify 與 Callback Notify 的執行時點不同 |

#### 4.4.2 Notify 與 Depex 的差異

- Depex 決定 PEIM 何時具備被 Dispatcher 執行的條件。
- Notify 用來在指定 PPI 已存在或稍後安裝時執行回呼。

若模組的主功能完全依賴某個 PPI，通常優先以 Depex 表達。若模組可先初始化，再於某 PPI 出現時補做動作，Notify 較合適。過度依賴 Notify 可能使執行順序難以追蹤，也容易形成重入或隱性相依。

#### 4.4.3 常見時序

```mermaid
sequenceDiagram
    participant A as PEIM A
    participant P as PEI Foundation
    participant B as PEIM B
    A->>P: InstallPpi(Guid X)
    B->>P: LocatePpi(Guid X)
    P-->>B: Interface Pointer
    B->>P: NotifyPpi(Guid Y)
    A->>P: InstallPpi(Guid Y)
    P-->>B: Invoke Notify Function
```

排查 PPI 問題時，建議記錄 GUID、Instance、Descriptor Flags、安裝者、查找者、Notify 類型及返回狀態。

### 4.5 Protocol Install、Locate、Open 與 Close

#### 4.5.1 Protocol 與 Handle Database

DXE 中的 Protocol 是由 GUID 識別的介面，安裝在 EFI Handle 上。一個 Handle 可安裝多個 Protocol，用來表示同一個 Controller、Image 或邏輯物件的能力集合。

常用服務及適用情境：

| Service | 適用情境 | 主要風險 |
|---|---|---|
| `InstallProtocolInterface()` | 在新或既有 Handle 發布單一 Protocol | 安裝失敗後的資源回收 |
| `InstallMultipleProtocolInterfaces()` | 一次發布多個相關 Protocol | 中途失敗時的完整性與回復 |
| `UninstallProtocolInterface()` | 移除不再提供的 Protocol | 仍被其他 Agent 開啟時可能失敗 |
| `LocateProtocol()` | 只需任一符合 GUID 的介面 | 多 Instance 時結果不代表特定 Controller |
| `LocateHandleBuffer()` | 列出符合條件的 Handle | 回傳 Buffer 由呼叫端釋放 |
| `HandleProtocol()` | 已知 Handle 時取得 Protocol | 不記錄 Driver Model 的開啟關係 |
| `OpenProtocol()` | Driver Binding 或需追蹤使用關係 | Attributes、AgentHandle、ControllerHandle 必須正確 |
| `CloseProtocol()` | 解除先前 Open 關係 | 參數需與 Open 時一致 |

#### 4.5.2 LocateProtocol 與 OpenProtocol 的選擇

`LocateProtocol()` 適合取得全域、單一 Instance 或不屬於特定 Controller 的服務。UEFI Driver Model 中若 Driver 要管理 Controller，應使用 `OpenProtocol()` 並選擇合適 Attributes，使 Core 能追蹤 Agent、Controller 與 Child Controller 關係。

常見 Attributes：

- `EFI_OPEN_PROTOCOL_GET_PROTOCOL`：取得介面，不宣告管理權。
- `EFI_OPEN_PROTOCOL_TEST_PROTOCOL`：只測試 Protocol 是否存在。
- `EFI_OPEN_PROTOCOL_BY_DRIVER`：Driver 宣告管理 Controller 的介面。
- `EFI_OPEN_PROTOCOL_BY_CHILD_CONTROLLER`：建立 Bus Driver 與 Child Handle 關係。
- `EFI_OPEN_PROTOCOL_EXCLUSIVE`：要求排他存取，使用前需確認對其他 Driver 的影響。

#### 4.5.3 Protocol Notification

當模組需要等待某 Protocol 安裝，可建立 Event 並以 `RegisterProtocolNotify()` 註冊。回呼被觸發後，仍需反覆取得尚未處理的 Handle，不宜假設一次通知只對應一個 Protocol Instance。

回呼函式應保持精簡，避免長時間阻塞、在高 TPL 執行不允許的服務，或再次觸發同一通知而形成遞迴。

### 4.6 Boot Service、Runtime Service 與 DXE Service

| 類別 | 提供者／入口 | 有效期間 | 典型功能 |
|---|---|---|---|
| PEI Services | PEI Foundation | PEI | PPI、HOB、PEIM、FV、PEI Memory |
| Boot Services | UEFI System Table 的 BootServices | DXE 至 ExitBootServices 成功前 | Memory、Event、Protocol、Image、Controller、Timer |
| Runtime Services | UEFI System Table 的 RuntimeServices | 開機期間與 OS Runtime | Variable、Time、Reset、Capsule、Virtual Address |
| DXE Services | DXE Services Table | DXE | GCD Memory／I/O Space、FV、Dispatcher |

#### 4.6.1 ExitBootServices 邊界

OS Loader 呼叫 `GetMemoryMap()` 取得 Memory Map 與 MapKey，再呼叫 `ExitBootServices()`。兩次呼叫之間若配置或釋放記憶體，MapKey 可能失效，因此 Loader 通常需要重新取得 Memory Map 後重試。

ExitBootServices 成功後：

- 不得再呼叫 Boot Services。
- Boot Services Code／Data 可由 OS 回收。
- Runtime Driver 只能依賴 Runtime Code／Data 與允許的 Runtime Services。
- 事件、Protocol 或一般 Pool 指標若仍被 Runtime 路徑引用，可能造成轉址後失效。

#### 4.6.2 Runtime Driver 注意事項

- 需要跨越 ExitBootServices 的函式與資料應放在正確 Runtime Memory Type。
- 若 OS 進行虛擬位址切換，需處理 `SetVirtualAddressMap()` 相關轉址需求。
- Runtime 路徑不應間接呼叫 Boot Services Library。
- Variable、Capsule、Reset 等介面需納入權限、輸入長度與非預期重入檢查。

### 4.7 Handle、Agent、Controller 與 Driver Binding

#### 4.7.1 基本角色

- Handle：Protocol 的容器，本身是不透明識別值。
- Agent Handle：發起 `OpenProtocol()` 的 Image，通常是 Driver Binding 所屬 Driver Image。
- Controller Handle：Driver 要管理的裝置或匯流排控制器。
- Child Handle：Bus Driver 建立的子裝置 Handle。

#### 4.7.2 Driver Binding 三個入口

| 函式 | 目的 | 檢查重點 |
|---|---|---|
| `Supported()` | 判斷 Driver 能否管理 Controller | 不應留下持續性副作用；暫時開啟的 Protocol 要關閉 |
| `Start()` | 開啟 Controller Protocol、初始化硬體、安裝新 Protocol | 任何失敗路徑都需逆序回收資源 |
| `Stop()` | 停止 Child 或 Controller 並解除 Protocol | Child 數量、Open 關係與 Uninstall 順序需一致 |

典型流程：

```mermaid
flowchart TD
    A[ConnectController] --> B[Supported]
    B -->|Unsupported| X[Try Next Driver]
    B -->|Supported| C[Start]
    C --> D[Open BY_DRIVER]
    D --> E[Initialize Controller]
    E --> F[Install Produced Protocol]
    F --> G[Optional Child Handles]
    G --> H[Stop]
    H --> I[Uninstall Child / Produced Protocol]
    I --> J[Close Consumed Protocol]
```

`Supported()` 成功不保證 `Start()` 一定成功，因為資源、硬體狀態或其他 Driver 的開啟關係可能在兩者之間改變。

### 4.8 生命週期、記憶體所有權與除錯

#### 4.8.1 所有權原則

每個配置、安裝、開啟與註冊動作，都應能回答下列問題：

1. 由誰建立？
2. 哪些模組可持有或修改？
3. 有效期限到哪一個階段或事件？
4. 成功與失敗路徑由誰回收？
5. Reset、S3 Resume、Capsule Update 或 ExitBootServices 後是否仍有效？

| 資源 | 建立方式 | 一般回收責任 |
|---|---|---|
| Pool／Pages | AllocatePool／AllocatePages | 配置者或明確接手所有權者 |
| LocateHandleBuffer 結果 | LocateHandleBuffer | 呼叫端 `FreePool()` |
| Event | CreateEvent／CreateEventEx | 建立者 `CloseEvent()`，或依事件生命週期保留 |
| Protocol Interface | InstallProtocolInterface | 安裝者在停止服務前 Uninstall |
| Open Protocol 關係 | OpenProtocol | 同一 Agent／Controller 依契約 CloseProtocol |
| GUID HOB 資料 | BuildGuidHob | 隨 HOB List 交接，消費端通常不釋放 |

#### 4.8.2 常見問題與排查

| 現象 | 優先觀測點 | 可能方向 | 驗證方式 |
|---|---|---|---|
| DXE 找不到 PEI 資料 | HOB List、GUID、資料長度 | HOB 未建立、建立太晚、GUID 不一致 | Dump HOB，確認 Build 與 Get 使用同一 GUID |
| PEIM 一直未執行 | Depex、PPI Database、Dispatcher Log | 必要 PPI 未安裝或 Depex 錯誤 | 開啟 PEI Dispatcher／PPI DEBUG Log |
| Notify 未進入 | Notify Flags、註冊時點、GUID | Descriptor 類型錯誤或回呼已在其他時點執行 | 記錄註冊與安裝順序 |
| Protocol 已安裝但 Locate 失敗 | Handle Database、Protocol GUID | GUID 實例不一致、安裝失敗未檢查 | 列出 Handle 與 Protocol，檢查 Status |
| UninstallProtocol 失敗 | OpenProtocolInformation | 仍有 Agent 開啟 | 查 Agent／Controller／Attributes 關係 |
| Driver Supported 成功但 Start 失敗 | Start 第一個失敗點 | Protocol 競爭、資源不足、硬體未就緒 | 保留完整 Status、Open 關係與硬體狀態 |
| ExitBootServices 偶發失敗 | Memory Map、MapKey | 取得 MapKey 後仍有配置或事件活動 | 重新 GetMemoryMap，追蹤配置來源 |
| OS Runtime 階段當機 | Runtime Memory、轉址清單 | 保存 Boot Services 指標或未轉換 Pointer | 檢查 Memory Type 與 Virtual Address Change |
| Warm Reset 才失敗 | 殘留狀態、Boot Mode、S3／Reset 路徑 | 初始化假設只適用 Cold Boot | 比對 Cold／Warm Log 與 HOB／Variable |

#### 4.8.3 建議除錯輸出

```text
[PEI][PPI] Install GUID=<guid> Ppi=<ptr> Flags=<flags> Status=<status>
[PEI][HOB] Build Type=<type> GUID=<guid> Base=<base> Size=<size>
[DXE][PROTO] Install Handle=<handle> GUID=<guid> If=<ptr> Status=<status>
[DXE][OPEN] Agent=<agent> Controller=<ctrl> GUID=<guid> Attr=<attr> Status=<status>
[DXE][DRV] Supported/Start/Stop Controller=<ctrl> RemainingChildren=<n> Status=<status>
[UEFI][EBS] MapKey=<key> DescriptorSize=<size> Count=<n> Status=<status>
```

Log 應避免只輸出「成功／失敗」，至少保留階段、GUID、Handle、位址、長度、Attributes 與 `EFI_STATUS`。

### 4.9 實作與排查入口

EDK II 常見查閱位置如下，實際路徑可能隨 branch 調整：

| 主題 | 常見位置／工具 |
|---|---|
| HOB 定義 | `MdePkg/Include/Pi/PiHob.h` |
| PEI Services 與 PPI Descriptor | `MdePkg/Include/Pi/PiPeiCis.h` |
| UEFI Boot／Runtime Services | `MdePkg/Include/Uefi/UefiSpec.h` |
| Driver Binding Protocol | `MdePkg/Include/Protocol/DriverBinding.h` |
| HOB Library | `MdePkg/Library/PeiHobLib`、`DxeHobLib` 或 branch 對應實作 |
| PEI Core | `MdeModulePkg/Core/Pei` |
| DXE Core 與 Handle Database | `MdeModulePkg/Core/Dxe` |
| UEFI Shell | `dh`、`drivers`、`devices`、`openinfo`、`memmap`、`dmpstore` |

建議原始碼搜尋：

```bash
grep -R "BuildGuidHob" -n --include='*.c' --include='*.h' .
grep -R "InstallPpi\|NotifyPpi\|LocatePpi" -n --include='*.c' .
grep -R "InstallProtocolInterface\|OpenProtocol" -n --include='*.c' .
grep -R "gEfiDriverBindingProtocolGuid" -n --include='*.inf' --include='*.c' .
```

### 4.10 驗證與測試重點

#### 4.10.1 測試矩陣

| 類別 | 建議覆蓋 |
|---|---|
| 開機型態 | Cold Boot、Warm Reset、AC Cycle、S3 Resume（若支援） |
| 韌體狀態 | 更新前、更新後、Recovery、設定恢復預設值 |
| 平台差異 | 不同 SKU、記憶體容量、CPU Stepping、PCIe 裝置組合 |
| 資源條件 | 低記憶體、裝置缺席、Protocol／PPI 延遲出現 |
| 錯誤注入 | HOB 長度錯誤、PPI 缺席、Protocol 被占用、Start 中途失敗 |
| OS 交接 | 多次 GetMemoryMap、ExitBootServices 重試、Runtime Variable／Reset |

#### 4.10.2 Pass／Fail 判定

- HOB List 可完整走訪，無零長度、越界、重疊或缺少結尾項目。
- 必要 PPI 與 Protocol 在預期階段出現，Instance 數量符合平台設計。
- Driver Binding 可完成 Connect、Disconnect、Reconnect，不殘留 Open 關係。
- 所有失敗路徑可回收已配置資源及已安裝介面。
- ExitBootServices 後無 Boot Services 存取，Runtime 功能在支援的 OS 上可重複執行。
- Cold Boot、Warm Reset 與不同 SKU 的 Log 差異均可由設計解釋。

### 4.11 安全性與相容性注意事項

- 將 HOB、Protocol 與 Runtime Service 的輸入視為跨信任邊界資料，檢查長度、版本、範圍與 Integer Overflow。
- 不透過 GUID HOB、Variable 或一般 DEBUG Log 暴露金鑰、密碼、Token 或敏感量測資料。
- 安裝或覆寫安全相關 Protocol 前，確認呼叫階段、SMM／MM 邊界及存取權限。
- 自訂 GUID 資料結構應具版本策略，避免 BIOS 更新後舊 DXE Driver 或 Capsule 路徑誤讀。
- Resource Descriptor 與 Memory Type 錯誤可能影響 DMA、記憶體保護與 OS 資源管理，需納入安全測試。
- 規格版本、EDK II branch、Silicon Package 與編譯器版本應一併記錄，避免只比較**名稱相同但語意不同**的介面。

### 4.12 本章檢查清單

- [ ] 已標示每個自訂 HOB 的 Producer、Consumer、GUID、Revision 與資料長度。
- [ ] HOB List 邊界、對齊、End of HOB List 與資源重疊已驗證。
- [ ] PPI 的 Install、Locate、Notify 與 Depex 關係有文件可回查。
- [ ] Protocol 安裝失敗與 Driver Start 中途失敗都有逆序回收流程。
- [ ] `OpenProtocol()` 的 Agent、Controller、Attributes 與 `CloseProtocol()` 成對。
- [ ] Driver Binding 的 Supported、Start、Stop 可重複執行且不殘留狀態。
- [ ] ExitBootServices 前後的服務使用與 Memory Type 已分界。
- [ ] Runtime Pointer 與 Virtual Address Change 處理符合平台需求。
- [ ] 已覆蓋 Cold Boot、Warm Reset、AC Cycle、更新前後及不同 SKU。
- [ ] Log 可識別 GUID、Handle、Instance、Address、Length、Attributes 與 EFI_STATUS。

### 4.13 本章重點

- HOB 用於 PEI 到 DXE 的資料交接；PPI 與 Protocol 分別服務 PEI 與 DXE／UEFI 執行環境。
- HOB 是資料契約，不應保存 Temporary RAM 指標，也不應取代需要持續互動的 Protocol。
- PPI Notify 與 PEIM Depex 解決的問題不同，應依相依是否為模組執行前提來選擇。
- Protocol 不只是一個函式指標；Handle、Agent、Controller 與 Open Attributes 共同描述 Driver Model 關係。
- ExitBootServices 是明確生命週期邊界，Runtime 路徑不可保留對 Boot Services Code／Data 的依賴。
- 排查時先確認階段，再沿著 Producer、Database、Consumer、所有權與回收順序縮小範圍。

### 4.14 參考資料

- UEFI Forum, UEFI Specification: https://uefi.org/specifications
- UEFI Forum, Platform Initialization Specification: https://uefi.org/specifications
- TianoCore EDK II: https://github.com/tianocore/edk2
- TianoCore EDK II Documentation: https://github.com/tianocore-docs
- 專案採用的 EDK II branch、Silicon Package、平台設計文件、Issue 與測試報告。

> 規格名稱、函式名稱與欄位定義應以專案採用的 UEFI／PI Specification 及 EDK II branch 為準。本章提供可重複使用的 BIOS 移植與排查架構，不取代平台供應商文件。
