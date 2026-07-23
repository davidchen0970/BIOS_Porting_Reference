# Part 2：平台與矽晶片初始化 導讀

> 第一次閱讀本 Part 時，建議先讀完本導讀，再進入各章節。

## 這一 Part 在做什麼

說明 CPU、Chipset／SoC、DRAM、Clock、Reset、Power Rail 與 GPIO 如何被帶到可供 DXE 使用的狀態。

## 為什麼需要這一 Part

新板 Bring-up 的早期問題多半跨越 Silicon、Board 與電源時序，若責任邊界不清楚，容易把硬體前置條件誤判為上層韌體問題。

## 讀完應能回答

- Platform、Silicon、Board Package 如何分工？
- DRAM Training 的輸入、輸出與快取資料是什麼？
- 不同 Reset Path 會重做哪些初始化？
- Clock、Power、GPIO 如何量測及驗證？

## 建議閱讀順序

先讀 Package 邊界，再依 CPU、Chipset、Memory、Clock／Reset／Power、GPIO 閱讀。

## 與其他 Part 的關係

本 Part 不獨立存在。遇到跨領域問題時，應回到 Part 1 判斷韌體階段，再依硬體初始化、OS 介面、安全、更新或測試等相鄰 Part 交叉查閱。

## 本 Part 章節

- [Platform、Silicon 與 Board Package 結構](06_platform_silicon_and_board_package_structure.md)
- [CPU 初始化與微碼更新](07_cpu_initialization_and_microcode_update.md)
- [Chipset、PCH 與 SoC 初始化](08_chipset_pch_and_soc_initialization.md)
- [Memory Reference Code 與 DRAM Training](09_memory_reference_code_and_dram_training.md)
- [Clock、Reset、Power Rail 與電源時序](10_clock_reset_power_rail_and_sequencing.md)
- [GPIO、Pinmux 與板級設定](11_gpio_pinmux_and_board_configuration.md)
