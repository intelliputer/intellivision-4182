# Safe-host mailbox workplan

## Purpose

Make a narrow, observable CP1610-to-6309 service boundary using 5 V 74LS TTL. The CP1610/ECS owns original hardware; the 6309 submits requests and receives bounded results.

Read [CP1610–6309 mailbox interface](../../doc/CP1610-6309-MAILBOX.md) before assigning any signals.

## Work

1. Establish the exact ECS attachment point, address-decode evidence, voltage levels, direction, reset behaviour, and bus timing from measurement—not connector assumptions.
2. Define revision-one registers: command, status, small data fields, request/acknowledge, error, and reset generation.
3. Draw the complete ownership table. Every shared line must name its driver, tri-state condition, pull state, and reset state.
4. Build the mailbox first on a bench fixture or local CPU-side loopback: 74LS address decode, 74LS245 direction control, latches/registers, synchronizers, and status LEDs/test points.
5. Prove one-way writes, then request/reply, invalid command handling, reset during transaction, timeouts, and 10,000+ transaction soak tests.
6. Implement minimal CP1610 and 6309 clients: `SERVICE_PING`, version read, and a short fixed payload.
7. Attach to an ECS only through the measured, buffered interface. Scope every bus-enable transition during reset and normal traffic.

## Acceptance gate

`SERVICE_PING` and bounded data exchange work repeatedly across reset/power cycles, neither side drives when unselected, and the original system still passes its baseline functional tests.

## Non-negotiable boundaries

- No raw CP1610, cartridge, STIC, PSG, or M6x09 bus connection.
- No direct 6309 access to ECS RAM or peripherals.
- Any need for a new service becomes a documented mailbox command, not a new uncontrolled wire.
