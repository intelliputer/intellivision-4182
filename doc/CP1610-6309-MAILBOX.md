# CP1610–6309 mailbox interface

## Purpose

The mailbox is a small, buffered command interface between the Master Component's CP1610 and the M6x09-II local computer's HD6309. The M6x09-II is the initial 6309 endpoint; the ECS carrier connects through only a measured, documented subset of its expansion header. The mailbox allows the 6309 to request work from ECS-aware CP1610 software without directly taking ownership of the ECS cartridge bus, SDCC, RAM, or AY-3-8917 pins.

The first use is safe access to the **real ECS AY-3-8917**: the 6309 asks for a sound-register update; CP1610-side code performs the physical ECS PSG write.

```text
M6x09-II HD6309 → local TTL mailbox → CP1610 service routine → ECS AY-3-8917
```

## Why a mailbox is needed

The CP1610 and 6309 have separate clocks, buses, programs, and reset behaviour. Allowing both to drive the ECS PSG or cartridge bus would create contention risk. The mailbox makes the daughterboard a cartridge-bus client only, while keeping the CP1610 as the sole electrical owner of the original ECS hardware.

This is a software-and-hardware contract. The mailbox hardware alone cannot make the CP1610 do anything; a CP1610 service routine must be executing.

## Required CP1610-side software

The CP1610 service routine may be supplied by a purpose-built cartridge ROM, a companion program, or another explicitly loaded ECS-aware program. It must:

1. Initialize the mailbox interface.
2. Poll its request flag, or respond to a proven interrupt path.
3. Read and validate each command and argument.
4. Perform the requested ECS operation through the CP1610's normal ECS-visible interface.
5. Return status/reply data and acknowledge the request.

Until such code is running, the mailbox is inert. The 6309 must detect that condition and avoid queuing work indefinitely.

## One-entry mailbox, revision one

All CPU-visible fields are 8 bits wide. The CPU address chosen for the CP1610 side must be measured and proven non-conflicting before implementation.

| Field | 6309 action | CP1610 action | Purpose |
| --- | --- | --- | --- |
| `CMD` | write | read | Command opcode. |
| `ARG0` | write | read | First argument, such as a PSG register number. |
| `ARG1` | write | read | Second argument, such as a PSG register value. |
| `REQUEST` | assert | observe/clear | 6309-to-CP1610 command-present handshake. |
| `ACK` | observe/clear | assert | CP1610-to-6309 command-complete handshake. |
| `STATUS` | read | write | Ready, error, service-present, and optional reply flags. |
| `REPLY` | read | write | Optional one-byte return value. |

## Basic handshake

```text
6309                                        CP1610
----                                        ------
wait until READY
write CMD, ARG0, ARG1
assert REQUEST  ------------------------->  synchronize and detect request
                                            latch/read CMD and arguments
                                            perform ECS operation
                                            write STATUS / optional REPLY
observe ACK      <------------------------- assert ACK
clear REQUEST
                                            observe request clear; clear ACK; set READY
```

The command and argument latches remain stable from `REQUEST` assertion until the CP1610 acknowledges completion. This prevents a partially updated command from being executed.

## Clock-domain safety

`REQUEST` and `ACK` cross independent CPU clock domains. They must not be used as raw asynchronous logic inputs.

- Synchronize each flag at the receiving side with two 74LS74 flip-flop stages.
- Use a level or toggle handshake, not a narrow pulse that the other CPU may miss.
- Keep the payload latches unchanged until the receiver has acknowledged them.
- Reset both handshake flags to a known idle state and expose `SERVICE_PRESENT` only after CP1610 driver initialization.

The first design should use a one-entry handshake rather than a dual-clock FIFO. A FIFO is an optimisation for later, after the basic protocol is proven with a logic analyser.

## Initial command set

| Opcode | Name | Arguments | CP1610 action |
| ---: | --- | --- | --- |
| `01` | `PSG_WRITE` | `ARG0` = AY register, `ARG1` = value | Write the indicated register on the ECS AY-3-8917. |
| `02` | `PSG_READ` | `ARG0` = AY register/port selector | Read a supported value and return it in `REPLY`. |
| `03` | `SERVICE_PING` | none | Return firmware/interface version and prove the service loop is active. |
| `04` | `INPUT_READ` | input selector | Return an ECS keyboard/controller value when the CP1610 driver supports it. |

Exact ECS PSG addresses and read/write behaviour are deliberately not hard-coded here. They must be verified on the target hardware and used only by the CP1610 service routine.

## Audio flow

```text
6309 music/game code
        |
        | PSG_WRITE commands
        v
TTL mailbox → CP1610 driver → existing ECS AY-3-8917 → original ECS audio mix
                                                                  |
                                                        measured audio tap/buffer
                                                                  |
                                                       3.5 mm TRS dual-mono output
```

This path uses the original ECS sound chip and its original analogue mixing context. It is the preferred non-invasive alternative to cutting tracks for direct PSG access.

## Performance implications

Individual PSG register writes are small transactions and music normally changes only a modest number of registers per frame/tick. A polled one-entry mailbox is therefore sufficient for initial note, volume, and noise updates.

For higher-rate effects, reduce overhead before touching ECS hardware:

1. Have the 6309 send only changed registers.
2. Add a `PSG_WRITE_BURST` command after the basic protocol is measured.
3. Add a small command FIFO only if profiling proves that the one-entry handshake is inadequate.

The CP1610 service cadence ultimately bounds latency. If it polls once per video frame, command response can approach one frame. Faster polling or a verified interrupt path may reduce latency, but must not destabilize the ECS program.

## Draft TTL implementation

| Function | Initial 74LS parts | Notes |
| --- | --- | --- |
| Command/argument/reply/status latches | 74LS374 | One octal register per independently written 8-bit field; exact count follows address map. |
| CP1610/6309 bus isolation | 74LS245 | Enable only for the active bus read/write transaction. |
| Register select | 74LS138 / 74LS139 | Decode the local 6309 and proven CP1610 mailbox addresses. |
| Handshake synchronization | 74LS74 | Two stages per crossing direction. |
| Handshake/control logic | 74LS00, 74LS02, 74LS04, 74LS08, 74LS32 | Reset, ready, request, acknowledge, and error state. |
| Optional command FIFO | 6116 SRAM plus counters/latches | Defer until performance testing justifies it. |

Do not connect CP1610 and 6309 data buses directly. The latches and controlled transceivers are the ownership boundary.

## Validation sequence

1. Verify daughterboard power/reset and that all CP1610-facing outputs are tri-stated while unselected.
2. Prove `SERVICE_PING` with a logic analyser, including reset and timeout cases.
3. Prove `PSG_WRITE` against one harmless ECS sound register and observe the real audio result.
4. Stress request/acknowledge transitions across both clocks without missed or duplicate commands.
5. Measure command latency and maximum sustainable command rate.
6. Add `PSG_READ` and input commands only after write operations are stable.
7. Only after this non-invasive path works, evaluate experimental direct AY-3-8917 access modifications.

## Non-goals

- The mailbox does not turn the 6309 into an ECS cartridge-bus master.
- The mailbox does not duplicate the AY-3-8917 in TTL.
- The mailbox does not work without CP1610-resident service software.
- The mailbox is not a substitute for measured cartridge-bus electrical and timing documentation.

## Related documents

- [Project roadmap](PROJECT-ROADMAP.md)
- [16-bit TTL video controller proposal](TTL-16BIT-VIDEO-CONTROLLER.md)
- [ECS-on-top MAGIC alternative](MAGIC-ALTERNATIVE.md)
- [Risk register](RISK-REGISTER.md)
