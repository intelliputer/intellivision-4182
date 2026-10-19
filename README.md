# Intellivision Computer Adaptor 4182 (ECS)

The Computer Adaptor 4182 is the core module of Mattel's Entertainment Computer System (ECS). It connects to an Intellivision Master Component; it is an expansion, not a standalone computer with its own processor.

## Development hierarchy

This is a capability-recovery project, not a reproduction of an unreleased Keyboard Component. It proceeds through evidence-backed gates so that the released ECS remains useful at every stage and no experimental board is permitted to contend for its bus or peripherals.

| Level | Deliverable | Boundary / exit condition |
| --- | --- | --- |
| 0 — Reference baseline | Preserve an unmodified ECS/Master Component; document target-board revision, power, connectors, audio, cassette, and bus observations. | Measurements and test artefacts exist before irreversible work. |
| 1 — Digital cassette source | Solid-state recorder substitute for `IN FROM TAPE`, `OUT TO TAPE`, and `REMOTE`; retain lossless recordings and decoded data when understood. | Repeated `CSAVE` / `CVRF` / `CLOAD` tests on original hardware. |
| 2 — Local 6309 computer | Use the working M6x09-II SBC, fitted with its HD6309, as the local computer core; an ECS carrier/interface board physically above the ECS supplies the mailbox and later peripherals. | Its existing RAM, ROM monitor, reset, clock, and console pass standalone acceptance tests without driving the host bus. |
| 3 — Safe host cooperation | Discrete 5 V 74LS TTL CP1610–6309 mailbox, with a CP1610 service program and explicit ownership/handshake rules. | `SERVICE_PING` and bounded data exchange are reliable; the 6309 remains a bus client, not a second bus master. |
| 4 — Reuse ECS services | Mailbox-controlled keyboard/input, printer, cassette, and the real ECS AY-3-8917; buffer the desired audio to the companion's 3.5 mm TRS output. | One service at a time passes measured, repeatable tests. |
| 5 — Independent workstation video | MAGIC-inspired, 6309-led discrete-TTL video controller, composite encoder, sibling composite-output board, and RCA output. | Stable timing and test patterns first; then efficient cards/tiles and moving objects rather than framebuffer-only animation. |
| 6 — Optional expansion research | Direct ECS PSG access, an additional real-AY-3-8917 board, or a Keyboard Component-style 44-contact/video-overlay investigation. | Only after a reversible modification plan, continuity map, timing measurements, tri-state ownership analysis, and a verified pass-through path. |

No FPGA, CPLD, GAL, or other programmable-logic device belongs in this project. The implementation palette is 5 V 74LS TTL logic, conventional CPU/memory/ROM parts, oscillators, and necessary analogue/video circuitry.

The detailed deliverables and safety rules are in [the project roadmap](doc/PROJECT-ROADMAP.md). The capability rationale and Keyboard Component evidence are in [Keyboard Component capability recovery](doc/KEYBOARD-COMPONENT-CAPABILITY-RECOVERY.md).

Practical, module-by-module bench sequences are collected in the [module workplans](modules/README.md). The [external-data workplan](modules/external-data/WORKPLAN.md) is the first active implementation task; its cassette compatibility gate precedes the new-computer modules.

## Architecture at a glance

```text
Master Component (CP1610) <--> ECS #4182 <--> decoded TTL mailbox <--> M6x09-II (HD6309)
          |                         |                                      |
          |                         +-- original ECS peripherals             +-- local ROM/RAM, ASSIST09, serial console
          |                              remain CP1610-owned                 +-- later: 16-bit TTL video path -> RCA
          +-- original system video/audio                                    +-- mailbox-requested ECS audio -> 3.5 mm TRS
```

The mailbox is the default integration point. It lets the 6309 request CP1610-controlled services without electrically taking over the cartridge bus, ECS PSG, input ports, cassette UART, or STIC.

## Local-computer baseline

The [M6x09-II SBC](../../cartheur/M6x09-II-SBC/README.md) is the preferred Level-2 core: it has an HD6309-compatible CPU socket, 32 KiB SRAM, a 16 KiB `27C128` ROM map, a 68B50-compatible serial interface, ASSIST09 monitor, 74LS00 glue logic, and a 32-pin expansion header. Retain one board at its proven 7.3728 MHz crystal (`E` ≈ 1.8432 MHz) as a recovery/reference system. With the stocked HD63C09RP, a separate blank performance board may use a 12 MHz crystal (`E` = 3 MHz), but it must divide `E` by two before the MC68B50's `RxC`/`TxC` inputs; the ACIA is rated to 1.5 MHz. Without that change, use a 6 MHz CPU crystal (`E` = 1.5 MHz). The 14.31818 MHz source remains for video timing only, not the stock CPU. The carrier must connect only to a measured, documented subset of the expansion header and must never back-power or drive the SBC bus during reset.

The [M6x09-I SBC](../../cartheur/M6x09-I-SBC/README.md) remains useful for firmware and peripheral experiments, but is not the target daughterboard baseline: its core address decode uses a GAL16V8 and its tick source is an 89C2051, both outside this project's discrete-logic rule.

## Design references

- [CP1610–6309 mailbox interface](doc/CP1610-6309-MAILBOX.md)
- [Third M6x09-II board feasibility and parts gate](doc/M6X09-II-THIRD-BOARD-FEASIBILITY.md)
- [Graphics integration paths](doc/GRAPHICS-INTEGRATION-PATHS.md)
- [Dual-M6x09 machine-intelligence proposal](doc/DUAL-M6X09-MACHINE-INTELLIGENCE-PROPOSAL.md)
- [TTL 16-bit video controller proposal](doc/TTL-16BIT-VIDEO-CONTROLLER.md)
- [Video performance analysis](doc/VIDEO-PERFORMANCE-ANALYSIS.md)
- [MAGIC alternative](doc/MAGIC-ALTERNATIVE.md)
- [Risk register](doc/RISK-REGISTER.md)

## Specifications

| Component | Specification |
| --- | --- |
| Processor | No separate CPU. ECS software runs on the Master Component's General Instrument CP1610. The adaptor's SDCC (Super-Duper Custom Chip) performs integration and bus-control logic. A photographed production board carries the custom marking `4182-0128 BG` (an ECS/Mattel marking, not a published orderable part number). The 6502 attribution sometimes associated with Intellivision computing concerns the separate, unreleased Keyboard Component—not the released ECS; the Keyboard Component service manual itself establishes two CPUs but does not identify them. |
| RAM | 2 KiB of additional 8-bit system RAM. Availability ranges from 1,984 bytes with no ECS EXEC/BASIC features in use to 2 bytes when all features, including BASIC, are used. |
| ROM | 12 KiB ECS EXEC/BASIC ROM, containing additional EXEC routines, ECS BASIC, and music-note graphics. |
| Sound | One GI AY-3-8917 sound chip: three tone channels with noise generation and two bidirectional 8-bit I/O ports. Mixed with the Master Component's AY-3-8914, this provides up to six simultaneous notes. The preliminary ECS guide reproduced by Blue Sky Rangers calls the added chip an AY-3-8914; later hardware-focused documentation and emulator modeling identify it as an AY-3-8917. |
| Cassette / printer | UART-based serial interface for program-controlled data transfer to or from a cassette recorder, or for printing. |
| Input ports | Two additional input ports, used by either the Computer Keyboard, two hand controllers (for four-player games), or the Music Synthesizer. |
| Power | Requires its own power supply to supplement the Master Component. |

## Notes on connections

The ECS reference identifies a cassette-recorder/printer interface and two extra input ports. The exact physical connector labels, pinouts, electrical levels, and the cassette data encoding are not specified by that source; they should be verified from an ECS service manual or by measurement before designing a compatible recorder/emulator.

## Source

Technical details are based on the scanned [Computer Module Guide](doc/Computer_Module_Guide.pdf), Blue Sky Rangers' [ECS technical overview](https://history.blueskyrangers.com/hardware/ecs.html), which cites a preliminary ECS User's Guide dated March 25, 1983; the [Intellivision Wiki ECS hardware notes](https://wiki.intellivision.us/index.php/Entertainment_Computer_System); and [MAME's Intellivision hardware model](https://github.com/mamedev/mame/blob/master/src/mame/mattel/intv.cpp). MAME models the ECS as an expansion of the CP1610-based Intellivision and models a 6502 separately for the unreleased Keyboard Component. The surviving service-manual evidence is more conservative: it reports two CPUs but does not identify them. See the [Keyboard Component capability-recovery evidence](doc/KEYBOARD-COMPONENT-CAPABILITY-RECOVERY.md).
