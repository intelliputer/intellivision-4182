# Module workplans

Each directory is an independently testable part of the grand plan. A later module may be designed or documented early, but it must not bypass the acceptance gate of a module it depends on.

| Order | Module | Purpose | May begin active integration after |
| --- | --- | --- | --- |
| 0 | [reference-baseline](reference-baseline/WORKPLAN.md) | Preserve and measure original hardware. | Never complete; it remains the control. |
| 1 | [external-data](external-data/WORKPLAN.md) | Solid-state cassette-compatible data source. | It is the first active implementation. |
| 2 | [local-6309](local-6309/WORKPLAN.md) | Establish a standalone, recoverable M6x09-II/HD6309 computer. | External-data acceptance. |
| 3 | [host-mailbox](host-mailbox/WORKPLAN.md) | Safe discrete-TTL CP1610-to-6309 cooperation. | Local-6309 acceptance and ECS signal measurements. |
| 4 | [ecs-services](ecs-services/WORKPLAN.md) | Reuse ECS input, cassette/printer, and AY audio through owned services. | Host-mailbox acceptance. |
| 5 | [ttl-video](ttl-video/WORKPLAN.md) | Independent MAGIC-inspired 74LS video surface and RCA output. | Local-6309 acceptance; it need not wait for host integration. |
| 6 | [machine-intelligence](machine-intelligence/WORKPLAN.md) | Second M6x09-II node for bounded asynchronous computation. | Coordinator M6x09-II and local-node mailbox proof. |
| 7 | [expansion-research](expansion-research/WORKPLAN.md) | Optional direct PSG, overlay/genlock, and Keyboard-Component-inspired experiments. | Every relevant ownership and reversibility gate. |

Shared constraints: 5 V discrete 74LS TTL logic; no FPGA, CPLD, GAL, or programmable logic; no raw pin-to-pin CPU-bus links; and retain at least one unmodified ECS/Master Component plus one known-good M6x09-II as references.

## Priority rule

Only `external-data` is an active build priority now. `reference-baseline` continues alongside it. The other modules may receive research notes and designs, but no ECS-facing integration begins until the required prior gate passes. `ttl-video` may be developed on its own bench after the local-6309 gate, because it does not need to drive the original system.

## Principal risks and their gates

| Risk | Containment | Gate that must pass |
| --- | --- | --- |
| Damage to scarce original hardware | Keep a reference pair unmodified; use passive breakouts and sacrificial cables. | Reference measurements and normal-operation test. |
| Incorrect cassette connector, level, or loading | Capture a real-recorder reference and measure each lead before attaching active electronics. | Repeated raw-capture playback and `CSAVE`/`CVRF`/`CLOAD`. |
| 6309 board is unstable or unrecoverable | Retain a known-good board/ROM; test the candidate standalone. | Monitor, RAM, ROM, serial, reset, power, and timing acceptance. |
| CP1610/ECS bus contention | Use decoded, buffered, tri-stated TTL mailbox registers only. | `SERVICE_PING` and soak/reset tests with no unselected driver. |
| Audio/video interference or incorrect ownership | Keep audio on TRS and companion video on RCA; make the original owner execute ECS services. | Measured output and one-service-at-a-time regression tests. |
| TTL video timing or VRAM contention | Give display fetch priority and make CPU access scheduled/registered. | Stable RCA soak and no-contention timing captures. |
| One intelligence node destabilizes the other | Use compact, synchronized request/reply mailboxes; no raw shared bus/RAM. | Reset, timeout, malformed-message, and sustained-job tests. |
| Irreversible expansion research | Require a spare/rework-authorised target, reversible splice plan, and pass-through proof. | Continuity, ownership, timing, and recovery evidence. |

## Dependency discipline

Passing a gate means the specified test is documented and repeatable on the named hardware, not merely observed once. A failure returns work to the module that owns the boundary; it must not be bypassed by adding a later module. Every module workplan states its own acceptance condition and non-negotiable boundaries.
