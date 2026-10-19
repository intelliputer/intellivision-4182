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

Create a new daughterboard centred on a Hitachi 6309-compatible CPU. Its purpose is to add a capable, independently programmable computer to the Intellivision/ECS environment, borrowing the Keyboard Component's ambition without requiring that unreleased system's hardware.

The 6309 is a different 8-bit CPU family from both the Master Component's CP1610 and the Keyboard Component's 6502. It cannot execute CP1610 or 6502 binaries directly.

### First architecture: isolated coprocessor with mailbox interface

The initial design should give the 6309 its own ROM/flash, RAM, clock, reset circuit, and address/data bus. A small CPLD/FPGA or equivalent bus-interface logic exposes a deliberately narrow mailbox/register interface to the CP1610 side.

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
4. Implement the CP1610-facing interface with electrically safe bus buffers and deterministic address decode.
5. Supply a minimal CP1610 client and 6309 monitor that exchange data through the mailbox.
6. Add higher-level services only after the mailbox is reliable: storage transfer, graphics/sound command queues, keyboard access, and development tools.

## Shared rules

- Preserve an original ECS and Master Component as reference hardware; do not use either as an undocumented development target.
- Capture measurements, schematics, firmware builds, test programs, and recordings in version control.
- Treat every unverified historical detail as a hypothesis, with its source and confidence level recorded.
- Keep original-ECS compatibility separate from extensions that require the 6309 daughterboard.

## Immediate next experiment

Make one known-good `CSAVE` recording with an ECS and a recorder from the guide's compatible class, digitize it at a lossless sample rate, and record the program name, ECS region (NTSC/PAL), recorder model, cable type, and volume settings. That capture is the reference artifact for the digital cassette-data work.
