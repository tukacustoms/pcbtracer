# Tuka Customs — Airsoft Tracer Unit (LM393 + NE555, 5 V LED Rail)

A **reliable, MCU-less** tracer unit tuned for competitive airsoft.  
Optical gate (IR LED + phototransistor) → **LM393** Schmitt comparator → **NE555** monostable → logic-level MOSFET → **UVA LED array @ 5 V** (single-LED branches w/ series resistors).  
Focus is **robust detection**, **repeatable flash width**, **easy tuning**, and **simple manufacturing** (SMD BOM).

![Block Diagram](docs/images/block-diagram.png)
*If the image is missing, add a screenshot of `docs/Tracer_Visual_Schematic.pdf`.*

---

## ✨ Highlights
- **No MCU required** (fastest bring-up; no firmware headaches).
- **Clean detection**: LM393 with **hysteresis** + RC filter (no double-flash).
- **Fixed flash time**: NE555 monostable (2–5 ms typical).
- **5 V rail** LED design: 1 LED + 1 resistor per branch → easy to scale brightness.
- **Battery-friendly**: Sizeable bulk cap near MOSFET; simulator-ready to check droop.
- **All SMD BOM**: production friendly; footprints sized for comfortable hand reflow.

---

## 📐 System Architecture

**Optics**  
- IR Emitter (940 nm) across the bore  
- Phototransistor + RPULL + C_S (sensor node **S**)

**Comparator (LM393)**  
- IN– ← **S**  
- IN+ ← Threshold pot (**VTH**)  
- Output pull-up to 5 V + **positive feedback (R_H)** for hysteresis

**Monostable (NE555)**  
- Triggered by LM393 OUT (LOW)  
- R_T & C_T set flash width (**~3 ms default**)  
- OUT drives MOSFET gate (100 Ω + 100 k pulldown)

**Power stage**  
- Logic-level N-MOSFET (e.g. IRLR7843)  
- **5 V LED rail** → N× branches (**LED + resistor**) → MOSFET drain  
- Bulk electrolytic (1000–4700 µF) + 0.1 µF close to MOSFET

See the printable **visual schematic**:  
[`docs/Tracer_Visual_Schematic.pdf`](docs/Tracer_Visual_Schematic.pdf)

![Optical Gate](docs/images/optical-gate.png)
*Add an image showing emitter ↔ receiver with a small shroud over the phototransistor.*

---

## 🧰 Bill of Materials (SMD)

- **Core BOM** (one per board): [`hardware/Tracer_BOM_SMD_Core.csv`](hardware/Tracer_BOM_SMD_Core.csv)  
- **Per-branch BOM** (copy per LED branch): [`hardware/Tracer_BOM_SMD_PerBranch.csv`](hardware/Tracer_BOM_SMD_PerBranch.csv)

**Key parts**
- **Comparator**: TI LM393DR (SOIC-8)  
- **Timer**: NE555DR (SOIC-8) or TLC555 (SOT-23-5)  
- **MOSFET**: IRLR7843 (DPAK/TO-252) or similar logic-level FET  
- **IR Optics**: 940 nm LED (1206) + phototransistor (PLCC-2)  
- **UVA LEDs**: 395–405 nm, 2835/3528 @ ~20 mA each (one resistor per LED)  
- **Protection**: SMBJ5.0A TVS on the 5 V rail  
- **Bulk**: 1000 µF/16 V low-ESR SMD can (near MOSFET)

---

## 🔬 Simulation 

**Formulae**
- Branch resistor: `R ≈ (5 − Vf) / I`  
  - Example: Vf = 3.3–3.4 V, I = 20–25 mA → **82–68 Ω**  
- 555 pulse: `t ≈ 1.1 × R_T × C_T` → 27 k × 100 nF ≈ **3 ms**  
- Rail droop estimate: `ΔV ≈ (I_pulse × t) / C`

---

## 🧪 Bring-Up & Tuning

1) **Comparator (LM393)**
- Start `RPULL = 68 k`, `C_S = 10 nF`, `R_H = 680 k`.  
- Adjust the pot until **OUT is HIGH** (beam intact) and you get **one clean LOW** on a quick beam break.
- Sunlight issues → increase `C_S` to 22 nF, reduce `RPULL` (56–47 k), or increase hysteresis (560 k).

2) **555**
- Default `t ≈ 3 ms` (R_T = 27 k, C_T = 100 nF).  
- For brightness, increase to 4–5 ms; for high ROF/cool running, 2–3 ms.

3) **LED array @ 5 V**
- Add branches to scale brightness before raising per-branch current.  
- Size `Cbulk` near MOSFET; keep rails tight and short.

---

## ✅ Current Status (Progress Log)

**v0 — Through-hole prototype (works):**
- IRLZ44N + 1000 µF bulk + LM358 preamp (initial tests)
- Detected BBs reliably; confirmed flash window with basic RC.

**v1 — Comparator + 555 (No MCU):**
- Switched to **LM393** (with hysteresis) + **NE555** monostable  
- Stable single trigger per shot, consistent flash length  
- Decision: go **5 V LED rail** with single-LED branches for simplicity

**v2 — SMD & Simulation:**
- Full **SMD BOM** and **visual schematic PDF**  
- LTspice models for droop/current and parameter sweeps  
- Wiring cheat-sheet + bring-up plan finalized

---

## 🧠 Problems & Learnings

- **Double-flash / chatter**: solved with LM393 hysteresis (`R_H ~ 560–680 k`) + sensor RC (`C_S 10–22 nF`) and better threshold setting.  
- **Brightness vs. battery**: flash width and branch count matter more than cranking current in a single LED; keep per-LED ≈ 20 mA and add branches.  
- **Rail sag**: put the **bulk cap at the MOSFET**; simulate ESR and wiring with `Rser` to avoid surprises.  
- **Sunlight immunity**: shroud the receiver, lower `RPULL`, bump `C_S`, and raise threshold slightly.

---

## 🔧 How to Build

1. Assemble SMD per the BOM (0603 passives, SOIC-8 logic, DPAK MOSFET).  
2. Route **logic/sensor ground** separately back to the **MOSFET source** (star).  
3. Place **Cbulk** and a **0.1 µF** right next to the MOSFET/LED rail.  
4. Run the **IR pair** across the bore; shroud the receiver.  
5. Set LM393 threshold; confirm one trigger per beam break.  
6. Set 555 to 2–5 ms; confirm flash width.  
7. Connect LED branches; confirm one flash per shot and stable brightness.

---

## 🗺️ Roadmap

- [ ] Add **KiCad** schematic + PCB (SMD footprints matching BOM).  
- [ ] Optional **LM393→555 LTspice** timing model (full chain).  
- [ ] Mechanical shell in Fusion 360 (shroud geometry + LED ring tilt).  
- [ ] Instrumented tests: droop vs. shot rate; thermal snapshots; sunlight torture test.  
- [ ] Version with **modulated IR** + band-pass for extreme sunlight immunity (still MCU-less).

---

## 🙌 Credits
Design & tests by **Tuka Customs**. 
