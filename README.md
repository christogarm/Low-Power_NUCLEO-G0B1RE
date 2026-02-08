# Low-Power Modes in NUCLEO-G0B1RE

Low-power modes allow saving power when the CPU does not need to remain running.

This device provides seven low-power modes:

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

The following table shows the typical measured current consumption.

<center>

| **Low-Power Mode**     | **Typical Current Consumption \*** |
| :--------------------- | :----------------------------------|
| Stop Mode (0 / 1)\*\*  | 105 / 3.45 [µA]                     |
| Standby Mode           | 0.25 [µA]                           |
| Shutdown Mode          | 47.5 [nA]                           |

</center>

> \* All low-power modes are measured under their lowest current consumption conditions .<br>
> \*\* Stop Mode values correspond to STOP 0 / STOP 1 respectively.
> This data is interpolated between VDD = 3.6 V and VDD = 3.0 V to obtain values at VDD = 3.3 V.


You can consult the theory current consumption in [STM32G0B1 Datasheet](https://www.st.com/resource/en/datasheet/stm32g0b1cc.pdf) in the section **5.3.5 Supply current characteristics** to review your particular case.

## How to test these modes?

Disconnect **JP3** and measure the current consumption to verify the low-power modes.

![Screenshot of Nucleo-G0B1RE from https://www.st.com/en/evaluation-tools/nucleo-g0b1re.html#.](img/Nucleo-G0B1RE-JP3.png)

### Serial Configuration Parameters

If you want to see the Date and Time of the RTC, configure the serial terminal with the following settings:

* Port: COMx (check Device Manager)
* Baud rate: 115200
* Data bits: 8
* Parity: None
* Stop bits: 1
* Flow control: None

## References

- STMicroelectronics. (2024). *RM0444: STM32G0x1 advanced Arm®-based 32-bit MCUs reference manual* (Rev. 6).  
  https://www.st.com/resource/en/reference_manual/rm0444-stm32g0x1-advanced-armbased-32bit-mcus-stmicroelectronics.pdf

- STMicroelectronics. (2024). *STM32G0B1xB/xC/xE Arm® Cortex®-M0+ 32-bit MCU datasheet* (DS13560 Rev. 5).  
  https://www.st.com/resource/en/datasheet/stm32g0b1cc.pdf
