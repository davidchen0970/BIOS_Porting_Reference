# Part 8：韌體更新、復原與維護 導讀

> 第一次閱讀本 Part 時，建議先讀完本導讀，再進入各章節。

## 這一 Part 在做什麼

描述映像從發佈、寫入、驗證、切換，到失敗復原、設定遷移與現場分析的生命週期。

## 為什麼需要這一 Part

工具顯示更新完成不代表平台安全可開機，還需確認 Reset、設定、交易完整性與 Recovery。

## 讀完應能回答

- Flash Layout 如何支援更新與復原？
- Capsule、BMC、Shell、Programmer 有何差異？
- Dual Image 如何判斷成功並回退？
- RMA 如何收集映像與 Crashdump？

## 建議閱讀順序

先讀 Flash Layout，再讀更新、Fail-safe、NVRAM Migration、RMA。

## 與其他 Part 的關係

本 Part 不獨立存在。遇到跨領域問題時，應回到 Part 1 判斷韌體階段，再依硬體初始化、OS 介面、安全、更新或測試等相鄰 Part 交叉查閱。

## 本 Part 章節

- [BIOS Image、Region 與 Flash Layout](39_bios_image_regions_and_flash_layout.md)
- [更新機制、Capsule 與離線燒錄](40_update_mechanisms_capsule_and_offline_flashing.md)
- [Dual Image、Fail-safe 與 Recovery Flow](41_dual_image_fail_safe_and_recovery_flow.md)
- [NVRAM Migration、設定保留與版本相容性](42_nvram_migration_setting_preservation_and_version_compatibility.md)
- [RMA、Field Service 與故障映像檔分析](43_rma_field_service_and_faulty_image_analysis.md)
