# Third alternative: a MAGIC-inspired companion computer

## Finding

`CCF10272011_00005.pdf` is a 79-page draft, marked confidential, for **MAGIC**: the *Mattel Advanced Graphics Interface Circuit*. It describes a proposed second-generation TV-game architecture, not the released ECS #4182.

The draft assumes a Motorola 68000 as its preferred CPU; specifies a 16-bit CPU interface, dynamically refreshed shared RAM, graphics and alpha generators, raster interrupts, composite-video timing, and external-video synchronization/mixing. In other words, it is a design direction that sits between the limited ECS and the unrealized Keyboard Component—not an ECS peripheral that can be reproduced by adding a single IC.

## Proposal

Build a **MAGIC-inspired companion computer**: a 6309-based computer with a discrete 5 V TTL video-and-memory controller. It may communicate with an Intellivision/ECS through a narrow, safe mailbox interface, but it must also be viable as a standalone system with its own video output.

FPGAs, CPLDs, and other programmable-logic devices are explicitly out of scope. The controller is to be built from SSI/MSI TTL parts—logic gates, counters, decoders, multiplexers, registers, shift registers, bus transceivers, and RAM/ROM—with analogue video circuitry where required.

```text
                         +-- optional, decoded mailbox -- Intellivision / ECS
                         |
6309 CPU <------8-bit command port------> TTL MAGIC-style controller <--> paired video RAM banks
   |                                              |
ROM / RAM / serial                                 +-- independent video output
                                                   +-- optional external-video sync / overlay experiment
```

This is intentionally different from the 6309 daughterboard:

| Route | Main purpose | CPU | Video owner | Relationship to ECS |
| --- | --- | --- | --- | --- |
| Digital cassette source | Preserve ECS tape workflows | none required | ECS / Master Component | Directly compatible accessory |
| 6309 daughterboard | Add a compact independent coprocessor | 6309 | Master Component remains owner initially | Mailbox-connected extension |
| MAGIC-inspired companion | Recreate the more ambitious Mattel design direction | 6309 | New TTL controller with 16-bit video datapath | Optional, constrained bridge |

## What the MAGIC draft contributes

- A preferred Motorola 68000 CPU and a 16-bit CPU data path in the original proposal. This project adapts the graphics architecture to the available 8-bit-bus 6309 instead of attempting to copy that CPU interface.
- Direct, arbitrated access to at least 16K 16-bit words of dynamic RAM, shared with the video system.
- A CPU clock derived from color subcarrier timing: approximately 7.16 MHz NTSC or 8.86 MHz PAL/SECAM in the proposal.
- A periodic timing interrupt near 60 Hz plus a programmable raster-line interrupt.
- 40 × 24 card-oriented display organization, 16 moving objects, alpha and background generators, scrolling/split-screen concepts, and high-resolution-card ideas.
- An explicit goal of external-video synchronization/mixing and provision for two video outputs.

These are architectural clues, not a complete implementation package. The document is a draft and its scanned diagrams/OCR are imperfect. It does not, by itself, establish a complete pinout, final register map, verified TTL schematic, or a production-compatible video implementation.

## Adapting MAGIC to a 6309

The 6309 is an 8-bit external-bus processor. It has useful 16-bit registers and a 16-bit address space, but it cannot perform a single 16-bit external memory transfer like the 68000 assumed by the MAGIC draft. The design must acknowledge that boundary rather than disguising it.

The solution is an **8-bit computer with a 16-bit video datapath**:

- The 6309 owns ordinary 8-bit program/data RAM, ROM, serial I/O, and the command interface.
- The controller owns video timing, display fetch, palette/state registers, and a 16-bit-wide video memory built from two matched 8-bit SRAM banks: low byte and high byte.
- The 6309 accesses controller state through 8-bit registers. It supplies a video address, then low byte, high byte, and a write/read command. TTL latches combine those byte operations into one 16-bit video-memory word operation.
- The controller arbitrates video-memory access. Display fetch has priority; CPU-originated commands are served during predetermined free slots or horizontal/vertical blanking. The first revision should use `VSTATUS` polling; a later revision may use the 6309's `HALT` input only after its bus-cycle behaviour is verified on hardware.
- For multi-byte updates, a FIFO made from TTL registers or small SRAM lets the 6309 queue writes. A latch-and-commit rule prevents the display engine from seeing a half-written word.

This is historically plausible and technically honest: many 8-bit computers used custom video hardware with a wider internal memory/datapath. It will not make the 6309 into a 68000, but it can deliver the MAGIC-style graphics features that benefit from 16-bit words.

### Suggested 6309-visible controller registers

| Register | Function |
| --- | --- |
| `VADDR_LO`, `VADDR_HI` | Video-memory word address latch. |
| `VDATA_LO`, `VDATA_HI` | Byte latches for one 16-bit video word. |
| `VCMD` | Read, write, increment-address, palette write, sprite/alpha-table selection, and display-control commands. |
| `VSTATUS` | Command-ready, blanking state, raster interrupt pending, and queue-full/empty flags. |
| `VIRQ_ACK` | Acknowledge frame/raster interrupts. |

The exact map remains a design task. Keep the first version small: word read/write, auto-increment, frame interrupt, raster interrupt, and display enable. Do not begin with the entire MAGIC register set.

## Recommended implementation boundary

Do not attempt to place a TTL controller as a second master on the original Intellivision cartridge bus in the first revision. The first version should use local RAM and own its own video path. Any ECS/Master Component connection should be electrically buffered and reduced to a mailbox/register protocol, exactly as for the 6309 concept.

That division prevents bus contention, avoids dependence on undocumented STIC timing, and makes the MAGIC-inspired machine useful even before original-hardware integration is solved.

## Preferred physical integration: above the ECS

The preferred mechanical form is a **cartridge-port daughterboard plugged into the ECS cartridge socket**. It is “on top of the ECS” as a stack, but it is not a piggyback replacement for the ECS SDCC, ROM, RAM, or sound chips.

```text
Intellivision Master Component
             |
            ECS
             |
 ECS cartridge socket ── 6309 + TTL MAGIC daughterboard ── RCA composite / 3.5 mm audio
                              |
                         local RAM / ROM / video RAM
```

The board's CP1610-facing circuitry is a passive client except when selected by a proven address decode. It may return/read mailbox registers and raise a deliberately buffered interrupt/handshake signal, but it must never drive address, data, or control lines outside a valid CP1610 bus cycle. The 6309 and TTL controller never become cartridge-bus masters.

### What the ECS contributes

- The physical host, Master Component CP1610 execution environment, and existing ECS software/keyboard ecosystem.
- A natural place to expose a CP1610-to-6309 mailbox/register window.
- Original ECS audio, if a separately measured analogue tap is routed to the audio-output section.

### What remains local to the daughterboard

- 6309, firmware ROM, CPU RAM, paired 16-bit video RAM, TTL video logic, and the composite encoder.
- RCA composite output through the video-composites board.
- 3.5 mm TRS audio output and any required ECS-audio buffer or additional-PSG re-creation.
- Its own reset supervision and power conditioning.

### Non-negotiable checks before connection

1. Measure and document every cartridge connector pin: voltage, direction, idle state, timing, and whether the ECS changes the signal from the Master Component.
2. Reserve and verify a non-conflicting address/register window. Do not guess from an emulator memory map alone.
3. Use 5 V TTL transceivers with explicit output-enable control so the daughterboard is electrically invisible when not selected.
4. Provide a cartridge pass-through strategy only after its address-selection and mechanical implications are designed; do not simply parallel two cartridge loads.
5. Do not rely on the ECS supply to power a 50-plus-package TTL system until its available regulated current, thermal margin, and back-feed behaviour have been measured. A separate regulated supply with a deliberate common ground may be required.
6. Route ECS analogue audio with a dedicated measured tap/wire; it should not be assumed to exist on the cartridge connector.

## Staged proof of concept

1. **Document extraction.** Manually transcribe the readable register diagrams, address map, timing requirements, and graphics modes into reviewable Markdown tables. Mark every inferred field as such.
2. **TTL timing board.** Build a small, testable 5 V TTL board from counters, decoders, registers, multiplexers, and a resistor DAC. First generate stable RGB/VGA-style debug timing, a frame interrupt, and a programmable raster interrupt; do not begin with composite-video compatibility.
3. **TTL graphics datapath.** Add static RAM, shift registers, latches, and bus transceivers for one simple card/framebuffer mode. Keep the first design in static RAM; add DRAM arbitration/refresh only after the timing and datapath are proven.
4. **Standalone computer.** Attach local RAM, ROM/flash, and a discrete 6309. Bring up a monitor and a graphics test suite through the 8-bit command port.
5. **MAGIC feature selection.** Add only demonstrably valuable features—sprites, alpha plane, scrolling, split screen, or external-video sync—one at a time, with test captures.
6. **Intellivision bridge.** Add a level-safe CP1610-facing mailbox only after the standalone system is reliable. Start with command/data exchange, not video-bus takeover.
7. **Optional ECS services.** Use the digital cassette project as a storage interchange and the ECS keyboard/controller ports as input sources through explicit adapters or protocol translation.

## Decision criteria

Choose this route when the goal is a new, historically informed 1980s-style computer/graphics system. Choose the 6309 route when the goal is a smaller, easier-to-bring-up coprocessor that leverages the existing Master Component display. Neither route should delay the cassette-data project, which remains the clearest compatibility milestone.

## Primary reference

[MAGIC working specification](CCF10272011_00005.pdf), especially its preview, RAM interface, CPU interface, TV timing/control, graphics-register, and die-estimate sections.

The concrete first-controller proposal and draft bill of materials are in [TTL 16-bit video controller](TTL-16BIT-VIDEO-CONTROLLER.md).
