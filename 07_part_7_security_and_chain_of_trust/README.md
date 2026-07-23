# Part 7：安全性與信任鏈 導讀

> 第一次閱讀本 Part 時，建議先讀完本導讀，再進入各章節。

## 這一 Part 在做什麼

建立映像驗證、量測、裝置認證、更新保護、SMM／MM 與弱點管理的完整安全模型。

## 為什麼需要這一 Part

安全功能保護的對象與生命週期不同，只確認 Enable 狀態不足以涵蓋 Provisioning、撤銷、Recovery 與負向測試。

## 讀完應能回答

- Secure Boot、TPM、Boot Guard 如何分工？
- DOE／SPDM 如何延伸裝置信任？
- Anti-rollback 與 Capsule 如何協作？
- SMM、Variable、Flash 如何限制寫入？

## 建議閱讀順序

依 Secure Boot、TPM、Trust Chain、Capsule、SMM／MM、Security Baseline 閱讀。

## 與其他 Part 的關係

本 Part 不獨立存在。遇到跨領域問題時，應回到 Part 1 判斷韌體階段，再依硬體初始化、OS 介面、安全、更新或測試等相鄰 Part 交叉查閱。

## 本 Part 章節

- [Secure Boot、Key Database 與映像檔驗證](33_secure_boot_key_database_and_image_verification.md)
- [TPM、Measured Boot 與 PCR](34_tpm_measured_boot_and_pcr.md)
- [Firmware Trust Chain、Boot Guard 與 Hardware Root of Trust](35_firmware_trust_chain_boot_guard_and_hardware_root_of_trust.md)
- [Capsule Update、Rollback Protection 與 Anti-rollback](36_capsule_update_rollback_protection_and_anti_rollback.md)
- [SMM、MM、Variable 與 Flash Protection](37_smm_mm_variable_and_flash_protection.md)
- [Security Baseline 與弱點處理流程](38_security_baseline_and_vulnerability_response.md)
