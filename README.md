# Tinyscope

> A pocket-sized, battery-powered oscilloscope for diagnosing vehicle electrical signals — with a live waveform display
> and wireless streaming to your phone.

## Overview

> Tinyscope is a two-channel handheld oscilloscope built specifically for automotive work. It captures fast ignition
> and sensor waveforms, renders them in real time on an on-board IPS display, and can stream the same data over Bluetooth Low Energy
> to a smartphone for a bigger screen and logging.

It is designed to safely probe signals such as:

- **Secondary ignition** (spark plug / coil) via a clip-on capacitive HT pickup — *never a direct connection*
- **Primary ignition** and coil drive
- **Tachometer** and **crank/cam** sensor signals
- **Vehicle speed sensor (VSS)** outputs
- **Injector** drive pulses
- General **sensor and logic-level** lines

## Why two processors?

> Ignition waveforms contain fast edges that an MCU's on-chip ADC cannot sample without aliasing, while wireless telemetry is a slow,
> bursty task. TinyScope splits the work:

- **STM32H743** (Cortex-M7) — the real oscilloscope engine: dual simultaneous-sampling 16-bit ADCs, deep capture RAM, hardware triggering, and the display.
- **nRF52840** (BLE module) — a dedicated wireless link to the phone. It never touches the analog path; it only relays capture buffers and accepts control commands.

## Features

- 2 analog channels — one general-purpose, one dedicated to the ignition pickup
- Simultaneous 16-bit sampling for true time-correlation between channels
- Hardware trigger with adjustable threshold
- Real-time waveform rendering on a 320×240 IPS LCD
- BLE streaming to a companion smartphone app
- USB-C charging with power-path (run while charging, even on a dead battery)
- Optional protected 12 V vehicle input for all-day use in the car
- On-board battery fuel gauge
- Floating (battery) operation for safe probing

## Specifications

| Parameter | Value |
|---|---|
| Channels | 2 (CH A general-purpose, CH B ignition pickup) |
| Acquisition MCU | STM32H743 (480 MHz Cortex-M7) |
| ADC | Dual 16-bit, simultaneous, up to ~3.6 MSPS each |
| Analog front-end bandwidth | ~1 MHz target (ignition waveform shape) |
| Input protection | TVS + clamp diodes, switchable attenuator |
| Wireless | Bluetooth LE 5 (Raytac MDBT50Q / nRF52840) |
| Display | 2.4–2.8" IPS LCD, 320×240, SPI (ILI9341) |
| Battery | 1S LiPo |
| Charger | BQ24075 (USB-C, power-path) |
| Vehicle input | 12 V protected input → TPS54360B-Q1 buck |
| Fuel gauge | MAX17048 (I²C) |
| Inputs | 2× BNC + capacitive HT pickup accessory |

> **Note:** TinyScope targets the *shape* of ignition and sensor waveforms, not MHz-class edge timing. It is a diagnostic tool,
> not a lab-grade instrument.

## ⚠️ Safety

> Working on vehicles involves real hazards. Read this before building or using TinyScope.

- **Never connect the instrument directly to a spark plug or HT lead.** Secondary ignition reaches tens of kilovolts. Use the capacitive pickup accessory only.
- The device is designed to run on its **internal battery so it floats** — this is the safe way to probe. When powered from the vehicle's 12 V instead, its ground is bonded to vehicle ground; do not simultaneously probe circuits referenced to a different ground.
- Observe correct input ranges and use the appropriate attenuator setting.
- This is a hobbyist/educational project. Use at your own risk.

## Controls

> The front panel uses five rotary encoders, one per oscilloscope control, so adjustments feel like a traditional scope.

Each control is a **Bourns PEC11R** 12 mm incremental (quadrature) encoder with 24 detents and an integrated momentary push switch.
The push switch on every knob doubles as a shortcut (e.g. recenter a position, toggle fine/coarse on a scale, or set the trigger to 50%).

| Encoder | Function | Push-switch shortcut |
|---|---|---|
| Vertical scale | Volts/division | Fine / coarse toggle |
| Vertical position | Trace offset | Recenter to zero |
| Horizontal scale | Time/division (timebase) | Fine / coarse toggle |
| Horizontal position | Trace time offset | Recenter |
| Trigger position | Trigger point along the sweep | Set to 50% |

**Reading them:** The high-use encoders are decoded by the STM32's hardware **timer encoder mode** (A/B into a timer's two channels, zero CPU overhead);
the rest are read via GPIO interrupts. All five push switches are plain GPIO inputs. Each A/B/switch line uses an RC filter for debounce.
Encoder common pins tie to GND.

## Power architecture

> The power input and charging stage handles three jobs: selecting a source, protecting against the automotive environment,
> and producing clean rails.

- **USB-C input** — 5 V bench/charging port (sink role).
- **12 V vehicle input** — fused, reverse-polarity protected (P-FET), transient-clamped (TVS), then stepped down by a wide-input automotive buck.
- **Source OR-ing** — a Schottky diode-OR feeds whichever source is present into the charger.
- **Charging & power path** — BQ24075 runs the system while charging the cell.
- **Fuel gauge** — MAX17048 reports state-of-charge over I²C.

## Status & roadmap

> This project is in the **hardware design phase**.

- [x] System architecture and block diagram
- [x] Power input & charging schematic
- [ ] Analog front-end schematic (CH A / CH B)
- [ ] Acquisition MCU schematic
- [ ] Wireless module schematic
- [ ] Display & user I/O schematic
- [ ] PCB layout
- [ ] First prototype bring-up
- [ ] STM32 acquisition firmware
- [ ] nRF52 BLE firmware
- [ ] Smartphone app

## Contributing

> Issues and pull requests are welcome. If you're reviewing the hardware, schematic feedback (especially on the analog front end and protection circuits)
> is especially valued.

## License

*TBD — choose a license (e.g. CERN-OHL for hardware, MIT/Apache-2.0 for firmware).*

## Disclaimer

> TinyScope is an open hardware project provided "as is", without warranty of any kind. The authors are not responsible for any damage to vehicles,
> equipment, or persons resulting from its construction or use.
