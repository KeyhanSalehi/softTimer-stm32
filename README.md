# softTimer-stm32

Lightweight, non-blocking software timer library for STM32 microcontrollers.  
Provides simple APIs for **timeouts**, **periodic tasks**, and **event scheduling** without blocking the CPU.  

---

## ✨ Features
- ✅ Compatible with all STM32 MCUs (uses `HAL_GetTick()` internally).  
- ✅ Minimal API (only two functions).  
- ✅ Macros for easy time conversion (`SEC_TO_MS`, `MIN_TO_MS`, `MS_TO_US`, `SEC_TO_US`).  
- ✅ Non-blocking – no `delay()` calls, CPU remains free.  
- ✅ Perfect for **bare-metal systems** where RTOS is not used.  
- ✅ Useful in **state machines**, **timeouts**, and **periodic task execution**.  

---

## 📂 File Structure
softTimer-stm32/
│
├── softTimer.h # API & Macros
└── README.md # Documentation

---

## 🚀 Getting Started

### 1. Add Library
Copy `softTimer.c` and `softTimer.h` into your project.  
Include in your code:
```c
#include "softTimer.h"

2. Initialize Timer
Simply create a variable to store the start time:

uint32_t myTimer = 0;
softTimer_reset(&myTimer);   // Start the timer

3. Check Timeout

Use softTimer_isElapsed() to check if a timeout has passed:

if (softTimer_isElapsed(&myTimer, SEC_TO_MS(2))) {
    // 2 seconds passed
    softTimer_reset(&myTimer);  // Restart timer if needed
}

🔧 API Reference
bool softTimer_isElapsed(uint32_t *timer, uint32_t timeout);
Checks if the given timeout (in ms) has elapsed since the last reset.

Returns true if elapsed, otherwise false.

void softTimer_reset(uint32_t *timer);

Resets timer to current system tick (HAL_GetTick()).

⏱️ Time Conversion Macros

SEC_TO_MS(sec) → Convert seconds to milliseconds

MIN_TO_MS(min) → Convert minutes to milliseconds

MS_TO_US(ms) → Convert milliseconds to microseconds

SEC_TO_US(sec) → Convert seconds to microseconds

📘 Examples
Example 1: Simple Timeout in Bare-Metal System

#include "softTimer.h"

uint32_t ledTimer = 0;

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

✅ Non-blocking alternative to HAL_Delay(1000)
✅ CPU is free to handle other tasks while waiting.

Example 2: State Machine with Timeout

#include "softTimer.h"

typedef enum {
    STATE_IDLE,
    STATE_WAIT,
    STATE_DONE
} app_state_t;

app_state_t state = STATE_IDLE;
uint32_t stateTimer = 0;

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
✅ Perfect for event-driven applications
✅ No delay(), state transitions are time-controlled

🎯 Where to Use

Bare-metal STM32 projects (without RTOS).

Implementing timeouts for communication protocols (UART, SPI, I2C).

Periodic tasks (LED blinkers, sensor polling, watchdog refresh).

State machines where non-blocking timing is needed.

Low-power systems where CPU must stay free for other tasks.
