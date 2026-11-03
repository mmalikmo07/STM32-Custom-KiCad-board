# STM32F103 Development Board

A 2-layer, 45 × 40 mm development board built around the STM32F103C8T6 (Arm Cortex-M3, 72 MHz), designed in KiCad 9 from schematic through layout to Gerbers. It's powered and programmed over USB, with SWD for debugging and breakout headers for UART and I²C.

This was my first full board. It began as a guided KiCad tutorial build, which is why the files are named `KiCad-STM32-tut`. Below are the design choices and what I'd change in the next revision.

<img width="577" alt="PCB layout" src="https://github.com/user-attachments/assets/3aaf17d7-7595-418f-a98f-a9981bb5419b" />
<img width="593" alt="3D render" src="https://github.com/user-attachments/assets/527df36a-2131-4360-8fd2-138caafa5b70" />
<img width="415" alt="Schematic detail" src="https://github.com/user-attachments/assets/59690d8c-61eb-478a-a39d-29c0ede3dbda" />

## Status

**Design complete, not yet fabricated.** The fab-ready Gerbers and drill files are in [`gerbers/`](gerbers/). The next steps are fabrication, assembly and bring-up (see below).

## Block overview

| Block | Implementation | Notes |
|---|---|---|
| MCU | STM32F103C8T6, LQFP-48 | 64 KB flash, 20 KB SRAM |
| Power | USB VBUS (5 V) → AMS1117-3.3 LDO (SOT-223) | 22 µF bulk capacitors on the input and output, as the AMS1117 needs for stability |
| Decoupling | 100 nF on the VDD pins, plus 10 µF bulk on 3.3 V | |
| Analogue supply | VDDA fed from 3.3 V through a 120 Ω ferrite bead, with 1 µF + 10 nF | Keeps digital switching noise off the ADC reference |
| Clock | 16 MHz crystal (3.2 × 2.5 mm), 10 pF load caps | The PLL gives 72 MHz SYSCLK, and the 48 MHz USB clock is derived from it |
| USB | Micro-B (Würth 629105150521), USB FS on PA11/PA12 | 1.5 kΩ pull-up on D+, which the F103 needs because it has no internal one |
| Boot | SPDT slide switch → 10 kΩ → BOOT0 | Selects the system bootloader or flash at reset |
| Debug / IO | 4-pin headers: SWD (SWDIO, SWCLK), USART1 (remapped to PB6/PB7), I²C2 (PB10/PB11, 1.5 kΩ pull-ups) | |
| Indicator | Power LED (0603) | |
| Mechanical | 4 × M2 mounting holes | |

**Layout:** 2 layers, with a solid GND pour on the bottom layer as the return plane. The top layer carries GND, 3.3 V and VBUS pours around the signal routing. Passives are mostly 0402.

## Rev B: what I'd change

These are the issues I found reviewing my own netlist:

1. **USB connector shield is floating.** The shield pads are unconnected. Tying them to GND through a ferrite bead or an RC network (e.g. 1 MΩ ∥ 4.7 nF) would improve EMI and ESD behaviour.
2. **No ESD protection on the USB lines.** Add a TVS array such as the USBLC6-2SC6 on D+/D− and VBUS.
3. **No input protection.** Add a polyfuse on VBUS and reverse-current protection if the board will ever be powered from the headers.
4. **Regulator choice.** An LDO is fine at this board's load. At around 100 mA it dissipates (5 − 3.3) × 0.1 ≈ 0.17 W. If I add peripherals that draw hundreds of mA, I'd move to a buck converter for efficiency.
5. **Connector.** USB-C with 5.1 kΩ CC pull-downs would make it work with modern cables and chargers.

## Bring-up plan

1. Visual inspection, then check continuity and look for shorts between 3.3 V, VBUS and GND
2. Power from a current-limited bench supply and check the 3.3 V rail and its ripple
3. Connect over SWD with an ST-Link and read the device ID
4. Flash a blinky over SWD, then test USB DFU through the BOOT0 switch
5. Check the 16 MHz HSE on a scope, then confirm the PLL by toggling a GPIO with MCO

## Files

```
KiCad-STM32-tut.kicad_pro / .kicad_sch / .kicad_pcb   KiCad 9 project
gerbers/                                             Gerbers, drill files, and a zipped fab package
3d-models/                                           STEP model for the USB connector
```

Tools: KiCad 9, STM32CubeMX (pin planning).
