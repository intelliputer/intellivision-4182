# Optional expansion-research workplan

## Purpose

Investigate high-risk, historically inspired extensions only after the stable modules work: direct ECS PSG access, a separate AY-3-8917 recreation, and Master/companion video overlay or Keyboard-Component-style external-video research.

This folder is deliberately not an implementation promise.

## Entry criteria

- Reference baseline is complete and preserved.
- External-data, local-6309, and host-mailbox acceptance gates are met where relevant.
- A written reversible modification plan, continuity map, ownership table, and recovery plan exist.
- Spare hardware or a sacrificial/rework-authorised board is explicitly identified.

## Research tracks

### Direct ECS AY-3-8917 access

1. Map all PSG-related traces/signals and prove normal ECS timing/ownership.
2. Design single-owner buffering/tri-state arbitration and verify it on a fixture.
3. Use reversible jumpers/splices first; measure register response and audio against unmodified ECS operation.
4. Revert and compare whenever a regression appears.

### AY-3-8917 recreation

1. Specify a real AY-3-8917, clock, reset, control interface, analogue mixer, and output stage.
2. Compare tone frequency, envelopes, noise, volume, and I/O behaviour with the ECS addition.
3. Keep it electrically separate until compatibility evidence justifies an adapter.

### Video overlay / external-video path

1. Measure Master Component `~MSYNC`, `CBLNK`, external-video behaviour, timing, voltage, and source impedance.
2. Develop a passive/isolated observation fixture and test the companion controller against recorded timings.
3. Build a separate mixer/genlock prototype; prove correct blanking and no loading before any installed modification.
4. Validate through the composite/RCA path. Never infer synchronization from a 6309 clock frequency.

## Acceptance gate

An experiment may graduate only with measured timing/electrical evidence, no regression to original functionality, a repeatable pass-through test, and a documented reversal procedure. Otherwise it remains research.
