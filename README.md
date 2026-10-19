# Intellivision Computer Adaptor 4182 (ECS)

The Computer Adaptor 4182 is the core module of Mattel's Entertainment Computer System (ECS). It connects to an Intellivision Master Component; it is an expansion, not a standalone computer with its own processor.

## Project goals

1. Build a solid-state, cassette-compatible digital data source for the ECS's `IN FROM TAPE`, `OUT TO TAPE`, and `REMOTE` connections.
2. Develop a 6309-based daughterboard as a new, isolated coprocessor system that communicates safely with the Master Component/ECS. This is an extension project, not a claim about original ECS hardware.

The requirements, safety boundaries, and staged deliverables are in [the project roadmap](doc/PROJECT-ROADMAP.md).

## Specifications

| Component | Specification |
| --- | --- |
| Processor | No separate CPU. ECS software runs on the Master Component's General Instrument CP1610. The adaptor's SDCC (Super-Duper Custom Chip) performs integration and bus-control logic. A photographed production board carries the custom marking `4182-0128 BG` (an ECS/Mattel marking, not a published orderable part number). The 6502 sometimes associated with Intellivision computing belonged to the separate, unreleased Keyboard Component—not the released ECS. |
| RAM | 2 KiB of additional 8-bit system RAM. Availability ranges from 1,984 bytes with no ECS EXEC/BASIC features in use to 2 bytes when all features, including BASIC, are used. |
| ROM | 12 KiB ECS EXEC/BASIC ROM, containing additional EXEC routines, ECS BASIC, and music-note graphics. |
| Sound | One GI AY-3-8917 sound chip: three tone channels with noise generation and two bidirectional 8-bit I/O ports. Mixed with the Master Component's AY-3-8914, this provides up to six simultaneous notes. The preliminary ECS guide reproduced by Blue Sky Rangers calls the added chip an AY-3-8914; later hardware-focused documentation and emulator modeling identify it as an AY-3-8917. |
| Cassette / printer | UART-based serial interface for program-controlled data transfer to or from a cassette recorder, or for printing. |
| Input ports | Two additional input ports, used by either the Computer Keyboard, two hand controllers (for four-player games), or the Music Synthesizer. |
| Power | Requires its own power supply to supplement the Master Component. |

## Notes on connections

The ECS reference identifies a cassette-recorder/printer interface and two extra input ports. The exact physical connector labels, pinouts, electrical levels, and the cassette data encoding are not specified by that source; they should be verified from an ECS service manual or by measurement before designing a compatible recorder/emulator.

## Source

Technical details are based on the scanned [Computer Module Guide](doc/Computer_Module_Guide.pdf), Blue Sky Rangers' [ECS technical overview](https://history.blueskyrangers.com/hardware/ecs.html), which cites a preliminary ECS User's Guide dated March 25, 1983; the [Intellivision Wiki ECS hardware notes](https://wiki.intellivision.us/index.php/Entertainment_Computer_System); and [MAME's Intellivision hardware model](https://github.com/mamedev/mame/blob/master/src/mame/mattel/intv.cpp). The latter models the ECS as an expansion of the CP1610-based Intellivision, while modeling a 6502 separately for the unreleased Keyboard Component.
