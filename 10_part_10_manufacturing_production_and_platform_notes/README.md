# Part 10：製造、量產與平台特定筆記 導讀

> 第一次閱讀本 Part 時，建議先讀完本導讀，再進入各章節。

## 這一 Part 在做什麼

把研發 BIOS 轉成可量產、可追溯、可維修的產品流程與靜態平台資料。

## 為什麼需要這一 Part

工站、版本、識別資料、權限與鎖定順序不一致，會造成量產及維修風險。

## 讀完應能回答

- Manufacturing Mode 如何控管？
- Board ID／SKU／FRU 如何判定？
- MAC／Serial／UUID 如何 Provision？
- Platform Notes 應記錄哪些靜態事實？

## 建議閱讀順序

依製造流程、平台辨識、Provisioning、Setup、Platform Notes 閱讀。

## 與其他 Part 的關係

本 Part 不獨立存在。遇到跨領域問題時，應回到 Part 1 判斷韌體階段，再依硬體初始化、OS 介面、安全、更新或測試等相鄰 Part 交叉查閱。

## 本 Part 章節

- [Manufacturing Mode、Factory Tool 與產線流程](50_manufacturing_mode_factory_tools_and_production_flow.md)
- [Board ID、SKU、FRU 與平台辨識](51_board_id_sku_fru_and_platform_identification.md)
- [MAC、Serial Number、UUID 與 Provisioning](52_mac_serial_number_uuid_and_provisioning.md)
- [BIOS Setup、客製選項與預設值管理](53_bios_setup_custom_options_and_default_management.md)
- [Platform Notes Template](54_platform_notes_template.md)
