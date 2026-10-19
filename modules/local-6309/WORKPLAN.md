# Local-6309 computer workplan

## Purpose

Bring up the M6x09-II as a self-contained HD6309 computer before it is allowed to meet the ECS. It is a local computer, not an ECS bus peripheral.

## Starting allocation

- Keep one populated M6x09-II as the untouched recovery/reference machine.
- Use a blank board for experiments; fit socketed HD63C09RP, 62256 SRAM, 27C128 ROM, MC68B50, and the documented support parts.
- Begin with the proven 7.3728 MHz configuration. A plain board may use 6 MHz (`E` = 1.5 MHz). A 12 MHz CPU crystal requires the documented ACIA clock-divider modification before use.

## Work

1. Inspect bare-board fabrication, continuity, socket orientation, and 5 V/GND planes before fitting ICs.
2. Assemble and power up in stages: regulation/decoupling, reset and clock, CPU/ROM/RAM, then serial interface.
3. Measure 5 V, reset release, crystal operation, CPU `E`, address/data activity, and current draw at each stage.
4. Boot ASSIST09 or equivalent monitor and retain a verified recovery image/checksum.
5. Run RAM tests, ROM checksum, serial-loopback and terminal tests, reset/power-cycle tests, and a sustained soak test.
6. Continuity-map the H2 expansion header and label direction/reset state for every proposed carrier signal.
7. If using the 12 MHz option, isolate the original ACIA clock feed, add the 74LS74 divide-by-two circuit, measure 3 MHz `E` and 1.5 MHz ACIA clock, and repeat all tests.

## Acceptance gate

The board boots a known monitor, passes RAM/ROM/serial/reset tests repeatedly, has recorded timing/power measurements, and its selected H2 signals are mapped. Until then, attach no ECS-facing circuitry.

## Do not do

- Do not use the stocked 16 MHz or 14.31818 MHz crystal as the HD63C09RP CPU crystal.
- Do not wire H2 directly to ECS signals or another M6x09-II bus.
- Do not discard the recovery monitor while developing project firmware.
