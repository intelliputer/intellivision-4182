# Dual-M6x09 machine-intelligence proposal

## Proposition

Build the Intellivision system as a small, layered computer rather than asking any one original board to become something it is not:

```text
Master Component --> ECS --> M6x09-II A --> M6x09-II B
  original console    service     coordinator    machine-intelligence node
```

The Master Component retains its CP1610/STIC identity. The ECS retains control of the released keyboard/controller, cassette/printer, and added-sound facilities. M6x09-II A—the **Third computer**—coordinates a new companion system: its local TTL video surface, game/application state, and safe requests for ECS services. M6x09-II B is a distinct **machine-intelligence node** with its own 6309, ROM, RAM, clock, monitor, and local program.

This is capability recovery, not a claim of binary or electrical compatibility with the unreleased Keyboard Component. It borrows the useful historical idea of an ambitious computing system surrounding the Master Component while preserving a clear electrical owner for every original resource.

## Why two 6309 computers

One 6309 can operate the companion video system, prepare shape commands, receive ECS input, and run game logic. A second 6309 gives the project a place to run longer-lived or more computationally irregular work without interrupting display cadence:

- pathfinding, planning, rule evaluation, and opponent behaviour;
- procedural maps, scenes, dialog, and asset preparation;
- data transformation, compression/decompression, and storage indexing;
- development-time analysis, diagnostics, and test generation;
- future specialised local hardware controlled by the intelligence node.

The point is not that two 8-bit CPUs form a 16-bit CPU. They do not. The gain comes from **separation of responsibilities and asynchronous work**, with a defined message boundary between the two systems.

## Node roles

| Node | Primary responsibility | May access | Must not access directly |
| --- | --- | --- | --- |
| Master Component | Original game console, CP1610/STIC execution | Its native system hardware | M6x09 buses or companion VRAM |
| ECS / CP1610 service | Keyboard/controllers, cassette/printer, real ECS PSG, optional STIC service | Original ECS-visible hardware | Direct companion VRAM or intelligence-node bus |
| M6x09-II A: coordinator | Game loop, companion graphics commands, video state, ECS mailbox client | Local TTL video interface; CP1610 mailbox | M6x09-II B raw bus; original hardware directly |
| M6x09-II B: intelligence | Planning, generation, analysis, background jobs | Its local RAM/ROM and node mailbox | ECS, Master Component, video VRAM, or M6x09-II A raw bus directly |

M6x09-II A is the only new computer allowed to request ECS services. M6x09-II B communicates with A, and A translates only approved requests into the CP1610–6309 mailbox protocol.

## Hardware topology

```text
                         original hardware domain
  Master Component <---> ECS <---> CP1610-resident service program
                                ^
                                | decoded CP1610–6309 mailbox
                                v
                         coordinator domain
  M6x09-II A (HD6309) <---> 74LS video interface --> TTL video controller --> RCA
         ^
         | selected A-side register window
         |                 local-node domain boundary
         v
  74LS dual-clock mailbox / optional later FIFO
         ^
         | selected B-side register window
         v
  M6x09-II B (HD6309) --> local intelligence program and local RAM/ROM
```

The boundaries are deliberate:

1. The CP1610–A mailbox protects original hardware.
2. The A–B mailbox protects the two independent 6309 buses.
3. The A-to-video interface protects video RAM and timing from arbitrary CPU bus traffic.

No boundary is a pin-for-pin bus connection.

## A–B node mailbox

The two M6x09-II boards have independent clocks, resets, and buses. Their `H2` connectors must therefore never be joined directly. The first inter-node link is a message device with a selected local register window on each side.

### Revision-one fields

| Field | A-to-B purpose | B-to-A purpose |
| --- | --- | --- |
| `CMD` | Job opcode | Completion/event opcode |
| `JOB_ID` | Identifies a request and result | Identifies the completed request |
| `ARG0..ARG3` | Small parameters, addresses, flags | Small results, status, flags |
| `REQUEST` | Job pending | Result/event pending |
| `ACK` | Consumer has captured message | Consumer has captured message |
| `STATUS` | Node ready, busy, error, reset generation | Node ready, busy, error, reset generation |

Use registered latches, controlled bus transceivers, and two-flip-flop synchronizers in both directions. A one-entry request/reply handshake is sufficient at first. Add a FIFO only after measurement shows that it is needed.

### Example jobs

| A-to-B command | B-to-A result | Meaning |
| --- | --- | --- |
| `PLAN_PATH` | `PATH_READY` | Find a path from compact map/state inputs. |
| `EVALUATE_STATE` | `ACTION_READY` | Select an action or policy result. |
| `GENERATE_REGION` | `ASSET_READY` | Create a map chunk or object description. |
| `DECOMPRESS_ASSET` | `BUFFER_READY` | Prepare data in B-local storage for staged transfer. |
| `DIAGNOSTIC` | `DIAGNOSTIC_REPLY` | Run a node self-test or return performance data. |

The payload starts compact. A returns a buffer handle only after B has placed an explicitly bounded result into a transfer mechanism. Do not introduce shared RAM as the first bulk-data solution.

## Graphics and game loop

The coordinator node owns presentation. The intelligence node never writes video RAM or the ECS directly.

```text
ECS input --> CP1610 service --> mailbox --> M6x09-II A game loop
                                                |
                                                +--> shape/object commands --> TTL video surface
                                                |
                                                +--> job request --> M6x09-II B
                                                |                         |
                                                |                         +--> decision / asset / path result
                                                |
                                                +--> PSG_WRITE --> CP1610 service --> real ECS AY-3-8917
```

This permits a game to draw shapes, tiles, objects, and companion bitmap scenes at its own cadence while B plans the next move, region, or response. A may apply B's result during a later frame, avoiding a requirement that intelligence work finish within a single video frame.

For an optional STIC-integrated display element, A sends an approved high-level command to the CP1610 service, such as a card/object or score update. The CP1610 retains electrical control of the STIC. The full companion picture remains on the independent RCA path until a separate, measured genlock/overlay project is proven.

## Software model

### Coordinator firmware on A

- Interrupt/frame task: present video state and consume controller/input events.
- Game task: maintain deterministic game/application state.
- ECS-service task: issue/receive mailbox operations for input, PSG, cassette, printer, or optional STIC requests.
- Node-link task: submit intelligence jobs, poll/receive completions, enforce timeouts.
- Asset task: stage data to the video controller without blocking the presentation loop.

### Intelligence firmware on B

- Node monitor and self-test.
- Message dispatcher with timeout/retry/error reporting.
- Job scheduler; initially one active job at a time.
- Local work buffers entirely in B RAM.
- Result serializer that returns compact, versioned messages to A.

No operating system is required for revision one. The important properties are fixed message formats, bounded jobs, versioned command sets, and recoverable resets.

## Clock and board configuration

The two M6x09-II boards do not need identical clocks.

- A benefits from the best accepted clock configuration because it coordinates graphics and transfer bandwidth.
- B benefits first from reliability, repeatability, and easy diagnostics; its clock can be the proven reference configuration until profiling proves a faster clock useful.
- Each node's serial interface and local timing must stay within its own CPU and MC68B50 specifications. Clock changes are node-local changes, not a way to synchronize video or the Master Component.

The companion video controller has its own clock/timing domain. Any future Master-video overlay requires measured synchronization to Master Component signals; it is not achieved by matching an M6x09 crystal frequency.

## Power, reset, and physical boundaries

- Each M6x09-II receives a measured 5 V power budget and local decoupling.
- Do not rely on the ECS connector to power both new boards and the TTL video system.
- A reset on B must result in a clear node-status change at A, not random messages or driven bus lines.
- A reset on A must leave the CP1610-facing and B-facing interfaces tri-stated or at defined idle states.
- Use a mechanically supported backplane/carrier; do not use a long, loose ribbon as an unbuffered CPU bus.
- Keep one known-good M6x09-II separate as a recovery/programming reference.

## Incremental implementation

1. Bring up M6x09-II A and B separately with their own monitors, RAM tests, and serial diagnostics.
2. Map each `H2` connector and build an A–B local mailbox loopback fixture with no ECS attached.
3. Prove one-way `PING`, then request/reply, reset detection, timeout, and sustained transfer tests.
4. Add one useful job, such as deterministic pathfinding over a small map.
5. Connect A—not B—to the CP1610–6309 mailbox and prove ECS input/PSG services.
6. Connect A to the companion TTL video controller and render B-provided object/path results.
7. Add bounded bulk-result transfer only after the compact-message path has been measured.
8. Consider specialised B-side hardware only after the two-node software/mailbox model is stable.

## Non-goals

- No shared raw 16-bit CPU bus.
- No FPGA, CPLD, GAL, or programmable logic.
- No direct B-to-ECS, B-to-STIC, or B-to-video-RAM electrical connection.
- No claim that this reproduces Keyboard Component hardware or software.
- No genlock/overlay claim before measured timing, video-level, and ownership work.

## Decision

The two-M6x09 configuration is a realistic way to give the project a true machine-intelligence computing surface using boards and 6309-family parts already in scope. It focuses complexity where it is manageable: small, observable TTL mailboxes between independently testable computers, rather than shared CPU buses or invasive control of original hardware.

## Related documents

- [Graphics integration paths](GRAPHICS-INTEGRATION-PATHS.md)
- [Third M6x09-II board feasibility](M6X09-II-THIRD-BOARD-FEASIBILITY.md)
- [CP1610–6309 mailbox interface](CP1610-6309-MAILBOX.md)
- [TTL 16-bit video controller proposal](TTL-16BIT-VIDEO-CONTROLLER.md)
- [Keyboard Component capability recovery](KEYBOARD-COMPONENT-CAPABILITY-RECOVERY.md)
