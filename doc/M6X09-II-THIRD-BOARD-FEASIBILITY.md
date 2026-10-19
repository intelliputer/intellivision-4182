# M6x09-II interface-development board: parts feasibility

## Decision

One of the three blank M6x09-II PCBs is feasible as the **mailbox/interface-development SBC**. It is not a fixed-speed production computer and it is not, by itself, the ECS cartridge interface.

There are also two populated M6x09-II boards at the desk. Keep those intact as known reference machines while the selected blank PCB is assembled and tested. This document calls that selected blank PCB the *interface-development board*; it does not assign an irreversible number to a physical board.

Its purpose is to run known-good 6309 firmware while an external, removable 74LS TTL carrier is developed and instrumented. This isolates interface mistakes from both the retained recovery SBC and the eventual performance/video SBC. The recommended maximum-speed configuration uses a sourced 12 MHz crystal plus a small ACIA clock-divider modification, subject to acceptance testing.

```text
third M6x09-II (normal-speed, socketed, instrumented)
                 |
                 | measured H2 expansion-header signals only
                 v
          removable 74LS mailbox-test carrier
                 |
                 +-- local loopback/logic-analyser test first
                 +-- CP1610/ECS connection only after the carrier passes
```

## Why this is the best use of the third board

- It preserves a known working 7.3728 MHz M6x09-II as the recovery/reference system.
- It avoids tying early mailbox experiments to the faster-clock board, where timing, ACIA configuration, and video work are already variables.
- A 12 MHz crystal produces the HD63C09RP's maximum specified `E` clock of 3 MHz. A 74LS74 divides that `E` clock by two before it reaches the MC68B50's `RxC`/`TxC` inputs, keeping the ACIA at its 1.5 MHz rated maximum.
- The M6x09-II's 32-pin `H2` header exposes enough board activity for measured expansion work, but a separate carrier owns all new decoding, latching, buffering, and handshaking.
- The carrier can be replaced after an error without reworking the SBC PCB.

## Important limitation

The M6x09-II's onboard logic maps its own RAM, ROM, and ACIA. Populating a blank M6x09-II does **not** add the CP1610 mailbox registers, bus buffers, cross-clock synchronizers, or safe ECS cartridge-port decode. Those belong on a separate 74LS carrier/interposer.

Do not connect the third board's raw expansion bus to the ECS. The first carrier must instead prove its local-side decode and register behaviour with no CP1610/ECS connection present.

## Video relationship determines the clock choice

Changing the 6309 crystal does **not** make the Master Component's STIC draw faster. The CP1610/STIC continues to generate its own display; the 6309 only changes the bandwidth available for writes to the companion controller or for mailbox commands.

| Intended visible result | Clock consequence | Recommended CPU-board choice |
| --- | --- | --- |
| Separate companion image on the project's RCA output | The TTL controller refreshes independently. CPU speed affects companion VRAM upload rate, not the Master Component image. | Use the 12 MHz / 3 MHz-`E` configuration with the ACIA divider if maximum companion framebuffer bandwidth is useful. |
| Companion screen with cards/sprites/command FIFO | Most visual work is done in TTL hardware; CPU clock has a smaller effect after initial asset upload. | The 12 MHz configuration is still useful, but video hardware features matter more than the last CPU increment. |
| Future text/graphics overlay synchronized with the Master Component | The video controller must synchronize to measured Master Component `~MSYNC`/`CBLNK`/external-video behaviour. A free-running 6309 clock does not establish genlock. | Keep CPU and video clocks architecturally separate. Do not choose 14.31818 MHz as a CPU crystal; use it only as a controller timing source if measurements prove that source relationship appropriate. |
| CP1610-controlled STIC graphics only | The CP1610 updates STIC state; 6309 work arrives through the mailbox. | CPU clock changes mailbox/data-preparation latency only; it does not change STIC scan-out or make the 6309 a STIC bus master. |

Therefore, the 12 MHz CPU modification is justified for the independent companion-video path and for general 6309 throughput. It must not be represented as an acceleration of the Master Component display. Any Master-video mixing/overlay is a separate, later gate requiring timing measurements and explicit video ownership.

## SBC population: one interface-development unit

The published M6x09-II manufacturing BOM establishes PCB compatibility. The local `parts-is-parts` inventory establishes the quantities below as of this review; physical count and part-marking inspection remain the assembly gate.

| Function | Part / board location | Qty. | Availability status | Notes |
| --- | --- | ---: | --- | --- |
| CPU | HD63C09RP, 40-pin DIP, `U4` | 1 | **9 inventoried** | Use a stocked DIP-40 socket; inspect the exact speed-grade marking. This development board does not need a faster clock. |
| Serial interface | MC68B50P, `U1` | 1 | **10 inventoried** | Use a stocked DIP-24 socket; needed for the monitor workflow. |
| Firmware | M27C128-10F1 EPROM, `U2` | 1 | **40 inventoried** | Use a stocked DIP-28 socket. Program a complete 16 KiB ASSIST09 image, not its raw 2 KiB monitor section. |
| Main memory | AS6C62256-55PIN SRAM, `U3` | 1 | **10 inventoried** | Use a stocked DIP-28 socket. |
| System timing part | 12.000 MHz, fundamental-mode, parallel-resonant HC-49/US **through-hole** crystal, 18 pF load, `X1` | 1 | **Source required** | Recommended maximum-speed clock for stocked HD63C09RP: `E` = 3 MHz. Example: Citizen `HC-49/U-S12000000ABJB` (18 pF load, 80 Ω maximum ESR) or equivalent. |
| Crystal load capacitors | 18 pF C0G/NP0 ceramic, radial 0.1-inch, `C2`, `C3` | 2 | **Source required / inspect existing stock** | Retain the published board values. The original 7.3728 MHz crystal also specifies 18 pF load, so this keeps the established oscillator topology rather than assuming a different load network. |
| ACIA clock divider | 74LS74, small daughtercard or documented board modification | 1 | **10 inventoried** | Divide CPU `E` (3 MHz) by two and drive both MC68B50 clock inputs at 1.5 MHz. The original direct `E` path to those inputs must be isolated; do not tie the divider to an active direct drive. |
| Fallback only | 5 MHz, 20 pF HC-49/US crystal (`251-05000`) | 1 | **10 inventoried** | Not the preferred build: it requires a recalculated load network and a rebuilt serial configuration, while delivering only `E` = 1.25 MHz. |
| Glue logic | SN74LS00N, `U5` | 1 | **118 inventoried** | Socketed. |
| Expansion connector | 2 × 16, 2.54 mm header, `H2` | 1 | **Not listed** | Mandatory; procure or physically locate it before assembly. |
| Serial connector | 1 × 6 FTDI-style header, `P1` | 1 | **Not listed** | Needed for the monitor and loader unless a documented alternative connector is designed. |
| Diode and 330 Ω resistor | 1N914, `D1`; RN60D3300FB14, `R1` | 1 each | **20 / 10 inventoried** | Sufficient for at least one board. |
| Reset, LED, remaining passives | Reset switch, LED, 2.2 kΩ SIP, 10 µF × 1, 18 pF × 2, 100 nF × 5 | 1 set | **Partially unlisted** | LEDs are stocked, but the listed inventory does not establish the switch, SIP network, or required capacitors. Do not substitute values without schematic review. |

### Minimum procurement/stock gate

The active ICs are already available in sufficient quantity. Do not start assembly until these missing/unverified items are physically resolved: **the 12 MHz / 18 pF crystal, C2/C3 18 pF C0G/NP0 capacitors, H2 header, P1 header, reset/passive set, and decoupling capacitors**. Before fitting the 12 MHz crystal, confirm the physical fit and scope oscillator start-up/frequency. Build and test the ACIA divider before isolating the original direct `E` path; at 1.5 MHz ACIA clock, ÷16 mode is 93,750 baud. Rebuild the monitor for that configuration and prove the host link. Do **not** use the stocked 16 MHz crystal with HD63C09RP.

## First mailbox-test carrier: preliminary parts

This is a plug-in 74LS test carrier, not the final ECS stack board. It provides a local register bank and handshake signals so firmware, the expansion-header map, and carrier timing can be proven before any CP1610 wiring.

| Function | 74LS part | Initial qty. | Purpose |
| --- | --- | ---: | --- |
| Local address/register decode | 74LS138, 74LS139 | 1 each | Choose a non-conflicting local I/O window after measuring the M6x09-II bus. |
| Command/status registers | 74LS273 | 5 | **29 inventoried.** Use edge-triggered registers for `CMD`, `ARG0`, `ARG1`, `STATUS`, `REPLY`; revise the mailbox schematic from its original 74LS374 proposal because 74LS374 is not listed in stock. |
| Data-bus isolation | 74LS245 | 2 | **20 inventoried.** Separate the M6x09-II bus from carrier register paths; outputs disabled when unselected. |
| Request/ack synchronizers | 74LS74 | 2 | **10 inventoried.** Two flip-flop stages for each of the two initial handshake directions. |
| Handshake/control gates | 74LS00, 74LS02, 74LS04, 74LS08, 74LS32 | 1 each | **118 / 54 / 69 / 10 / 10 inventoried.** Reset, ready, request, acknowledge, and error control. |
| Pull-ups, LEDs, test points, decoupling | 10 kΩ resistors, LEDs, 100 nF capacitors | 1 set | **10 kΩ × 100 and LEDs are inventoried; 100 nF capacitors are not listed.** One 100 nF capacitor at every TTL IC; expose all enables and handshakes. |
| Carrier connector | Mating 2 × 16 connector plus mechanical support | 1 | **Not listed.** No loose jumper-wire bus for the sustained test. |

The inventoried 74LS stock makes this carrier direction feasible. Exact quantities remain provisional until its schematic and the M6x09-II `H2` continuity map exist. The register change to 74LS273 must be captured in that schematic; it is not a pin-for-pin 74LS374 substitution.

## Assembly and test order

1. Fit the 12 MHz / 18 pF crystal and C2/C3 18 pF C0G/NP0 capacitors; scope oscillator start-up and frequency before installing the CPU.
2. Build and test the 74LS74 `E ÷ 2` ACIA-clock daughtercard, then isolate the original direct `E` path to the ACIA and verify 1.5 MHz at both clock inputs.
3. Rebuild ASSIST09 for the 93,750-baud ACIA configuration, then program and verify the complete 16 KiB EPROM image; retain its checksum and complete programmer readback.
4. Pass standalone acceptance: 5 V rail, reset waveform, 3 MHz `E` clock, RAM test, measured serial monitor/load, and a simple expansion-header logic-analyser capture.
4. Continuity-map every `H2` contact and record signal direction, idle state, voltage, reset state, and timing. Do not rely on inferred pin functions.
5. Build the carrier with only its local address decoder, one 74LS374, 74LS245, pull-ups, LEDs, and test points. Demonstrate a local write/read loopback.
6. Add the full latch bank and request/acknowledge logic; stress it from 6309 firmware with a logic analyser.
7. Add the CP1610-facing side only after the local carrier has a written register map, reset behaviour, and passing contention tests.

## Acceptance criteria

The third board is ready to meet the ECS carrier only when it:

- boots ASSIST09 and performs repeatable RAM and serial-load tests;
- exposes a documented, measured `H2` map;
- keeps all carrier outputs inactive during SBC reset and when unselected;
- passes at least one million local mailbox write/read/handshake iterations without an error, duplicate acknowledgement, or uncommanded output-enable;
- retains a removable recovery path: the carrier may be unplugged and the SBC remains independently usable.

## Board allocation

| Hardware | Proposed role | Clock configuration |
| --- | --- | --- |
| Populated desk board A | Recovery/reference SBC | Leave at its proven configuration; do not alter for interface experiments. |
| Populated desk board B | Known-good comparison SBC | Leave intact until a blank-board replacement passes acceptance. |
| Blank PCB A | Performance/video candidate | Source a 12 MHz crystal; `E` = 3 MHz is the stocked HD63C09RP limit. The video generator keeps its separate 14.31818 MHz master clock. |
| Blank PCB B | Mailbox/interface-development SBC | Maximum-speed stock-CPU configuration: 12 MHz / 18 pF crystal, `E` = 3 MHz, plus 74LS74 ACIA `E ÷ 2` daughtercard, rebuilt monitor, and removable TTL carrier. |
| Blank PCB C | Reserve/spare | Remains uncommitted until the two experimental roles have bench results. |

## Related documents

- [Project roadmap](PROJECT-ROADMAP.md)
- [CP1610–6309 mailbox interface](CP1610-6309-MAILBOX.md)
- [Risk register](RISK-REGISTER.md)
- [Video performance analysis](VIDEO-PERFORMANCE-ANALYSIS.md)
