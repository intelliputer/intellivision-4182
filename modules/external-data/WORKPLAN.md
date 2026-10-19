# ECS external-data / digital-cassette workplan

## Objective

Make a solid-state external-data device that behaves as a compatible cassette recorder for the ECS #4182 `IN FROM TAPE`, `OUT TO TAPE`, and (where useful) `REMOTE` interconnects. It must preserve original ECS operation and create reproducible digital copies of programs and data.

This is the first active hardware project. Do not connect an MCU, DAC, sound card, or other active circuit to the ECS until Phase 1 is complete.

## Definition of done

On an unmodified Master Component and ECS, the device can:

- load several known-good programs repeatedly;
- receive a `CSAVE` recording, retain it losslessly, and replay it;
- pass `CVRF` when applicable;
- retain a raw preservation capture for every test recording; and
- fail safely: no external-data circuit may back-power or damage the ECS when either side is switched off.

`REMOTE` control is optional for the first compatible release. It is a separate milestone, not a reason to delay proven data input/output.

## Bench kit

Before beginning, assemble:

- a known-good Master Component, ECS, cartridges/software that exercise `CSAVE`, `CLOAD`, and `CVRF`, and a compatible cassette recorder;
- the original cable(s), plus a sacrificial extension or correctly identified breakout—not a cut ECS cable;
- oscilloscope with ×10 probes, DMM, and a logic/audio capture method that does not load the signal appreciably;
- isolated or battery-powered recording/playback equipment if bench ground loops are possible;
- labelled raw-capture storage and a notebook or test log;
- a current-limited 5 V bench supply for the future digital device.

Keep one ECS/Master Component pair as the untouched reference system. Make measurements through the recorder cable/breakout, never by modifying that reference pair.

## Phase 0 — Test records and log

**Goal:** establish repeatable material before interpreting the format.

1. Record the ECS, board revision/serial information, software title and program name, television region, recorder model, cable, and all recorder volume/tone settings.
2. Make at least three real-recorder saves of a small, recognisable program and prove each with `CVRF`/`CLOAD`.
3. Give each recording a durable identifier, for example `ECS-001-A`.
4. Create a directory per identifier for photos, notes, scope screenshots, raw captures, and test results.

**Exit condition:** at least one known-good physical recording can be loaded more than once and is fully documented.

## Phase 1 — Passive connector and electrical characterization

**Goal:** identify every conductor and its electrical behaviour without an active substitute attached.

1. Identify plug type, conductor count, shell/common connection, and which ECS lead corresponds to `IN FROM TAPE`, `OUT TO TAPE`, and `REMOTE`.
2. With the ECS and recorder off, use continuity only on the sacrificial breakout to establish conductor mapping. Do not infer tip/ring/sleeve assignments from connector appearance.
3. With the real recorder connected and then disconnected, measure DC voltage at every signal conductor in idle, save, load, and remote states.
4. Scope the ECS output while saving: amplitude, offset, frequency content, burst/envelope shape, and the effect of recorder volume.
5. Scope the recorder output while loading: amplitude, offset, polarity if identifiable, clipping threshold, and ECS input sensitivity versus recorder volume.
6. Characterize `REMOTE` separately: open-circuit voltage, polarity, current, state transitions, and whether it is a contact closure, a voltage input, or something else.
7. Measure only with high-impedance instruments. Record probe type, ground reference, and the connected equipment for every capture.

**Deliverables:** connector/pin map, idle-voltage table, waveform captures, and a provisional interface schematic marked with measured versus assumed facts.

**Stop condition:** any unexpected DC voltage, heat, unstable operation, or uncertain common/return means stop and re-check the breakout before proceeding.

## Phase 2 — Preservation capture and protocol study

**Goal:** preserve recordings before attempting to decode or regenerate them.

1. Capture the real recorder's playback through a known, documented input path at a lossless rate and without automatic gain control, noise reduction, or filtering.
2. Preserve the unedited raw capture as the master; record capture device, gain, rate, bit depth, channels, and cable path.
3. Make multiple captures of the same tape at different known-good volume settings.
4. Compare silence, leader, header, payload, and trailing sections visually and by measured frequency/timing.
5. Test whether replay from the capture path can load the ECS before attempting any decoding. This proves the analogue replay chain independently of the format decoder.
6. Only then document modulation, framing, bit order, checksums, names, and error behaviour. Label every conclusion with its supporting capture IDs.

**Exit condition:** a raw, lossless capture played back through a controlled analogue path can load at least one known-good save.

## Phase 3 — Playback-only prototype

**Goal:** replace the recorder only on the ECS input path.

Build the smallest reversible prototype that emits the proven waveform into `IN FROM TAPE`.

- Use AC coupling, a defined output impedance, gain trim/attenuation, and output protection.
- Do not power the prototype from an undocumented ECS connector pin.
- Provide a physical mute/disable state at power-up and reset.
- Begin with a single fixed, known-good capture stored locally; a user interface and file browser can wait.
- Monitor the output on the scope at the ECS end of the actual cable, not only at the source board.

Test order:

1. Confirm no DC component or transient reaches the ECS input at prototype power-up/down.
2. Replay one raw capture at the measured target level.
3. Establish a successful `CLOAD` at least five times without changing the setup.
4. Repeat after power cycles and cable reconnects.
5. Test the lowest and highest workable output levels, then record a conservative default.

**Exit condition:** five consecutive loads of the same program on original hardware, with retained scope captures and settings.

## Phase 4 — ECS output capture

**Goal:** record `OUT TO TAPE` safely and make its output reusable.

1. Add a high-impedance, protected capture input; it must not materially change a real recorder's behaviour when connected in parallel.
2. Save from the ECS while capturing the waveform. Keep the raw waveform even if a decoder is also implemented.
3. Compare its timing and level to a real-recorder recording of the same save.
4. Replay the captured output through the Phase-3 playback path and prove `CVRF`/`CLOAD`.
5. Repeat with several program sizes and names, including an intentionally bad/aborted save if it can be performed safely.

**Exit condition:** an ECS save captured by the device can be replayed and loaded repeatedly without using magnetic tape.

## Phase 5 — Digital library and decoded representation

**Goal:** turn proven captures into a usable storage system without losing preservation data.

- Store raw recordings as immutable masters.
- Create a decoded/container format only after the framing and integrity behaviour are understood.
- Associate every decoded object with the source capture ID, tool/firmware version, and checksum.
- Preserve failed or ambiguous captures; they are protocol evidence.
- Add selection controls only after the one-recording path is reliable: buttons/display, serial console, or removable storage are all acceptable.

**Exit condition:** at least three independently created saves are catalogued, replayable, and traceable to raw captures.

## Phase 6 — `REMOTE` control (optional)

Implement only after Phases 1–5 pass.

1. Reproduce the measured control function with an electrically isolated or otherwise proven-safe interface.
2. Default to the inactive/non-driving state at reset and power loss.
3. Verify that the device does not source current into the ECS remote line unless the measurement record explicitly requires it.
4. Test starts, stops, reset recovery, and operation with remote absent.

**Exit condition:** remote operation improves usability without changing data-path reliability or affecting a real recorder attached to the reference setup.

## Required test log for every run

| Field | Record |
| --- | --- |
| Test ID and date | Unique run identifier and date/time |
| Hardware | Master Component, ECS, recorder/device revision, cable/breakout |
| Software | Cartridge/program name and save identifier |
| Device settings | Output level, capture gain, selected file, firmware revision |
| Result | `CSAVE`, `CVRF`, `CLOAD`, pass/fail and observed symptom |
| Evidence | Capture/screenshot filenames and scope settings |
| Next action | Repeat, change one variable, or investigate |

## Safety rules

- Change one variable per test.
- Never assume audio-style connectors share a standard pinout or signal level.
- Never connect two driven outputs together; use a deliberately designed, measured mixing/isolation network if comparison is needed.
- Treat all unmeasured ECS signals as potentially damaging to modern low-voltage inputs.
- Keep raw captures and measurement notes under version control or in a backed-up archival location before editing/decoding them.
- Do not begin the 6309, mailbox, video, or machine-intelligence hardware because the cassette work exists; begin those only after this workplan’s defined exit condition is met.

## First bench session

1. Photograph and label the ECS, recorder, cable, and intended breakout.
2. Prove one real-recorder `CSAVE` followed by `CVRF` and `CLOAD`.
3. Create test ID `ECS-001-A` and complete the log entry.
4. Capture the real-recorder playback losslessly.
5. Make only passive breakout/continuity observations. End the session before attaching any active replacement circuit.
