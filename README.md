NV Diamond Qubit Project
Oct 26, 2026 · @Charlie
How to use this doc
This is the build plan only; theory lives in a separate file. Each stage is a milestone. Each item with an ID (S1-04, D2) is one ticket: the bold line is the ticket title, the text under it is the description, and Done when is the acceptance criteria. Decision tickets (D-) block purchases, so close those first.
Project at a glance
Stage
Goal
What you have at the end
Rough cost
Status
1. Uncut Gem
See the spin resonance
Palm-sized board; OLED shows a dip near 2.87 GHz that reacts to a magnet
about $175 to $250, plus diamond (quote pending)
Ready to start
2. Cleaner instrument
Measure with it
Bench setup; laptop shows clean, calibrated dip pairs
about $4,500 in the published build
Needs BOM
3. Pulsed control
Control it as a qubit
Rabi oscillation curve you produce on purpose
Not scoped
Scope after Stage 1
4. Research grade
Single NV, few-qubit register
Reference only
Tens of thousands of dollars
Out of scope
Every stage runs the same chain and upgrades one weak link: green light in, microwaves sweep, red light out, electronics read the brightness.
Safety (applies to every stage)
• Treat the green LED like a laser for your eyes; never look into it or aim it at anyone.
• Avoid reflective surfaces near the light path.
• Disconnect power before touching the board; check for shorts before every power-up.
• Keep microwave antennas away from your face while running.
• Neodymium magnets pinch, chip, and wipe cards; keep them away from electronics you care about.
• Power from an isolated or protected USB supply, not a bare laptop port.
• The repo's SAFETY.md also says not to work alone and to keep one hand away from live equipment.
Decisions (close before ordering)
D1 · Diamond: powder now or bulk now Stage 1 is designed around cheap microdiamond powder (many tiny crystals, random orientations, smeared signal). A bulk plate (one crystal, fixed orientations, clean signal) costs much more and is the Stage 2 upgrade. Decided Oct 26, 2026: microdiamond powder now, bulk plate in Stage 2. Status: closed.
D2 · Diamond supplier Decided Oct 27, 2026: Adámas Nanotechnologies (Raleigh, NC), Low Nitrogen Microdiamond with NV centers, SKU FMDNVLN1-10mg (150 to 200 µm, 20 to 30 ppm N). Of their three low-N grades it has the highest ODMR contrast (10.2%, measured by Adámas under green LED excitation, which matches this build); FMDNVLN2 and FMDNVLN3 trade contrast for narrower lines, which matters in Stage 2, not here. No list price online; a UK reseller lists a similar 10 mg grade near £373. Quote request drafted in Gmail to info@adamasnano.com (not sent). Element Six sells bulk plates only, so it moves to Stage 2. Done when: price, lead time and shipping to NM recorded.
D3 · Light source: LED or laser Decided Oct 27, 2026: green LED, Cree C503B-GCN-CY0C0791 (5 mm, 527 nm, 30° beam, 3.2 V typical / 4.0 V max at 20 mA, 30 mA max) with a 100 Ω ¼ W series resistor: I = (5 V − 3.2 V) / 100 Ω ≈ 18 mA. Wire it always on: anode to header 5V_SUPP1 pin 4 (5 V), cathode through the resistor to pin 5 (GND). That skips Q4, which has no base resistor in the V2 schematic, and the stock firmware never drives GPIO16 (LASER_CTRL) anyway. Trade-off accepted: less light on the diamond than a laser, so a smaller dip. Status: closed.
D4 · RF hardware Decided Oct 27, 2026. On V2 the synthesizer (ADF4351) and both amplifier stages are on the board, so there is no separate RF board or SMA cable to buy.
• Amplifier U2/U3: NXP MMG3014NT1, as in the authors' newer production BOM (ProdFiles/Uncut Gem3.csv) and the schematic. The GALI-84 in the older BOM CSV needs 5.4 to 6.2 V at the device; this board feeds the amps 5 V through a diode (D2/D3), about 4.6 V, so a GALI-84 would be under-biased.
• MMG3014NT1 is end-of-life (last-time buy passed June 2026). LCSC showed 19 in stock (C2151399): buy 4 to 6 now. Fallback: Qorvo SBB5089Z (LCSC C19631614, 964 in stock), same SOT-89 pinout, runs straight from 5 V; with it, fit 0 Ω links instead of D2/D3.
• D2/D3: Comchip CDBU0530 (0603, 0.5 A Schottky). Each amp draws about 135 mA, too much for a 1N4148-class diode.
• D8 (USB 5 V series diode, SOD-123F): B5819W SL, 1 A Schottky (LCSC C8598). The schematic calls it a zener; a zener would be wrong there.
• Antenna: J4 (SMA) is not fitted in the authors' BOM; solder the enameled wire loop straight to the J4 pads.
Status: closed.
D5 · Power supply Decided Oct 27, 2026: any USB-A power bank (2 A or more) plus a short USB-A to USB-C data cable. Expected draw is roughly 0.4 to 0.5 A (two amps at about 135 mA each, the ADF4351, the ESP32), so for laptop logging use a USB 3 port or a powered hub, not a 500 mA USB 2 port. V2 USB-C quirk (from the PCB file): only one side of the connector is wired (CC2 and the second D+/D− pair are unconnected). USB data works in one plug orientation only, and a USB-C to C charger may power it in one orientation only. If the board doesn't power up or no serial port appears, flip the plug; an A-to-C cable always delivers 5 V. Status: closed.
D6 · V2 assembly route Decided Oct 27, 2026: JLCPCB PCB plus top-side SMT assembly. V2 uses 0.5 mm-pitch QFNs (ADF4351, CP2102N), which are impractical to hand-solder; all SMD parts sit on the top side. Upload files: the BOM and CPL are in the vault at qc/nv-diamond/jlcpcb/: UncutGem-V2-BOM-JLCPCB.csv and UncutGem-V2-CPL-JLCPCB.csv (90 placements, built from the PCB file, not the stale BOM CSV); the Gerbers zip is the repo's hardware/PCB/ProdFiles folder zipped, sent in chat on Oct 27. Board: 2-layer, 1.6 mm, 82.4 × 82.4 mm octagon. Left out on purpose: J1 to J4 (unused SMAs) and R3/R4 (schematic note: do not populate). Hand-solder the through-hole parts (5V_SUPP1, OLED socket, photodiode socket). In JLCPCB's preview, check the rotation of U1, U2/U3, U6, U8, Q4 to Q6 and P1 before paying. My rough estimate for 2 assembled boards: $120 to $180 before shipping and import duties; the ADF4351 alone is about $17 per board. Done when: JLCPCB quote saved and order placed.
Stage 1: Uncut Gem V2
What it is
One palm-sized board (V2, Rev 0.2), USB-C powered and factory-assembled. The on-board ESP32 tells the on-board ADF4351 microwave chip which frequency to make; two on-board amplifiers boost it; a wire loop antenna soldered to the board radiates it at a few grains of diamond in epoxy. A green LED makes the diamond glow red; a red filter passes only that glow to a photodiode; two op-amp stages turn the photodiode current into a voltage; the ESP32 reads it at each frequency and draws brightness vs frequency on a 1.3" OLED.
What it looks like when done
• Board on the bench in a dimmed room, green LED lit, diamond glowing faintly red through red film.
• OLED: "Calibrating..." on boot, then a mostly flat trace with a shallow dip near the middle (2.87 GHz). With powder the dip is small, around a percent of total brightness or less.
• Magnet brought close: the dip widens and spreads. Magnet removed: it shrinks back.
• Light blocked: the magnet no longer changes anything.
Parts (BOM)
V2 purchase list. Prices are approximate USD from the listed source as of Oct 27, 2026.
Item
Part number
Qty
Source
Approx. cost
Notes
Assembled V2 board
Uncut Gem V2 Rev 0.2, JLCPCB PCBA
2
JLCPCB
$120 to $180 for 2 (estimate)
Per D6; files in vault
Spare amplifiers
NXP MMG3014NT1 (LCSC C2151399)
2 to 4
LCSC
about $4 each
End-of-life; per D4
NV diamond
Adámas FMDNVLN1-10mg
1
Adámas (quote)
pending
Per D2
Green LED
Cree C503B-GCN-CY0C0791
3
DigiKey
about $1 each
Per D3
LED resistor
100 Ω, ¼ W, through-hole
3
any
under $1
Per D3
Photodiode
Vishay BPW34
2
DigiKey or Mouser
about $1 to $2 each
Plugs into the PD1 socket; D4 on the board is not fitted
Red filter
LEE 106 Primary Red, 21 × 48 in sheet
1
LEE Filters USA
$12
Blocks green, passes red
OLED
1.3" 128×64 I2C, SH1106 driver, pins VCC-GND-SCL-SDA, 3 to 5 V
1 to 2
any
about $6 to $10
Firmware uses Adafruit SH110X (SH1106), not SSD1306. Many modules are wired GND-VCC, which would reverse power on this board
Headers
2×3 male header (2.54 mm), 1×4 and 1×3 female sockets
1 each
any
about $3
Hand-solder
Antenna wire
Enameled copper magnet wire, about 0.5 mm (24 AWG)
1 spool
any
about $8
Enameled only
Epoxy
Clear 2-part epoxy
1
any
about $8
For the diamond prism
Power
USB-A power bank (2 A or more) plus USB-A to USB-C data cable
1
any
$0 to $25
Per D5
Tools (not from the guide): soldering iron and solder, multimeter, flush cutters, tweezers, small magnet, red film or red-passing goggles, a way to dim the room. No hot air or stencil needed if JLCPCB assembles the board.
Tickets
S1-01 · Read the repo build docs Read BuildGuide, hardware/DiamondMount, firmware, and science folders and the open issues in the Uncut Gem repo. Note anything that changes the steps below. Done when: notes added to this doc; D3 and D4 can be answered.
S1-02 · Order parts Order everything in the purchase list once D1 to D6 are closed. Done when: all parts received and checked against the list.
S1-03 · Order assembled boards Upload the Gerbers zip plus the BOM and CPL from qc/nv-diamond/jlcpcb/ to JLCPCB (PCB plus top-side assembly, qty 2), fix any rotations flagged in the preview, and order. Done when: boards received.
S1-04 · Build the diamond prism Embed the diamond in epoxy on the photodiode with red filter gel between them, following hardware/DiamondMount. Diamond sinks in most epoxies, so expect to layer it. Done when: prism cured, photodiode leads intact, diamond sits in the light path.
S1-05 · Solder through-hole parts Solder the 2×3 header at 5V_SUPP1, the 1×4 socket at OLED1 and the 1×3 socket at PD1. Leave C34/C35 off; they only turn the op-amps into integrators. Done when: no solder bridges under magnification.
S1-06 · Inspect the assembled board Check the QFNs (U1, U8) and the amps (U2, U3) for bridges and correct orientation. Done when: no visible defects.
S1-07 · Continuity check With no power, check that 5 V, 3.3 V and GND are not shorted to each other. Done when: no shorts.
S1-08 · Fit standoffs Bolt in brass PCB standoffs before placing boards. Done when: board sits level and stable.
S1-09 · First power-up Power from USB with the A-to-C cable. Done when: PWR LED lights; 5 V at 5V_SUPP1 pin 2, 3.3 V at the LM1117 output (U6), 2.5 V at the R13/R14 divider (FAKE_GND). Any wrong reading: unplug and find the short.
S1-10 · Flash firmware Install the Adafruit SH110X and Adafruit GFX libraries, then compile and upload the Arduino IDE firmware from the repo's firmware folder over the board's USB-C (CP2102N; use the BOOT/EN buttons if auto-reset fails). If no serial port appears, flip the USB-C plug. Done when: upload succeeds and serial output appears.
S1-11 · Fit the OLED Plug the 1.3" SH1106 module into OLED1, checking that its pin 1 is VCC. Done when: the display lights at power-up.
S1-12 · RF lock check Power up. Done when: the on-board LOCK LED lights (ADF4351 locked to the 10 MHz oscillator).
S1-13 · Connect the light source Solder the 100 Ω resistor to the LED cathode; connect the anode to 5V_SUPP1 pin 4 and the resistor to pin 5 (always on, per D3). Done when: it lights at power-up.
S1-14 · Connect the photodiode Plug in the connector from the diamond prism. Done when: seated, polarity correct.
S1-15 · Connect the antenna Solder a small enameled wire loop to the J4 pads (center pad and ground), positioned right beside the diamond. Done when: joint inspected, no shorts.
S1-16 · Final short check and boot Done when: OLED shows "Calibrating..." on power-up.
S1-17 · Align light and diamond Hold the light source on stiff enameled wire; adjust while watching through red film. Done when: the diamond glows bright red.
S1-18 · First sweep Dim the room, power up, look at the lower half of the OLED. The sweep runs 2.614 to 3.126 GHz in 4 MHz steps (128 points). If no pattern, open the serial port and confirm data arrives. Done when: a dip near 2.87 GHz is visible.
S1-19 · Magnet test Bring a small magnet toward the diamond slowly. Done when: the dip visibly widens or spreads and returns when the magnet is removed.
S1-20 · Control test Block the light to the diamond with a small paper cutout and repeat S1-19. Done when: the magnet response disappears (proves the signal is fluorescence, not electrical pickup).
S1-21 · Log sweeps to a laptop (suggested addition) Capture serial output to CSV with a timestamp and the room temperature, and plot it. The published build has no calibration procedure or expected signal levels, so your logs become the baseline for Stage 2. Done when: a script in the repo saves and plots a sweep with and without a magnet.
Troubleshooting
• No power or no serial port: flip the USB-C plug (only one orientation is wired); use an A-to-C cable.
• No LOCK LED: check 3.3 V at U6 and the 10 MHz oscillator (X1).
• Blank display: check the OLED pin order (VCC-GND-SCL-SDA) and that it is an SH1106 module.
• No display pattern: check serial output, the photodiode connector, and the op-amp.
• No red glow: re-aim the LED; check its polarity and the resistor.
• No magnet response: confirm the sweep covers 2.87 GHz, then the antenna joint, then diamond alignment. Check the repo's open issues.
Stage 2: Cleaner instrument
What it is
The same chain, upgraded at every weak point: a bulk diamond plate instead of powder (clean, separate dips), better light collection, a lock-in amplifier that pulls the tiny signal out of noise by modulating the microwaves and listening only at that rate, and a coil that makes a known magnetic field for calibration.
What it looks like when done
• A bench setup: diamond plate in a mount with a coil around it, light source, photodiode, Red Pitaya board, laptop.
• On the laptop: sharp dips in clean pairs (up to four pairs). Rotating the magnet slides them around.
• A plot of dip splitting vs coil current that is a straight line.
• Sensitivity near 1 µT/√Hz in the published version.
Tickets
S2-01 · Extract the Stage 2 BOM Read the LED-based teaching magnetometer paper and list every part, supplier, and price. Done when: a Stage 2 BOM table exists in this doc with a total.
S2-02 · Buy the bulk NV diamond plate Done when: plate received with its spec sheet (nitrogen and NV concentration, orientation).
S2-03 · Build the optics Light focusing and red collection per the paper. Done when: red signal at the photodiode is clearly higher than with the Stage 1 prism.
S2-04 · Set up the lock-in (Red Pitaya) Modulate the microwaves at a few kHz and demodulate the photodiode signal at that rate. Done when: a sweep shows the resonance with visibly lower noise than an unmodulated sweep.
S2-05 · Build and characterize the calibration coil Done when: field per amp is known (measured or calculated from turns and radius).
S2-06 · Resolved spectrum Done when: separate dip pairs are visible and move as the magnet angle changes.
S2-07 · Calibration line Step coil current, record dip positions, plot splitting vs current. Done when: the plot is a straight line and its slope is recorded.
S2-08 · Sensitivity measurement Done when: a sensitivity figure in µT/√Hz is measured and compared with the paper's.
S2-09 · Temperature readout Track the midpoint of a dip pair while temperature changes. Done when: midpoint drift vs temperature is logged over at least a 5 K swing.
Stage 3: Pulsed control
What it is
Stages 1 and 2 run everything continuously. Stage 3 times everything: a green pulse to reset, a microwave pulse of chosen length to rotate the spin, a green pulse to read. A fixed bias magnet separates the two dips so you drive only one of them, which gives a clean two-level qubit. The ESP32 is too slow for this, so it needs a fast microwave switch, a nanosecond pulse sequencer, and light that can be switched quickly.
What it looks like when done
• Same bench plus switch and timing hardware.
• On the laptop: brightness vs microwave pulse length is a wavy curve that oscillates and slowly fades (Rabi oscillation). Follow-ups produce decaying fringes (Ramsey) and a longer-lived echo signal.
Tickets
S3-01 · Scope the pulsed hardware Pick and price a fast microwave switch, a pulse sequencer (check whether the Red Pitaya can do it), gateable light, and a bias magnet mount. Done when: Stage 3 BOM with costs added here.
S3-02 · Isolate one transition Position the bias magnet so the two dips are well separated. Done when: one dip can be driven with no visible effect on the other.
S3-03 · Build the pulse sequence Green reset, microwave pulse of length t, green readout; repeat and average. Done when: the sequence runs and timing is verified on a scope.
S3-04 · Rabi oscillation Sweep pulse length t. Done when: brightness oscillates with t and the π-pulse length is recorded.
S3-05 · Ramsey fringes Two half-flip pulses separated by a wait. Done when: decaying fringes recorded and the decay time noted.
S3-06 · Spin echo Add a full-flip pulse in the middle of the wait. Done when: decay time measured and longer than in S3-05.
S3-07 · T1 measurement Reset, wait, read. Done when: relaxation curve recorded.
Stage 4: Research grade (out of scope)
A confocal microscope focuses light onto a single NV center, single-photon detectors count its emission, and nearby carbon-13 nuclear spins serve as extra qubits for a few-qubit register. Kept here for reference; no tickets.
Links
• Uncut Gem repository (AGPL-3.0)
• Uncut Gem paper (arXiv)
• LED-based teaching magnetometer paper (arXiv)
