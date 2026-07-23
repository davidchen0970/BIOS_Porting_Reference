# Part 3：EDK II、建置系統與設定資料 導讀

> 第一次閱讀本 Part 時，建議先讀完本導讀，再進入各章節。

## 這一 Part 在做什麼

串起 EDK II 原始碼樹、描述檔、PCD、工具鏈、映像產物、Setup Variable 與 NVRAM。

## 為什麼需要這一 Part

許多「修改未生效」源自 Library、PCD、FDF、Toolchain 或 Default Store，而不是 Driver 本身。

## 讀完應能回答

- DSC、DEC、INF、FDF 各自控制什麼？
- PCD 與 Build Flag 的覆寫優先序為何？
- AutoGen、Link、GenFds 產物如何追查？
- Variable、FTW、Default 與 Migration 如何協作？

## 建議閱讀順序

依 Workspace、描述檔、PCD、建置流程、NVRAM 的順序閱讀。

## 與其他 Part 的關係

本 Part 不獨立存在。遇到跨領域問題時，應回到 Part 1 判斷韌體階段，再依硬體初始化、OS 介面、安全、更新或測試等相鄰 Part 交叉查閱。

## 本 Part 章節

- [EDK II Workspace 與 Package 結構](12_edk_ii_workspace_and_package_structure.md)
- [DSC、DEC、INF 與 FDF 設定](13_dsc_dec_inf_and_fdf_configuration.md)
- [PCD、Build Flag 與平台組態](14_pcd_build_flags_and_platform_configuration.md)
- [工具鏈、建置流程與映像檔產出](15_toolchain_build_flow_and_image_output.md)
- [Setup Variable、NVRAM 與預設值管理](16_setup_variable_nvram_and_default_management.md)
