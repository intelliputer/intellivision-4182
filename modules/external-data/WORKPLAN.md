# ECS external-data / digital-cassette workplan

## Objective

Make a solid-state external-data device that behaves as a compatible cassette recorder for the ECS #4182 `IN FROM TAPE`, `OUT TO TAPE`, and (where useful) `REMOTE` interconnects. It must preserve original ECS operation and create reproducible digital copies of programs and data.

This is the first active hardware project. Do not connect an MCU, DAC, sound card, or other active circuit to the ECS until Phase 1 is complete.

## Implementation rule

The finished external-data device is a project-built circuit, not an assembly of third-party development, audio, storage, ADC/DAC, or converter boards. It may use normal production ICs, sockets, discrete components, an SD socket, and a project PCB or wire-wrap assembly; all controller, storage, conversion, analogue-conditioning, power, and connector circuitry belongs on that assembly. The Analog Discovery 2 remains a test instrument only, never a permanent part of the device.

## Revision-zero integration platform

The principal revision-zero carrier is a Vector `8801-6` wire-wrap card in the stripped Hewlett-Packard 1611A main-board assembly. The card has been physically test-fitted and fits the assembly's 100-pin slot exactly. The HP board is deliberately treated as a **powered mechanical backplane**, not as an S-100 computer or a reused HP signal bus: its legacy circuitry has been stripped, and its revised power input carries only +5 V and ground from the salvaged Hewlett-Packard 5036A supply.

Use the 8801-6 for the full external-data prototype so that its analogue input, analogue playback output, controller/SD circuitry, and power entry can be physically separated. The smaller Vector `8015-1` remains the preferred carrier for isolated subcircuit experiments, such as a filter, output-driver, or protection network.

Before a project card is populated, make and retain a slot pin map. Only independently verified +5 V and ground contacts may be used at first; all other contacts are `NC`. A later inter-card connection must have a named function, a documented pin assignment, a defined ground return, and buffer/isolation appropriate to that function. It must never become an unqualified shared CPU or video bus.

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

- a known-good Master Component, ECS, and cartridges/software that exercise `CSAVE`, `CLOAD`, and `CVRF`;
- the original cable(s), plus a sacrificial extension or correctly identified breakout—not a cut ECS cable;
- oscilloscope with ×10 probes, DMM, and a logic/audio capture method that does not load the signal appreciably;
- the Analog Discovery 2, WaveForms, and its local datasheet;
- labelled raw-capture storage and a notebook or test log;
- the stripped HP 1611A main-board assembly, its 100-pin card slot, the fitted Vector `8801-6`, and the salvaged HP 5036A-derived +5 V supply;
- a current-limited 5 V bench supply for isolated subcircuit work and for comparison against the HP supply.

Keep one ECS/Master Component pair as the untouched reference system. Make measurements through a passive breakout, never by modifying that reference pair.

Build the ECS-side 3.5 mm straight-through data leads from [the cable-harness sheet](CABLE-HARNESS.md): one from the future device playback output to `IN FROM TAPE`, and one from `OUT TO TAPE` to the future device capture input. `REMOTE` stays unconnected until its electrical behaviour is measured.

The period [Rodney cassette-interface reference](RODNEY-TAPE-INTERFACE-REFERENCE.md) is a useful analogue-front-end pattern: separate AC-coupled transmit and receive chains, defined gain/loading, and a testable digital slicer. It is not an ECS circuit; its values, approximately 3 kHz / 100 bit/s format, and connector assignments are unproven for this project.

The AD2 captures the ECS output directly, then supplies controlled replay through a protected test box; those results define the final playback and capture circuits.

The current custom-board inventory assessment is [revision-zero BoM](BOM.md). It distinguishes confirmed IC stock from components whose exact selection must wait for the baseline captures.

## Phase 0A — Powered-backplane qualification

**Goal:** qualify the stripped HP/8801 assembly as a safe project-card carrier without connecting it to the ECS.

1. Photograph the stripped HP board, revised power plug, slot, card orientation/keying, and fitted 8801-6.
2. Create a numbered map of all 100 slot contacts. Mark only measured +5 V and ground contacts as allocated; mark every other contact `NC`.
3. With power disconnected, verify continuity within each nominated +5 V group and each nominated ground group, and verify that the groups are not shorted together.
4. Energize the assembly with no project card installed. Measure the +5 V value, polarity, ripple/noise, and any unexpected voltage on every contact intended for use.
5. Add a card-local fuse or resettable protector, reverse-polarity protection where appropriate, connector-entry bulk capacitance, and 100 nF decoupling at every IC before fitting active circuitry.
6. Prove the first populated card does not overheat or exceed its planned supply current with the ECS entirely disconnected.

**Exit condition:** a dated slot/power map and no-load/controlled-load measurements show a correctly polarised, stable +5 V rail; the protected 8801-6 card can power safely without the ECS attached.

**Stop condition:** any unexpected powered contact, uncertain slot orientation, abnormal ripple, heat, smell, or unexplained current draw stops platform work until the map and supply path are corrected.

## Phase 0 — Test records and log

**Goal:** establish repeatable material before interpreting the format.

1. Record the ECS, board revision/serial information, software title and program name, television region, cable/breakout, and AD2 configuration.
2. Make at least three `CSAVE` operations of a small, recognisable program while recording raw AD2 captures of `OUT TO TAPE`.
3. Give each recording a durable identifier, for example `ECS-001-A`.
4. Create a directory per identifier for photos, notes, scope screenshots, raw captures, and test results.

**Exit condition:** three repeatable raw `CSAVE` waveform captures exist and are fully documented.

## Phase 1 — Passive connector and electrical characterization

**Goal:** identify every conductor and its electrical behaviour without an active substitute attached.

1. Identify plug type, conductor count, shell/common connection, and which ECS lead corresponds to `IN FROM TAPE`, `OUT TO TAPE`, and `REMOTE`.
2. With the ECS off, use continuity only on the sacrificial breakout to establish conductor mapping. Do not infer tip/ring/sleeve assignments from connector appearance.
3. Measure DC voltage at every ECS signal conductor in idle, save, load, and remote states before attaching any active source.
4. Scope the ECS output while saving: amplitude, offset, frequency content, burst/envelope shape, and repeatability across runs.
5. Use AD2 Wavegen only through the protected test box to explore the ECS input: amplitude, offset, polarity, clipping threshold, and acceptance range.
6. Characterize `REMOTE` separately: open-circuit voltage, polarity, current, state transitions, and whether it is a contact closure, a voltage input, or something else.
7. Measure only with high-impedance instruments. Record probe type, ground reference, and the connected equipment for every capture.

**Deliverables:** connector/pin map, idle-voltage table, waveform captures, and a provisional interface schematic marked with measured versus assumed facts.

**Stop condition:** any unexpected DC voltage, heat, unstable operation, or uncertain common/return means stop and re-check the breakout before proceeding.

## Phase 2 — Preservation capture and protocol study

**Goal:** preserve recordings before attempting to decode or regenerate them.

1. Capture the ECS `CSAVE` waveform directly with the AD2 through the documented passive breakout.
2. Preserve the unedited raw capture as the master; record capture device, gain, rate, bit depth, channels, and cable path.
3. Make multiple captures of the same `CSAVE` operation and compare their repeatability.
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

1. Add a high-impedance, protected capture input based on the AD2 measurement evidence.
2. Save from the ECS while capturing the waveform. Keep the raw waveform even if a decoder is also implemented.
3. Compare its timing and level to the retained AD2 master capture of the same save.
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

**Exit condition:** remote operation improves usability without changing data-path reliability or introducing an unmeasured drive condition.

## Required test log for every run

| Field | Record |
| --- | --- |
| Test ID and date | Unique run identifier and date/time |
| Hardware | Master Component, ECS, device/fixture revision, cable/breakout, AD2 configuration |
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

1. Photograph and label the ECS, cables, passive breakout, and AD2 connections.
2. Perform one `CSAVE` while capturing `OUT TO TAPE` with the AD2; do not attach Wavegen yet.
3. Create test ID `ECS-001-A` and complete the log entry.
4. Save the raw AD2 capture and scope image with its settings.
5. Make only passive breakout/continuity observations. End the session before attaching any active replacement circuit.

## Path to a full circuit

After the first known-good capture, follow the circuit-design sequence in [Rodney cassette-interface reference](RODNEY-TAPE-INTERFACE-REFERENCE.md): derive the ECS electrical contract, prove raw-waveform playback, make separately testable playback and capture stages, then add `REMOTE`. A PCB/enclosure is a final deliverable after—not before—the measured interface table, successful round-trip tests, and a complete schematic/BoM/acceptance sheet.
