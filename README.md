# STM32F411RE — Bare-Metal UART Ring Buffer

> Interrupt-driven UART communication with a circular buffer implementation on STM32F411RE (Nucleo-64), written entirely in bare-metal C — no HAL, no CubeMX, no abstraction layers.

---

## Hardware

| Component | Detail |
|---|---|
| MCU | STM32F411RE (Nucleo-64) |
| Core | ARM Cortex-M4 @ 100MHz |
| Peripherals used | USART2, GPIO |
| Toolchain | GCC ARM, STM32CubeIDE / Makefile |

---

## What This Project Demonstrates

- **Circular (ring) buffer** — fixed-size FIFO with head/tail pointer management and modulo wrap-around
- **Interrupt-driven UART RX** — data received via `USART2_IRQHandler`, written into the ring buffer from ISR context
- **Volatile correctness** — shared buffer state between ISR and main loop declared `volatile` to prevent compiler reordering
- **Bare-metal peripheral init** — direct register manipulation using `stm32f4xx.h` (RCC, GPIO, USART, NVIC)
- **No HAL dependency** — every register configured by hand using bit shifts and masks

---

## Key Concepts

```
USART2 RX → RXNE interrupt → ISR writes to ring buffer → main loop reads
```

- Buffer is a fixed array with `head` (write) and `tail` (read) indices
- Wrap-around handled via modulo: `head = (head + 1) % BUFFER_SIZE`
- Full/empty detection via index comparison
- ISR only writes; main loop only reads — no mutex needed on Cortex-M4 with atomic index updates

---

## Project Structure

```
├── Core/
│   ├── Src/
│   │   ├── main.c          # Main loop, buffer read logic
│   │   ├── uart.c          # USART2 init, IRQ handler
│   │   └── ring_buffer.c   # Circular buffer implementation
│   ├── Inc/
│   │   ├── uart.h
│   │   └── ring_buffer.h
├── Startup/
│   └── startup_stm32f411xe.s
├── STM32F411RETx_FLASH.ld  # Linker script
└── README.md
```

---

## Register-Level UART Init (No HAL)

```c
// Enable clocks
RCC->APB1ENR |= RCC_APB1ENR_USART2EN;
RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;

// PA2 = TX, PA3 = RX → Alternate Function 7
GPIOA->MODER  |=  (2U << 4) | (2U << 6);   // AF mode
GPIOA->AFR[0] |=  (7U << 8) | (7U << 12);  // AF7 = USART2

// Baud rate: 100MHz / (16 * 9600) ≈ 651
USART2->BRR = 0x028B;

// Enable RXNE interrupt, TX, RX, USART
USART2->CR1 |= USART_CR1_RXNEIE | USART_CR1_TE | USART_CR1_RE | USART_CR1_UE;

// Enable IRQ in NVIC
NVIC_EnableIRQ(USART2_IRQn);
```

---

## Build & Flash

```bash
# Build (Makefile-based)
make

# Flash via ST-Link
st-flash write build/output.bin 0x08000000
```

---

## Skills Demonstrated

| Area | Detail |
|---|---|
| C Language | Pointers, volatile, structs, function pointers, undefined behavior awareness |
| Memory | Stack vs heap, fixed-size buffers, ISR-safe data sharing |
| Peripherals | USART, GPIO, NVIC — direct register access |
| Firmware patterns | Circular buffer, interrupt-driven I/O, polling fallback |
| Toolchain | GCC ARM, linker scripts, startup assembly |

---

## Target Roles

This project is part of a portfolio targeting **automotive and industrial embedded** roles (KPIT, Bosch, Continental) requiring:
- Bare-metal C on ARM Cortex-M
- Production-quality firmware patterns
- No reliance on middleware or HAL abstraction

---

## Author

**Ansari**
Embedded Systems Engineer — Vi Microsystems
Hardware: STM32F411RE · STM32H723ZG · ESP32 · RP2040 · LoRa
