# External-data device — revision-zero BoM

## Scope

This is a **custom-board** raw-waveform recorder/replayer: no development boards, audio boards, SD modules, ADC/DAC modules, or converter boards. It captures `ECS OUT TO TAPE`, retains the raw waveform, and replays it into `ECS IN FROM TAPE`.

It is a planning BoM, not a release-to-build BoM. The AD2 `TEST` baseline determines final input range, sample rate, filter corners, DAC resolution, output impedance, and playback-driver choice.

Inventory status below means **found in** `parts-is-parts/inventory/immediate.csv` on 2026-10-19. “Not listed” means it was not found in that inventory file; it is not proof that the part is absent elsewhere.

## Confirmed immediate stock

| Qty | Part / stock number | Role | Status / design note |
| ---: | --- | --- | --- |
| 1 | `ATMEGA328P-PU` | Bare 5 V controller | **10 in stock.** Candidate only; it needs external persistent storage for useful raw captures. |
| 1 | `ATS16B-E`, 16 MHz crystal | Controller clock | **5 in stock.** Suitable nominal clock for the ATmega candidate. |
| 1 | 28-pin DIP socket | ATmega socket | **8 in stock** (`123-47-628-41-001000`); `D0828-42` wire-wrap sockets are also listed ×8. |
| 1 | `MCP3008` | 10-bit SPI ADC | **6 in stock.** Candidate capture ADC. Its sample-rate and reference suitability are AD2-test decisions. |
| 1 | 16-pin DIP socket | MCP3008 socket | **36 wire-wrap sockets in stock** (`D0816-42`). |
| 1 | `KA358` | Dual op-amp | **630 in stock.** Use for high-impedance capture buffering, bias, and active filtering; not assumed to be the final low-impedance playback driver. |
| 0–1 | `LM339AN` | Quad comparator | **25 in stock.** Optional input slicer/diagnostic threshold; raw ADC capture is the revision-one priority. |
| 1 | `LM336Z` | 2.49 V reference | **17 in stock.** Provisional reference for the MCP4921 playback DAC: with its gain set to 1×, it gives a nominal 0–2.49 V DAC range. Confirm accuracy/noise and final playback scaling during the AD2 gate. |
| 1 | `SN74LS14N` | Schmitt inverter | **23 in stock.** Optional reset/mute/status conditioning. |
| 1 | `SN74LS74AN` | Dual flip-flop | **10 in stock.** Optional deterministic mute/reset latch. |
| 0–1 | `SN74HCT125N` | Tri-state buffer | **10 in stock.** Optional 5 V logic isolation; do not treat it as a universal 3.3 V SD level shifter. |
| 2–4 | `2N2222A` | NPN transistor | **100 in stock.** Mute, indicator, or optional `REMOTE` open-collector experiments. |
| 0–2 | `2N7000` | N-channel MOSFET | **100 in stock.** Optional switching/mute/remote work. |
| 4–8 | `1N4148` | Small-signal diode | **100 in stock.** Input protection/clamping experiments. |
| 1–2 | `1N5819` | Schottky diode | **100 in stock.** Power reverse-polarity or clamp option. |
| 1 | `10TC405` / `PS1024ALBLK` | Power or user switch | **123 / 10 in stock.** Choose mechanical form during enclosure work. |
| 2 | LEDs + 330 Ω resistors | Power/activity indication | LEDs in stock; `RN60D3300FB14` ×10 is listed. |
| as needed | 4.7 kΩ / 10 kΩ resistors | Pull-ups, dividers, bias prototypes | **100 each in stock** (`MFR25SFRF52-4K7`, `MF1/4DCT52R1002F`). Final values are not yet fixed. |
| 1 | Vector `8801-6` board | Full revision-zero wire-wrap card | **4 in stock; selected as the main prototype carrier.** It fits the stripped HP 1611A assembly's 100-pin slot exactly. Use only the independently verified +5 V and ground contacts initially; the legacy S-100-style edge arrangement is not a project bus. Datasheet: `datasheets/vector-8801-6.pdf`. |
| 0–1 | Vector `8015-1` Vectorbord® Circbord™ | Isolated subcircuit experiment carrier | **User-selected when needed.** Verified as a 4.00 × 4.00 × 0.062 in FR-4 board with 0.1 in grid, 0.042 in non-plated holes, and isolated 0.080 in pads on one side. It accepts Vector R32 wire-wrap socket pins and is specified for up to 72 16-pin DIPs. Reserve it for filter, output-driver, or protection experiments rather than the full prototype. Datasheet: `datasheets/vector-8015-1.pdf`. |
| as needed | Wire-wrap pins, sockets, Kynar wire | Prototype construction | Stock is listed; verify quantities before allocation. |

## Required but not yet confirmed in immediate inventory

These are required for a complete standalone device. Do not buy final values before the AD2 baseline unless marked “mechanical”.

| Qty | Required item | Minimum requirement | Why it is needed | Selection gate |
| ---: | --- | --- | --- | --- |
| 1 | `MCP4921-I/P` DAC IC | 12-bit, single-channel SPI DAC, 8-pin PDIP, 2.7–5.5 V supply, external reference, selectable 1×/2× output gain | **Selected provisional DAC.** Directly interfaces to the ATmega328P and can use the stocked LM336Z reference. Its rail-to-rail output is still followed by reconstruction, AC-coupling, amplitude-setting, and ECS-protection circuitry. | Confirm replay bandwidth, level, and impedance with the AD2 baseline before purchasing/populating. Datasheet: `datasheets/22248a.pdf`. |
| 1 | Playback output driver | Cleanly supply at least 3 Vpp into 100 Ω with margin (design target: ≥20 mA peak), at the measured frequency range, from the selected supply | A DAC/KA358 alone may not drive the ECS requirement. | Measured ECS load and output-amplitude window. |
| 1 | microSD card socket | Bare PCB-mount socket, not a breakout board | Persistent raw-capture library. | Storage format and enclosure decision. |
| 1 | microSD card | Industrial or known-good card, modest capacity is sufficient | Holds raw captures and metadata. | Controller/file-system firmware plan. |
| 1 | 3.3 V regulator | Sized for SD-card write bursts; include input/output capacitors | SD cards require 3.3 V. | Storage circuit choice. |
| 1 | SD logic-level interface | Defined 5 V-controller to 3.3 V-SD translation, including safe card-to-controller return level | Protects the SD card and establishes reliable SPI signalling. | Exact controller and socket selected. |
| 1 | Protected 5 V card-power entry | Accept the verified +5 V/ground contacts from the HP 1611A/5036A platform; isolate it from ECS power and include adequate current margin | Powers the standalone device without using any ECS power pin. | Powered-backplane qualification; final fuse/protector and current budget. |
| 2 | 3.5 mm mono TS PCB/panel jack | One playback output, one capture input | `IN FROM TAPE` and `OUT TO TAPE` connections. | Mechanical layout; cable harness already defined. |
| 0–1 | 2.5 mm mono TS PCB/panel jack | `REMOTE` only | Optional revision; omit from initial board if no remote function. | Remote-line measurements. |
| 1 | ISP/programming connector | Bare 2×3 0.1-inch header or equivalent | Programs a socketed ATmega without a development board. | Controller finalisation. |
| 1 | Reset pushbutton | Momentary normally-open | Local controller recovery. | Board layout. |
| several | Decoupling capacitors | 100 nF per IC plus local bulk capacitance | Required for stable digital/analogue operation. | Final IC count and regulator datasheets. |
| several | Analogue capacitors/resistors/trimmer | Coupling, anti-alias/reconstruction filters, bias, gain, output attenuation | Establishes the measured ECS waveform and protection behaviour. | AD2 voltage, impedance, and spectrum results. |
| 1 | Enclosure, strain relief, labels | Physical protection and connector identification | Makes the device safe to use at the ECS. | Board and connector placement. |

The revision-zero card receives its +5 V through the stripped HP 1611A main-board assembly from its salvaged HP 5036A-derived supply, rather than through an ECS connector. The power path remains a qualification item: see the powered-backplane gate in [the workplan](WORKPLAN.md). A card-local fuse/protector, reverse-polarity protection where appropriate, connector-entry bulk capacitance, and per-IC decoupling are still required.

## Candidate parts that are not automatically approved

| In-stock part | Possible use | Why it remains conditional |
| --- | --- | --- |
| `LM380N` ×10 | Playback power/output amplifier | Must be tested for clean low-frequency waveform reproduction, gain, startup transient, supply requirements, and the measured ECS load. |
| `OPA551PA` ×10 | High-current analogue stage | Its supply and output requirements must be checked against the final power architecture; do not assume it is a 5 V drop-in driver. |
| `LM324AN` ×25 | Additional filtering/buffering | Useful for bench work, but has similar non-rail-to-rail limits to the KA358 family. |
| Existing EEPROM/SRAM/EPROM | Firmware or small buffer experiments | None replaces the required removable/persistent raw-capture storage by itself. |
| Resistor-ladder DAC made from stock resistors | Emergency playback experiment | The listed resistor values/precision do not yet establish a matched R-2R ladder; do not treat this as the final DAC. |

## Not in revision one

- `REMOTE` driver and 2.5 mm jack;
- decoded cassette-file format;
- display, keypad, or sophisticated user interface;
- hardware FSK decoder/slicer as the primary capture method;
- direct ECS power, any modified ECS track, or any third-party board.

## Build gate

The MCP4921-I/P is selected as the provisional DAC, but do not purchase/populate its surrounding output driver or final filter-value set until the `ECS-TEST-001` through `ECS-TEST-003` measurements exist. The first purchase/selection review should answer:

1. What voltage, offset, and source/load behaviour does `OUT TO TAPE` show?
2. What replay level and impedance does `IN FROM TAPE` accept?
3. What capture/playback sample rate and converter resolution preserve the waveform sufficiently for `CLOAD TEST` and `CVRF TEST`?
4. Is the ATmega328P data-rate/storage path sufficient, or does the bare-controller choice need to change?

Only then promote this revision-zero document to an exact, orderable BoM.
