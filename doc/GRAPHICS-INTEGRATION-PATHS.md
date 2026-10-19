# Graphics integration paths

## Purpose

This note defines how a 6309 application can draw game surfaces, shapes, text, and moving objects while respecting the Master Component/ECS ownership boundaries.

The intent is to recover the useful workstation/game-graphics capability associated with the Keyboard Component class of system—high-resolution graphics memory plus an external-video/overlay path—without claiming to duplicate its undocumented circuitry or allowing the 6309 to drive the Master Component bus.

## The program sees a surface API

6309 software should issue drawing commands to an abstract surface, not directly manipulate ECS or STIC pins.

```text
SURFACE_SELECT(target)
CLEAR(colour)
FILL_RECT(x, y, width, height, colour)
LINE(x0, y0, x1, y1, colour)
BLIT(source, destination, size)
SET_TILE(cell, tile, attributes)
SET_OBJECT(id, x, y, pattern, attributes)
PRESENT()
```

The graphics library selects one of the backends below. The command vocabulary can be shared, but their capabilities and timing cannot be assumed identical.

## 1. Companion surface — primary path

This is the main path for new 6309 games and graphics applications.

```text
6309 program in M6x09-II RAM/ROM
        |
        | local 6309 bus through measured H2 header
        v
74LS carrier decode / register interface / command FIFO
        |
        +--> paired VRAM and TTL cards, objects, palette, raster logic
        |
        v
TTL composite encoder --> video-composites board --> RCA --> RCA-to-HDMI converter
```

`FILL_RECT`, `LINE`, and `BLIT` initially become CPU writes to the controller's `STREAM_WRITE` register path. Once the basic framebuffer works, the hardware should gain cards/tiles, moving objects, and a command FIFO. At that point, a game updates small attribute/position lists while the TTL controller performs repeated pixel work.

The M6x09-II clock affects only how quickly this program can prepare commands or upload new VRAM data. It does **not** alter Master Component video timing or STIC drawing speed.

### ECS-assisted companion renderer

This is the intended integrated-game arrangement. The ECS is not asked to paint the companion framebuffer; it removes the surrounding compatibility work so the 6309/TTL path can concentrate on drawing.

| Responsibility | Owner | Why |
| --- | --- | --- |
| Game loop, shape list, asset decompression, scene state | 6309 | Local RAM and direct access to the companion command interface. |
| Rectangle/line/blit execution, cards, objects, palette, scan-out | Local TTL controller | Keeps repeated pixel work out of both CPUs. |
| Keyboard/controllers | ECS/CP1610 service, returned through mailbox | Reuses original ports and avoids unmeasured electrical takeover. |
| Music and sound effects | ECS AY-3-8917, requested through mailbox | Reuses the real PSG and its original mixing context. |
| Cassette/printer and ECS BASIC-facing services | ECS/CP1610 service | Preserves released-ECS compatibility and protocol ownership. |
| Optional STIC status/scoreboard/card updates | CP1610 service | Keeps the Master Component's graphics hardware under its native owner. |

The critical rule is **commands, not pixels, cross the mailbox**. A game might send `PSG_WRITE`, `INPUT_READ`, or an occasional `STIC_CARD_UPDATE` to the ECS, while its `FILL_RECT`, `LINE`, `BLIT`, and object commands stay on the fast local 6309-to-TTL path. This is how integration reduces software and electrical complexity without making the CP1610 a slow framebuffer copier.

## 2. Master Component STIC surface — mailbox service path

This is a constrained compatibility/cooperation path, not a high-resolution raster path.

```text
6309 graphics request
        |
        v
74LS CP1610–6309 mailbox
        |
        v
CP1610-resident service program
        |
        v
normal, measured CP1610-visible STIC/EXEC operations
        |
        v
Master Component RF/video output
```

The mailbox request might be `STIC_CARD_UPDATE`, `STIC_OBJECT_UPDATE`, or `STIC_COLOUR_UPDATE`. The CP1610 service validates it and performs the actual STIC-visible update. This retains one electrical owner of the Master Component hardware.

This backend maps well to cards and moving objects. It does **not** turn the STIC into a general pixel-addressable drawing surface: arbitrary lines, filled polygons, and bitmap blits must be approximated by cards/objects or rendered on the companion surface instead.

The initial mailbox hardware does not yet implement these commands. Add them only after `SERVICE_PING`, audio, and input services are stable and the CP1610/STIC access method is measured.

## 3. Synchronized companion overlay — later research path

The Keyboard Component evidence indicates use of `CBLNK` and the external-video path for a genlocked text display. That establishes a historical direction, not a ready-to-copy circuit.

```text
6309 --> companion VRAM / TTL renderer
                       |
Master timing/video --> measured sync/key capture --> TTL timing lock + keyed analogue mixer
                       |
                       v
                 combined video source --> video-composites board --> RCA
```

For this path, the companion renderer must measure and synchronize to the Master Component's `~MSYNC`, `CBLNK`, and external-video behaviour. The composite encoder/mixer—not the 6309 CPU clock—determines line/frame phase and which source is visible. The 14.31818 MHz timing source may be useful inside the companion controller, but cannot be presumed phase-locked to the Master Component.

This is a separate hardware gate. Do not connect raw TTL pixels, the M6x09-II bus, or an unmeasured composite signal to the Master Component external-video pins.

## Runtime integration

An application that combines Intellivision-era input/audio with companion graphics follows this path:

```text
ECS keyboard/controllers --> CP1610 service --> mailbox --> 6309 game loop
                                                        |
                                                        +--> game state / shape commands
                                                        |       |
                                                        |       +--> companion surface --> RCA
                                                        |
                                                        +--> optional STIC card/object requests
                                                        |
                                                        +--> PSG_WRITE requests --> CP1610 --> real ECS AY-3-8917
```

The first playable arrangement uses companion RCA video, real ECS sound through the mailbox, and input returned through the mailbox. The Master Component may remain connected and running, but its picture is not the companion game's video output until the later overlay path is proven.

## Machine-intelligence compute surface on the M6x09-II bus

The machine-intelligence program may also use a dedicated **compute surface** attached to the M6x09-II external bus. This is a local Third-computer peripheral, not an ECS or Master Component bus device.

```text
machine-intelligence program on 6309
               |
               v
M6x09-II H2 expansion bus --> 74LS decode/buffer --> compute surface
                                                     | local memory / specialised logic / optional local CPU
                                                     v
                                               result/status mailbox
               |
               +--> CP1610–6309 mailbox --> ECS services --> Master Component hardware
```

The compute surface should expose a narrow memory-mapped command/status interface, for example `MI_CMD`, `MI_ARG0..N`, `MI_REQUEST`, `MI_STATUS`, and `MI_REPLY`/result-buffer address. The 6309 remains the only bus master. A compute-surface module may have local memory or its own processor, but it receives work through buffered registers or a FIFO; it must never drive the M6x09-II bus except during a selected, direction-controlled read.

### Preferred compute-surface implementation: second M6x09-II

A second M6x09-II can supply the compute surface with a second HD6309, local ROM/RAM, clock, monitor, and serial diagnostic path. This creates a small two-node system without changing the Master Component/ECS electrical boundary.

```text
Master Component <-- ECS <-- CP1610–6309 mailbox <-- M6x09-II A: game/video coordinator
                                                        |
                                                        | 74LS dual-clock node mailbox
                                                        v
                                               M6x09-II B: machine-intelligence node
```

- **M6x09-II A** owns the companion video-controller interface and is the only M6 node permitted to issue ECS mailbox requests.
- **M6x09-II B** runs the machine-intelligence workload against its own local RAM/ROM. It returns compact decisions, paths, generated assets, or command lists to A.
- The inter-M6 mailbox uses separate 74LS245-buffered bus interfaces, register/FIFO storage, and 74LS74 synchronizers in each direction. Each node sees only its own selected register window.
- Never connect the two `H2` address/data/control buses pin-for-pin. They are independent CPU buses with independent clocks and reset states.
- Begin with compact request/reply messages. A shared bulk-memory window is a later feature requiring explicit arbitration or true dual-port memory; it is not the first hardware revision.

This gives the machine-intelligence program three clean destinations:

| Destination | Use |
| --- | --- |
| Local TTL video surface | Shapes, sprites, cards, and raster output. |
| Local compute surface | Search, planning, data transformation, AI-specific state, or other work that benefits from specialised local hardware. |
| ECS mailbox | Inputs, sound, storage/printer, and optional CP1610-owned STIC updates. |

The carrier requirements are the same as for the video and mailbox interfaces: 74LS address decode, controlled 74LS245 data isolation, registered command data, explicit owner enables, reset-to-tristate behaviour, separate power budgeting, and logic-analyser-visible handshakes.

## Hardware integration gates

1. M6x09-II standalone monitor/RAM/serial acceptance.
2. Carrier register/FIFO loopback acceptance with no ECS connected.
3. Companion fixed pattern, then VRAM and card/object rendering, to RCA.
4. CP1610 mailbox `SERVICE_PING`, input, and ECS-PSG services.
5. Optional STIC card/object service experiments, with CP1610 as sole STIC owner.
6. Optional Master-video synchronization and keyed-overlay measurements.

## Non-negotiable boundaries

- The 6309 never directly drives the CP1610, STIC, ECS PSG, or cartridge bus.
- The companion's direct-RCA mode remains independently testable even if the Master Component is absent.
- STIC cooperation is command-level and CP1610-owned.
- Genlock/overlay work is not enabled by a faster CPU crystal; it requires measured timing and a separately designed video mixer.

## Related documents

- [Project roadmap](PROJECT-ROADMAP.md)
- [CP1610–6309 mailbox interface](CP1610-6309-MAILBOX.md)
- [TTL 16-bit video controller proposal](TTL-16BIT-VIDEO-CONTROLLER.md)
- [Keyboard Component capability recovery](KEYBOARD-COMPONENT-CAPABILITY-RECOVERY.md)
- [Third M6x09-II board feasibility](M6X09-II-THIRD-BOARD-FEASIBILITY.md)
