# Draft purchase BoM

**Status:** planning draft, assembled 2026-10-20 from the module workplans and
their linked BoMs.  It is a procurement aid, not a released schematic or an
authorisation to connect unmeasured circuitry to an ECS.

The inventory statements in this document are only those recorded in the
project documents (principally the 2026-10-19 immediate-inventory snapshot).
Before ordering, physically count stock and check package, pin spacing and
voltage ratings.

## Purchase now — baseline and active external-data work

These purchases are either passive test equipment/cabling or mechanical items;
they do not depend on the still-unmeasured cassette signal levels.

| Qty. | Item / minimum specification | Used by | Reason / status |
| ---: | --- | --- | --- |
| 2 | 3.5 mm (1/8 in) mono **TS**, screened, straight-through leads, 3–5 ft | `reference-baseline`, `external-data` | One `PLAY → IN FROM TAPE`, one `OUT TO TAPE → CAPTURE`.  Do not substitute TRS/TRRS leads; continuity-test each lead. |
| 2 | Passive inline 3.5 mm TS breakouts, or parts to make them (male/female TS connectors, screened two-conductor cable, labelled tip/sleeve pads) | `reference-baseline` | One per data path.  First revision has no active/passive components beyond the straight-through connection and test pads. |
| 1 | Current-limited 5 V bench supply | `external-data` | Required for isolated subcircuit tests and to qualify the salvaged HP supply.  Do not use ECS power. |
| 1 | Card-local 5 V protection set: fuse or resettable protector, reverse-polarity protection as appropriate, connector-entry bulk capacitor(s) | `external-data` | Mandatory before active circuitry on the Vector 8801-6 card.  Select ratings after the backplane current budget is measured. |
| 25–50 | 100 nF ceramic decoupling capacitors, ≥25 V, through-hole, 2.54/5.08 mm lead spacing to suit carriers | `external-data`, mailbox prototype | No immediate inventory is recorded.  Buy a small compatible lot now; use one at every IC. |
| 1 | Reset pushbutton, momentary normally-open, through-hole/panel compatible | `external-data` | Required for the standalone controller card; exact mounting follows layout. |
| 1 | ISP header, 2×3, 2.54 mm, through-hole | `external-data` | Required to program the socketed ATmega328P without a development board. |

Existing confirmed stock covers the initial external-data controller/analogue
experiments: ATmega328P, 16 MHz crystal, MCP3008, KA358, LM336Z, sockets,
small-signal parts, resistors, LEDs, wire-wrap supplies and Vector 8801-6.
The Analog Discovery 2 is test equipment only, not a product component.

## Purchase after cassette baseline captures

Do **not** release these for order until `ECS-TEST-001` through `ECS-TEST-003`
have established waveform amplitude/offset, bandwidth, and source/load
behaviour.  The listed quantities are per external-data device.

| Qty. | Item / provisional requirement | Decision still needed |
| ---: | --- | --- |
| 1 | MCP4921-I/P, 12-bit SPI DAC, 8-pin PDIP | Selected provisionally, but replay resolution/bandwidth must be proven first. |
| 1 | Playback output driver, ≥3 Vpp into 100 Ω with ≥20 mA peak margin | Choose only after measured ECS input load and accepted level window. |
| 1 | PCB-mount microSD socket plus 1 known-good microSD card | Socket mechanics and firmware/storage approach remain open. |
| 1 | 3.3 V regulator with its specified input/output capacitors | Must tolerate SD write-current bursts. |
| 1 set | Defined 5 V ↔ 3.3 V SPI level-shifting circuit | Must protect the SD card in both directions; an HCT125 is not automatically suitable. |
| 2 | 3.5 mm mono TS PCB/panel jacks | Device playback and capture connections. |
| 0–1 | 2.5 mm mono TS PCB/panel jack | `REMOTE` only; omit for revision one unless its polarity/current are measured. |
| 1 set | Analogue filter, coupling, bias, gain, attenuation and protection parts | Values and driver topology are measurement-gated. |
| 1 | Enclosure, cable strain relief and labels | Choose after connector and carrier layout are fixed. |

## Next bench build — M6x09-II interface SBC and mailbox carrier

This is the earliest non-ECS-facing digital build.  The active ICs are already
recorded as stocked; the following gaps need resolution before assembly.

| Qty. | Item / exact requirement | Applies to | Notes |
| ---: | --- | --- | --- |
| 1 | 12.000 MHz HC-49/US through-hole crystal, fundamental, parallel resonant, 18 pF load, ≤80 Ω ESR | Interface-development SBC `X1` | Example specified in the feasibility study: Citizen `HC-49/U-S12000000ABJB`, or electrically/mechanically equivalent.  Do not use the stocked 16 MHz crystal with HD63C09RP. |
| 2 | 18 pF C0G/NP0 ceramic capacitors, radial 0.1 in | SBC `C2`, `C3` | Stock must be inspected; source if absent. |
| 1 | 2×16, 2.54 mm expansion header | SBC `H2` | Mandatory; confirm board header orientation/style before ordering. |
| 1 | 1×6 FTDI-style, 2.54 mm header | SBC `P1` | Monitor/loader connection unless the design documents another connector. |
| 1 set | Reset switch, LED, 2.2 kΩ SIP network, 10 µF capacitor | SBC | These are not fully confirmed in inventory.  Preserve published values pending schematic check. |
| 1 | Mating 2×16 connector plus mechanical support | Mailbox-test carrier | Required; do not use loose jumper wire as a sustained expansion-bus connection. |
| 1 set | Test points and a logic-analyser header | Mailbox-test carrier | Expose enables, handshakes, reset and key clock signals. |
| 1 small lot | 100 nF ceramic capacitors | SBC/carrier | The SBC calls for 5; the carrier needs one per installed TTL IC.  A 25–50 lot covers initial builds with spares. |

The 6309 CPU, 62256 SRAM, 27C128 EPROM, MC68B50, 74LS74 divider,
74LS273 latch bank, 74LS245 transceivers, decode/control 74LS logic, 10 kΩ
pull-ups, LEDs, diode and 330 Ω resistor are documented as already in stock.

## Deferred design purchase — independent TTL video controller

This is a planning allowance only.  It should not be ordered as a full build
until the display mode, composite standard/encoder, power budget, board split,
and validated `video-composites` input revision are frozen.  It is independent
of the original ECS video path.

| Qty. | Item | Planning notes |
| ---: | --- | --- |
| 1 | M6x09-II performance SBC fitted with suitable HD6309 | Retain one separately proven recovery SBC. |
| 2 | 62256, 32K×8, 5 V asynchronous SRAM, 55 ns or faster | 16-bit VRAM low/high banks. |
| 1 | 14.31818 MHz, 5 V TTL oscillator | NTSC timing proposal; do not substitute a CPU crystal. |
| See note | 74LS00 ×3; 02 ×2; 04 ×3; 08 ×2; 32 ×2; 74 ×4; 138 ×2; 139 ×2; 153 ×2; 157 ×3; 163 ×8; 189 ×3; 245 ×8; 283 ×6; 374 ×12; 688 ×4 | Initial controller estimate; buy a small spare margin only after the schematic/netlist exists. |
| 80 | 100 nF ceramic decoupling capacitors | Initial controller allowance, one per IC. |
| 4 | Bulk 5 V capacitors | Value/rating follow board power design. |
| 1 set | 5 V regulator/power entry, fuse and reverse-polarity protection | Size from measured total current. |
| 1 | Validated `video-composites` board/revision | Must accept the chosen controller's baseband composite profile. |
| 1 | RCA female jack | Sole video-output connector. |
| 1 | 3.5 mm stereo female TRS jack | Separate dual-mono audio output. |
| 1 set | Composite encoder, DAC/resistor networks, analogue buffer, connector/test-point/header set | **TBD**: depends on NTSC/PAL and the chosen encoder topology. |

The optional direct-ECS-PSG and new AY-3-8917 branches are deliberately
excluded from purchasing: each requires its own measured/reversibility gate.

## Not a procurement item yet

- An ECS cartridge/CP1610-side connector, bus interposer, or direct bus
  attachment: pinout, timing, decode and ownership are not yet measured.
- A `REMOTE` cable or driver: build/connect only after the target recorder and
  its control polarity/current are characterised.
- A final external-data PCB: the current carrier is a wire-wrap prototype and
  the analogue requirements are not yet determined.
- Any FPGA, CPLD, GAL or programmable-logic part: these are out of scope.

## Recommended order sequence

1. Buy the baseline cabling/breakout supplies, small decoupling lot, protected
   card-power parts, ISP/reset parts, and the interface-SBC mechanical/timing
   gaps.
2. Capture and document the ECS cassette baseline; qualify the HP/Vector
   power path with the ECS disconnected.
3. Finalise and order the measurement-gated external-data analogue/storage
   parts.
4. Complete local 6309 and mailbox-carrier acceptance before any ECS-facing
   hardware work.
5. Freeze the video schematic and then turn its planning allowance into an
   orderable, reference-designator-level BoM.

## Source documents

- [External-data revision-zero BoM](../modules/external-data/BOM.md)
- [External-data workplan](../modules/external-data/WORKPLAN.md)
- [Cassette cable harness](../modules/external-data/CABLE-HARNESS.md)
- [M6x09-II parts feasibility](M6X09-II-THIRD-BOARD-FEASIBILITY.md)
- [TTL 16-bit video controller](TTL-16BIT-VIDEO-CONTROLLER.md)
