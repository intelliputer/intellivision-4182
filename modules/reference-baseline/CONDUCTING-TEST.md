# Conducting test: ECS `TEST` cassette baseline

## Purpose

Produce the first three authoritative raw captures of the ECS cassette output without building or connecting a digital-cassette circuit. This is the entry test for the external-data module.

The test uses the Computer Module Guide's own Volume Balance program. The guide explicitly says to save, verify, and reload it under the name `TEST` to check cassette hookup. Here, the Analog Discovery 2 (AD2) captures the native `CSAVE TEST` waveform directly.

## Test boundaries

- This procedure observes `OUT TO TAPE`; it does **not** drive `IN FROM TAPE`.
- Do not connect AD2 Wavegen, AD2 digital I/O, the Minisette, a PC audio jack, or any other active source during this test.
- Use a passive 3.5 mm TS breakout only. It must be continuity-tested before use.
- The ECS/Master Component remains unmodified.

## Equipment

- Master Component, ECS 4182, keyboard, controllers, correct power supplies, and display.
- Analog Discovery 2 with WaveForms and [its datasheet](../../datasheets/AnalogDiscovery2NIDatasheet.pdf).
- One 3.5 mm TS passive breakout connected to `OUT TO TAPE`; no cut original cable.
- AD2 flywires/probes, DMM, camera, and test log.

## Program under test

Enter the following exactly as printed on page 10 of the Computer Module Guide. In particular, the guide prints `PRIN C`, not `PRINT C`.

The same listing is preserved as [TEST.bas](TEST.bas).

```basic
10 V=15
20 P=200
30 C=1
40 CALL HUSH
50 CALL TONE
60 PRIN C
70 C=4
80 CALL HUSH
90 CALL TONE
100 PRIN C
110 GOTO 30
```

Photograph the entered listing on the screen or retain a manual transcription in the test record. Do not “correct” the guide's spelling from an OCR transcription.

## Pre-flight checklist

- [ ] Reference hardware identity, ECS revision, power supplies, software/program name, date, and operator recorded.
- [ ] ECS connector and breakout photographed and labelled.
- [ ] Breakout continuity passed: ECS tip reaches labelled signal pad; ECS sleeve reaches labelled return pad; no tip/sleeve short.
- [ ] AD2 scope inputs only: `1+` to signal pad, `1−` to the identified sleeve/return pad.
- [ ] Wavegen and all AD2 digital I/O are disconnected.
- [ ] AD2 input range and trigger settings recorded before the first capture.
- [ ] A new capture directory exists: `ECS-TEST-001/`.

## Procedure

### A. Establish the guide's audio baseline

1. Boot the ECS and select BASIC as directed by the guide.
2. Enter the program above, pressing `RTN` after every line.
3. Type `RUN` and press `RTN`.
4. Adjust the ECS balance control until the two alternating tones sound equal. Record the control position and any observation about tone quality.
5. Press `ESC` to stop the program. Leave `TEST` in memory.

### B. Capture `ECS-TEST-001`

1. In WaveForms Scope, begin with a conservative voltage range and no assumptions about DC offset. Confirm that the trace is stable in idle.
2. Arm a single capture. Use an edge trigger only after a manual capture has identified an appropriate threshold.
3. At the ECS, enter `CSAVE TEST` and press `RTN`.
4. Save the unedited AD2 capture in its native form and export a lossless data form such as CSV. Save a scope screenshot showing scale, sample rate, trigger, and time base.
5. Record idle DC, peak-to-peak voltage, offset, leader duration, burst/envelope shape, visible dominant frequencies, and total captured duration. Do not alter the source capture.

### C. Repeatability captures

1. Without editing the BASIC program or moving the breakout, repeat `CSAVE TEST` for `ECS-TEST-002` and `ECS-TEST-003`.
2. Record every changed variable, including AD2 range/trigger changes.
3. Compare the three traces for amplitude, polarity, leader length, burst spacing, and total timing. Preserve differences; do not average them away.

## Results record

| Field | `ECS-TEST-001` | `ECS-TEST-002` | `ECS-TEST-003` |
| --- | --- | --- | --- |
| Date/time |  |  |  |
| ECS / power identity |  |  |  |
| AD2 serial / WaveForms version |  |  |  |
| Scope range / sample rate / trigger |  |  |  |
| Idle DC / offset |  |  |  |
| Peak-to-peak voltage |  |  |  |
| Leader duration |  |  |  |
| Payload duration |  |  |  |
| Raw-capture filename |  |  |  |
| Screenshot filename |  |  |  |
| Observations / anomalies |  |  |  |

## Pass condition

The test passes when three labelled, unedited `CSAVE TEST` captures exist with sufficient metadata to reproduce the setup and compare their timing/level. A successful load is **not** required in this observation-only test.

## What happens next

Use these captures to design the protected AD2 Wavegen test box. Only after that fixture is measured at the ECS connector may a scaled replay be attempted with `CLOAD TEST` and then `CVRF TEST`. The result of that replay—not an assumed cassette format—sets the playback-circuit requirements.

## Source

[Computer Module Guide](../../doc/Computer_Module_Guide.pdf), printed page 10 (Volume Balance program) and printed page 11 (instruction to use it as the cassette/printer test program, saving it as `TEST`).
