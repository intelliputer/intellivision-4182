# Project roadmap

This repository has two related, but independently deliverable, hardware projects. The first preserves compatibility with the released ECS #4182. The second is a new computer subsystem; it must not be represented as original ECS hardware.

**Implementation priority:** complete the digital cassette-data source through repeated save, verify, and load tests on unmodified original hardware before building a 6309 carrier, video controller, or machine-intelligence node. It is the first compatibility product and the project’s reference for removable data.

## 1. Digital cassette-data source

Build a modern, solid-state substitute for the external cassette recorder supported by the ECS. It must behave like a compatible recorder at the ECS connectors, while storing recordings as digital files.

### Compatibility boundary

| ECS connector | Recorder connector | Direction | Device responsibility |
| --- | --- | --- | --- |
| `IN FROM TAPE` | `EAR` | recorder to ECS | Play a correctly levelled and timed data waveform. |
| `OUT TO TAPE` | `MIC` | ECS to recorder | Capture the ECS waveform without clipping or altering its timing. |
| `REMOTE` | `REM` | control | Detect/obey remote motor control, if implemented. |

The Computer Module guide specifies a recorder frequency response of 500–5,000 Hz (±3 dB), at least 3 V peak-to-peak output at 2,400 Hz and 4,800 Hz into a 100 Ω load, and a tape speed of 1⅞ ips (±3%). These are interface requirements, not yet a complete protocol specification.

### Deliverables

1. Capture known-good `CSAVE` recordings as lossless WAV files, retaining the original sample rate and channels.
2. Characterize the physical interface: plug dimensions and pinouts, DC bias, impedance, signal level, remote-control polarity, and the effect of PAL versus NTSC timing.
3. Reverse-engineer the on-tape framing, modulation, bit order, checksums, program-name encoding, and `CSAVE`/`CLOAD`/`CVRF` behavior.
4. Define an archival file format: raw WAV as the preservation master, plus a documented decoded representation when the format is understood.
5. Build a cassette emulator with nonvolatile storage, playback gain control, protected recording input, and a hardware or software remote-control option.
6. Validate it against original ECS hardware: save, verify, and reload several programs repeatedly, including a recording made on a real compatible recorder.

## 2. 6309 computer daughterboard

Create an ECS carrier/interface board, physically stacked above the ECS, which hosts or mates to the existing M6x09-II single-board computer fitted with a Hitachi 6309-compatible CPU. Its purpose is to add a capable, independently programmable computer to the Intellivision/ECS environment, borrowing the Keyboard Component's ambition without requiring that unreleased system's hardware.

The 6309 is a different 8-bit CPU family from the Master Component's CP1610 and cannot execute CP1610 binaries directly. MAME models the separate Keyboard Component with a 6502, but its surviving service manual only establishes a two-CPU arrangement, not the CPU identities; no historical binary compatibility is being claimed either way.

### First architecture: M6x09-II local computer with isolated mailbox interface

The M6x09-II provides the initial 6309's ROM, RAM, clock, reset, serial console, and local address/data bus. Its verified base configuration is 32 KiB SRAM at `$0000`–`$7FFF`, 16 KiB ROM at `$C000`–`$FFFF`, a 68B50-compatible serial interface, ASSIST09 monitor, and a 32-pin expansion header. Keep one board at its proven 7.3728 MHz crystal (`E` ≈ 1.8432 MHz) as the recovery system. The stocked HD63C09RP permits at most a 12 MHz crystal input, producing `E` = 3 MHz. A 12 MHz performance board must isolate the original direct `E` connection to the MC68B50 and feed its `RxC`/`TxC` from a 74LS74 `E ÷ 2` divider (1.5 MHz); otherwise use a 6 MHz CPU crystal (`E` = 1.5 MHz). Do not fit the stocked 16 MHz crystal or use the 14.31818 MHz video source as this CPU's crystal: both exceed the documented CPU limit. The ECS carrier adds only the deliberately narrow mailbox/register interface to the CP1610 side: discrete 5 V 74LS bus transceivers, address decoders, registers/latches, gates, and interrupt/handshake logic.

```text
Master Component CP1610 <--safe, decoded mailbox--> ECS carrier <---> M6x09-II / HD6309
          |                                                                    |
          +-------------------- ECS peripherals remain CP1610-owned ----------+
```

This makes the 6309 a coprocessor, not a second bus master. It prevents bus contention and lets either processor run independently. Direct control of the Intellivision bus, RAM, STIC, PSGs, cassette UART, or cartridge signals must remain disabled until the required ownership, voltage, direction, and timing rules are measured and documented.

### Existing-board boundary

- The M6x09-II is a local computer, not a peripheral to be driven by the ECS. Its power, reset, and bus remain autonomous.
- The carrier may attach only through a measured expansion-header pinout. Before any connection, document each pin's direction, reset state, voltage, timing, and contention tolerance.
- The recovery M6x09-II's HD6309, ASSIST09 monitor, ROM/RAM map, and serial workflow are reused unchanged for initial bring-up. A later project ROM may add mailbox drivers without removing the recovery monitor until replacement boot recovery is proven.
- Use distinct board identities for reference and performance configurations. The performance-board acceptance record must include oscillator marking, measured `E` frequency, CPU marking/speed grade, ROM checksum, ACIA control word and measured baud, 5 V rail under load, and expansion-bus timing captures.
- M6x09-I is a useful software/peripheral development platform, but its GAL16V8 address decoder and 89C2051 tick generator exclude it from the discrete-TTL daughterboard baseline.

### Deliverables

1. Establish electrical constraints: connector/pinout, supply rails, logic thresholds, timing, reset behavior, and power budget.
2. Define a read/write register map for command, status, data, interrupt/handshake, and reset; publish it before firmware development.
3. Re-run and record M6x09-II standalone acceptance: HD6309, RAM, ROM/ASSIST09, serial console, reset release, and clock; retain it as the recovery baseline.
4. Continuity-map and document the M6x09-II expansion header before assigning carrier signals; do not infer the complete header pinout from board function alone.
5. Implement the CP1610-facing carrier interface with electrically safe 5 V TTL bus buffers, deterministic address decode, and discrete handshake logic.
6. Supply a minimal CP1610 client and 6309 monitor/ASSIST09-resident program that exchange data through the mailbox.
7. Add higher-level services only after the mailbox is reliable: storage transfer, graphics/sound command queues, keyboard access, and development tools.

## 3. ECS audio integration or additional-PSG re-creation

Provide the companion system's audio through a 3.5 mm TRS female output (dual-mono initially). The project may integrate original mixed audio, use the original ECS AY-3-8917 through CP1610 mailbox commands, experimentally add direct access to that existing PSG, or build an ECS additional-PSG re-creation around a real AY-3-8917 and TTL control circuitry.

1. Measure and document a post-mix, pre-RF/baseband audio tap on the target original hardware, including level, impedance, DC bias, and noise.
2. Build an AC-coupled buffer that presents the integrated mono signal safely on both TRS channels without tying driven outputs together.
3. Implement CP1610-side mailbox services that program the original ECS AY-3-8917 on behalf of the 6309; this is the preferred non-invasive control path.
4. If direct PSG access is pursued, map every AY-3-8917 signal, add buffered single-owner arbitration, and use documented, reversible splice/jumper arrangements before cutting any track.
5. In parallel, specify an ECS additional-PSG re-creation board: a real AY-3-8917, clock, reset, register interface, analogue output, mixer, and controller-port behaviour.
6. Compare frequency, volume, noise, register response, and output level against original ECS audio before calling a direct-access modification or re-creation compatible.
7. Treat the existing Master Component AY-3-8914 path separately: the additional-PSG re-creation supplies only the ECS-added three channels, not a standalone six-channel reconstruction.

The CP1610-facing command protocol is specified in [the CP1610–6309 mailbox interface](CP1610-6309-MAILBOX.md).

## Shared rules

- Preserve an original ECS and Master Component as reference hardware; do not use either as an undocumented development target.
- Use discrete 5 V TTL/SSI/MSI logic; FPGAs, CPLDs, and other programmable-logic devices are out of scope.
- Capture measurements, schematics, firmware builds, test programs, and recordings in version control.
- Treat every unverified historical detail as a hypothesis, with its source and confidence level recorded.
- Keep original-ECS compatibility separate from extensions that require the 6309 daughterboard.

## Immediate next experiment

Make one known-good `CSAVE` recording with an ECS and a recorder from the guide's compatible class, digitize it at a lossless sample rate, and record the program name, ECS region (NTSC/PAL), recorder model, cable type, and volume settings. That capture is the reference artifact for the digital cassette-data work.

Do not connect an active digital device to either data lead until the connector, common/return, levels, and recorder-side loading have been measured. The first interface fixture should be passive: a correctly identified breakout and observation lead for the real recorder, followed by playback-only emulation, then capture, then `REMOTE` control.
