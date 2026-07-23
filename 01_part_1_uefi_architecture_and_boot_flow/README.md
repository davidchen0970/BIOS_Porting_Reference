# Part 1：UEFI 架構與開機流程 導讀

> 第一次閱讀本 Part 時，建議先讀完本導讀，再進入各章節。

## 這一 Part 在做什麼

建立從 CPU Reset 到 OS Loader 的全貌，先理解 SEC、PEI、DXE、BDS、Runtime 的責任與交接。

## 為什麼需要這一 Part

開機問題會因階段不同而有完全不同的記憶體、服務與觀測方式。這一 Part 是後續所有章節的共同基礎。

## 讀完應能回答

- UEFI 與 PI 規格如何分工？
- 各韌體階段的輸入、輸出與停止點是什麼？
- PPI、Protocol、HOB、Event、Handle 如何使用？
- ExitBootServices 前後的責任如何改變？

## 建議閱讀順序

依編號閱讀。先建立架構全貌，再進入映像配置、階段介面與 Boot Manager。

## 與其他 Part 的關係

本 Part 不獨立存在。遇到跨領域問題時，應回到 Part 1 判斷韌體階段，再依硬體初始化、OS 介面、安全、更新或測試等相鄰 Part 交叉查閱。

## 本 Part 章節

- [UEFI PI 架構與韌體執行階段](01_uefi_pi_architecture_and_firmware_phases.md)
- [SEC、PEI、DXE 與 BDS 開機流程](02_sec_pei_dxe_and_bds_boot_flow.md)
- [Firmware Volume、FFS 與映像檔配置](03_firmware_volume_ffs_and_image_layout.md)
- [HOB、Protocol、PPI 與 UEFI Service](04_hob_protocol_ppi_and_uefi_services.md)
- [Boot Manager、Boot Option 與開機順序](05_boot_manager_boot_option_and_boot_order.md)
