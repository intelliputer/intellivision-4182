# Reference-baseline workplan

## Purpose

Create the measurement record that makes all later work reversible and falsifiable. The reference ECS and Master Component are controls, not development targets.

## Work

1. Assign identities to the Master Component, ECS, power supply, cables, cartridges, and one untouched M6x09-II.
2. Photograph both sides of every board and connector before modification; record board markings and visible rework.
3. Prove normal operation: boot, controller/keyboard operation, ECS sound, and a known cassette save/load using a real recorder.
4. Record supply voltages under idle and active load, connector pin observations, audio levels, and cassette waveform captures.
5. Keep a measurement log with instrument, probe, grounding, board identity, date, and capture filename.
6. Make a continuity map only on sacrificial cables, extension boards, or spare boards—never by cutting the reference harness.

## Deliverables

- inventory/identity sheet and photographic record;
- baseline functional-test log;
- power, audio, cassette, and connector measurement records;
- labelled raw captures and scope screenshots.

## Acceptance gate

Another person can distinguish measured facts from assumptions, reproduce the normal ECS test, and restore the reference system to its documented condition.

## Rules

- No track cuts, drilled holes, or active external drivers on the reference pair.
- Any surprising voltage, heat, or behaviour pauses the experiment until documented.
- Keep this module active for the life of the project.
