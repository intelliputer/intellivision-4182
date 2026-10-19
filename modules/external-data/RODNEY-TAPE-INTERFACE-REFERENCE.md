# Rodney cassette-interface reference

## Status and scope

This note captures the useful engineering pattern from the period cassette interface preserved in the sibling `aiventure-rodney` project. It is a **reference circuit**, not an ECS schematic and not a parts list to reproduce unchanged.

Source evidence:

- [scanned Chapter 13 circuit page](../../../cartheur/aiventure-rodney/spr-book/1301.png) (Figure 13-1);
- [period parts list](../../../cartheur/aiventure-rodney/build-of-materials/Parts-List.csv) (`Mini-tape interface circuit`); and
- [OCR/transcription evidence](../../../cartheur/aiventure-rodney/spr-book-analysis/ocr.tsv), pages 1300–1303.

The source describes an Intel 8085 cassette interface, not an Intellivision ECS device. It uses software-generated approximately 3 kHz tone bursts at roughly 100 bit/s. Those modulation, framing, program-ROM, connector, and level choices are not ECS facts.

## Reusable architecture

The circuit deliberately separates the two directions:

```text
digital transmit source --> gain / output conditioning --> AC coupling --> recorder AUX / ECS data input

recorder earphone / ECS data output --> AC coupling --> gain / limiting / slicing --> digital receive input
```

That is exactly the right starting architecture for this project. The eventual solid-state device should preserve the same functional separation, but use values selected from ECS measurements.

### Transmit-side lessons

- Generate the complete data waveform digitally, but present it through an analogue output stage.
- AC-couple the output so the target sees no imposed DC bias.
- Define the output impedance and provide a measured amplitude adjustment or calibrated fixed setting.
- Measure at the far end of the real ECS cable, where cable and input loading are included.
- Keep power-up/reset output muted or in a known idle state.

### Receive-side lessons

- AC-couple the recorder/ECS output before applying gain or threshold detection.
- Use high-impedance observation/capture first, so the interface does not load the ECS output.
- Separate analogue conditioning from the eventual digital decoder: gain, limiting/protection, and a clean thresholded/captured signal are independent concerns.
- Provide test points before and after every important stage: connector, AC-coupled input, amplified waveform, slicer output, and digital capture input.
- Preserve the analogue waveform even if a decoder succeeds; it is the archival and debugging master.

### Component pattern worth retaining

The source circuit uses one LM324 quad op amp, AC-coupling capacitors, resistor-defined gain/bias networks, and two small signal diodes. The documented values are useful as a *scale reference* for low-speed cassette audio conditioning:

| Source part class | Values/count in the Rodney circuit | ECS use now |
| --- | --- | --- |
| Quad op amp | 1 × LM324 | Candidate for bench conditioning only; suitability and headroom are to be measured. |
| Coupling capacitors | 0.1 µF × 3, 0.001 µF × 1 | Starting range for experiments, not selected ECS values. |
| Resistors | 100 Ω, 220 Ω, 820 Ω; 1 kΩ × 3; 10 kΩ × 6; 22 kΩ; 1 MΩ | Starting range for gain, bias, loading, and output experiments. |
| Signal diodes | 1N458 × 2 | Example limiter/detector parts; choose modern equivalents only after signal polarity/levels are known. |

The exact original values must not be copied into an ECS device without verification. In particular, do not presume that an LM324 powered from 5 V can safely provide the required clean amplitude into the ECS-side load. The ECS guide’s recorder requirement calls for at least 3 V peak-to-peak at 2.4 kHz and 4.8 kHz into 100 Ω; the completed output stage must be tested against that load, not merely observed unloaded.

## What does not transfer

- 8085 `SID`/`SOD` wiring, firmware, PROM arrangement, and program-memory addresses.
- The source's approximately 3 kHz / 100 bit/s encoding, leader duration, and byte format.
- The physical connector assignment or common/return arrangement.
- Any assumption about `REMOTE`; the Rodney reference has no ECS-compatible remote-control design.
- Any claim that the original ECS accepts the Rodney circuit's amplitude, bias, impedance, or polarity.

## Circuit-design research sequence

The following work turns the reference architecture into an ECS-specific full circuit.

### 1. Establish the ECS electrical contract

Using a sacrificial breakout and a real compatible recorder, measure and record:

| Question | Required observation |
| --- | --- |
| Connectors | Plug geometry, every conductor, common/return, shielding, and which cable reaches each ECS label. |
| ECS transmit | Idle DC, waveform amplitude/offset, source behaviour, frequency spectrum, and load sensitivity during `CSAVE`. |
| ECS receive | Working recorder waveform at the ECS cable end, amplitude/offset/polarity, and working-level range during `CLOAD`. |
| Loading | Behaviour with recorder connected/disconnected and with safe known test loads; no assumed 100 Ω termination. |
| Remote | Idle/control voltage, current, polarity, and transition behaviour, measured separately. |
| Safety | Power-up/down and ground-reference behaviour for both systems. |

**Output:** a measured-interface table. No schematic value is final until it cites this table.

### 2. Prove the waveform before decoding it

1. Capture several known-good `CSAVE` recordings losslessly.
2. Reproduce one raw waveform with a laboratory playback chain and prove repeated ECS loads.
3. Sweep amplitude, offset (through AC coupling), source impedance, and cable arrangement one variable at a time.
4. Record the workable window and select a conservative nominal setting.

**Output:** a playback signal specification: waveform source format, level at the ECS connector, output impedance, coupling capacitor, protection, mute state, and acceptance test.

### 3. Design the transmit/playback stage

Use the Rodney output-side pattern as the starting block diagram, not as the circuit:

```text
digital sample/tone generator
  -> DAC or filtered one-bit source
  -> gain/buffer stage
  -> AC-coupling and defined series/output impedance
  -> optional clamp/protection and mute
  -> ECS IN FROM TAPE
```

Build this on a small test fixture with a jumper-selectable gain/output network. Test into the measured ECS input and into equivalent loads. Choose the final amplifier/buffer only after confirming voltage swing, current, distortion, startup transient, and 5 V compatibility.

### 4. Design the receive/capture stage

Start with a high-impedance raw capture path, then add a separately testable decode path:

```text
ECS OUT TO TAPE
  -> input protection
  -> AC coupling / high-impedance buffer
  -> raw ADC or comparator observation point
  -> optional gain / band limiting / adaptive threshold
  -> digital capture or decoder
```

The first revision need not decode in hardware. A captured waveform that can be replayed successfully is already a valid storage result. Add the comparator/slicer only if it improves observed decoding reliability; retain raw capture regardless.

### 5. Add remote only after both data paths pass

Implement `REMOTE` as an isolated third subcircuit. Its transistor, optocoupler, relay, or open-drain choice follows the measured polarity/current requirement. It must default inactive and must not share an unmeasured signal return with the audio/data path.

### 6. Freeze a circuit revision

A complete circuit package requires:

- connector/pinout drawing and cable assembly;
- transmit and receive schematics with all component values and test points;
- power, ground, protection, shielding, and enclosure strategy;
- digital waveform source/capture specification and firmware revision;
- raw/decoded storage format definition;
- BoM with exact parts and alternates;
- bring-up sequence and acceptance-test sheet;
- scope captures demonstrating every stated level/timing limit.

Only then draw a PCB or make a permanent enclosure.

## Immediate next research action

Perform the AD2 reference-baseline capture sequence: make and document a passive breakout, then capture three ECS `CSAVE` waveforms. Replay a scaled capture only through the protected AD2 Wavegen test box. That evidence set decides whether the Rodney-style LM324 conditioning is adequate, needs a stronger output driver, or should be replaced by a different analogue front end.
