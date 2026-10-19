# TTL video-controller workplan

## Purpose

Build an independent MAGIC-inspired video surface from 74LS TTL, RAM, ROM, and conventional analogue circuitry. It is not a faster STIC and does not initially overlay the Master Component picture.

Read [TTL 16-bit video controller proposal](../../doc/TTL-16BIT-VIDEO-CONTROLLER.md) and [video performance analysis](../../doc/VIDEO-PERFORMANCE-ANALYSIS.md).

## Boundaries

- The controller owns its paired 8-bit VRAM; the paired RAMs make a 16-bit video word, not a 16-bit CPU.
- The M6x09-II submits commands through a controlled interface; it does not share the live video RAM bus.
- Video leaves through the project’s separate `video-composites` board to RCA. Audio does not travel on RCA.
- NTSC/PAL conversion is outside this module; an RCA-to-HDMI converter handles display compatibility.
- The HP/Vector carrier may connect to a video daughterboard with a defined harness as roles are assigned. Carry only protected power, reset, registered command/status signals, and the final conditioned composite/audio connection where needed. Keep the video clock, pixel path, VRAM buses, and timing-critical TTL local to the daughterboard.

## Work

1. Freeze a revision-one display mode, colour strategy, memory map, timing budget, 5 V power estimate, and the carrier-versus-daughterboard ownership boundary. If a harness is used, assign every conductor, its return path, voltage level, direction, termination/protection, and maximum intended cable length.
2. Build the clock/reset and horizontal/vertical timing chain on a test fixture. Verify stable sync, blanking, and test points before attaching VRAM.
3. Add display-address generation, paired VRAM, and serial/parallel pixel output. Prove static patterns, then RAM-read consistency and no bus contention.
4. Add the TTL/composite encoder stage and test through the existing video-composites board to its RCA output.
5. Add CPU command registers, status (`VSTATUS`), and a defined arbitration policy: display fetch always wins; writes occur only in documented free slots.
6. Implement the smallest command set: clear, fill, plot/line or tile/object update, and bounded transfer.
7. Measure command latency, frame stability, VRAM margins, output levels, and current draw. Only then add moving objects, cards/tiles, palette, or sprites.

## Acceptance gate

The controller produces stable RCA test patterns for a long soak, preserves display timing while the 6309 submits commands, and has no simultaneous VRAM drivers. A demo draws a bounded scene without changing original Master Component video behaviour.

## Do not do

- Do not connect it to `~MSYNC`, `CBLNK`, or external-video signals until the optional overlay module is approved.
- Do not use FPGA/CPLD/GAL logic or rely on an unmeasured shared-memory shortcut.
