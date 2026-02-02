# Low-Power Modes in NUCLEO-G0B1RE

Low-power modes allow saving power when the CPU does not need to remain running.

This device provides seven low-power modes:
* **Low-Power Run Mode**
  * System clock frequency is reduced below 2 MHz.
  * Code is executed from SRAM or Flash memory.
  * The regulator operates in low-power mode.

* **Sleep Mode**
  * CPU clock is stopped.
  * Peripherals can continue running if their clocks are enabled.
  * The CPU wakes up on an interrupt or event.

* **Low-Power Sleep Mode**
  * CPU clock is stopped.
  * Selected peripherals can remain active.

* **Stop Mode**
  * SRAM content is retained.
  * All clocks in the VCORE domain are stopped.
  * LSI and LSE oscillators can remain running.
  * RTC and TAMP peripherals can stay active.

* **Standby Mode**
  * VCORE domain is powered off.
  * SRAM content can be preserved if configured.
  * LSI and LSE oscillators can remain running.
  * RTC can remain active.

* **Shutdown Mode**
  * Lowest power consumption mode.
  * VCORE domain is powered off.
  * Only the LSE oscillator can remain running.
  * Supply voltage monitoring is disabled.


## Power Consumption Summary

The following table shows typical measured current consumption.  
This data is based on maximum clock frequency, execution from RAM or Flash memory, and RTC enabled.
<center>

| **Low-Power Mode**     | **Typical Current <br>Consumption** |
| :--------------------- | :------------------------------ |
| Low-Power Run Mode     | 515 [µA]                          |
| Sleep Mode             | 1.9 [mA]                          |
| Low-Power Sleep Mode   | 70 [µA]                           |
| Stop Mode (0 / 1)\*    | 300 / 7.8 [µA]                    |
| Standby Mode\*\*       | 2.5 [µA]                          |
| Shutdown Mode          | 550 [nA]                          |

</center>

> \* Stop mode includes **STOP 0 and STOP 1** configurations.  
> \*\* Standby mode can preserve SRAM.

You can consult the theory current consumption in [STM32G0B1 Datasheet](https://www.st.com/resource/en/datasheet/stm32g0b1cc.pdf) in the section **5.3.5 Supply current characteristics** to review your particular case.

## How to test these modes?

Disconnect **JP3** and measure the current consumption to verify the low-power modes.

![Screenshot of Nucleo-G0B1RE from https://www.st.com/en/evaluation-tools/nucleo-g0b1re.html#.](img/Nucleo-G0B1RE-JP3.png)



## References

- STMicroelectronics. (2024). *RM0444: STM32G0x1 advanced Arm®-based 32-bit MCUs reference manual* (Rev. 6).  
  https://www.st.com/resource/en/reference_manual/rm0444-stm32g0x1-advanced-armbased-32bit-mcus-stmicroelectronics.pdf

- STMicroelectronics. (2024). *STM32G0B1xB/xC/xE Arm® Cortex®-M0+ 32-bit MCU datasheet* (DS13560 Rev. 5).  
  https://www.st.com/resource/en/datasheet/stm32g0b1cc.pdf
