# Reference-baseline workplan

## Purpose

Create the measurement record that makes all later work reversible and falsifiable. The reference ECS and Master Component are controls, not development targets.

## Work

1. Assign identities to the Master Component, ECS, power supply, cables, cartridges, and one untouched M6x09-II.
2. Photograph both sides of every board and connector before modification; record board markings and visible rework.
3. Prove normal operation: boot, controller/keyboard operation, ECS sound, and a `CSAVE` waveform capture using the passive breakout and AD2.
4. Record supply voltages under idle and active load, connector pin observations, audio levels, and cassette waveform captures.
5. Keep a measurement log with instrument, probe, grounding, board identity, date, and capture filename.
6. Make a continuity map only on sacrificial cables, extension boards, or spare boards—never by cutting the reference harness.

## Deliverables

- inventory/identity sheet and photographic record;
- baseline functional-test log;
- power, audio, cassette, and connector measurement records;
- labelled raw captures and scope screenshots.

## External-data pre-development characterization

This is a reference-baseline activity, not external-data-device development. Use the [Analog Discovery 2 — NI Edition datasheet](../../datasheets/AnalogDiscovery2NIDatasheet.pdf) and a passive, labelled cable breakout to establish the actual cassette electrical contract before designing an emulator circuit.

### Fixture rules

- Use only the AD2 differential analogue scope inputs for first observation. Connect `1+` to the measured signal and `1−` only after the cable return/common has been identified.
- Do not connect AD2 3.3 V digital I/O to the ECS.
- Do not connect Wavegen directly to an ECS data input. Wavegen must pass through an AC-coupled, current-limited, variable-attenuation test box.
- Do not power the ECS or Master Component from the AD2 supplies.
- Make no active connection until the passive breakout has been continuity-mapped and photographed.

### Capture sequence

1. Record the ECS/Master Component, software, power, cable/breakout, AD2 serial/configuration, and test identifier.
2. Capture `ECS DATA OUT` during a successful `CSAVE`. Save the raw AD2 data, scope image, sample rate, voltage range, trigger, and probe/cable connection record.
3. After the controlled-input gate, capture the signal arriving at `ECS DATA IN` during a successful AD2 replay; do not presume it is the same amplitude or polarity as the output capture.
4. Measure and record idle DC, peak-to-peak voltage, offset, leader, burst/envelope timing, dominant frequencies, and repeatability across at least three runs.
5. Keep the captures unedited as preservation masters. Derived WAV, CSV, decoded data, and plots must identify their raw source capture.

### Controlled input exploration

Only after the capture sequence:

1. Connect AD2 Wavegen `W1` through the test box to `ECS DATA IN`; monitor before and at the ECS end with the two scope channels.
2. Start at zero offset and minimum amplitude. Test 2.4 kHz and 4.8 kHz signals first, increasing level only within the measured range.
3. Record the accepted/rejected level window, source/loading behaviour, and all settings. A tone-response test is not proof of data compatibility.
4. Replay a measured `CSAVE` waveform only after its scaling, AC coupling, and protection are documented. Attempt `CLOAD`/`CVRF` with one changed variable per run.

### Canonical first cassette test: `TEST`

Use the Computer Module Guide's own **Volume Balance** BASIC program as the first cassette test program. It is a particularly good baseline because the guide explicitly directs the owner to save it, verify it, and reload it under the name `TEST` after using it to balance the Master Component and Computer Module sound.

The complete bench procedure, exact program listing, evidence naming, and results table are in [Conducting test: ECS `TEST` cassette baseline](CONDUCTING-TEST.md).

1. Enter the program directly from printed page 10 of the guide; photograph or transcribe the entered listing as test evidence rather than relying on OCR of the scan.
2. Run it and set the ECS balance control so its alternating Master Component/ECS tones are equal in perceived level. Record the balance-control position.
3. Stop the program with `ESC`; retain the program in memory.
4. Issue `CSAVE TEST` while capturing `OUT TO TAPE` with the AD2. This is capture series `ECS-TEST-001`.
5. Repeat twice with no program changes (`ECS-TEST-002`, `ECS-TEST-003`) and compare leader, payload duration, amplitude, and timing.
6. Once protected Wavegen replay is ready, clear/reset the program memory as appropriate, issue `CLOAD TEST`, and replay a scaled raw capture. Then use `CVRF TEST` as the integrity check.

This test is intentionally small, known to the ECS guide, and easy to re-enter. It exercises the real `CSAVE`/`CLOAD`/`CVRF` path without requiring a custom program or a tape transport.

### Baseline exit evidence

Before external-data hardware development starts, retain:

- connector and return/common mapping;
- at least three raw `CSAVE` waveform captures, plus `CLOAD` captures once controlled replay succeeds;
- a measured voltage/timing table for both directions;
- the AD2 fixture diagram and WaveForms settings/files;
- the safe initial Wavegen test-box schematic/value record; and
- a log showing whether a captured waveform can be replayed successfully.

Failure to load a synthetic replay is useful evidence; it means the work remains in this baseline investigation rather than being solved by prematurely changing the recorder or emulator design.

## Acceptance gate

Another person can distinguish measured facts from assumptions, reproduce the normal ECS test, and restore the reference system to its documented condition.

## Rules

- No track cuts, drilled holes, or active external drivers on the reference pair.
- Any surprising voltage, heat, or behaviour pauses the experiment until documented.
- Keep this module active for the life of the project.
