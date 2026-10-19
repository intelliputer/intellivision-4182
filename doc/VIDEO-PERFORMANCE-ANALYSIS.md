# 6309 / TTL video performance analysis

## Question

Can one 6309 driving a 16-bit-wide TTL video-memory controller match the drawing speed of the Intellivision Master Component?

## Short answer

It can match the television refresh rate, but a framebuffer-only version cannot efficiently repaint a full 320 × 240 screen every 60 Hz frame. To match the Master Component's *perceived* drawing speed, the TTL controller needs hardware drawing features—cards/tiles, moving objects, palette changes, and/or command queuing—so that the 6309 changes state rather than pushing all pixels.

## Architectural boundary

There is one 6309. Its external bus is 8 bits wide. The video subsystem is 16 bits wide internally:

```text
6309 8-bit bus → TTL command registers → 16-bit video bus → paired 8-bit SRAMs
```

The paired SRAMs hold one low byte and one high byte for every video-memory word. The 6309 sends a word as two byte writes. In streaming mode, the high-byte write commits the word and advances the video address automatically.

Two 6309s would not create a coherent 16-bit CPU bus; they would add ownership and synchronization problems. The correct acceleration is in the video hardware.

## Baseline framebuffer calculation

### Assumptions

| Parameter | Value | Status |
| --- | ---: | --- |
| Framebuffer | 320 × 240 pixels | Proposed first controller target |
| Pixel format | 4 bits per pixel | Proposed first controller target |
| Framebuffer size | 38,400 bytes | Derived: 320 × 240 × 4 / 8 |
| Video word | 16 bits / four pixels | Proposed paired-SRAM organisation |
| Words per framebuffer | 19,200 | Derived: 38,400 / 2 |
| Upload method | `STREAM_WRITE`: low byte then high byte | Proposed register-interface optimisation |
| Recovery CPU clock | 1.8432 MHz `E` | Proven M6x09-II configuration: 7.3728 MHz oscillator divided by four |
| Performance CPU clock candidate | 3 MHz `E` | 12 MHz crystal divided by four; documented limit of the stocked HD63C09RP and requires a separately accepted board |
| I/O cost assumption | 5 CPU cycles per byte write | Optimistic estimate; must be measured on the actual board |

### Theoretical ceiling

```text
Recovery unit:    1.8432 MHz / 5 = 368,640 byte writes/s = 9.6 frame uploads/s theoretical
Performance unit: 3.0000 MHz / 5 = 600,000 byte writes/s = 15.6 frame uploads/s theoretical
```

Those figures exclude loop overhead, register setup, status polling, arbitration waits, interrupt work, and useful game/application code. A sensible planning range is **4–7 full-screen updates per second** on the recovery clock and **8–12** on an accepted performance board.

The performance crystal is not a speed-grade override: with the stocked HD63C09RP, do not exceed 12 MHz crystal input / 3 MHz `E`. Acceptance must verify the CPU marking/specification, `E` clock, 5 V rail under load, ROM/RAM operation, ACIA divisor and actual baud, expansion-bus timing, and mailbox/video operation. Even then, 60 Hz full-screen updates remain implausible once real software and contention are included.

## Comparison with the Master Component

The Master Component's CP1610 runs at roughly 0.895 MHz, yet games can animate smoothly because the STIC performs display work in hardware. The CPU updates display cards and moving-object state; it does not repaint a framebuffer for every frame.

| Method | CPU work per animated object | Full-screen bandwidth required | Likely visual result |
| --- | --- | --- | --- |
| 6309 framebuffer only | Many pixel/word writes | High | Fine for partial updates; full-screen motion is limited. |
| CP1610 + STIC | Position/attribute/card updates | Low | Responsive 60 Hz-style object motion within STIC limits. |
| 6309 + TTL card/sprite engine | Position/attribute/card updates | Low | Can meet or exceed STIC-style perceived animation once the hardware engine exists. |

The objective therefore is not “make the 6309 write pixels faster than the STIC.” The objective is “move repeated pixel work into the TTL controller,” as the STIC did.

## Recommended feature order

1. Prove the 16-bit framebuffer path and `STREAM_WRITE` protocol.
2. Measure actual write bandwidth with a small 6309 benchmark.
3. Add a card/tile map so a background update changes an entry, not every pixel.
4. Add a moving-object engine with position, pattern, palette, priority, and enable registers.
5. Add a command FIFO so the 6309 can queue work during display fetch.
6. Add palette and raster-register changes for low-bandwidth effects.
7. Re-run the same benchmark suite after each feature.

## Measurements required

- Exact 6309 part number, speed grade, oscillator marking, `E` clock frequency, and bus-cycle timing for both recovery and performance boards.
- Performance-board serial/ACIA control word, measured baud, ROM checksum, and supply voltage under sustained load.
- 74LS propagation-delay margin at the selected controller clock and board trace length.
- SRAM access time, output-enable time, write-pulse timing, and margin under display fetch.
- Number and placement of CPU service slots per scan line/frame.
- Actual `VSTATUS` polling and `STREAM_WRITE` instruction-cycle counts.
- Full-frame upload time, 1 KiB upload time, tile-table update time, and sprite-register update time.
- Composite-output timing impact, if any, on usable VRAM service slots.

## Acceptance criteria

The controller is ready to move beyond framebuffer-only experiments when it can:

- refresh a stable picture at the chosen composite timing;
- demonstrate repeatable, contention-free streaming writes;
- perform a measured full-screen upload at its documented rate;
- update a representative tile/sprite scene at the frame rate without requiring a full-screen rewrite;
- preserve correct video and audio output while the 6309 is under sustained transfer load.

## Related documents

- [16-bit TTL video controller proposal](TTL-16BIT-VIDEO-CONTROLLER.md)
- [MAGIC-inspired companion alternative](MAGIC-ALTERNATIVE.md)
- [Project roadmap](PROJECT-ROADMAP.md)
