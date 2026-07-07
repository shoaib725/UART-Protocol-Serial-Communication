# UART Protocol: Implementation, Simulation & Comparative Analysis

A study of the UART protocol implemented and verified across three complementary embedded development environments: **Keil µVision (8051 Assembly)**, **Proteus Design Suite** (circuit simulation), and **Arduino (Tinkercad Circuits)**.

Both implementations transmit the string `"Embedded Systems & IOT"` at **9600 baud, 8 data bits, no parity, 1 stop bit (8N1)**, allowing a direct, apples-to-apples comparison of register-level vs. library-abstracted UART programming.

## Overview

UART (Universal Asynchronous Receiver Transmitter) is an asynchronous, full-duplex serial protocol using only TX and RX lines. This project explores UART from two ends of the abstraction spectrum:

- **8051 Assembly (Keil µVision):** Direct manipulation of the `SCON`, `TMOD`, `TH1`, and `SBUF` registers, with manual polling of the `TI` flag.
- **Arduino (Tinkercad, C++):** The same functionality via the built-in `Serial` library, hiding all register-level configuration.

**Proteus Design Suite** bridges the two, using the HEX file compiled by Keil to simulate an AT89C51 circuit and verify the transmitted data on a Virtual Terminal — a hardware-free validation step used in real embedded workflows.

## Tools & Platforms

| Platform | Role | Verified Output |
|---|---|---|
| **Keil µVision** | 8051 Assembly UART development & debugging | Disassembly window, register states (SCON, TMOD, TH1, SBUF, TI) |
| **Proteus Design Suite** | Circuit-accurate simulation of AT89C51 using Keil's compiled HEX | Virtual Terminal showing transmitted string |
| **Tinkercad Circuits** | Arduino UNO UART implementation in C++ | Serial Monitor showing repeated string output |

## Implementation

### 8051 Assembly (Keil µVision)

```asm
org 0000h

mov scon, #50h      ; UART Mode 1, 8-bit UART, REN enabled
mov tmod, #20h      ; Timer 1, Mode 2 (8-bit auto-reload)
mov th1, #0fdh      ; Reload value for 9600 baud @ 11.0592 MHz
setb tr1            ; Start Timer 1
mov dptr, #message  ; Load message base address

transmit:
clr a
next_char:
movc a, @a+dptr     ; Load character from code memory
jz transmit         ; Repeat on null terminator
mov sbuf, a         ; Load byte into transmit buffer

wait_transmit:
jnb ti, wait_transmit ; Poll until transmission complete
clr ti               ; Clear transmit flag for next byte
inc dptr             ; Advance to next character
sjmp next_char       ; Repeat

message: db 'Embedded Systems & IOT', 00h
end
```

### Arduino (Tinkercad, C++)

```cpp
void setup() {
  Serial.begin(9600); // Configure UART: 9600 baud, 8N1
}

void loop() {
  Serial.println("Embedded Systems & IOT");
  delay(1000); // Repeat once per second
}
```

## Comparative Analysis

| Aspect | Keil / 8051 Assembly | Arduino (Tinkercad) |
|---|---|---|
| Programming Level | Low-level, register-based | High-level, library-based |
| Lines of Code | ~19 lines | ~6 lines |
| UART Setup | Manual `SCON`, `TMOD`, `TH1` config | `Serial.begin(9600)` |
| Data Transmission | Manual `SBUF` load + `TI` polling | `Serial.println()` |
| Learning Curve | Steep | Gentle |
| Development Time | Higher | Lower |
| Hardware Control | Full, precise | Limited to library scope |
| Portability | Tied to 8051 register map | Portable across Arduino boards |
| Best Suited For | Embedded systems education, optimization | Rapid prototyping, IoT |

**Key finding:** The Arduino implementation uses roughly **68% fewer lines of code** than the equivalent Assembly version, trading fine-grained hardware control for development speed.

## Results

All three implementations were verified against the same success criterion — correct, repeated transmission of `"Embedded Systems & IOT"` at 9600-8N1 — and matched byte-for-byte:

- **Keil µVision:** Verified via disassembly/register watch (`SCON`, `TMOD`, `TH1`, `TI` toggling correctly).
- **Proteus:** Verified via Virtual Terminal, no corrupted or dropped characters.
- **Tinkercad:** Verified via Serial Monitor, string printed once per second as coded.

## Challenges & Learnings

- **Baud rate accuracy** depends on using the correct crystal frequency (11.0592 MHz) for an exact 9600 baud `TH1` reload value.
- **Forgetting to clear the `TI` flag** after each byte causes the polling loop to stall on the first character.
- **Incorrect null-termination** of the message string causes the transmit loop to overrun and print garbage.
- **Proteus requires the exact HEX file** for the correct microcontroller (AT89C51) — mismatches silently produce no output.
- **Fair LOC comparison** required excluding comments/blank lines to reflect genuine complexity differences.

## Author

**Pathan Mohammad Shoaib Khan**
B.Tech, Electronics and Communication Engineering

## References

- 8051 Microcontroller Architecture and Serial Communication — Keil µVision documentation
- Proteus Design Suite — Labcenter Electronics, VSM documentation
- Arduino Serial Library Reference — Arduino.cc official documentation
- Tinkercad Circuits — Autodesk online circuit simulation environment
