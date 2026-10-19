# Project roadmap

This repository has two related, but independently deliverable, hardware projects. The first preserves compatibility with the released ECS #4182. The second is a new computer subsystem; it must not be represented as original ECS hardware.

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

Create a new cartridge-port daughterboard, physically stacked above the ECS, centred on a Hitachi 6309-compatible CPU. Its purpose is to add a capable, independently programmable computer to the Intellivision/ECS environment, borrowing the Keyboard Component's ambition without requiring that unreleased system's hardware.

The 6309 is a different 8-bit CPU family from the Master Component's CP1610 and cannot execute CP1610 binaries directly. MAME models the separate Keyboard Component with a 6502, but its surviving service manual only establishes a two-CPU arrangement, not the CPU identities; no historical binary compatibility is being claimed either way.

### First architecture: isolated coprocessor with mailbox interface

The initial design should give the 6309 its own ROM/flash, RAM, clock, reset circuit, and address/data bus. Discrete 5 V TTL logic exposes a deliberately narrow mailbox/register interface to the CP1610 side: bus transceivers, address decoders, registers/latches, gates, and interrupt/handshake logic.

```text
Master Component CP1610 <--safe, decoded mailbox--> interface logic <---> 6309 + local RAM/ROM
          |                                                                    |
          +-------------------- ECS peripherals remain CP1610-owned ----------+
```

This makes the 6309 a coprocessor, not a second bus master. It prevents bus contention and lets either processor run independently. Direct control of the Intellivision bus, RAM, STIC, PSGs, cassette UART, or cartridge signals must remain disabled until the required ownership, voltage, direction, and timing rules are measured and documented.

### Deliverables

1. Establish electrical constraints: connector/pinout, supply rails, logic thresholds, timing, reset behavior, and power budget.
2. Define a read/write register map for command, status, data, interrupt/handshake, and reset; publish it before firmware development.
3. Prototype the daughterboard in isolation and verify the 6309's RAM, ROM/flash, serial console, reset, and clock behavior.
4. Implement the CP1610-facing interface with electrically safe 5 V TTL bus buffers, deterministic address decode, and discrete handshake logic.
5. Supply a minimal CP1610 client and 6309 monitor that exchange data through the mailbox.
6. Add higher-level services only after the mailbox is reliable: storage transfer, graphics/sound command queues, keyboard access, and development tools.

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
