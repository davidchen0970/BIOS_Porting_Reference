# Part 5：ACPI、SMBIOS 與作業系統介面 導讀

> 第一次閱讀本 Part 時，建議先讀完本導讀，再進入各章節。

## 這一 Part 在做什麼

說明韌體如何向 OS 描述硬體、拓樸、電源與平台資料，以及 Runtime Hand-off。

## 為什麼需要這一 Part

BIOS 內裝置可用不代表 OS 可正確接管，Table、AML、Runtime Attribute 或 Hand-off 錯誤都可能造成 OS 異常。

## 讀完應能回答

- ACPI Table 與 AML 如何建立？
- Sleep、Wake、GPE 如何分工？
- SMBIOS 資料來源如何整合？
- OS Loader 與 Runtime Service 如何交接？

## 建議閱讀順序

依 ACPI 架構、ASL／AML、Power State、SMBIOS、Runtime Hand-off 閱讀。

## 與其他 Part 的關係

本 Part 不獨立存在。遇到跨領域問題時，應回到 Part 1 判斷韌體階段，再依硬體初始化、OS 介面、安全、更新或測試等相鄰 Part 交叉查閱。

## 本 Part 章節

- [ACPI 架構與 Table 產生流程](23_acpi_architecture_and_table_generation.md)
- [DSDT、SSDT、AML 與 ASL 除錯](24_dsdt_ssdt_aml_and_asl_debugging.md)
- [Power State、Sleep、Wake 與 Runtime Power Management](25_power_state_sleep_wake_and_runtime_power_management.md)
- [SMBIOS Type 與平台資訊](26_smbios_types_and_platform_information.md)
- [UEFI Runtime Service 與 OS Hand-off](27_uefi_runtime_services_and_os_handoff.md)
