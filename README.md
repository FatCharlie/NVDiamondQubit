# NV Diamond Qubit Project

## How to use this doc

This is the build plan only; theory lives in a separate file. Each stage is a **milestone**. Each item with an ID (S1-04, D2) is **one ticket**: the bold line is the ticket title, the text under it is the description, and **Done when** is the acceptance criteria. Decision tickets (D-) block purchases, so close those first.

## Project at a glance

| Stage                 | Goal                          | What you have at the end                                                 | Rough cost                          | Status              |
| --------------------- | ----------------------------- | ------------------------------------------------------------------------ | ----------------------------------- | ------------------- |
| 1. Uncut Gem          | See the spin resonance        | Palm-sized board; OLED shows a dip near 2.87 GHz that reacts to a magnet | about £115                          | Ready to start      |
| 2. Cleaner instrument | Measure with it               | Bench setup; laptop shows clean, calibrated dip pairs                    | about $4,500 in the published build | Needs BOM           |
| 3. Pulsed control     | Control it as a qubit         | Rabi oscillation curve you produce on purpose                            | Not scoped                          | Scope after Stage 1 |
| 4. Research grade     | Single NV, few-qubit register | Reference only                                                           | Tens of thousands of dollars        | Out of scope        |

Every stage runs the same chain and upgrades one weak link: **green light in, microwaves sweep, red light out, electronics read the brightness.**

## Safety (applies to every stage)

- Treat the green LED like a laser for your eyes; never look into it or aim it at anyone.
- Avoid reflective surfaces near the light path.
- Disconnect power before touching the board; check for shorts before every power-up.
- Keep microwave antennas away from your face while running.
- Neodymium magnets pinch, chip, and wipe cards; keep them away from electronics you care about.
- Power from an isolated or protected USB supply, not a bare laptop port.
- The repo's SAFETY.md also says not to work alone and to keep one hand away from live equipment.

## Decisions (close before ordering)

**D1 · Diamond: powder now or bulk now**
Stage 1 is designed around cheap microdiamond powder (many tiny crystals, random orientations, smeared signal). A bulk plate (one crystal, fixed orientations, clean signal) costs much more and is the Stage 2 upgrade. Default: powder now, bulk in Stage 2.
**Done when:** choice recorded here.

**D2 · Diamond supplier**
Find a source for NV microdiamond suitable for ODMR. Sigma-Aldrich lists fluorescent nanodiamonds, but suitability at this concentration is unconfirmed. Check the repo's issues and the paper's acknowledgments for where the authors got theirs.
**Done when:** supplier, part number, price, and shipping to NM recorded.

**D3 · Light source: LED or laser**
The paper uses a green Superbrite LED (about 525 to 532 nm); the repo README mentions a laser module. Check what the V2 build guide specifies.
**Done when:** part number recorded and added to the BOM.

**D4 · RF hardware**
The paper names the ADF4351 and GALI-84+ parts, but the exact amplifier board and SMA cable/connectors come from the repo's hardware folder BOM.
**Done when:** exact amplifier board, SMA cable, and connectors recorded in the BOM.

**D5 · Power supply**
Pick an isolated or protected 5 V USB supply or battery pack.
**Done when:** part chosen.

## Stage 1: Uncut Gem

### What it is

A palm-sized circuit board. An ESP32 tells a microwave chip which frequency to make; an amplifier boosts it; a wire antenna radiates it at a blob of diamond powder in epoxy. A green LED makes the diamond glow red; a red filter passes only that glow to a photodiode; an op-amp turns the photodiode current into a voltage; the ESP32 reads it at each frequency and draws brightness vs frequency on a small OLED screen.

### What it looks like when done

- Board on the bench in a dimmed room, green LED lit, diamond glowing faintly red through red film.
- OLED: "Calibrating..." on boot, then a mostly flat trace with a shallow dip near the middle (2.87 GHz). With powder the dip is small, around a percent of total brightness or less.
- Magnet brought close: the dip widens and spreads. Magnet removed: it shrinks back.
- Light blocked: the magnet no longer changes anything.

### Parts (BOM)

Prices are the authors' approximate UK figures; add US shipping and import costs.

| Item | Part / spec | Approx. cost (GBP) | Notes |
| --- | --- | --- | --- |
| Microcontroller | ESP32 dev board | 10 | Runs sweep, ADC, display; Arduino IDE firmware |
| Microwave synthesizer | ADF4350/ADF4351 module | 15 to 25 | 35 MHz to 4.4 GHz, SPI-controlled |
| Microwave amplifier | Two GALI-84+ (about 16 dB each), 40 dB total | 10 | Exact board per D4 |
| Photodiode amp | BPW34 photodiode, TL082 dual op-amp, two 1 kOhm resistors | up to 5 | Use a DIP socket for the op-amp |
| Green light source | Superbrite LED or laser module | 2 to 5 | Per D3 |
| Red filter | Red filter gel | a few | Blocks green, passes red |
| Display | OLED screen | 5 |  |
| PCB | Gerbers from repo Prod_files | about 5 per unit | Zip the folder for JLCPCB or PCBWay |
| NV diamond | Microdiamond with NV centers | about 10 | Per D1, D2 |
| Antenna and mount | Enameled copper wire, epoxy | 10 to 15 | Enameled only; bare wire shorts |
| Power | Protected USB supply or battery pack | free to 5 | Per D5 |

**Tools (not from the guide):** soldering iron and solder, multimeter, flush cutters, short SMA cable, brass PCB standoffs, small magnet, red film or red-passing goggles, a way to dim the room.

### Tickets

**S1-01 · Read the repo build docs**
Read BuildGuide, hardware/DiamondMount, firmware, and science folders and the open issues in the Uncut Gem repo. Note anything that changes the steps below.
**Done when:** notes added to this doc; D3 and D4 can be answered.

**S1-02 · Order parts**
Order everything in the BOM once D1 to D5 are closed.
**Done when:** all parts received and checked against the BOM.

**S1-03 · Order PCBs**
Zip Prod_files and order from JLCPCB or PCBWay.
**Done when:** boards received.

**S1-04 · Build the diamond prism**
Embed the diamond in epoxy on the photodiode with red filter gel between them, following hardware/DiamondMount. Diamond sinks in most epoxies, so expect to layer it.
**Done when:** prism cured, photodiode leads intact, diamond sits in the light path.

**S1-05 · Build the transimpedance stage**
Solder the op-amp socket and TL082, then resistors and capacitors. Add C1 and C2 only for an integrator. Trim leads.
**Done when:** no solder bridges under magnification.

**S1-06 · Solder sockets**
Sockets for the ESP32, the OLED, and the 3x2 pin 5 V/GND block.
**Done when:** all seated and continuity-checked.

**S1-07 · Solder the ADF4351 socket**
The 5x2 socket goes on the back so the front stays flat.
**Done when:** socket fitted, no bridges.

**S1-08 · Fit standoffs**
Bolt in brass PCB standoffs before placing boards.
**Done when:** board sits level and stable.

**S1-09 · First power-up (ESP32 only, RF boards off)**
Check rails with a multimeter.
**Done when:** steady 5 V at the supply socket, 3.3 V at the ADF4351 socket, and 5 V / 2.5 V / 0 V at the R1-R2 divider. Any wrong reading: find the short before continuing.

**S1-10 · Flash firmware**
Compile and upload the Arduino IDE firmware from the repo's firmware folder.
**Done when:** upload succeeds and serial output appears.

**S1-11 · Mount RF boards**
ADF4351 on the back, amplifier on the front, joined with a short SMA cable.
**Done when:** both mounted, cable seated.

**S1-12 · RF lock check**
Attach a short antenna, power up.
**Done when:** the ADF4351 LOCK LED lights.

**S1-13 · Connect the light source**
Wire the LED or laser to the 5 V/GND socket, observing polarity.
**Done when:** it lights at power-up.

**S1-14 · Connect the photodiode**
Plug in the connector from the diamond prism.
**Done when:** seated, polarity correct.

**S1-15 · Connect the antenna**
To the amplifier output by SMA or a soldered wire, positioned right beside the diamond.
**Done when:** joint inspected, no shorts.

**S1-16 · Final short check and boot**
**Done when:** OLED shows "Calibrating..." on power-up.

**S1-17 · Align light and diamond**
Hold the light source on stiff enameled wire; adjust while watching through red film.
**Done when:** the diamond glows bright red.

**S1-18 · First sweep**
Dim the room, power up, look at the lower half of the OLED. The sweep runs 2.614 to 3.126 GHz in 4 MHz steps (128 points). If no pattern, open the serial port and confirm data arrives.
**Done when:** a dip near 2.87 GHz is visible.

**S1-19 · Magnet test**
Bring a small magnet toward the diamond slowly.
**Done when:** the dip visibly widens or spreads and returns when the magnet is removed.

**S1-20 · Control test**
Block the light to the diamond with a small paper cutout and repeat S1-19.
**Done when:** the magnet response disappears (proves the signal is fluorescence, not electrical pickup).

**S1-21 · Log sweeps to a laptop (suggested addition)**
Capture serial output to CSV with a timestamp and the room temperature, and plot it. The published build has no calibration procedure or expected signal levels, so your logs become the baseline for Stage 2.
**Done when:** a script in the repo saves and plots a sweep with and without a magnet.

### Troubleshooting

- No LOCK LED: check the 3.3 V rail at the ADF4351 socket and the SPI wiring.
- No display pattern: check serial output, the photodiode connector, and the op-amp socket.
- No red glow: re-aim the light source; check polarity.
- No magnet response: confirm the sweep covers 2.87 GHz, then the antenna joint, then diamond alignment. Check the repo's open issues.

## Stage 2: Cleaner instrument

### What it is

The same chain, upgraded at every weak point: a bulk diamond plate instead of powder (clean, separate dips), better light collection, a lock-in amplifier that pulls the tiny signal out of noise by modulating the microwaves and listening only at that rate, and a coil that makes a known magnetic field for calibration.

### What it looks like when done

- A bench setup: diamond plate in a mount with a coil around it, light source, photodiode, Red Pitaya board, laptop.
- On the laptop: sharp dips in clean pairs (up to four pairs). Rotating the magnet slides them around.
- A plot of dip splitting vs coil current that is a straight line.
- Sensitivity near 1 µT/√Hz in the published version.

### Tickets

**S2-01 · Extract the Stage 2 BOM**
Read the LED-based teaching magnetometer paper and list every part, supplier, and price.
**Done when:** a Stage 2 BOM table exists in this doc with a total.

**S2-02 · Buy the bulk NV diamond plate**
**Done when:** plate received with its spec sheet (nitrogen and NV concentration, orientation).

**S2-03 · Build the optics**
Light focusing and red collection per the paper.
**Done when:** red signal at the photodiode is clearly higher than with the Stage 1 prism.

**S2-04 · Set up the lock-in (Red Pitaya)**
Modulate the microwaves at a few kHz and demodulate the photodiode signal at that rate.
**Done when:** a sweep shows the resonance with visibly lower noise than an unmodulated sweep.

**S2-05 · Build and characterize the calibration coil**
**Done when:** field per amp is known (measured or calculated from turns and radius).

**S2-06 · Resolved spectrum**
**Done when:** separate dip pairs are visible and move as the magnet angle changes.

**S2-07 · Calibration line**
Step coil current, record dip positions, plot splitting vs current.
**Done when:** the plot is a straight line and its slope is recorded.

**S2-08 · Sensitivity measurement**
**Done when:** a sensitivity figure in µT/√Hz is measured and compared with the paper's.

**S2-09 · Temperature readout**
Track the midpoint of a dip pair while temperature changes.
**Done when:** midpoint drift vs temperature is logged over at least a 5 K swing.

## Stage 3: Pulsed control

### What it is

Stages 1 and 2 run everything continuously. Stage 3 times everything: a green pulse to reset, a microwave pulse of chosen length to rotate the spin, a green pulse to read. A fixed bias magnet separates the two dips so you drive only one of them, which gives a clean two-level qubit. The ESP32 is too slow for this, so it needs a fast microwave switch, a nanosecond pulse sequencer, and light that can be switched quickly.

### What it looks like when done

- Same bench plus switch and timing hardware.
- On the laptop: brightness vs microwave pulse length is a wavy curve that oscillates and slowly fades (Rabi oscillation). Follow-ups produce decaying fringes (Ramsey) and a longer-lived echo signal.

### Tickets

**S3-01 · Scope the pulsed hardware**
Pick and price a fast microwave switch, a pulse sequencer (check whether the Red Pitaya can do it), gateable light, and a bias magnet mount.
**Done when:** Stage 3 BOM with costs added here.

**S3-02 · Isolate one transition**
Position the bias magnet so the two dips are well separated.
**Done when:** one dip can be driven with no visible effect on the other.

**S3-03 · Build the pulse sequence**
Green reset, microwave pulse of length t, green readout; repeat and average.
**Done when:** the sequence runs and timing is verified on a scope.

**S3-04 · Rabi oscillation**
Sweep pulse length t.
**Done when:** brightness oscillates with t and the π-pulse length is recorded.

**S3-05 · Ramsey fringes**
Two half-flip pulses separated by a wait.
**Done when:** decaying fringes recorded and the decay time noted.

**S3-06 · Spin echo**
Add a full-flip pulse in the middle of the wait.
**Done when:** decay time measured and longer than in S3-05.

**S3-07 · T1 measurement**
Reset, wait, read.
**Done when:** relaxation curve recorded.

## Stage 4: Research grade (out of scope)

A confocal microscope focuses light onto a single NV center, single-photon detectors count its emission, and nearby carbon-13 nuclear spins serve as extra qubits for a few-qubit register. Kept here for reference; no tickets.

## Links

- [Uncut Gem repository](https://github.com/QuantumVillage/UncutGem) (AGPL-3.0)
- [Uncut Gem paper (arXiv)](https://arxiv.org/html/2509.18329v1)
- [LED-based teaching magnetometer paper (arXiv)](https://arxiv.org/html/2512.03106v2)
