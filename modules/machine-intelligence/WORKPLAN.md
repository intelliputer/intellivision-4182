# Machine-intelligence node workplan

## Purpose

Add a second M6x09-II/HD6309 as a local asynchronous computation node. M6x09-II A is the coordinator and owns the video interface and ECS mailbox. M6x09-II B runs planning, generation, analysis, and other bounded jobs.

Read the [dual-M6x09 machine-intelligence proposal](../../doc/DUAL-M6X09-MACHINE-INTELLIGENCE-PROPOSAL.md).

## Work

1. Bring up A and B independently under the local-6309 workplan; retain a third known-good board or recovery path.
2. Define the A-to-B command ABI: `CMD`, `JOB_ID`, arguments, request, acknowledge, status, error, protocol version, and maximum payload.
3. Build a selected register window on each board with registered latches, 74LS245 bus isolation, and two-flip-flop synchronizers for all cross-clock status signals.
4. Prove `PING`, request/reply, restart detection, timeout, queue-full, and invalid-message tests with no ECS or video attached.
5. Implement one deterministic bounded job, such as pathfinding on a small fixed map. Record execution time and result checksums.
6. Let A convert the result to local game/video commands. B must never write companion VRAM, ECS, STIC, or AY hardware directly.
7. Add compact asset/result transfer only after the message link survives soak and reset tests. Add a FIFO only when measured traffic requires it.

## Acceptance gate

B can reset, hang, or return an invalid response without corrupting A, ECS services, or video output. A can apply a valid result on a later frame without blocking display timing.

## Non-goals

- Two 6309s are not a 16-bit CPU and must not be presented as one.
- Never join the two H2 buses pin-for-pin or create raw shared RAM as revision one.
- This node is a later capability, not a dependency for digital cassette compatibility.
