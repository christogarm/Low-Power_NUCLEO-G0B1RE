# Standby Mode

This mode provides the lowest power consumption, with POR, BOR, and PDR active.

Use `HAL_PWR_EnterSTANDBYMode()` to enter Standby mode.

When this mode is activated, the following features are disabled:
- VCORE is switched off (SRAM can be preserved); therefore, the PLL, HSI16, and HSE are switched off.
- I/O configuration is lost (pull-up, pull-down, or no resistor must be reconfigured).
- All peripheral registers are lost, except for some peripherals such as the RTC domain.

## Wake-up Sources
To exit Standby mode, the following events or interrupts can be used:
- WKUPx pin edge
- RTC event
- TAMP event
- External reset on the NRST pin
- IWDG reset
- BOR

## Reset Behavior
Standby mode always generates a system reset.
The application restarts from the reset vector.
Standby entry can be detected using PWR and RCC reset flags.

## Memory Retention
All other peripheral registers are lost, except the following peripherals:
- SRAM
- Backup registers
- BOR
- LSI
- RTC
- TAMP
- IWDG

## Current Consumption

<center>

| **Active Feature** | **Current Consumption** | **Measured Current Consumption** |
|--------------------|-------------------------|----------------------------------|
| Nothing            | 1.2 µA                  | 1.9 µA                           |
| SRAM               | 2.35 µA                 | 2.4 µA                           |
| RTC with LSI       | 1.75 µA                 | 2.0 µA                           |
| IWDG               | 1.55 µA                 | 2.0 µA                           |
| SRAM & IWDG & RTC  | 3.25 µA                 | 2.5 µA                           |

</center>

> The maximum theoretical current consumption is specified with a 3.3 V supply voltage (VDD) and disable ultra low-power.
> This data is interpolated between VDD = 3.6 V and VDD = 3.0 V to obtain values at VDD = 3.3 V.

## Ultra-low-power enable (ENB_ULP)

This configuration enables or disables **periodical sampling of the supply voltage** in Stop and Standby modes for detecting **PDR and BOR reset conditions**.

When ultra-low-power mode is enabled (`HAL_PWREx_EnablePORMonitorSampling();`), the supply voltage is not continuously monitored. If the voltage drops below the minimum operating condition between two samples, a **BOR or PDR reset may not be detected**.

This may result in **unreliable system behavior**, as the device could continue operating outside its specified voltage range. For this reason, this mode should only be used when a **stable and well-controlled power source** is guaranteed.

The following table shows the measured current consumption with **Ultra-low-power mode enabled**:

<center>

| **Active Feature** | **Measured Current Consumption** |
|--------------------|----------------------------------|
| Nothing            | 1.0 µA                           |
| SRAM & IWDG & RTC  | 1.6 µA                           |

</center>

## Important Notes

- If you use a debugger, it must be disabled to achieve correct Standby current consumption, you can use `HAL_DBGMCU_DisableDBGStandbyMode();`.
- If you use the IWDG, remember to diseable `IWDG_STDBY` to freeze the IWDG counter while the MCU is in Standby mode.
- If you use the RTC, you should use the RTC backup registers to verify whether the RTC configuration has been preserved or needs to be reconfigured.

## References

- STMicroelectronics. (2024). *RM0444: STM32G0x1 advanced Arm®-based 32-bit MCUs reference manual* (Rev. 6).  
  https://www.st.com/resource/en/reference_manual/rm0444-stm32g0x1-advanced-armbased-32bit-mcus-stmicroelectronics.pdf

- STMicroelectronics. (2024). *STM32G0B1xB/xC/xE Arm® Cortex®-M0+ 32-bit MCU datasheet* (DS13560 Rev. 5).  
  https://www.st.com/resource/en/datasheet/stm32g0b1cc.pdf