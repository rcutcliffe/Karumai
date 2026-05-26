# Karumai Island Telephone Network Simulator

A single-file HTML/JavaScript simulator of a fictional island PSTN, accurate to Australian Post Office / Telecom Australia engineering practice of the 1960s–1980s. It plays real audio through the Web Audio API and shows a detailed trace of every signal exchanged between exchanges.

---

## The Network

**Karumai Island** — area code (074), population ~180,000 — is served by nine exchanges connected via a mix of MFC-M trunk signalling, decadic junctions, and Step/Strowger selectors.

| Code | Name | Type | Lines | Signalling |
|------|------|------|-------|------------|
| PKM | Port Karumai Main | ARK511-D | 30 | Decadic junction to PKT |
| PKS | Port Karumai South | ARK-M | 200 | MFC-K to parent PKT |
| PKT | Port Karumai Tandem | ARF tandem | — | Trunk only, Reg-LP |
| STF | Stenfield | ARF Reg-LM | 800 | MFC-M |
| HAL | Halverton | ARF Reg-LM | 600 | MFC-M |
| DWK | Denwick | ARF Reg-LP | 400 | MFC-M |
| MRW | Murrawa | Step/Strowger | 200 | Decadic, no DSR |
| COR | Cape Orris | Step/Strowger | 30 | Decadic, DSR |
| TBK | Talbeck | ARF Reg-LP | 200 | MFC-M |

Direct routes exist between STF↔HAL and DWK↔TBK. All other inter-exchange traffic routes via the PKT tandem.

---

## Signalling Accurately Modelled

### MFC-M (per HXF003 — APO 1969)
- Forward frequencies F0–F5: 1380 / 1500 / 1620 / 1740 / 1860 / 1980 Hz
- Backward frequencies F0–F4: 1140 / 1020 / 900 / 780 / 660 Hz
- Digit 0 = signal 10 = F3+F4 (1740+1860 Hz)
- One continuous compelled digit stream end-to-end
- Seizure + digit 1 sent simultaneously; 1A1 is the first backward signal
- GV absorbs prefix digits (1+2), returns 1A1 after routing
- GIV/SL return 3A1; final SL returns 3A3
- 1A7 + re-apply + 2A2 restart for tandem routing
- 1A9 + re-apply + 3A7 for Step exchange destinations
- Reg-LM: 2A7 waiting place after thousands digit (suppressed on tandem restart)
- ARF common control marker delay 80–200 ms random at each switching stage
- CLASS signal (Group 2, sig 2) sent after 3A3; B-series follows

### MFC-K (per ETP0185 — Telecom Australia 1970, ARK-M only)
- Off-hook: PKT KS sends Group 6 signal 11 requesting subscriber category
- ARK-M KMR responds K1 (subscriber unrestricted)
- Local call: PKT KS sends digits → KMR responds U1/U3/B-series/G7-12/U4
- Non-local: KMR sends U5 → PKT sends G7-11 → KMR sends U4; FUR prepared

### ARK-D (PKM)
- Off-hook seizes circuit directly to PKT Reg.ELP
- PKT provides dial tone; subscriber decadic travels to PKT register
- Terminating: GV signals 1A9 → re-apply → 3A7; PKT outpulses all 7 digits decadically on PKM junction; PKM Reg-D receives all digits, extracts tens+units, passes to marker M; M tests and connects subscriber line; M and Reg-D release

### Step/Strowger Exchanges
- MRW: 5 group selectors + final selector; first 2 digits select PKT junction on outgoing calls
- COR: DSR (Discriminating Selector Repeater) as first stage, then 1st group selector, then final selector
- Originating → PKT: decadic on junction → PKT REG-I converts to MFC-M for GV stage
- Terminating from PKT: PKT GV signals 1A9 + re-apply + 3A7; decadic outpulsing on junction (4 digits after stripping 3); selector stages step on each digit as it arrives
- Per-pulse Strowger click audio (layered noise bursts: ratchet + body thud + metallic ring)
- Selector hunting sounds between digits

### Tones (Australian standard)
- Dial tone: 400 Hz (Step exchanges: wobbly LFO-modulated); ARK-M: 400+450 Hz
- Ringing: 400 Hz, 0.4s/0.2s/0.4s/2.0s cadence
- Busy: 425 Hz, 375 ms on/off
- Congestion: 425 Hz fast
- Answer: 500 ms pause then 900 Hz 2-second TCAF burst

---

## Source Documents

- **HXF003** — APO 1969: *MFC Signalling ARF Exchanges*. Definitive frequency tables, signal sequences, call scenarios.
- **ETP0185** — Telecom Australia 1970: *MFC Signalling ARK-M Exchanges*. MFC-K scheme, K/U/3A/B signal series, ARK-M off-hook sequence.

---

## Installation

No installation required. The simulator is a single self-contained HTML file with no external dependencies beyond Google Fonts (loaded from CDN for display purposes only — the simulator works without internet access, fonts will fall back gracefully).

1. Download `karumai.html`
2. Open it in any modern web browser (Chrome, Firefox, Edge, Safari)
3. That's it

> **Note:** The Web Audio API requires a user gesture before audio can play. Click any button or dial pad key to initialise audio on first use.

---

## Usage

### Network Map tab
Shows the island geography with all nine exchanges, inter-exchange routes, circuit counts, and number ranges. Click any exchange node to auto-populate a random call from that exchange to a random destination and jump to the Active Call tab.

### Active Call tab

**To place a call:**
1. Select an originating exchange from the *Originating Exchange* dropdown
2. Select a destination exchange from the *Destination Number* dropdown — this auto-fills a random valid number for that exchange
3. Or type a number manually using the dial pad or keyboard (digits 0–9, Backspace to delete)
4. Click **CALL** or press Enter

**During a call:**
- The oscilloscope displays the active tone frequencies in real time
- The frequency bar strip lights up each active MFC-M or MFC-K frequency
- The Signals panel shows the current forward and backward signals
- The Exchange Dialogue trace log shows every signal exchanged with timestamps, frequencies, and descriptions
- Click **CLEAR** or press the hang-up button to abort the call at any point

**Speed control:** The slider adjusts simulation speed from 0.5× (half speed, useful for studying the signalling) to 5× (fast, useful for watching many calls).

### Exchange Status tab
Shows circuit utilisation bars for every inter-exchange route, updated in real time as calls are placed and released.

### Call History tab
Records the last 60 calls with originating exchange, dialled number, route taken, call duration, and result (answered / busy / congestion / aborted).

---

## Notes for Telephony Enthusiasts

- The simulator models **one call at a time** — it is not a traffic simulation
- Random outcomes: ~8% chance of busy (1B2), ~4% chance of congestion on any route
- Circuit utilisation is tracked and released after ~18 seconds (simulated hold time)
- The compelled MFC-M signalling sequence is accurate: each forward signal is held until the backward acknowledgement arrives
- Marker delays at ARF/ARK exchanges are randomised 80–200 ms to reflect real common-control behaviour
- Strowger selector audio uses layered noise synthesis to approximate the characteristic ratchet, body thud, and metallic ring of a uniselector or group selector in operation

---

## Contributing

Pull requests and issues welcome, particularly from those with access to original APO/Telecom Australia engineering documentation. Areas of ongoing interest:

- Additional exchange types (ARF Reg-LM with subscriber trunk dialling, PABX junctions)
- Operator-assisted call scenarios
- Trunk offer / alternative routing on congestion
- Additional islands / extended network topology

---

## Licence

MIT — do whatever you like with it, attribution appreciated.
