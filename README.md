# BIOS／UEFI 移植技術參考文件

**語言**： [English](README.en.md) | [繁體中文](README.md)

> 本技術文件以 **EDK II** 與 UEFI PI 架構為核心，涵蓋 x86／ARM 平台的 UEFI 韌體移植、板級支援、系統整合、量產驗證與問題排查。目標讀者為具備基礎 C 語言、計算機架構及韌體開發概念的工程師。各章節依開機流程、平台初始化、作業系統介面、安全性、韌體維護及測試等主題編排，可作為新平台 Bring-up 與既有產品維護的技術索引。

### 章節索引

#### Part 1：UEFI 架構與開機流程

- [UEFI PI 架構與韌體執行階段](01_part_1_uefi_architecture_and_boot_flow/01_uefi_pi_architecture_and_firmware_phases.md)
- [SEC、PEI、DXE 與 BDS 開機流程](01_part_1_uefi_architecture_and_boot_flow/02_sec_pei_dxe_and_bds_boot_flow.md)
- [Firmware Volume、FFS 與映像檔配置](01_part_1_uefi_architecture_and_boot_flow/03_firmware_volume_ffs_and_image_layout.md)
- [HOB、Protocol、PPI 與 UEFI Service](01_part_1_uefi_architecture_and_boot_flow/04_hob_protocol_ppi_and_uefi_services.md)
- [Boot Manager、Boot Option 與開機順序](01_part_1_uefi_architecture_and_boot_flow/05_boot_manager_boot_option_and_boot_order.md)

#### Part 2：平台與矽晶片初始化

- [Platform、Silicon 與 Board Package 結構](02_part_2_platform_and_silicon_initialization/06_platform_silicon_and_board_package_structure.md)
- [CPU 初始化與微碼更新](02_part_2_platform_and_silicon_initialization/07_cpu_initialization_and_microcode_update.md)
- [Chipset、PCH 與 SoC 初始化](02_part_2_platform_and_silicon_initialization/08_chipset_pch_and_soc_initialization.md)
- [Memory Reference Code 與 DRAM Training](02_part_2_platform_and_silicon_initialization/09_memory_reference_code_and_dram_training.md)
- [Clock、Reset、Power Rail 與電源時序](02_part_2_platform_and_silicon_initialization/10_clock_reset_power_rail_and_sequencing.md)
- [GPIO、Pinmux 與板級設定](02_part_2_platform_and_silicon_initialization/11_gpio_pinmux_and_board_configuration.md)

#### Part 3：EDK II、建置系統與設定資料

- [EDK II Workspace 與 Package 結構](03_part_3_edk_ii_build_system_and_configuration/12_edk_ii_workspace_and_package_structure.md)
- [DSC、DEC、INF 與 FDF 設定](03_part_3_edk_ii_build_system_and_configuration/13_dsc_dec_inf_and_fdf_configuration.md)
- [PCD、Build Flag 與平台組態](03_part_3_edk_ii_build_system_and_configuration/14_pcd_build_flags_and_platform_configuration.md)
- [工具鏈、建置流程與映像檔產出](03_part_3_edk_ii_build_system_and_configuration/15_toolchain_build_flow_and_image_output.md)
- [Setup Variable、NVRAM 與預設值管理](03_part_3_edk_ii_build_system_and_configuration/16_setup_variable_nvram_and_default_management.md)

#### Part 4：周邊裝置與匯流排初始化

- [PCIe Enumeration、Resource Allocation 與 Option ROM](04_part_4_peripheral_and_bus_initialization/17_pcie_enumeration_resource_allocation_and_option_rom.md)
- [USB Host Controller 與開機裝置](04_part_4_peripheral_and_bus_initialization/18_usb_host_controller_and_boot_devices.md)
- [SATA、NVMe、eMMC 與儲存裝置](04_part_4_peripheral_and_bus_initialization/19_sata_nvme_emmc_and_storage_devices.md)
- [SMBus、I2C、SPI 與 LPC／eSPI](04_part_4_peripheral_and_bus_initialization/20_smbus_i2c_spi_and_lpc_espi.md)
- [Network Stack、PXE、HTTP Boot 與 iSCSI](04_part_4_peripheral_and_bus_initialization/21_network_stack_pxe_http_boot_and_iscsi.md)
- [Graphics、GOP 與主控台初始化](04_part_4_peripheral_and_bus_initialization/22_graphics_gop_and_console_initialization.md)

#### Part 5：ACPI、SMBIOS 與作業系統介面

- [ACPI 架構與 Table 產生流程](05_part_5_acpi_smbios_and_os_interfaces/23_acpi_architecture_and_table_generation.md)
- [DSDT、SSDT、AML 與 ASL 除錯](05_part_5_acpi_smbios_and_os_interfaces/24_dsdt_ssdt_aml_and_asl_debugging.md)
- [Power State、Sleep、Wake 與 Runtime Power Management](05_part_5_acpi_smbios_and_os_interfaces/25_power_state_sleep_wake_and_runtime_power_management.md)
- [SMBIOS Type 與平台資訊](05_part_5_acpi_smbios_and_os_interfaces/26_smbios_types_and_platform_information.md)
- [UEFI Runtime Service 與 OS Hand-off](05_part_5_acpi_smbios_and_os_interfaces/27_uefi_runtime_services_and_os_handoff.md)

#### Part 6：BMC、管理控制器與主機通訊

- [BIOS／UEFI 與 BMC 互動](06_part_6_bmc_management_controller_and_host_communication/28_bios_uefi_and_bmc_interaction.md)
- [KCS、BT、SSIF 與 eSPI 通訊](06_part_6_bmc_management_controller_and_host_communication/29_kcs_bt_ssif_and_espi_communication.md)
- [IPMI、Redfish Host Interface 與 OEM Command](06_part_6_bmc_management_controller_and_host_communication/30_ipmi_redfish_host_interface_and_oem_commands.md)
- [MCTP、PLDM 與韌體資料交換](06_part_6_bmc_management_controller_and_host_communication/31_mctp_pldm_and_firmware_data_exchange.md)
- [POST Code、SEL、Event 與 Telemetry](06_part_6_bmc_management_controller_and_host_communication/32_post_code_sel_event_and_telemetry.md)

#### Part 7：安全性與信任鏈

- [Secure Boot、Key Database 與映像檔驗證](07_part_7_security_and_chain_of_trust/33_secure_boot_key_database_and_image_verification.md)
- [TPM、Measured Boot 與 PCR](07_part_7_security_and_chain_of_trust/34_tpm_measured_boot_and_pcr.md)
- [Firmware Trust Chain、Boot Guard 與 Hardware Root of Trust](07_part_7_security_and_chain_of_trust/35_firmware_trust_chain_boot_guard_and_hardware_root_of_trust.md)
- [Capsule Update、Rollback Protection 與 Anti-rollback](07_part_7_security_and_chain_of_trust/36_capsule_update_rollback_protection_and_anti_rollback.md)
- [SMM、MM、Variable 與 Flash Protection](07_part_7_security_and_chain_of_trust/37_smm_mm_variable_and_flash_protection.md)
- [Security Baseline 與弱點處理流程](07_part_7_security_and_chain_of_trust/38_security_baseline_and_vulnerability_response.md)

#### Part 8：韌體更新、復原與維護

- [BIOS Image、Region 與 Flash Layout](08_part_8_firmware_update_recovery_and_maintenance/39_bios_image_regions_and_flash_layout.md)
- [更新機制、Capsule 與離線燒錄](08_part_8_firmware_update_recovery_and_maintenance/40_update_mechanisms_capsule_and_offline_flashing.md)
- [Dual Image、Fail-safe 與 Recovery Flow](08_part_8_firmware_update_recovery_and_maintenance/41_dual_image_fail_safe_and_recovery_flow.md)
- [NVRAM Migration、設定保留與版本相容性](08_part_8_firmware_update_recovery_and_maintenance/42_nvram_migration_setting_preservation_and_version_compatibility.md)
- [RMA、Field Service 與故障映像檔分析](08_part_8_firmware_update_recovery_and_maintenance/43_rma_field_service_and_faulty_image_analysis.md)

#### Part 9：Debug、效能與測試

- [BIOS Debug Methodology](09_part_9_debugging_performance_and_testing/44_bios_debug_methodology.md)
- [Serial Log、POST Code、Trace 與 Debug Toolkit](09_part_9_debugging_performance_and_testing/45_serial_log_post_code_trace_and_debug_toolkit.md)
- [常見開機失敗模式與問題排查](09_part_9_debugging_performance_and_testing/46_common_boot_failure_patterns_and_troubleshooting.md)
- [S3、S4、S5、Warm Reset 與 Cold Boot 測試](09_part_9_debugging_performance_and_testing/47_power_cycle_reset_and_sleep_state_testing.md)
- [Boot Time、記憶體用量與效能調校](09_part_9_debugging_performance_and_testing/48_boot_time_memory_usage_and_performance_tuning.md)
- [BIOS General Test Matrix](09_part_9_debugging_performance_and_testing/49_bios_general_test_matrix.md)

#### Part 10：製造、量產與平台特定筆記

- [Manufacturing Mode、Factory Tool 與產線流程](10_part_10_manufacturing_production_and_platform_notes/50_manufacturing_mode_factory_tools_and_production_flow.md)
- [Board ID、SKU、FRU 與平台辨識](10_part_10_manufacturing_production_and_platform_notes/51_board_id_sku_fru_and_platform_identification.md)
- [MAC、Serial Number、UUID 與 Provisioning](10_part_10_manufacturing_production_and_platform_notes/52_mac_serial_number_uuid_and_provisioning.md)
- [BIOS Setup、客製選項與預設值管理](10_part_10_manufacturing_production_and_platform_notes/53_bios_setup_custom_options_and_default_management.md)
- [Platform Notes Template](10_part_10_manufacturing_production_and_platform_notes/54_platform_notes_template.md)

#### Part 11：附錄

- [常用縮寫與術語](11_part_11_appendices/A01_common_abbreviations_and_terms.md)
- [常用工具與指令參考](11_part_11_appendices/A02_common_tools_and_commands_reference.md)
- [Log 收集 Package Template](11_part_11_appendices/A03_log_collection_package_template.md)
- [Bring-up 與 Acceptance Checklist](11_part_11_appendices/A04_bring_up_and_acceptance_checklist.md)
- [BIOS Release Checklist](11_part_11_appendices/A05_bios_release_checklist.md)
- [Documentation Template](11_part_11_appendices/A06_documentation_template.md)
