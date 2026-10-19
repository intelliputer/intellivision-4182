# Keyboard Component capability recovery

## Project intent

The ECS-mounted 6309/TTL companion system is intended to recover useful **Keyboard Component-class capabilities** that the released ECS did not fully provide. It is not a claim that the board is an original Keyboard Component, an instruction-compatible 6502 system, or a reproduction of its unreleased hardware.

The practical target is feature recovery: a programmable computer environment with substantial local capability, persistent storage, keyboard/input access, printer/output support, stronger graphics, and audio integration—while preserving the Master Component and ECS as usable host hardware.

## Historical distinction

The released ECS is a CP1610-hosted expansion with ROM, 2 KiB of additional RAM, an extra PSG, cassette/printer interface, and additional input ports. The unreleased Keyboard Component was a different, more ambitious design with a separate Computer II board, display-oriented hardware, an internal cassette mechanism, and its own power system. The two should not be conflated.

The Keyboard Component service manual's self-test names `CPU1-UP` and `CPU2-UP`, which is good direct evidence for a dual-processor design. It does **not** identify those processors, their clock rates, or the capacity/type of its graphics RAM. A 6502-based implementation may be reported elsewhere, but it is not an assumption on which this project should depend until it is tied to board evidence or a suitable primary source.

The 6309 is neither a CP1610 nor a 6502. Compatibility must be provided at the interface/protocol level where it is valuable, not assumed at the binary level.

## Capability matrix

| Capability to recover | ECS baseline | Companion-system approach | Initial proof | Compatibility boundary |
| --- | --- | --- | --- | --- |
| Independent programmable compute | No separate CPU | One 6309 with local ROM and RAM, isolated from the CP1610 bus | 6309 monitor, serial console, RAM test | New 6309 software; no historical CPU or binary compatibility implied |
| Larger usable memory | 2 KiB ECS RAM, heavily shared with BASIC/EXEC | Local 6309 RAM plus dedicated 16-bit video RAM | Memory test and stable video fetch | Local memory is not Master Component RAM |
| Keyboard/input use | ECS keyboard/controller ports via ECS sound-chip I/O | CP1610 mailbox input service first; later explicit adapters/protocol translation | Read one key/controller state through mailbox | Do not electrically take over ECS input ports without measurement |
| Program/data persistence | Cassette recorder/printer interface | Solid-state cassette-compatible digital source, archival WAV/decoded files, optional 6309-local storage | `CSAVE`/`CLOAD`/`CVRF` with known-good recordings | ECS tape protocol must be measured, not invented |
| Printer/output support | ECS UART/printer path | CP1610 mailbox service for original printer path; optional 6309-native output later | Print a known test line through original-compatible path | Printer electrical/protocol details require verification |
| Text/graphics workstation display | Master Component STIC display; ECS adds no separate display controller | TTL 16-bit video controller with cards/tiles, moving objects, palette/raster features, and composite output | Fixed pattern, framebuffer, then card/sprite demos | New composite output is separate from STIC/RF video; a later genlocked-overlay experiment is a separate feature |
| Audio and music | Extra ECS AY-3-8917 mixed with Master Component audio | Use original ECS PSG through mailbox; integrate audio output; optional additional-PSG re-creation | CP1610 mailbox `PSG_WRITE` drives real ECS audio | The re-creation adds only the ECS PSG path, not the complete original mix |
| Host/co-processor cooperation | No general second-CPU service | CP1610–6309 TTL mailbox and optional command FIFO | `SERVICE_PING`, then safe PSG write | First revision: 6309 is not a cartridge-bus master |
| Keyboard Component-class host interface | ECS cartridge pass-through exposes the Master Component port | Preserve the physical pass-through; study a measured 44-contact Keyboard-style interface only after the mailbox design works | Continuity map and passive pass-through test | Do not assume an ordinary one-to-one ribbon, bus timing, or termination |
| Development environment | ECS BASIC and EXEC routines | 6309 monitor, storage tools, diagnostics, graphics/audio command libraries | Reproducible ROM build and diagnostics | New software ecosystem, not a promise of historical Keyboard Component software support |

## Implementation principles

1. **Recover capabilities in layers.** Bring up a functional 6309 computer before attempting high-performance graphics, audio replication, or direct ECS modifications.
2. **Use the original hardware where it is already strong.** The CP1610/ECS owns its original bus, PSG, input, cassette, and printer interfaces until measurements support a different arrangement.
3. **Add new resources locally.** The 6309, RAM, video RAM, TTL graphics logic, composite encoder, and local firmware belong on the daughterboard.
4. **Prefer protocol compatibility to electrical takeover.** A mailbox command to the CP1610 is usually safer than a track cut or shared peripheral bus.
5. **Keep experimental modifications separable.** Any ECS track cut, drilled splice, or direct-PSG access must be documented and reversible where practical.
6. **Prove one capability at a time.** Each capability needs a test program, a measured interface contract, and a documented pass condition.

## What the Keyboard Component references establish

The sibling reference repository, [`cartheur/intellivision-2609`](../../../cartheur/intellivision-2609/keyboard/README.md), provides an evidence-led starting point rather than a speculative feature list:

- The Computer II assembly, keyboard, tape-control assembly/cassette deck, and dedicated power supply are documented subassemblies.
- The service self-test reports `CPU1-UP` and `CPU2-UP`, and separately tests high-resolution graphics RAM and a keyboard I/O port. This establishes the capability categories, not component identities or RAM capacity.
- The Master Component remains present. The Keyboard Component connects through its cartridge port, provides a separate power lead, and retains a pass-through cartridge receptacle.
- The connector analysis records a 44-contact, 0.1-inch cartridge interface carrying `DB0`–`DB15`, control, timing, video, and power signals. Its documented use of `CBLNK` and the external-video path for genlocked text is particularly relevant to any future overlay work.

The source material does **not** publish a complete harness conductor map, board schematic, CPU type, CPU clock, graphics-RAM specification, cassette encoding, or video timing. These remain investigation items, not design facts.

### Consequence for this project

The mailbox architecture remains the low-risk first integration: it gives the 6309 access to services controlled by the CP1610 without contending for the Master Component bus. Once it is proven, a separate advanced experiment may investigate a Keyboard Component-style 44-contact interface and genlocked overlay. That experiment needs an intact-unit continuity map, measured bus timing, defined tri-state ownership, and an independently tested pass-through path before it is allowed near a working console.

## Priority sequence

1. Digital cassette-data source and preservation captures.
2. 6309 local monitor, ROM/RAM, reset, and serial diagnostics.
3. CP1610–6309 mailbox with `SERVICE_PING`.
4. Mailbox-controlled use of the original ECS AY-3-8917 and audio output buffer.
5. TTL video timing, composite encoder, and framebuffer proof.
6. Card/tile and moving-object hardware to achieve STIC-like perceived animation efficiency.
7. Keyboard/input and printer services through verified CP1610/ECS interfaces.
8. Optional direct-ECS modifications and additional-PSG re-creation only after the non-invasive paths are stable.
9. Optional Keyboard Component-style bus/video-overlay research, beginning with a continuity and timing study—not a live second bus master.

## Evidence to develop

- The [Keyboard Component service-manual findings](../../../cartheur/intellivision-2609/keyboard/README.md) and [connector analysis](../../../cartheur/intellivision-2609/keyboard/CONNECTOR-ANALYSIS.md), with the scanned manual image treated as authoritative where OCR differs.
- Original Keyboard Component board photographs, ROMs, and schematics where legally and practically available.
- A verified feature list separated into: released hardware, prototype/unreleased hardware, and project-defined additions.
- CP1610/ECS cartridge-bus, audio, tape, printer, and input-port measurements from the actual target revision.
- Bench results for every claimed recovered capability.

## Related documents

- [Project roadmap](PROJECT-ROADMAP.md)
- [CP1610–6309 mailbox interface](CP1610-6309-MAILBOX.md)
- [16-bit TTL video controller proposal](TTL-16BIT-VIDEO-CONTROLLER.md)
- [Video performance analysis](VIDEO-PERFORMANCE-ANALYSIS.md)
- [Risk register](RISK-REGISTER.md)
