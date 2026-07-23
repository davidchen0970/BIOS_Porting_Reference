# Part 9：Debug、效能與測試 導讀

> 第一次閱讀本 Part 時，建議先讀完本導讀，再進入各章節。

## 這一 Part 在做什麼

建立問題分類、證據收集、決策樹、電源循環、效能量測與測試 Scope。

## 為什麼需要這一 Part

若缺乏一致方法，團隊會用不同 Log 與 Pass／Fail 標準，導致問題難以重現與回歸。

## 讀完應能回答

- 如何分類 Hang、Reset、Crash、Device Missing？
- 如何關聯 Serial、POST、MCA、AER、Crashdump？
- 如何量測 Boot Time 與資源？
- 測試 Scope 如何和 A04／A05 Gate 分開？

## 建議閱讀順序

先讀 Methodology 與 Toolkit，再讀失敗模式、電源測試、效能、Test Matrix。

## 與其他 Part 的關係

本 Part 不獨立存在。遇到跨領域問題時，應回到 Part 1 判斷韌體階段，再依硬體初始化、OS 介面、安全、更新或測試等相鄰 Part 交叉查閱。

## 本 Part 章節

- [BIOS Debug Methodology](44_bios_debug_methodology.md)
- [Serial Log、POST Code、Trace 與 Debug Toolkit](45_serial_log_post_code_trace_and_debug_toolkit.md)
- [常見開機失敗模式與問題排查](46_common_boot_failure_patterns_and_troubleshooting.md)
- [S3、S4、S5、Warm Reset 與 Cold Boot 測試](47_power_cycle_reset_and_sleep_state_testing.md)
- [Boot Time、記憶體用量與效能調校](48_boot_time_memory_usage_and_performance_tuning.md)
- [BIOS General Test Matrix](49_bios_general_test_matrix.md)
