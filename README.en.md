# BIOS/UEFI Porting Technical Reference

**Language**: [English](README.en.md) | [Traditional Chinese](README.md)

> This technical reference is based on **EDK II** and the UEFI PI architecture. It covers UEFI firmware porting, board support, system integration, production validation, and troubleshooting for x86 and ARM platforms. It is intended for firmware engineers with foundational knowledge of C, computer architecture, and firmware development. The chapters are organized around boot flow, platform initialization, operating system interfaces, security, firmware maintenance, and testing, providing a practical index for new-platform bring-up and existing-product maintenance.

---

## Table of Contents

### Part 1: UEFI Architecture and Boot Flow

- [UEFI PI Architecture and Firmware Execution Phases](01_part_1_uefi_architecture_and_boot_flow/01_uefi_pi_architecture_and_firmware_phases.md)
- [SEC, PEI, DXE, and BDS Boot Flow](01_part_1_uefi_architecture_and_boot_flow/02_sec_pei_dxe_and_bds_boot_flow.md)
- [Firmware Volumes, FFS, and Image Layout](01_part_1_uefi_architecture_and_boot_flow/03_firmware_volume_ffs_and_image_layout.md)
- [HOBs, Protocols, PPIs, and UEFI Services](01_part_1_uefi_architecture_and_boot_flow/04_hob_protocol_ppi_and_uefi_services.md)
- [Boot Manager, Boot Options, and Boot Order](01_part_1_uefi_architecture_and_boot_flow/05_boot_manager_boot_option_and_boot_order.md)

### Part 2: Platform and Silicon Initialization

- [Platform, Silicon, and Board Package Structure](02_part_2_platform_and_silicon_initialization/06_platform_silicon_and_board_package_structure.md)
- [CPU Initialization and Microcode Updates](02_part_2_platform_and_silicon_initialization/07_cpu_initialization_and_microcode_update.md)
- [Chipset, PCH, and SoC Initialization](02_part_2_platform_and_silicon_initialization/08_chipset_pch_and_soc_initialization.md)
- [Memory Reference Code and DRAM Training](02_part_2_platform_and_silicon_initialization/09_memory_reference_code_and_dram_training.md)
- [Clock, Reset, Power Rails, and Power Sequencing](02_part_2_platform_and_silicon_initialization/10_clock_reset_power_rail_and_sequencing.md)
- [GPIO, Pin Multiplexing, and Board Configuration](02_part_2_platform_and_silicon_initialization/11_gpio_pinmux_and_board_configuration.md)

### Part 3: EDK II, Build System, and Configuration Data

- [EDK II Workspace and Package Structure](03_part_3_edk_ii_build_system_and_configuration/12_edk_ii_workspace_and_package_structure.md)
- [DSC, DEC, INF, and FDF Configuration](03_part_3_edk_ii_build_system_and_configuration/13_dsc_dec_inf_and_fdf_configuration.md)
- [PCDs, Build Flags, and Platform Configuration](03_part_3_edk_ii_build_system_and_configuration/14_pcd_build_flags_and_platform_configuration.md)
- [Toolchains, Build Flow, and Image Output](03_part_3_edk_ii_build_system_and_configuration/15_toolchain_build_flow_and_image_output.md)
- [Setup Variables, NVRAM, and Default Management](03_part_3_edk_ii_build_system_and_configuration/16_setup_variable_nvram_and_default_management.md)

### Part 4: Peripheral and Bus Initialization

- [PCIe Enumeration, Resource Allocation, and Option ROMs](04_part_4_peripheral_and_bus_initialization/17_pcie_enumeration_resource_allocation_and_option_rom.md)
- [USB Host Controllers and Boot Devices](04_part_4_peripheral_and_bus_initialization/18_usb_host_controller_and_boot_devices.md)
- [SATA, NVMe, eMMC, and Storage Devices](04_part_4_peripheral_and_bus_initialization/19_sata_nvme_emmc_and_storage_devices.md)
- [SMBus, I2C, SPI, and LPC/eSPI](04_part_4_peripheral_and_bus_initialization/20_smbus_i2c_spi_and_lpc_espi.md)
- [Network Stack, PXE, HTTP Boot, and iSCSI](04_part_4_peripheral_and_bus_initialization/21_network_stack_pxe_http_boot_and_iscsi.md)
- [Graphics, GOP, and Console Initialization](04_part_4_peripheral_and_bus_initialization/22_graphics_gop_and_console_initialization.md)

### Part 5: ACPI, SMBIOS, and Operating System Interfaces

- [ACPI Architecture and Table Generation](05_part_5_acpi_smbios_and_os_interfaces/23_acpi_architecture_and_table_generation.md)
- [DSDT, SSDT, AML, and ASL Debugging](05_part_5_acpi_smbios_and_os_interfaces/24_dsdt_ssdt_aml_and_asl_debugging.md)
- [Power States, Sleep, Wake, and Runtime Power Management](05_part_5_acpi_smbios_and_os_interfaces/25_power_state_sleep_wake_and_runtime_power_management.md)
- [SMBIOS Types and Platform Information](05_part_5_acpi_smbios_and_os_interfaces/26_smbios_types_and_platform_information.md)
- [UEFI Runtime Services and OS Hand-off](05_part_5_acpi_smbios_and_os_interfaces/27_uefi_runtime_services_and_os_handoff.md)

### Part 6: BMC, Management Controllers, and Host Communication

- [BIOS/UEFI and BMC Interaction](06_part_6_bmc_management_controller_and_host_communication/28_bios_uefi_and_bmc_interaction.md)
- [KCS, BT, SSIF, and eSPI Communication](06_part_6_bmc_management_controller_and_host_communication/29_kcs_bt_ssif_and_espi_communication.md)
- [IPMI, Redfish Host Interface, and OEM Commands](06_part_6_bmc_management_controller_and_host_communication/30_ipmi_redfish_host_interface_and_oem_commands.md)
- [MCTP, PLDM, and Firmware Data Exchange](06_part_6_bmc_management_controller_and_host_communication/31_mctp_pldm_and_firmware_data_exchange.md)
- [POST Codes, SEL, Events, and Telemetry](06_part_6_bmc_management_controller_and_host_communication/32_post_code_sel_event_and_telemetry.md)

### Part 7: Security and Chain of Trust

- [Secure Boot, Key Databases, and Image Verification](07_part_7_security_and_chain_of_trust/33_secure_boot_key_database_and_image_verification.md)
- [TPM, Measured Boot, and PCRs](07_part_7_security_and_chain_of_trust/34_tpm_measured_boot_and_pcr.md)
- [Firmware Chain of Trust, Boot Guard, and Hardware Root of Trust](07_part_7_security_and_chain_of_trust/35_firmware_trust_chain_boot_guard_and_hardware_root_of_trust.md)
- [Capsule Updates, Rollback Protection, and Anti-rollback](07_part_7_security_and_chain_of_trust/36_capsule_update_rollback_protection_and_anti_rollback.md)
- [SMM, MM, Variables, and Flash Protection](07_part_7_security_and_chain_of_trust/37_smm_mm_variable_and_flash_protection.md)
- [Security Baseline and Vulnerability Response](07_part_7_security_and_chain_of_trust/38_security_baseline_and_vulnerability_response.md)

### Part 8: Firmware Update, Recovery, and Maintenance

- [BIOS Images, Regions, and Flash Layout](08_part_8_firmware_update_recovery_and_maintenance/39_bios_image_regions_and_flash_layout.md)
- [Update Mechanisms, Capsules, and Offline Flashing](08_part_8_firmware_update_recovery_and_maintenance/40_update_mechanisms_capsule_and_offline_flashing.md)
- [Dual Images, Fail-safe Design, and Recovery Flow](08_part_8_firmware_update_recovery_and_maintenance/41_dual_image_fail_safe_and_recovery_flow.md)
- [NVRAM Migration, Setting Preservation, and Version Compatibility](08_part_8_firmware_update_recovery_and_maintenance/42_nvram_migration_setting_preservation_and_version_compatibility.md)
- [RMA, Field Service, and Faulty Image Analysis](08_part_8_firmware_update_recovery_and_maintenance/43_rma_field_service_and_faulty_image_analysis.md)

### Part 9: Debugging, Performance, and Testing

- [BIOS Debugging Methodology](09_part_9_debugging_performance_and_testing/44_bios_debug_methodology.md)
- [Serial Logs, POST Codes, Traces, and Debugging Toolkit](09_part_9_debugging_performance_and_testing/45_serial_log_post_code_trace_and_debug_toolkit.md)
- [Common Boot Failure Patterns and Troubleshooting](09_part_9_debugging_performance_and_testing/46_common_boot_failure_patterns_and_troubleshooting.md)
- [S3, S4, S5, Warm Reset, and Cold Boot Testing](09_part_9_debugging_performance_and_testing/47_power_cycle_reset_and_sleep_state_testing.md)
- [Boot Time, Memory Usage, and Performance Tuning](09_part_9_debugging_performance_and_testing/48_boot_time_memory_usage_and_performance_tuning.md)
- [BIOS General Test Matrix](09_part_9_debugging_performance_and_testing/49_bios_general_test_matrix.md)

### Part 10: Manufacturing, Production, and Platform-specific Notes

- [Manufacturing Mode, Factory Tools, and Production Flow](10_part_10_manufacturing_production_and_platform_notes/50_manufacturing_mode_factory_tools_and_production_flow.md)
- [Board ID, SKU, FRU, and Platform Identification](10_part_10_manufacturing_production_and_platform_notes/51_board_id_sku_fru_and_platform_identification.md)
- [MAC Addresses, Serial Numbers, UUIDs, and Provisioning](10_part_10_manufacturing_production_and_platform_notes/52_mac_serial_number_uuid_and_provisioning.md)
- [BIOS Setup, Custom Options, and Default Management](10_part_10_manufacturing_production_and_platform_notes/53_bios_setup_custom_options_and_default_management.md)
- [Platform Notes Template](10_part_10_manufacturing_production_and_platform_notes/54_platform_notes_template.md)

### Part 11: Appendices

- [Common Abbreviations and Terms](11_part_11_appendices/A01_common_abbreviations_and_terms.md)
- [Common Tools and Command Reference](11_part_11_appendices/A02_common_tools_and_commands_reference.md)
- [Log Collection Package Template](11_part_11_appendices/A03_log_collection_package_template.md)
- [Bring-up and Acceptance Checklist](11_part_11_appendices/A04_bring_up_and_acceptance_checklist.md)
- [BIOS Release Checklist](11_part_11_appendices/A05_bios_release_checklist.md)
- [Documentation Template](11_part_11_appendices/A06_documentation_template.md)
