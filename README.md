# Class D Audio Amplifier

> EN 2111 — Electronic Circuit Design | University of Moratuwa  
> Department of Electronic and Telecommunication Engineering  
> Batch 23 — Group 14

A fully designed, simulated, and breadboard-implemented Class-D audio amplifier featuring a pre-amplifier with active Baxandall tone control, high-frequency PWM generation, a half-bridge MOSFET output stage with bootstrap gate driver, and a 4th-order Butterworth LC low-pass output filter.

---

## Team

| Name | Index | Task |
|---|---|---|
| De Mel D. J. | 230121D | MOSFET Gate Driver Design |
| Rahman M. F. A. | 230507R | PWM Generation Design |
| Rahul B. | 230508V | Pre-Amplifier Design |
| Ratheeshan A. R. | 230539P | Low-Pass Filter Design |

---

## System Architecture

```
Audio In → Pre-Amp → PWM Generator → Gate Driver → Half-Bridge → LC Filter → Speaker
(NE5532)   (AD8065 + LM311D)  (IR2104)     (IRF530 x2)  (4th-order Butterworth)
```

---

## Repository Structure

```
class-d-audio-amplifier/
├── report/                  # Final submitted report (PDF)
├── simulation/
│   ├── preamp/              # LTspice files — Pre-amplifier & tone control (Rahul)
│   ├── pwm/                 # LTspice files — Triangular wave gen & comparator (Abdul Rahman)
│   ├── gate-driver/         # LTspice files — IR2104 half-bridge driver (Deelaka)
│   └── filter/              # LTspice files — 4th-order LC Butterworth filter (Rakesh)
├── pcb/
│   ├── schematic/           # Altium Designer schematic files (.SchDoc, .PrjPcb)
│   └── layout/              # PCB layout files + Gerbers
└── docs/
    └── references/          # Datasheets and application notes
```

---

## Subsystem Summaries

### 1. Pre-Amplifier (`simulation/preamp/`)
- Built around the **NE5532** dual op-amp (5 nV/√Hz noise, 10 MHz BW)
- Inverting gain stage: **Av = −15 (≈ 23.5 dB)**, AC-coupled at 7.23 Hz lower cutoff
- Active **Baxandall tone control** with bass (±12 dB shelf below 200 Hz) and treble adjustment
- Volume control potentiometer for modulation index matching
- Simulated THD: **0.243%** at 500 Hz, 750 mV peak input

### 2. PWM Generation (`simulation/pwm/`)
- **Triangular wave generator**: AD8065 relaxation oscillator + inverting integrator
  - f ≈ 354 kHz (R = 3.9 kΩ, C = 330 pF, β = 0.5)
  - Integrator: Ri = 33 kΩ, Ci = 47 pF, 100 kΩ DC feedback resistor
- **Comparator**: LM311D (replaced LM393 for faster switching at 350 kHz)
- PWM duty cycle: `D(t) = 0.5 + V_audio(t) / (2 × V_tri_pk)`
- Final switching frequency reduced to **150 kHz** for IR2104 compatibility

### 3. Gate Driver (`simulation/gate-driver/`)
- **IR2104** half-bridge gate driver (final iteration)
  - Integrated dead-time generation (520 ns fixed)
  - Internal PWM inversion for complementary HS/LS outputs
  - Bootstrap capacitor: **100 nF** (Cboot > 10 × Cg = 1.8 nF)
  - VDD bypass capacitor: **1 µF** (CVDD > 10 × Cboot)
  - Bootstrap diode: **1N5819** Schottky
- MOSFETs: **IRF530** (selected over IRFZ44N for lower Qg = 26 nC and Ciss = 670 pF at 300 kHz)
- Required gate drive: Isource = 313 mA (within 83 ns, 5% of half-period)

### 4. Output Filter (`simulation/filter/`)
- **4th-order passive LC Butterworth low-pass filter** (LC ladder topology)
- Cutoff frequency: **20 kHz**, −40 dB/decade rolloff
- Designed for RS/RL mismatch (RS = 0.5 Ω, RL = 8 Ω → RL/RS ≈ 16)
- Component values: **L1 = 100 µH, C2 = 1.56 µF, L3 = 68 µH, C4 = 330 nF**
- Zobel network (future improvement): R = 5.6 Ω, C = 100 µF for variable speaker load compensation

---

## Key Design Decisions

| Decision | Chosen | Reason |
|---|---|---|
| Triangular wave IC | AD8065 (over NE555) | NE555 has DC offset and non-linear (exponential) waveform |
| Comparator | LM311D (over LM393) | Faster switching, cleaner transitions at 350 kHz |
| Gate driver | IR2104 (over LTC4444) | Integrated dead-time, simpler implementation; cost/time tradeoff |
| MOSFET | IRF530 (over IRFZ44N) | Lower Qg and Ciss → less switching loss at high frequency |
| Filter order | 4th-order (over 2nd-order) | Steeper rolloff needed after reducing fsw to 150 kHz |
| Filter type | Passive LC (over RC/active) | Zero dissipation, handles full output current, best ripple rejection |

---

## How to Contribute (For Group Members)

1. **Fork** this repository to your own GitHub account
2. **Clone** your fork locally:
   ```bash
   git clone https://github.com/<your-username>/class-d-audio-amplifier.git
   ```
3. Add the upstream remote:
   ```bash
   git remote add upstream https://github.com/230539P/class-d-audio-amplifier.git
   ```
4. Work inside **your assigned subfolder** only (see table above)
5. Push to your fork and open a **Pull Request** to `230539P/class-d-audio-amplifier`

To pull in updates from the main repo:
```bash
git fetch upstream
git merge upstream/main
```

---

## Tools Used

- **LTspice** — Circuit simulation
- **Altium Designer** — PCB schematic and layout
- **LaTeX (XeLaTeX)** — Report generation
- **Tektronix TBS1000C** — Oscilloscope measurements

---

## References

1. Texas Instruments — NE5532 Datasheet
2. Texas Instruments — LM311 Datasheet
3. Analog Devices — AD8065 Datasheet
4. Infineon — IRF530N Datasheet
5. Infineon — IR2104 Datasheet
6. Analog Devices — AN-1070: Class D Audio Amplifier Basics
7. Texas Instruments — Class-D Audio Amplifier Design Basics (SLAA701A)
8. Chris Bowick — *RF Circuit Design*, Reed Elsevier, 1982
