# Part 6：BMC、管理控制器與主機通訊 導讀

> 第一次閱讀本 Part 時，建議先讀完本導讀，再進入各章節。

## 這一 Part 在做什麼

整理 BIOS、BMC、CPLD 與 Host OS 的開機進度、設定、事件及資料交換。

## 為什麼需要這一 Part

伺服器開機是多元件協作，初始化時機、Timeout、Retry、降級與 Log 對時都會直接影響問題定位。

## 讀完應能回答

- 各管理通道何時初始化？
- BMC 未就緒時 BIOS 如何降級？
- IPMI、Redfish、MCTP、PLDM 如何選用？
- POST、SEL、Event 如何建立共同時間線？

## 建議閱讀順序

先讀整體互動，再依通道與協定閱讀，最後閱讀事件與 Telemetry。

## 與其他 Part 的關係

本 Part 不獨立存在。遇到跨領域問題時，應回到 Part 1 判斷韌體階段，再依硬體初始化、OS 介面、安全、更新或測試等相鄰 Part 交叉查閱。

## 本 Part 章節

- [BIOS／UEFI 與 BMC 互動](28_bios_uefi_and_bmc_interaction.md)
- [KCS、BT、SSIF 與 eSPI 通訊](29_kcs_bt_ssif_and_espi_communication.md)
- [IPMI、Redfish Host Interface 與 OEM Command](30_ipmi_redfish_host_interface_and_oem_commands.md)
- [MCTP、PLDM 與韌體資料交換](31_mctp_pldm_and_firmware_data_exchange.md)
- [POST Code、SEL、Event 與 Telemetry](32_post_code_sel_event_and_telemetry.md)
