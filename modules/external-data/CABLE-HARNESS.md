# ECS cassette cable harness

## Established connector facts

The ECS Computer Module Guide specifies three separate, straight-through cables, 3–5 feet long:

| ECS jack | Recorder jack | Cable specified by guide | Modern physical description | Signal direction |
| --- | --- | --- | --- | --- |
| `IN FROM TAPE` | `EAR` / digital-device playback output | Mini-phone plug at both ends | 3.5 mm / 1/8-inch mono phone-plug cable | Playback source → ECS |
| `OUT TO TAPE` | `MIC` / digital-device capture input | Mini-phone plug at both ends | 3.5 mm / 1/8-inch mono phone-plug cable | ECS → capture device |
| `REMOTE` | `REM` | Sub-mini-phone plug at both ends | 2.5 mm / 3/32-inch mono phone-plug cable | Motor-control connection |

The guide says the cable ends are interchangeable: this establishes **straight-through**, not crossover, wiring. It also specifies 3–5 foot cable lengths.

The guide’s recorder recommendation is a recorder with `EAR`, `MIC`, and `REM` jacks. It does not publish a tip/sleeve electrical schematic or voltage/impedance values; those remain measurement tasks.

## Build specification

Make each data cable from flexible, screened single-channel cable with **two-conductor mono TS plugs**, not stereo TRS audio leads.

```text
3.5 mm TS data cable, both directions

plug A tip    -------------------------------- plug B tip
plug A sleeve -------------------------------- plug B sleeve

2.5 mm TS remote cable

plug A tip    -------------------------------- plug B tip
plug A sleeve -------------------------------- plug B sleeve
```

- `tip` is the centre conductor/signal lead.
- `sleeve` is the shield/return lead.
- There is no tip-to-sleeve connection inside a correct cable.
- Do not use a crossover cable, stereo patch lead, Y-adapter, or commoned pair of data cables.
- Use moulded leads only if a continuity test proves the same mapping; many modern 3.5 mm leads are TRS/TRRS and are unsuitable.

## Cables to make now

### 1. Playback / load cable

```text
Digital device PLAY output  -- 3.5 mm TS straight-through --  ECS IN FROM TAPE
```

For baseline work, this terminates at the AD2 Wavegen test box—not directly at AD2 W1. The final solid-state device will provide the playback endpoint.

### 2. Record / save cable

```text
ECS OUT TO TAPE  -- 3.5 mm TS straight-through --  Digital device CAPTURE input
```

For baseline work, terminate this at the passive breakout and AD2 differential scope input. The final device needs a protected, high-impedance capture stage.

### 3. Remote cable

```text
Recorder REM  -- 2.5 mm TS straight-through --  ECS REMOTE
```

Do not build/connect this one until the target recorder is confirmed to have a `REM` jack and its control polarity/current have been measured. The cable form is known; the electrical behaviour is not.

## Pre-connection continuity test

With cables unplugged from all equipment:

1. Verify tip-to-tip continuity, ideally under 1 Ω.
2. Verify sleeve-to-sleeve continuity, ideally under 1 Ω.
3. Verify tip-to-sleeve is open at both ends and across both cables.
4. Label both ends: `EAR → IN`, `OUT → MIC`, or `REM → REM`.
5. Photograph or log cable length, plug type, wire colour, and test result.

## AD2 measurement breakout

Do not cut the three working cables. Make a separate passive inline breakout for each signal when measurements begin. It must expose labelled `TIP` and `SLEEVE` test pads, preserve straight-through continuity, and have no resistors, capacitors, LEDs, or powered electronics in its first revision.

Only after the breakout is continuity-tested may the Analog Discovery 2 differential scope inputs be attached. The Wavegen connection is a later test-box connection to `IN FROM TAPE`, not a third cable tied into the recorder cable.

## Source

The pin/cable directions and connector classes are from the local [Computer Module Guide](../../doc/Computer_Module_Guide.pdf): cassette hook-up instructions and cassette-recorder specifications. The guide directs `EAR → IN FROM TAPE`, `MIC → OUT TO TAPE`, and `REM → REMOTE`; it calls for two mini-phone cables and one sub-mini-phone cable.
