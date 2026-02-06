# Stop Mode

In **Stop 0** and **Stop 1** modes, the device achieves very low power consumption while **retaining SRAM and register contents**.  
The CPU clock is stopped and most peripherals are disabled.  
Unlike Standby mode, the device **does not reset** when exiting Stop mode.

## Enter Stop Mode
To enter Stop mode, use the following HAL function:

```c
HAL_PWR_EnterSTOPMode(uint32_t Regulator, uint8_t STOPEntry);
```
### Regulator Selection

The `Regulator` parameter specifies which internal voltage regulator remains active during Stop mode:
  * Main regulator (`Regulator = PWR_MAINREGULATOR_ON`)
  * Low-Power Regulator (`Regulator = PWR_LOWPOWERREGULATOR_ON`)

**STOP 0:** uses Main Regulator. <br>
**STOP 1:** uses Low-Power Regulator.

As a result:
* STOP1 provides lower current consumption.
* STOP0 provides a faster wake-up time.

### Stop Entry Method 
* The `STOPEntry` parameter defines how the CPU enters Stop mode:
  * WFI - Wait For Interrupt (`STOPEntry = PWR_STOPENTRY_WFI`)
  * WFE - Wait For Event (`STOPEntry = PWR_STOPENTRY_WFE`)

Behavior when exiting Stop mode:
* WFI: the CPU wakes up, executes the corresponding interrupt service routine (ISR), and then continues execution in the main program.

* WFE: the CPU wakes up and continues execution in the main program without executing an ISR, unless an interrupt is also pending.

## System Clock After Wake-Up

When exiting Stop mode, the system clock is automatically switched to HSI.
If another clock source (such as PLL or HSE) is required, it must be reconfigured by software after wake-up.

## Wake-Up Sources
The following events or interrupts can be used to exit Stop mode on STM32G0B1 devices:

* BOR (Brown-out Reset)
* PVD (Programmable Voltage Detector)
* CSS (Clock Security System)
* USART / LPUART
* I2C
* COMP (Comparator)
* LPTIM
* USB
* GPIO external interrupt (EXTI)
* RTC event
* TAMP event

Note: The availability of some wake-up sources depends on the peripheral configuration and EXTI mapping.
## Current Consumption

<center>

| **STOP Mode** | **Current Consumption** | **Measured Current Consumption** |
|---------------|-------------------------|----------------------------------|
| STOP 0        | 105 µA \*               | 112.5 µA                         |
| STOP 1        | 3.95 µA                 | 5.1 µA                           |

</center>

> \* Current consumption corresponds to STOP 0 mode only, with no additional peripherals enabled.
> This data is interpolated between VDD = 3.6 V and VDD = 3.0 V to obtain values at VDD = 3.3 V.

The current consumption values are specified at **VDD = 3.3 V**, **25 °C**, with **RTC enabled** and **Flash memory powered down**, unless otherwise stated.

## Other Low-Power Characteristics

### Flash memory powered down (FPD_STOP)

This configuration determines whether the Flash memory is put in **power-down mode** or remains in **idle mode** when the device enters Stop mode.

If Flash memory remains **idle** (`HAL_PWREx_DisableFlashPowerDown(PWR_FLASHPD_STOP);`), the current consumption increases, but the **wake-up latency is reduced**.

If Flash memory is **powered down** (`HAL_PWREx_EnableFlashPowerDown(PWR_FLASHPD_STOP);`), the current consumption is lower, but an **additional wake-up latency** is introduced while the Flash memory becomes ready. This process is handled automatically by the hardware.

The following table shows the measured current consumption with **Flash memory in idle mode**:

<center>

| **STOP Mode** | **Measured Current Consumption** |
|---------------|----------------------------------|
| STOP 0        | 125.3 µA                         |
| STOP 1        | 13.4 µA                          |

</center>

### Ultra-low-power enable (ENB_ULP)

This configuration enables or disables **periodical sampling of the supply voltage** in Stop and Standby modes for detecting **PDR and BOR reset conditions**.

When ultra-low-power mode is enabled (`HAL_PWREx_EnablePORMonitorSampling();`), the supply voltage is not continuously monitored. If the voltage drops below the minimum operating condition between two samples, a **BOR or PDR reset may not be detected**.

This may result in **unreliable system behavior**, as the device could continue operating outside its specified voltage range. For this reason, this mode should only be used when a **stable and well-controlled power source** is guaranteed.

The following table shows the measured current consumption with **Ultra-low-power mode enabled**:

<center>

| **STOP Mode** | **Measured Current Consumption** |
|---------------|----------------------------------|
| STOP 0        | 112.6 µA                         |
| STOP 1        | 4.2 µA                           |

</center>

## Important Notes

- Disable all **unused peripherals** to reduce current consumption.
- When using a debugger, it must be disabled to achieve accurate Stop mode current measurements. This can be done using `HAL_DBGMCU_DisableDBGStopMode();`.
- If the IWDG is used, ensure that the `IWDG_STOP` option byte is disabled to freeze the IWDG counter while the MCU is in Stop mode.

## References

- STMicroelectronics. (2024). *RM0444: STM32G0x1 advanced Arm®-based 32-bit MCUs reference manual* (Rev. 6).  
  https://www.st.com/resource/en/reference_manual/rm0444-stm32g0x1-advanced-armbased-32bit-mcus-stmicroelectronics.pdf

- STMicroelectronics. (2024). *STM32G0B1xB/xC/xE Arm® Cortex®-M0+ 32-bit MCU datasheet* (DS13560 Rev. 5).  
  https://www.st.com/resource/en/datasheet/stm32g0b1cc.pdf