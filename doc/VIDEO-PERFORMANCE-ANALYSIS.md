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
| CPU clock example | 3.58 MHz | Working estimate; actual permitted 6309 speed must be confirmed |
| I/O cost assumption | 5 CPU cycles per byte write | Optimistic estimate; must be measured on the actual board |

### Theoretical ceiling

```text
3.58 MHz / 5 cycles per byte write = 716,000 byte writes per second
716,000 / 38,400 bytes per frame = 18.6 complete frame uploads per second
```

That figure excludes loop overhead, register setup, status polling, arbitration waits, interrupt work, and useful game/application code. A sensible early estimate is **10–15 full-screen updates per second**.

If the selected 6309 and the finished board prove reliable at 7.16 MHz, the theoretical transfer ceiling approximately doubles. This does not automatically make 60 Hz full-screen updates feasible, especially once real software and contention are included.

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

- Exact 6309 part number, speed grade, clock frequency, and bus-cycle timing.
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
