# Risk register

This register covers the ECS-mounted 6309 / discrete-TTL MAGIC-inspired companion system, including composite video and ECS-audio integration. Ratings are provisional and must be revised after bench measurements.

| ID | Risk | Likelihood | Impact | Early warning / trigger | Mitigation | Closure evidence |
| --- | --- | --- | --- | --- | --- | --- |
| R01 | Composite encoder is unstable, has incorrect level, or fails to preserve colour/sync. | High | High | Rolling image, lost lock, wrong black level, missing colour, or bright-scene clipping through the RCA-to-HDMI converter. | Prove a fixed colour-bar/pattern encoder before adding VRAM or the CPU; scope the waveform into a 75-ohm load; validate with the selected video-composites board. | Stable output through the selected converter for colour bars, black/white levels, and moving test patterns. |
| R02 | 74LS propagation delays leave insufficient margin at the chosen clock. | Medium | High | Temperature-sensitive corruption, one-pixel errors, unstable counters, or timing that fails only with all logic installed. | Keep complex datapaths at or below 7.15909 MHz; use synchronous counters, short traces, a ground plane, local decoupling, and measured timing budgets. | Logic-analyser/scope traces show setup/hold margin at worst-case supply and temperature. |
| R03 | Video-RAM arbitration permits two drivers or misses display fetches. | Medium | High | VRAM gets hot, corrupted pixels, random display failures, or CPU writes that alter unrelated words. | Use one explicit RAM owner at a time, transceivers with output-enable control, registered handoff, and `VSTATUS` polling before any CPU-stall scheme. | Stress test shows no contention and no corruption during sustained CPU writes plus active display. |
| R04 | The 6309 cannot upload enough framebuffer data for the intended visuals. | High | Medium | Full-screen animation is visibly below target frame rate. | Use `STREAM_WRITE`; measure transfer bandwidth early; implement tiles/cards, moving objects, palette changes, and a command FIFO instead of relying on full-frame redraws. | Benchmark results meet the documented update budget for the chosen demo/game workload. |
| R05 | Cartridge-bus pinout, timing, or address decode is misunderstood. | Medium | Critical | ECS/Master Component fails to boot, crashes when the board is connected, or a bus line is driven when unselected. | Measure every relevant connector signal; use buffered, tri-stated TTL interfaces; select a mailbox window only after testing; never make the 6309 a bus master. | The daughterboard remains electrically inactive when unselected and passes read/write/boot tests on reference hardware. |
| R06 | ECS power is insufficient for the TTL board or back-feeds through the connector. | High | Critical | Brownouts, hot regulator/connector, unstable video, resets, or voltage present on an unpowered system. | Make a measured current and thermal budget; use a separately regulated 5 V supply if required; add fuse, reverse-polarity protection, and deliberate common-ground design. | Current, ripple, temperature, and power-up/down behaviour are within measured limits under sustained load. |
| R07 | Large board stack damages or intermittently connects through the ECS cartridge socket. | Medium | High | Connector flex, intermittent resets, image changes when touched, or inability to fit in an enclosure. | Use mechanical support/standoffs, a controlled interconnect, strain relief, and defer cartridge pass-through. | Repeated insertion, vibration, and warm-operation tests remain fault-free. |
| R08 | The video-composites board is mistaken for a digital-video encoder. | Medium | High | TTL RGB/sync is routed directly into `VIDEO_IN`, producing no usable composite signal. | Maintain the boundary: TTL controller -> composite encoder -> video-composites `VIDEO_IN` -> RCA. | Verified baseband composite waveform exists at `VIDEO_IN` before output-board integration. |
| R09 | ECS audio tap has incompatible level, DC bias, impedance, or unacceptable hum/noise. | Medium | Medium | Distortion, hum, low volume, noise coupled from video clocks, or audio lost when the ECS is reset. | Measure a post-mix/pre-RF tap; AC-couple and buffer it; use controlled audio return routing and dual-mono isolation resistors. | TRS output has documented level/noise and clean audio under video/CPU stress. |
| R10 | ECS additional-PSG re-creation differs from original ECS behaviour. | Medium | Medium | Incorrect pitch, volume, noise, I/O-port operation, or incompatible software behaviour. | Use original ECS audio integration first; measure original clock, register response, mixing, and port behaviour before declaring the re-creation compatible. | A defined comparison suite matches the original within agreed electrical and audible tolerances. |
| R13 | Direct access modification to the existing ECS AY-3-8917 causes PSG or ECS bus contention, or damages a board trace. | Medium | Critical | Hot ICs, loss of ECS sound/input operation, resets, or a PSG pin driven by both interfaces. | Prefer mailbox-mediated CP1610 control first; map all PSG signals; use sockets/headers, buffers/transceivers, explicit owner enables, and documented removable jumpers before any cut/splice. | Scope confirms exclusive signal ownership in both modes; continuity/functional tests pass before and after the modification. |
| R11 | The draft MAGIC document is treated as a complete production specification. | High | Medium | Design decisions rely on illegible/OCR-derived fields or undocumented assumptions. | Mark every imported field with confidence and source; keep a separate project-defined register map and schematic set. | All implemented behaviour traces to either a verified measurement or an explicitly approved new design choice. |
| R12 | No adequate test equipment is available when failures occur. | Medium | High | Timing issues are diagnosed by visual symptoms only. | Make an oscilloscope, logic analyser, 75-ohm termination, known-good RCA-to-HDMI converter, and reference ECS hardware prerequisites for integration work. | Required tools and repeatable test fixtures are available before video/bus integration. |

## Gated build order

Do not begin a later gate until the stated earlier gate has passed.

1. **Power and mechanical gate:** R06 and R07.
2. **Composite-output gate:** R01 and R08 using fixed patterns only.
3. **TTL timing and VRAM gate:** R02 and R03.
4. **6309 performance gate:** R04.
5. **ECS cartridge-bus gate:** R05.
6. **Audio integration/additional-PSG re-creation gate:** R09 and R10.
7. **Direct-PSG modification gate:** R13, only after the non-invasive mailbox path is proven.
8. **Feature-expansion gate:** R11 and R12 remain active throughout.

## Review cadence

Review this register after every board revision, after each integration gate, and immediately following any unexpected reset, excessive temperature, distorted composite output, bus contention, or audio-noise event.

## Related documents

- [Project roadmap](PROJECT-ROADMAP.md)
- [MAGIC-inspired companion alternative](MAGIC-ALTERNATIVE.md)
- [16-bit TTL video controller proposal](TTL-16BIT-VIDEO-CONTROLLER.md)
- [Video performance analysis](VIDEO-PERFORMANCE-ANALYSIS.md)
