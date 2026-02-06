# Shutdown Mode

This mode allows to achieve the lowest power consumption.

SRAM and register contents are lost except for registers in the RTC domain. The POR/PDR and BOR are not available in Shutdown mode. 

Use `HAL_PWREx_EnterSHUTDOWNMode()` to enter Shutdown mode.

When this mode is activated, the following features are disabled:
- VCORE is switched off; therefore, the PLL, HSI16, and HSE are switched off.
- I/O configuration is lost (pull-up, pull-down, or no resistor must be reconfigured).
- All peripheral registers are lost, except for some peripherals such as the RTC domain.

## Wake-up Sources
To exit Shutdown mode, the following events or interrupts can be used:
- WKUPx pin edge
- RTC Interrupt
- TAMP Interrupt
- External reset on the NRST pin

## Reset Behavior
Shutdown mode always generates a system reset.
The application restarts from the reset vector.


## Current Consumption

<center>

| **RTC** | **Max Theoretical Current<br> Consumption at 25 °C** | **Practical Current<br>Consumption** |
|---------|------------------------------------------------------|--------------------------------------|
| Disable | 435 nA                                               | 1.9 nA                               |
| Enable  | 520 nA*                                              | 2.4 nA**                             |



</center>

\* This current is clocked by LSE bypass at 32.768 kHz <br>
\*\* This current is clocked by LSE crystal at 32.768 kHz

The maximum theoretical current consumption is specified with a 3.3 V supply voltage (VDD).

## Important Notes

- If you use the RTC, you should use the RTC backup registers to verify whether the RTC configuration has been preserved or needs to be reconfigured.

## References

- STMicroelectronics. (2024). *RM0444: STM32G0x1 advanced Arm®-based 32-bit MCUs reference manual* (Rev. 6).  
  https://www.st.com/resource/en/reference_manual/rm0444-stm32g0x1-advanced-armbased-32bit-mcus-stmicroelectronics.pdf

- STMicroelectronics. (2024). *STM32G0B1xB/xC/xE Arm® Cortex®-M0+ 32-bit MCU datasheet* (DS13560 Rev. 5).  
  https://www.st.com/resource/en/datasheet/stm32g0b1cc.pdf