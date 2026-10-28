# NV Diamond Qubit Project

Oct 26, 2026 · @Charlie

## *WARNING: AI Generated, I will start editing this as I put it together*

## How to use this doc

This is the build plan only; theory lives in a separate file. Each stage is a **milestone**. Each item with an ID (S1-04, D2) is **one ticket**: the bold line is the ticket title, the text under it is the description, and **Done when** is the acceptance criteria. Decision tickets (D-) block purchases, so close those first.

## Project at a glance

| Stage | Goal | What you have at the end | Rough cost | Status |
| --- | --- | --- | --- | --- |
| 1. Uncut Gem on Pi | See the spin resonance | Pi 4 plus RF modules; a plot shows a dip near 2.87 GHz that reacts to a magnet | about $300 to $360 including the diamond | Ready to start |
| 2. Cleaner instrument | Measure with it | Bench setup; laptop shows clean, calibrated dip pairs | about $4,500 in the published build | Needs BOM |
| 3. Pulsed control | Control it as a qubit | Rabi oscillation curve you produce on purpose | Not scoped | Scope after Stage 1 |
| 4. Research grade | Single NV, few-qubit register | Reference only | Tens of thousands of dollars | Out of scope |

Every stage runs the same chain and upgrades one weak link: **green light in, microwaves sweep, red light out, electronics read the brightness.**

## Safety (applies to every stage)

- Treat the green LED like a laser for your eyes; never look into it or aim it at anyone.
- Avoid reflective surfaces near the light path.
- Disconnect power before touching the board; check for shorts before every power-up.
- Keep microwave antennas away from your face while running.
- Neodymium magnets pinch, chip, and wipe cards; keep them away from electronics you care about.
- Power the Pi from its official supply; power down before rewiring anything on the header.
- The repo's SAFETY.md also says not to work alone and to keep one hand away from live equipment.

## Decisions (close before ordering)

**D1 · Diamond: powder now or bulk now** Stage 1 is designed around cheap microdiamond powder (many tiny crystals, random orientations, smeared signal). A bulk plate (one crystal, fixed orientations, clean signal) costs much more and is the Stage 2 upgrade. Decided Oct 26, 2026: microdiamond powder now, bulk plate in Stage 2. Revised Oct 28, 2026: a single \~800 µm NV crystal instead of powder, on Adámas's recommendation. One crystal has only four NV axis directions, so a magnet gives clean, separate dip pairs instead of one smeared dip; a polished plate stays the Stage 2 upgrade. Status: closed.

**D2 · Diamond supplier** Decided Oct 27, 2026: Adámas Nanotechnologies (Raleigh, NC), Encapsulated 800 µm NV microdiamond, **SKU ENMDNV800, $200**: one \~800 µm crystal sealed in a clear polymer block about 12.5 × 12.5 × 7 mm, shipped with its own ODMR spectrum. Recommended by Marco Torelli at Adámas for this kind of build; it replaces the powder grade FMDNVLN1-10mg. Reply drafted in Gmail asking about lead time, shipping, polymer transparency, crystal orientation and the conditions behind their ODMR data (not sent). Element Six sells bulk plates only, so it moves to Stage 2. **Done when:** lead time and shipping to NM recorded and the order placed.

**D3 · Light source: LED or laser** Decided Oct 27, 2026: green LED, **Cree C503B-GCN-CY0C0791** (5 mm, 527 nm, 30° beam, 3.2 V typical / 4.0 V max at 20 mA, 30 mA max) with a **100 Ω ¼ W** series resistor: I = (5 V − 3.2 V) / 100 Ω ≈ 18 mA. Wire it always on: anode to Pi header pin 4 (5 V), cathode through the resistor to GND. Trade-off accepted: less light on the diamond than a laser, so a smaller dip. **Status:** closed.

**D4 · Platform and RF hardware** Decided Oct 27, 2026: your Raspberry Pi 4 Model B replaces the Uncut Gem's ESP32 and custom board. The RF side reuses the module pair the Uncut Gem V1 used.

- **Synthesizer:** "35M-4.4GHz ADF4351 development board": the bare board with a 10-pin control header, 25 MHz onboard reference, SMA outputs, 5 V in at about 110 to 145 mA. Buy the bare board, not versions with an STM32 controller or touchscreen. Output is close to 0 dBm over most of its range. Its logic is 3.3 V, so it wires straight to the Pi. About $25 to $45.
- **Amplifier:** 30 to 4000 MHz, 40 dB broadband amplifier board, 5 V at 135 mA, about +20 dBm at 1 dB compression; the same type V1 used. About $6 to $15. Gain at 2.87 GHz isn't published; expect less than 40 dB near the top of its range, which is still plenty.
- **Antenna:** an SMA male pigtail with a small enameled-wire loop soldered on, placed at the diamond.
- **V2 custom board:** shelved. The JLCPCB files stay in the vault at `qc/nv-diamond/jlcpcb/`.

**Status:** closed.

**D5 · Power supply** Decided Oct 27, 2026: the official Raspberry Pi 4 USB-C supply (5.1 V, 3 A) powers everything. The ADF4351 board, amp board and LED draw about 300 mA from the Pi's 5 V pins (2 and 4); the ADS1115 and op-amp take a few mA from 3.3 V (pin 1). **Status:** closed.

**D6 · Photodiode amplifier and ADC** Decided Oct 27, 2026. The Pi has no analog inputs and its pins are 3.3 V only, so:

- **ADC: ADS1115 module** (16-bit, I2C at address 0x48, up to 860 samples/s, ranges down to ±0.256 V), Adafruit 1085 or a generic module, powered from 3.3 V.
- **Amplifier: one MCP6002 stage on perfboard** (DIP-8, rail-to-rail, 1.8 to 6 V supply, 1 pA input bias), powered from 3.3 V. The photodiode runs at zero bias with 10 MΩ ∥ 10 pF feedback: V = I × 10 MΩ, so 100 nA gives 1 V; bandwidth 1 / (2π · 10 MΩ · 10 pF) ≈ 1.6 kHz. The output can't exceed 3.3 V, which is exactly the ADC's safe input range. The original two-stage TL082 circuit needed a 5 V rail and a 2.5 V midpoint and could overdrive a 3.3 V input; the ADS1115's gain settings replace its second stage.

**Status:** closed.

**D7 · Software** Decided Oct 27, 2026: Python on the Pi, in the vault at `qc/nv-diamond/pi/`: `adf4351.py` (register math and SPI driver; register words match the open-source adf435x library at 5,049 test frequencies), `ads1115.py`, and `sweep.py` (sweeps, averaging, CSV, PNG plot; `--sim` runs without hardware). Each point is read with microwaves on and off and reported as contrast C = (V\_on − V\_off) / V\_off, which cancels LED and temperature drift. **Status:** closed; first real-hardware test is S1-06.

## Stage 1: Uncut Gem on a Raspberry Pi

### What it is

Your Pi 4 runs a Python script that tells an ADF4351 synthesizer board which microwave frequency to make; an amplifier board boosts it; a wire loop antenna radiates it at a single diamond crystal sealed in a polymer block. A green LED makes the diamond glow red; a red filter passes only that glow to a photodiode; a one-stage op-amp turns the photodiode current into a voltage; an ADS1115 converts it for the Pi, which logs brightness vs frequency and plots it.

### What it looks like when done

- On the bench in a dimmed room: the Pi, two small RF boards joined by an SMA cable, and a perfboard with the op-amp; green LED lit, diamond glowing faintly red through red film.
- On the monitor: a plot of contrast vs frequency, flat with a dip near 2870 MHz. Compare its depth with the ODMR spectrum Adámas ships with the crystal.
- Magnet brought close: the dip splits into up to four separate pairs, one per NV axis, each at 2870 ± 28 · B∥ MHz (B∥ is the field along that axis, in mT). Magnet removed: it comes back.
- Light blocked: the dip disappears.

### Wiring

Pi header pins are physical pin numbers. Match the ADF4351 board by its silkscreen labels; pin order differs between board versions.

| From | To | Notes |
| --- | --- | --- |
| Pi pin 2 (5 V) | ADF4351 board 5 V in, amp board 5 V | About 280 mA together |
| Pi pin 4 (5 V) | LED anode | LED cathode, then 100 Ω, then GND |
| Pi pin 6 (GND) | ADF4351 GND, amp GND, ADS1115 GND, op-amp pin 4 | One common ground |
| Pi pin 19 (GPIO10, MOSI) | ADF4351 DATA |  |
| Pi pin 23 (GPIO11, SCLK) | ADF4351 CLK |  |
| Pi pin 24 (GPIO8, CE0) | ADF4351 LE | Add 10 kΩ from LE to GND |
| Pi pin 22 (GPIO25) | ADF4351 CE | Chip enable, driven high by the script |
| Pi pin 18 (GPIO24) | ADF4351 LD | Lock detect, read by the script |
| Pi pin 1 (3.3 V) | ADS1115 VDD, op-amp pin 8 | Leave the ADF4351 board's own 3.3 V pin unconnected |
| Pi pin 3 (GPIO2, SDA) | ADS1115 SDA |  |
| Pi pin 5 (GPIO3, SCL) | ADS1115 SCL | ADS1115 ADDR to GND (address 0x48) |
| ADF4351 RF out (SMA) | Amp IN (SMA) | SMA male-male cable |
| Amp OUT (SMA) | Antenna pigtail | Loop right beside the diamond |

**Photodiode amplifier (MCP6002 pins: 1 OUTA, 2 INA−, 3 INA+, 4 VSS, 5 INB+, 6 INB−, 7 OUTB, 8 VDD):** pin 8 to 3.3 V and pin 4 to GND with 100 nF across them; pin 3 to GND; BPW34 cathode to pin 2, anode to GND; 10 MΩ and 10 pF in parallel from pin 1 to pin 2; pin 1 to ADS1115 A0; unused half: pin 5 to GND, pin 6 tied to pin 7.

### Parts (BOM)

Prices are approximate USD as of Oct 27, 2026. Total about $300 to $360 including the $200 diamond, before shipping.

| Item | Part number | Qty | Source | Approx. cost | Notes |
| --- | --- | --- | --- | --- | --- |
| Raspberry Pi 4 Model B and official supply | owned | 1 |  | $0 | Per D4, D5 |
| Synthesizer | 35M-4.4GHz ADF4351 development board (bare, 25 MHz reference) | 1 | AliExpress or similar | about $25 to $45 | Per D4 |
| RF amplifier | 30 to 4000 MHz, 40 dB board, 5 V | 1 | AliExpress or similar | about $6 to $15 | Per D4 |
| SMA cable | SMA male to SMA male, 10 to 15 cm | 1 | any | about $5 | Synth to amp |
| Antenna pigtail | SMA male to bare wire (RG316) | 1 | any | about $5 | Solder the loop to it |
| ADC | ADS1115 module (Adafruit 1085 or generic) | 1 | Adafruit, DigiKey, Amazon | about $5 to $15 | Per D6 |
| Op-amp | Microchip MCP6002-I/P (DIP-8) | 2 | DigiKey | under $1 each | Per D6 |
| Small parts | 10 MΩ, 1 MΩ and 10 kΩ ¼ W resistors; 10 pF and 100 nF ceramic capacitors | a few each | any | about $3 | 1 MΩ is the fallback if 10 MΩ saturates |
| Build supplies | Perfboard, DIP-8 socket, female-female jumper wires | 1 set | any | about $10 |  |
| NV diamond | Adámas ENMDNV800 (800 µm crystal in a 12.5 × 12.5 × 7 mm polymer block) | 1 | Adámas | $200 plus shipping | Per D2 |
| Green LED | Cree C503B-GCN-CY0C0791 | 3 | DigiKey | about $1 each | Per D3 |
| LED resistor | 100 Ω, ¼ W | 3 | any | under $1 | Per D3 |
| Photodiode | Vishay BPW34 | 2 | DigiKey or Mouser | about $1 to $2 each | Two wires to the perfboard |
| Red filter | LEE 106 Primary Red sheet | 1 | LEE Filters USA | $12 | Blocks green, passes red |
| Antenna wire | Enameled copper magnet wire, about 0.5 mm (24 AWG) | 1 spool | any | about $8 | Enameled only |
| Epoxy | Clear 2-part epoxy | 1 | any | about $8 | Optional; holds the block, gel and photodiode together |

**Tools:** soldering iron and solder, multimeter, flush cutters, tweezers, small magnet, red film or red-passing goggles, a way to dim the room, and a monitor or SSH session for the Pi.

### Tickets

**S1-01 · Set up the Pi** Enable SPI and I2C, install the packages listed in `qc/nv-diamond/pi/README.md`, and copy the `pi/` folder to the Pi. Done when: `python3 sweep.py --sim --plot` saves a PNG with a dip at 2870 MHz.

**S1-02 · Order parts** Order everything in the parts list once D1 to D7 are closed. Done when: all parts received and checked against the list.

**S1-03 · Build the photodiode amplifier** Build the MCP6002 stage on perfboard per the wiring section. Done when: with the photodiode covered the output reads near 0 V, and room light raises it.

**S1-04 · Mount the diamond** Stand the polymer block on one 12.5 mm face, with a square of red gel between it and the photodiode, the photodiode centered under the crystal. Aim the LED at the crystal through a side face so little green reaches the photodiode directly. A dab of epoxy or tape holds it all together. Done when: block, gel and photodiode are fixed, and the crystal sits in the LED's beam.

**S1-05 · Wire the ADS1115** Done when: `i2cdetect -y 1` shows 0x48 and the reading changes when you shade the photodiode.

**S1-06 · Wire the ADF4351 board** Connect 5 V, GND, DATA, CLK, LE (with 10 kΩ to GND), CE and LD per the wiring table. Done when: the lock-detect pin reads high after setting 2614, 2870 and 3126 MHz from Python.

**S1-07 · Add the amplifier and antenna** Join synth to amp with the SMA cable, power the amp from Pi pin 2, and fit the antenna pigtail with a small loop pressed against the block face nearest the crystal. The crystal sits a few millimeters inside the block, so keep the loop flat on the polymer. Done when: no shorts; the loop is centered on the crystal.

**S1-08 · Connect the light source** LED anode to Pi pin 4, cathode through 100 Ω to GND (always on, per D3). Done when: it lights at power-up.

**S1-09 · Align light and diamond** Hold the LED on stiff enameled wire; adjust while watching through red film. Done when: the diamond glows red and the ADC reading sits well above the dark level.

**S1-10 · First sweep** Dim the room and run `python3 sweep.py --sweeps 10 --plot` (2614 to 3126 MHz in 4 MHz steps). Done when: the plot shows a dip near 2870 MHz.

**S1-11 · Magnet test** Bring a small magnet toward the diamond slowly and sweep again. Done when: the dip splits into separate pairs that move as you turn the magnet, and the single dip returns when the magnet is removed.

**S1-12 · Control test** Block the light to the diamond with a small paper cutout and sweep again. Done when: the dip disappears (proves the signal is fluorescence, not electrical pickup).

**S1-13 · Baseline log** Run sweeps with `--temp` and `--note`, with and without a magnet, and commit the CSVs and plots to the repo. The published build has no expected signal levels, so these become the baseline for Stage 2. Done when: a labeled set of sweeps is saved.

### Troubleshooting

- Nothing at 0x48: check SDA/SCL, 3.3 V, ADDR to GND, and that I2C is enabled.
- Script warns the PLL is unlocked: check CE is high, the LE pull-down, 5 V at the synth board, and that DATA and CLK aren't swapped; confirm it's the bare board with a 25 MHz reference.
- Reading stuck near 3.3 V: the photodiode amp is saturated; swap 10 MΩ for 1 MΩ or shade stray light.
- Reading stuck near 0 V: photodiode reversed or not lit; the cathode goes to op-amp pin 2.
- No red glow: re-aim the LED; check its polarity and the resistor.
- No dip: check the amp board has power, the SMA joints are tight and the loop is close; narrow the sweep (`--start 2820 --stop 2920 --step 0.5`) and add sweeps.

## Stage 2: Cleaner instrument

### What it is

The same chain, upgraded at every weak point: a bulk diamond plate instead of powder (clean, separate dips), better light collection, a lock-in amplifier that pulls the tiny signal out of noise by modulating the microwaves and listening only at that rate, and a coil that makes a known magnetic field for calibration.

### What it looks like when done

- A bench setup: diamond plate in a mount with a coil around it, light source, photodiode, Red Pitaya board, laptop.
- On the laptop: sharp dips in clean pairs (up to four pairs). Rotating the magnet slides them around.
- A plot of dip splitting vs coil current that is a straight line.
- Sensitivity near 1 µT/√Hz in the published version.

### Tickets

**S2-01 · Extract the Stage 2 BOM** Read the LED-based teaching magnetometer paper and list every part, supplier, and price. Done when: a Stage 2 BOM table exists in this doc with a total.

**S2-02 · Buy the bulk NV diamond plate** Done when: plate received with its spec sheet (nitrogen and NV concentration, orientation).

**S2-03 · Build the optics** Light focusing and red collection per the paper. Done when: red signal at the photodiode is clearly higher than with the Stage 1 mount.

**S2-04 · Set up the lock-in (Red Pitaya)** Modulate the microwaves at a few kHz and demodulate the photodiode signal at that rate. Done when: a sweep shows the resonance with visibly lower noise than an unmodulated sweep.

**S2-05 · Build and characterize the calibration coil** Done when: field per amp is known (measured or calculated from turns and radius).

**S2-06 · Resolved spectrum** Done when: separate dip pairs are visible and move as the magnet angle changes.

**S2-07 · Calibration line** Step coil current, record dip positions, plot splitting vs current. Done when: the plot is a straight line and its slope is recorded.

**S2-08 · Sensitivity measurement** Done when: a sensitivity figure in µT/√Hz is measured and compared with the paper's.

**S2-09 · Temperature readout** Track the midpoint of a dip pair while temperature changes. Done when: midpoint drift vs temperature is logged over at least a 5 K swing.

## Stage 3: Pulsed control

### What it is

Stages 1 and 2 run everything continuously. Stage 3 times everything: a green pulse to reset, a microwave pulse of chosen length to rotate the spin, a green pulse to read. A fixed bias magnet separates the two dips so you drive only one of them, which gives a clean two-level qubit. The Pi and ADS1115 are too slow for this, so it needs a fast microwave switch, a nanosecond pulse sequencer, and light that can be switched quickly.

### What it looks like when done

- Same bench plus switch and timing hardware.
- On the laptop: brightness vs microwave pulse length is a wavy curve that oscillates and slowly fades (Rabi oscillation). Follow-ups produce decaying fringes (Ramsey) and a longer-lived echo signal.

### Tickets

**S3-01 · Scope the pulsed hardware** Pick and price a fast microwave switch, a pulse sequencer (check whether the Red Pitaya can do it), gateable light, and a bias magnet mount. Done when: Stage 3 BOM with costs added here.

**S3-02 · Isolate one transition** Position the bias magnet so the two dips are well separated. Done when: one dip can be driven with no visible effect on the other.

**S3-03 · Build the pulse sequence** Green reset, microwave pulse of length t, green readout; repeat and average. Done when: the sequence runs and timing is verified on a scope.

**S3-04 · Rabi oscillation** Sweep pulse length t. Done when: brightness oscillates with t and the π-pulse length is recorded.

**S3-05 · Ramsey fringes** Two half-flip pulses separated by a wait. Done when: decaying fringes recorded and the decay time noted.

**S3-06 · Spin echo** Add a full-flip pulse in the middle of the wait. Done when: decay time measured and longer than in S3-05.

**S3-07 · T1 measurement** Reset, wait, read. Done when: relaxation curve recorded.

## Stage 4: Research grade (out of scope)

A confocal microscope focuses light onto a single NV center, single-photon detectors count its emission, and nearby carbon-13 nuclear spins serve as extra qubits for a few-qubit register. Kept here for reference; no tickets.

## Links

- [Uncut Gem repository](https://github.com/QuantumVillage/UncutGem) (AGPL-3.0)
- [Uncut Gem paper (arXiv)](https://arxiv.org/html/2509.18329v1)
- [LED-based teaching magnetometer paper (arXiv)](https://arxiv.org/html/2512.03106v2)
