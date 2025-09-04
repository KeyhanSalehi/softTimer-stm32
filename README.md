# softTimer-stm32
Lightweight, non-blocking **software timer library** for STM32 microcontrollers.  
Provides simple APIs for **timeouts**, **periodic tasks**, and **event scheduling** without blocking the CPU.

---

## ✨ Features
- ✅ Header-only (just include `softTimer.h`, no `.c` file needed).
- ✅ Compatible with all STM32 MCUs (uses `HAL_GetTick()` internally).
- ✅ Minimal API (only two inline functions).
- ✅ Macros for easy time conversion (`SEC_TO_MS`, `MIN_TO_MS`, `MS_TO_US`, `SEC_TO_US`).
- ✅ Non-blocking – no `delay()` calls, CPU remains free.
- ✅ Perfect for **bare-metal systems** where RTOS is not used.
- ✅ Useful in **state machines**, **timeouts**, and **periodic task execution**.

---

## 📂 File Structure
```
softTimer-stm32/
│
├── softTimer.h  # Header-only API & Macros
└── README.md    # Documentation
```

---

## 🚀 Getting Started

### 1. Add Library
Copy `softTimer.h` into your project.  
Include in your code:
```c
#include "softTimer.h"
```

### 2. Initialize Timer
Create a timer variable and reset it:
```c
softTimer_t myTimer = 0;
softTimer_reset(&myTimer); // Start the timer
```

### 3. Check Timeout
Check if the given timeout has elapsed:
```c
if (softTimer_isElapsed(&myTimer, SEC_TO_MS(2))) {
    // 2 seconds passed
    softTimer_reset(&myTimer); // Restart timer if needed
}
```

---

## 🔧 API Reference

```c
void softTimer_reset(softTimer_t *timer);
```
Resets timer to current system tick (`HAL_GetTick()` or user-defined).

```c
bool softTimer_isElapsed(softTimer_t *timer, uint32_t timeout);
```
Checks if the given timeout (in ms) has elapsed since the last reset.  
Returns `true` if elapsed, otherwise `false`.

---

## ⏱️ Time Conversion Macros
- `SEC_TO_MS(sec)` → Convert seconds to milliseconds
- `MIN_TO_MS(min)` → Convert minutes to milliseconds
- `MS_TO_US(ms)` → Convert milliseconds to microseconds
- `SEC_TO_US(sec)` → Convert seconds to microseconds

---

## 📘 Examples

### Example 1: Simple Timeout in Bare-Metal System
```c
#include "softTimer.h"

softTimer_t ledTimer = 0;

int main(void) {
    HAL_Init();
    softTimer_reset(&ledTimer);
    while (1) {
        if (softTimer_isElapsed(&ledTimer, SEC_TO_MS(1))) {
            HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13); // Toggle LED every 1s
            softTimer_reset(&ledTimer);
        }
    }
}
```
- ✅ Non-blocking alternative to `HAL_Delay(1000)`
- ✅ CPU is free to handle other tasks while waiting.

### Example 2: State Machine with Timeout
```c
#include "softTimer.h"

typedef enum {
    STATE_IDLE,
    STATE_WAIT,
    STATE_DONE
} app_state_t;

app_state_t state = STATE_IDLE;
softTimer_t stateTimer = 0;

int main(void) {
    HAL_Init();
    softTimer_reset(&stateTimer);
    while (1) {
        switch (state) {
            case STATE_IDLE:
                // Do some work
                softTimer_reset(&stateTimer);
                state = STATE_WAIT;
                break;
            case STATE_WAIT:
                // Wait for 5 seconds without blocking
                if (softTimer_isElapsed(&stateTimer, SEC_TO_MS(5))) {
                    state = STATE_DONE;
                }
                break;
            case STATE_DONE:
                // Work completed
                // Reset to idle or stay here
                state = STATE_IDLE;
                break;
        }
    }
}
```
- ✅ Perfect for event-driven applications
- ✅ No blocking `delay()`, state transitions are time-controlled

### Example 3: UART Timeout Handling
```c
#include "softTimer.h"
#include "stm32f4xx_hal.h" // Example for STM32F4, adjust for your MCU

UART_HandleTypeDef huart2; // Example UART handle
softTimer_t uartTimer = 0;
uint8_t rxBuffer[10];
bool dataReceived = false;

void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart) {
    dataReceived = true; // Data received, stop waiting
}

int main(void) {
    HAL_Init();
    // Initialize UART (example configuration for STM32F4)
    huart2.Instance = USART2;
    huart2.Init.BaudRate = 115200;
    huart2.Init.WordLength = UART_WORDLENGTH_8B;
    huart2.Init.StopBits = UART_STOPBITS_1;
    huart2.Init.Parity = UART_PARITY_NONE;
    huart2.Init.Mode = UART_MODE_RX;
    HAL_UART_Init(&huart2);

    softTimer_reset(&uartTimer);
    HAL_UART_Receive_IT(&huart2, rxBuffer, 10); // Start UART receive

    while (1) {
        if (dataReceived) {
            // Process received data
            dataReceived = false;
            softTimer_reset(&uartTimer);
            HAL_UART_Receive_IT(&huart2, rxBuffer, 10); // Restart UART receive
        } else if (softTimer_isElapsed(&uartTimer, SEC_TO_MS(2))) {
            // Timeout: no data received in 2 seconds
            HAL_UART_AbortReceive_IT(&huart2); // Stop UART
            softTimer_reset(&uartTimer);
            HAL_UART_Receive_IT(&huart2, rxBuffer, 10); // Restart UART receive
        }
    }
}
```
- ✅ Non-blocking UART timeout handling
- ✅ Resets UART receive operation on timeout

---

## 🎯 Where to Use
- Bare-metal STM32 projects (without RTOS).
- Implementing timeouts for communication protocols (UART, SPI, I2C).
- Periodic tasks (LED blinkers, sensor polling, watchdog refresh).
- State machines where non-blocking timing is needed.
- Low-power systems where CPU must stay free for other tasks.

---

## 📜 License
This library is open-source under the **MIT License**.  
You are free to use, modify, and distribute in commercial or personal projects.

---

## 💡 Tip
Replace `HAL_GetTick()` by redefining `SOFTTIMER_TICK()` macro if you want to use a custom tick source (e.g., SysTick, timer peripheral, or simulation).
