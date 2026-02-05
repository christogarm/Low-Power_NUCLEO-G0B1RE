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

| **Active Feature** | **Max Theoretical Current<br> Consumption at 25 °C** | **Practical Current<br>Consumption** |
|--------------------|------------------------------------------------------|--------------------------------------|
| Nothing            | 1.8 µA                                               | 1.9 µA                               |
| SRAM               | 6.2 µA                                               | 2.4 µA                               |
| RTC with LSI       | 3.3 µA                                               | 2.0 µA                               |
| IWDG               | 3.0 µA                                               | 2.0 µA                               |
| SRAM & IWDG & RTC  | 7.1 µA                                               | 2.5 µA                               |

</center>

The maximum theoretical current consumption is specified with a 3.6 V supply voltage (VDD).

## Important Notes

- If you use a debugger, it must be disabled to achieve correct Standby current consumption, you can use `HAL_DBGMCU_DisableDBGStandbyMode();`.
- If you use the IWDG, remember to diseable `IWDG_STDBY` to freeze the IWDG counter while the MCU is in Standby mode.
- If you use the RTC, you should use the RTC backup registers to verify whether the RTC configuration has been preserved or needs to be reconfigured.

## References

- STMicroelectronics. (2024). *RM0444: STM32G0x1 advanced Arm®-based 32-bit MCUs reference manual* (Rev. 6).  
  https://www.st.com/resource/en/reference_manual/rm0444-stm32g0x1-advanced-armbased-32bit-mcus-stmicroelectronics.pdf

- STMicroelectronics. (2024). *STM32G0B1xB/xC/xE Arm® Cortex®-M0+ 32-bit MCU datasheet* (DS13560 Rev. 5).  
  https://www.st.com/resource/en/datasheet/stm32g0b1cc.pdf