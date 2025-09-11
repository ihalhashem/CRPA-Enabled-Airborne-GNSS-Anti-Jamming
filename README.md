# CRPA-Enabled Airborne GNSS Anti-Jamming

## Introduction
<p align="justify">
Airborne GNSS robustness study built in STK. A civil airliner flies a low-altitude (10 kft MSL) route over Abu Dhabi while a commercial rooftop jammer radiates at GPS L1. I compare a conventional single-element FRPA against a compact 2-element CRPA (MVDR beamformer) and quantify the effect on received C/N₀ and C/(N+I). The primary stress case uses a <b>20 W jammer with a 10 dBi panel</b>.
</p>

## Table&nbsp;of&nbsp;Contents
* [Scenario](#scenario)
* [FRPA Baseline](#frpa-baseline)
* [CRPA Receiver](#crpa-receiver)
* [Jammer & J/S Model](#jammer--js-model)
* [Results](#results)
* [How to Reproduce (STK)](#how-to-reproduce-stk)
* [Assumptions & Notes](#assumptions--notes)
* [Repository Structure](#repository-structure)

---

<a id="scenario"></a>
## Scenario

### Overview
<p align="justify">
Aircraft at 10 kft MSL (landing/loiter representative) over a rectangular Area Target. Constellation is GPS L1 C/A (1.57542 GHz). Links are auto-selected by maximum elevation. The jammer sits on a rooftop inside the area.
</p>

### Key Parameters

| Item | Value |
|---|---|
| Frequency | 1.57542 GHz (GPS L1 C/A) |
| Aircraft altitude | 10 kft MSL |
| Duration | ~30 min around 09:00–09:30 UTC |
| Link selection | Receive-constrained, Max Elevation |
| Propagation | Free-space, light-time delay ON |
| Metrics | Per-satellite C/N₀, C/(N+I), Rx gain, J/S vs. range |

*(optional figure placeholders)*  
`![Scenario](figs/scenario_overview.png)`  
`![Route](figs/route_3d.png)`

---

<a id="frpa-baseline"></a>
## FRPA Baseline

### Overview
<p align="justify">
Single-element RHCP patch on the aircraft (FRPA) used as the non-adaptive reference.
</p>

### Setup
* Antenna: embedded RHCP patch (FRPA)
* Nominal gain: 3–5 dBic (broad beam)
* Polarization: RHCP
* System noise temperature: 290 K

*(optional)*  
`![FRPA Median C/N0](figs/median_cn_frpa.png)`

---

<a id="crpa-receiver"></a>
## CRPA Receiver

### Overview
<p align="justify">
Compact <b>4-element circular array</b> (0.5 λ spacing) with <b>MVDR</b> beamforming. The main lobe tracks the strongest satellite; an adaptive <b>null is steered at the jammer</b> (object-based null provider).
</p>

### Array & Beamformer
* Elements: 4 (circular), spacing = **0.5 λ**
* Backlobe suppression: 20 dB
* Beam Direction: **Automatic** (max-elevation satellite)
* Null Direction: **Object → Jammer**, az/el limits ±90°
* Beamformer: **MVDR**, constraint = **3 dB**

*(optional)*  
`![CRPA Gain vs Time](figs/rx_gain_crpa.png)`

---

<a id="jammer--js-model"></a>
## Jammer & J/S Model

### Jammer (reference case)
| Parameter | Value |
|---|---|
| Location | Rooftop inside Area Target |
| Power | **20 W** (43 dBm) |
| Antenna | **10 dBi panel** |
| EIRP | **+53 dBm = +23 dBW** |
| Polarization | Linear (defaults to STK dipole/panel), mismatch to RHCP included by STK |

### J/S sanity line (free-space, co-pol)
<p align="justify">
I also plot a closed-form J/S vs. range curve to contextualize the STK results:
</p>

\[
\mathrm{J/S\ (dB)} \approx \left(EIRP_\mathrm{jam}\;[\mathrm{dBW}] - L_\mathrm{fs}(R)\;[\mathrm{dB}]\right) - P_\mathrm{sig,iso}\;[\mathrm{dBW}]
\]

*Where \(L_\mathrm{fs}(R)=32.45+20\log_{10}(f_\mathrm{MHz})+20\log_{10}(R_\mathrm{km})\).*

*(optional)*  
`![JSR vs Range](figs/jsr_vs_range.png)`

---

<a id="results"></a>
## Results

### What to look at
* **Median C/N₀ and C/(N+I) per satellite** (robust to brief adaptive nulls).  
* **Rx gain vs. time** shows CRPA main-lobe and narrow adaptive nulls toward the jammer.  
* **Min C/N₀** is shown only for stress awareness (over-emphasizes short nulls), so the headline comparisons are based on **medians**.

### Headline Findings (typical run)
* With FRPA, median per-satellite **C/N₀ ≈ 33–45 dB-Hz** under 20 W/10 dBi stress.
* With CRPA (4-el, MVDR), medians improve to **~40–56 dB-Hz**, maintaining tracking margin.
* Adaptive nulls are **brief and narrow**; navigation continuity is preserved while close to the jammer.

*(drop your actual images here)*  
`![Median C/N0 FRPA vs CRPA](figs/median_cn_compare.png)`  
`![Min C/N0 (awareness)](figs/min_cn_compare.png)`  
`![Rx Gain Traces](figs/rx_gain_traces.png)`

---

<a id="how-to-reproduce-stk"></a>
## How to Reproduce (STK)

1. **Objects**
   - *Constellations*: GPS IIF/III/IIR/IIRM transmitters.
   - *Aircraft*: 10 kft MSL route over Area Target.
   - *Jammer*: Facility on rooftop; Transmitter → Complex Model (20 W), Antenna → Panel 10 dBi.

2. **CommSystem**
   - *Link Definition*: Constraining **Receive**; Selection **Max Elevation**.
   - *Interference*: Assign **Jammer** constellation.
   - *Receiver (FRPA run)*: Complex Receiver → Antenna **Patch** / FRPA equivalent.
   - *Receiver (CRPA run)*:
     - Antenna → **Phased Array** → 4 elements, **circular**, **0.5 λ** spacing.
     - **Beam Direction Provider**: Automatic (strongest satellite).
     - **Null Direction Provider**: Object → **Jammer**.
     - **Beam Former**: **MVDR**, constraint **3 dB**.

3. **Reports/Graphs**
   - *Per-satellite* **C/N₀** and **C/(N+I)**.
   - Receiver **Gain** vs. time.
   - Export CSV → plot medians in Python/Matlab.

---

<a id="assumptions--notes"></a>
## Assumptions & Notes
* Rx noise temperature nominal **290 K** (hot case sensitivity can be swept).
* GPS EIRP per block uses conservative (minimum) values from open literature.
* Polarization mismatch (panel vs RHCP) and off-boresight losses are handled by STK’s antenna models.
* Medians are used for headline comparisons; mins are provided for stress awareness only.

---

<a id="repository-structure"></a>
## Repository Structure
