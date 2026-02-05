# Standby Mode

This mode provides the lowest power consumption, with POR, BOR, and PDR active.

Use `HAL_PWR_EnterSTANDBYMode()` to enter Standby mode.

When this mode is activated, the following features are disabled:
- VCORE is switched off (SRAM can be preserved); therefore, the PLL, HSI16, and HSE are switched off.
- I/O configuration is lost (pull-up, pull-down, or no resistor must be reconfigured).
- All peripheral registers are lost, except for some peripherals such as the RTC domain.

To exit Standby mode, the following events or interrupts can be used:
- WKUPx pin edge
- RTC event
- TAMP event
- External reset on the NRST pin
- IWDG reset
- BOR

After exiting Standby mode, the program counter restarts and the program is executed again from the reset vector.

On the other hand, the following peripherals, clocks, or features can remain active:
- SRAM
- Backup registers
- BOR
- LSI
- RTC
- TAMP
- IWDG

## Current Consumption

<center>

| **Active Feature** | **Theoretical Current<br>Consumption** | **Practical Current<br>Consumption** |
|--------------------|----------------------------------------|--------------------------------------|
| Nothing            | 515 µA                                 | 1.9 µA                               |
| SRAM               | 515 µA                                 | 2.4 µA                               |
| RTC with LSI       | 515 µA                                 | 2.0 µA                               |
| IWDG               | 515 µA                                 | 2.0 µA                               |
| SRAM & IWDG & RTC  | 515 µA                                 | 2.5 µA                               |


</center>

## References

- STMicroelectronics. (2024). *RM0444: STM32G0x1 advanced Arm®-based 32-bit MCUs reference manual* (Rev. 6).  
  https://www.st.com/resource/en/reference_manual/rm0444-stm32g0x1-advanced-armbased-32bit-mcus-stmicroelectronics.pdf

- STMicroelectronics. (2024). *STM32G0B1xB/xC/xE Arm® Cortex®-M0+ 32-bit MCU datasheet* (DS13560 Rev. 5).  
  https://www.st.com/resource/en/datasheet/stm32g0b1cc.pdf