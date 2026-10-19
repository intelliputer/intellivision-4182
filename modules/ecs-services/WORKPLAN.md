# ECS-services and audio workplan

## Purpose

Reuse the released ECS where it is strongest: input, cassette/printer facilities, and its actual AY-3-8917 sound chip. The CP1610 service program remains the electrical owner; the 6309 requests work through the mailbox.

Audio output for the companion is a 3.5 mm stereo female jack carrying buffered dual-mono initially. Video is separate and goes only to RCA via the companion video path.

## Work

1. After mailbox acceptance, choose one service only: start with a version/status command, then keyboard/controller input, then PSG write/readback where safely supported.
2. Define a versioned command record for each service: request fields, result fields, maximum latency, errors, and ownership.
3. Write a CP1610 client/service routine that uses existing ECS paths; write matching 6309 test software.
4. For audio, locate and measure a post-mix/pre-RF or otherwise suitable analogue tap: level, impedance, DC bias, noise, and response.
5. Build an AC-coupled buffer with a known input load and output protection; feed the same mono signal to left and right TRS channels without tying active sources together.
6. Compare tones/noise/envelopes from the real ECS AY against expected commands and record output level at the jack.
7. Add cassette/printer requests only after the standalone external-data module is proven; never let a mailbox experiment disturb the cassette electrical interface.

## Acceptance gate

Each service has repeatable hardware tests, a documented command definition, bounded failure behaviour, and no regression in standalone ECS operation. Audio must be clean and safe at the TRS jack before it is connected to external amplification.

## Deferred choices

Direct access to the existing AY-3-8917 and a new AY-3-8917 recreation are optional research. Neither is required to use the original chip by CP1610-owned mailbox commands.
