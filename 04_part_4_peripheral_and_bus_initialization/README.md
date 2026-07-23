# Part 4：周邊裝置與匯流排初始化 導讀

> 第一次閱讀本 Part 時，建議先讀完本導讀，再進入各章節。

## 這一 Part 在做什麼

說明 Controller 探索、資源配置、Driver Binding、Child Handle、Device Path 與 Boot Device 的形成。

## 為什麼需要這一 Part

裝置缺失可能位於 Link、Enumeration、BAR、Driver、Protocol 或 Boot Option 任一層，需要端到端觀察。

## 讀完應能回答

- PCIe／CXL 如何列舉及配置資源？
- USB／Storage 如何形成 Block I/O 與 Device Path？
- 低速匯流排如何處理 Ownership、Timeout 與 Recovery？
- Network 與 Graphics 如何進入 UEFI Boot 流程？

## 建議閱讀順序

先讀 PCIe 資源模型，再依產品所用 USB、Storage、低速匯流排、Network、Graphics 選讀。

## 與其他 Part 的關係

本 Part 不獨立存在。遇到跨領域問題時，應回到 Part 1 判斷韌體階段，再依硬體初始化、OS 介面、安全、更新或測試等相鄰 Part 交叉查閱。

## 本 Part 章節

- [PCIe Enumeration、Resource Allocation 與 Option ROM](17_pcie_enumeration_resource_allocation_and_option_rom.md)
- [USB Host Controller 與開機裝置](18_usb_host_controller_and_boot_devices.md)
- [SATA、NVMe、eMMC 與儲存裝置](19_sata_nvme_emmc_and_storage_devices.md)
- [SMBus、I2C、SPI 與 LPC／eSPI](20_smbus_i2c_spi_and_lpc_espi.md)
- [Network Stack、PXE、HTTP Boot 與 iSCSI](21_network_stack_pxe_http_boot_and_iscsi.md)
- [Graphics、GOP 與主控台初始化](22_graphics_gop_and_console_initialization.md)
