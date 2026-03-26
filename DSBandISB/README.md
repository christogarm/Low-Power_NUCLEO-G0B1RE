# STM32 – STOP 1 Low Power Mode with DSB and ISB

Low power firmware for STM32 microcontroller implementing **STOP 1** mode, with explicit use of the `__DSB()` and `__ISB()` memory barriers to guarantee a safe and correct entry into the low power state.

---

## Overview

This firmware implements a normal / low power operation cycle controlled by a button on PC13. When the user presses the button, the system disables non-essential peripherals, sets all GPIOs to analog mode, powers down Flash memory, and enters **STOP 1** mode. A falling-edge interrupt on PC13 wakes the system up and resumes normal operation.

---

## Required Hardware

| Component | Description |
|---|---|
| MCU | STM32 (compatible with STM32G0/L0/L4 family) |
| LED | Connected to `LED_GREEN_Pin` (status blink) |
| Button | Connected to **PC13** (falling-edge input) |
| UART | USART2 at 115200 baud (date/time output to console) |
| IWDG | Independent watchdog active during normal operation |
| RTC | Real-time clock sourced from LSI |

---

## Configured Peripherals

### System Clock (`SystemClock_Config`)
- Source: **HSI** (High Speed Internal), no PLL
- Voltage regulator at scale 1 (`PWR_REGULATOR_VOLTAGE_SCALE1`)
- LSI enabled to feed the RTC and IWDG

### USART2
- Baudrate: **115200**
- Format: 8 bits, no parity, 1 stop bit
- Mode: TX/RX, no hardware flow control

### RTC
- Hour format: **24-hour**
- Source: LSI (prescalers: AsynchPrediv=127, SynchPrediv=255)
- Initial date/time: Wednesday 28/01/2026 at 00:00:00

### IWDG
- Prescaler: 256
- Reload: 500 (~5 s timeout)
- Refreshed continuously in the main loop and during button wait loops

### GPIOs
- **PC13**: falling-edge interrupt (button + STOP wake-up pin)
- **LED_GREEN_Pin**: push-pull output, high speed
- All other pins: analog mode, no pull (minimum leakage current)

---

## Operation Flow

```
Start
  └─> Peripheral initialization
  └─> Wait for PC13 button release
  └─> Main loop:
        ├─ Toggle LED every `delay` ms
        ├─ Print date/time via UART every 10 cycles
        ├─ Refresh IWDG
        └─ If valid press detected on PC13:
              ├─ Debounce (50 ms)
              ├─ Wait for button release
              ├─ GPIOs → analog mode
              ├─ Disable GPIO clocks (except GPIOC)
              ├─ De-initialize USART2
              ├─ Disable debugger in STOP mode
              ├─ Suspend SysTick
              ├─ Enable EXTI4_15 IRQ (PC13 as wake-up source)
              ├─ Enable Flash power-down in STOP
              ├─ Enter STOP 1 → enterSTOP1Mode_Safe()
              └─ On wake-up:
                    ├─ Resume SysTick
                    ├─ Re-initialize GPIO and USART2
                    └─ Set delay to 100 ms
```

---

## Key Function: `enterSTOP1Mode_Safe()`

This function implements the correct and safe sequence to enter STOP 1 mode on an ARM Cortex-M core, with explicit use of the **DSB** and **ISB** barriers.

```c
void enterSTOP1Mode_Safe(void) {
    /* Configure STOP 1 mode with low-power regulator */
    MODIFY_REG(PWR->CR1, PWR_CR1_LPMS, PWR_LOWPOWERMODE_STOP1);

    /* Set SLEEPDEEP bit in SCB to enable STOP mode entry */
    SET_BIT(SCB->SCR, SCB_SCR_SLEEPDEEP_Msk);

    /* DSB: Data Synchronization Barrier
       Ensures all memory writes (including PWR->CR1 and SCB->SCR
       above) are fully completed before the next instruction
       is executed. */
    __DSB();

    /* ISB: Instruction Synchronization Barrier
       Flushes the CPU pipeline and forces subsequent instructions
       to be re-fetched from already-updated memory, guaranteeing
       that WFI executes with the correct hardware configuration. */
    __ISB();

    /* WFI: Wait For Interrupt
       The CPU enters low-power mode and remains suspended until
       an enabled interrupt occurs (PC13 EXTI falling edge). */
    __WFI();

    /* On wake-up, clear SLEEPDEEP to restore normal operation */
    CLEAR_BIT(SCB->SCR, SCB_SCR_SLEEPDEEP_Msk);
}
```

### Why are DSB and ISB necessary?

| Barrier | Type | Purpose in this context |
|---|---|---|
| `__DSB()` | Memory | Ensures writes to `PWR->CR1` and `SCB->SCR` are visible to the hardware before `__WFI()` executes |
| `__ISB()` | Pipeline | Flushes the Cortex-M pipeline so no speculative instructions execute with stale configuration |

Without these barriers, the processor may execute `__WFI()` before the hardware has applied the STOP 1 configuration, leading to undefined behavior or higher-than-expected current consumption.

---

## Current Reduction Strategy

Before entering STOP 1, the firmware applies the following measures:

1. **GPIOs set to analog mode** — all inactive pins are configured as analog (`MODER = 0xFFFFFFFF`), eliminating leakage current from floating inputs.
2. **GPIO clocks disabled** — clocks for GPIOA, B, D, E, and F are turned off. Only GPIOC remains active as it contains the wake-up pin PC13.
3. **USART2 de-initialized** — `HAL_UART_DeInit()` is called to fully disconnect the peripheral.
4. **Debugger disabled** — `HAL_DBGMCU_DisableDBGStopMode()` prevents the debugger from keeping unnecessary clocks alive.
5. **SysTick suspended** — `HAL_SuspendTick()` disables the periodic system tick interrupt.
6. **Flash powered down** — `HAL_PWREx_EnableFlashPowerDown(PWR_FLASHPD_STOP)` shuts down Flash memory during STOP, significantly reducing current consumption.

---

## Wake-up

The system wakes exclusively via the **EXTI4_15** interrupt associated with **PC13** (falling edge). This IRQ is explicitly enabled before entering STOP mode and is handled in the `EXTI4_15_IRQHandler`.

> **Note:** During STOP 1, the IWDG continues running on LSI. If time in STOP exceeds the IWDG timeout (~5 s with the current configuration), the system will reset. You can freeze the IWDG counter in STOP mode by configuring the **`IWDG_STOP` Option Byte**.

---

## UART Output

Every 10 cycles of the main loop, the current RTC time and date are printed:

```
00:01:35  28/01/2026
```

The `_write()` function redirects `printf()` to USART2 via `HAL_UART_Transmit()`.

---

## Build Notes

- Generated with **STM32CubeIDE / STM32CubeMX**
- HAL version compatible with STM32G0xx or STM32L4xx depending on the project's `main.h`
- Ensure `USE_HAL_DRIVER` and the correct MCU family are defined in the compiler flags

---

## License

Copyright © 2026 STMicroelectronics. All rights reserved.  
Distributed under the terms of the `LICENSE` file included in the software component.
