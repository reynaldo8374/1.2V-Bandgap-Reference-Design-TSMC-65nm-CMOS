# 1.2 V Bandgap Reference in TSMC 65 nm CMOS

Complete schematic-to-post-layout implementation of a Banba-style CMOS bandgap reference, including PVT and Monte Carlo verification, full-custom layout, physical verification, parasitic extraction, and extracted-circuit simulation.

**Status:** Schematic, layout, DRC/LVS review, parasitic extraction, and post-layout verification completed.

---

## Table of Contents

- [1. Project Overview](#1-project-overview)
- [2. Design Objectives and Reported Performance](#2-design-objectives-and-reported-performance)
- [3. Operating Principle](#3-operating-principle)
- [4. Circuit Implementation](#4-circuit-implementation)
- [5. Pre-Layout Verification](#5-pre-layout-verification)
- [6. Layout Implementation](#6-layout-implementation)
- [7. Physical Verification](#7-physical-verification)
- [8. Parasitic Extraction Flow](#8-parasitic-extraction-flow)
- [9. Post-Layout Verification](#9-post-layout-verification)
- [10. Pre-Layout vs. Post-Layout Comparison](#10-pre-layout-vs-post-layout-comparison)
- [11. Known Tool and PDK Compatibility Notes](#11-known-tool-and-pdk-compatibility-notes)
- [12. Design Limitations](#12-design-limitations)
- [13. Future Design Improvements](#13-future-design-improvements)
- [14. Repository Structure](#14-repository-structure)
- [15. Tools and Technology](#15-tools-and-technology)
- [16. References](#16-references)
- [17. Author](#17-author)

---

## 1. Project Overview

This repository documents a **1.2 V bandgap reference implemented in TSMC 65 nm CMOS**. The circuit uses a Banba-style current-summing core with a **24:1 PNP emitter-area ratio**, an operational amplifier that regulates the internal nodes \(V_X\) and \(V_Y\), a startup circuit, and a resistor network that independently sets the temperature balance and output-voltage scale.

The documented flow covers transistor-level design, PVT and Monte Carlo verification, full-custom layout, Pegasus DRC/LVS/ERC review, Quantus RC extraction, and extracted-SPICE simulation in Cadence ADE. All numerical results quoted in this README are taken from the committed simulation screenshots.

---

## 2. Design Objectives and Reported Performance

### 2.1 Design Conditions

| Parameter | Value |
|---|---:|
| Technology | TSMC 65 nm CMOS |
| Nominal supply voltage | 2.5 V |
| Target reference voltage | 1.2 V |
| Temperature range | -40 °C to 125 °C |
| Process corners | TT, SS, SF, FF, FS |
| Monte Carlo samples | 1000 |
| Circuit simulator | Cadence Spectre |
| DRC/LVS | Cadence Pegasus |
| Parasitic extraction | Cadence Quantus |

### 2.2 Electrical Performance Summary

| Metric | Pre-Layout | Post-Layout | Conditions |
|---|---:|---:|---|
| Reference voltage | 1.2000 V | 1.2001 V | TT, 27 °C |
| TT temperature coefficient | 2.66 ppm/°C | 2.55 ppm/°C | -40 °C to 125 °C |
| Line regulation | 15.51 mV/V | 15.42 mV/V | VDD = 2.0 V to 3.0 V, TT, 27 °C |
| Supply current | 87.16 µA | 85.17 µA | VDD = 2.5 V, TT, 27 °C |
| Power consumption | 217.9 µW | 212.9 µW | VDD = 2.5 V, TT, 27 °C |
| Low-frequency PSRR | -36.88 dB | -36.9861 dB | TT, 27 °C |
| MC process mean | 1.20021 V | 1.20023 V | 1000 samples, 27 °C |
| MC process standard deviation | 1.96 mV | 1.935 mV | 1000 samples, 27 °C |
| MC mismatch mean | 1.20005 V | 1.20001 V | 1000 samples, 27 °C |
| MC mismatch standard deviation | 445.86 µV | 545.19 µV | 1000 samples, 27 °C |

### 2.3 Post-Layout PSRR Robustness

The following low-frequency values were read from the post-layout PSRR markers.

#### Process-Corner Variation at 27 °C

| Corner | PSRR |
|---|---:|
| TT | -36.9861 dB |
| SS | -36.6821 dB |
| SF | -36.1072 dB |
| FF | -37.1939 dB |
| FS | -37.7213 dB |

#### Temperature Variation at TT

| Temperature | PSRR |
|---|---:|
| -40 °C | -38.8769 dB |
| 27 °C | -36.9861 dB |
| 125 °C | -34.5079 dB |

The post-layout reference voltage and temperature coefficient are **final retuned results**, not an untouched before-and-after parasitic comparison. After extraction, the PTAT-path resistor was adjusted to restore the temperature slope, and the output resistor was then adjusted to center the TT output at 27 °C near 1.2 V.

---

## 3. Operating Principle

The derivation below follows the low-voltage current-summing bandgap formulation described by Behzad Razavi in *The Design of a Low-Voltage Bandgap Reference*. The architecture first combines PTAT and CTAT **currents**, then converts their sum into the output voltage through a separate resistor.

### 3.1 Generation of the PTAT Component

The amplifier regulates the common PMOS gate voltage so that:

$$
V_X \approx V_Y
$$

The two PNP devices carry comparable current densities but use an emitter-area ratio \(N\). With \(N=24\) in this implementation, their base-emitter-voltage difference is:

$$
\Delta V_{BE}
=
V_{BE1}-V_{BE2}
=
V_T \ln(N)
$$

where:

$$
V_T=\frac{kT}{q}
$$

Because \(V_T\) is proportional to absolute temperature, \(\Delta V_{BE}\) is PTAT. In this circuit, \(\Delta V_{BE}\) appears across \(R_4\), producing:

$$
I_{\mathrm{PTAT}}
=
\frac{\Delta V_{BE}}{R_4}
=
\frac{V_T\ln(N)}{R_4}
$$

### 3.2 CTAT Current and Core Current

The base-emitter voltage of the unit PNP has a negative temperature coefficient. With \(R_5=R_{18}\), the two regulated branches retain the symmetry required by the Banba core, and the mirrored core current can be written to first order as:

$$
I_{\mathrm{CORE}}
\approx
\frac{V_{BE1}}{R_{18}}
+
\frac{V_T\ln(N)}{R_4}
$$

The first term is CTAT and the second is PTAT. Their temperature slopes are selected to cancel near the center of the specified temperature range.

The condition for first-order temperature cancellation is:

$$
\frac{\mathrm{d}V_{BE1}}{\mathrm{d}T}
+
\frac{R_{18}}{R_4}
\frac{k}{q}\ln(N)
\approx 0
$$

This relation explains the tuning strategy used in the design: \(R_4\) sets the PTAT weighting and therefore the temperature slope, while \(R_5\) and \(R_{18}\) remain matched.

### 3.3 Output-Voltage Scaling

The third PMOS branch copies the core current into the output resistor \(R_7\). The resulting reference voltage is:

$$
V_{\mathrm{REF}}
\approx
R_7 I_{\mathrm{CORE}}
$$

or, after substitution:

$$
V_{\mathrm{REF}}
\approx
R_7
\left(
\frac{V_{BE1}}{R_{18}}
+
\frac{V_T\ln(N)}{R_4}
\right)
$$

An equivalent and more useful design form is:

$$
V_{\mathrm{REF}}
\approx
\frac{R_7}{R_{18}}
\left[
V_{BE1}
+
\frac{R_{18}}{R_4}V_T\ln(N)
\right]
$$

This separation is important:

| Component | Primary design role |
|---|---|
| \(R_4\) | Adjusts PTAT weighting and temperature coefficient |
| \(R_5, R_{18}\) | Preserve the matched Banba core branches |
| \(R_7\) | Scales and centers the final reference voltage |
| \(N=24\) | Sets \(\Delta V_{BE}=V_T\ln(N)\) |
| Operational amplifier | Forces \(V_X\approx V_Y\) |
| Startup circuit | Prevents the zero-current equilibrium after power-up |

The output resistor scales the summed current without changing the first-order PTAT-to-CTAT cancellation condition. This is why the post-layout tuning was performed in two steps: \(R_4\) was adjusted first for minimum temperature coefficient, followed by \(R_7\) for the final 1.2 V centering.

### 3.4 Main Nonidealities

Finite amplifier gain produces an error between \(V_X\) and \(V_Y\), while amplifier offset is amplified by the resistor ratio. Its first-order output contribution scales approximately as:

$$
\left|\Delta V_{\mathrm{REF,OS}}\right|
\approx
\frac{R_7}{R_{18}}
\left(
1+\frac{R_{18}}{R_4}
\right)
\left|V_{\mathrm{OS}}\right|
$$

The output also depends on current-mirror output resistance and on differences between the drain-source voltages of the core and output PMOS devices. These effects motivate adequate channel length, sufficient amplifier gain, and post-layout verification over process and temperature.

For the PSRR plots in this repository, the displayed quantity is:

$$
\mathrm{PSRR}_{\mathrm{dB}}(f)
=
20\log_{10}
\left|
\frac{V_{\mathrm{OUT}}(f)}
{V_{\mathrm{DD}}(f)}
\right|
$$

More-negative values indicate better supply rejection. Low-frequency PSRR is strongly influenced by amplifier gain and internal supply feedthrough, while high-frequency behavior is increasingly determined by device capacitances and internal-node coupling. The core also admits a zero-current equilibrium, so a startup circuit is required to guarantee the intended operating point after power-up.

The temperature coefficient reported throughout this project is:

$$
TC
=
\frac{
V_{\mathrm{REF,max}}-V_{\mathrm{REF,min}}
}{
V_{\mathrm{REF}}(27^\circ\mathrm{C})
\left(T_{\max}-T_{\min}\right)
}
\times 10^6
\quad
\left[\mathrm{ppm}/^\circ\mathrm{C}\right]
$$

with:

$$
T_{\min}=-40^\circ\mathrm{C},
\qquad
T_{\max}=125^\circ\mathrm{C}
$$

---

## 4. Circuit Implementation

### 4.1 Complete Bandgap Reference Schematic

<p align="center">
  <img src="img/prelayout/bgr_schematic.png" width="95%">
</p>

The complete circuit contains the Banba bandgap core, bipolar devices, resistor network, output-current mirror, operational amplifier, compensation network, and startup circuitry.

The final resistor network visible in the schematic uses approximately:

| Component | Approximate Function | Nominal Resistance |
|---|---|---:|
| PTAT-path resistor | Sets \(\Delta V_{BE}\)-derived current | approximately 12.4 kΩ |
| Core branch resistor 1 | CTAT/core branch scaling | approximately 73.5 kΩ |
| Core branch resistor 2 | Matched core branch | approximately 73.5 kΩ |
| Output resistor | Converts summed current into VREF | approximately 75.2 kΩ |
| BJT ratio | Generates \(\Delta V_{BE}\) | 24:1 |

The temperature coefficient was tuned using the core PTAT-path resistor. After the temperature slope was optimized, the output resistor was used to center the TT, 27 °C output near 1.2 V.

### 4.2 Operational Amplifier

<p align="center">
  <img src="img/prelayout/opamp_schematic.png" width="95%">
</p>

The operational amplifier regulates the Banba core by forcing:

\[
V_X \approx V_Y
\]

Its biasing, gain, and stability must remain sufficient across process and temperature variation because an incorrect loop operating point directly disturbs the PTAT/CTAT current balance.

---

## 5. Pre-Layout Verification

### 5.1 Simulation Scope

The pre-layout design was verified using:

- nominal DC operating-point analysis,
- temperature sweep from -40 °C to 125 °C,
- TT/SS/SF/FF/FS process corners,
- line-regulation sweep from 2.0 V to 3.0 V,
- startup transient simulation with three supply-ramp speeds,
- AC PSRR simulation, and
- 1000-sample Monte Carlo process and mismatch analyses.

### 5.2 Temperature Sweep Across Process Corners

<p align="center">
  <img src="img/prelayout/temperature_sweep_all_corners.png" width="95%">
</p>

The pre-layout response remains close to 1.2 V across the complete temperature range. The different corner curves reflect process-dependent changes in BJT voltage, MOS bias currents, and resistor behavior.

### 5.3 Temperature Coefficient

<p align="center">
  <img src="img/prelayout/temperature_coefficient_all_corners.png" width="90%">
</p>

The nominal TT temperature coefficient is approximately:

\[
TC_{\mathrm{pre,TT}} = 2.66\ \mathrm{ppm}/^\circ\mathrm{C}
\]

This result was used as the baseline during post-layout resistor retuning.

### 5.4 Line Regulation

<p align="center">
  <img src="img/prelayout/line_regulation_tt_27c.png" width="90%">
</p>

The supply voltage was swept from 2.0 V to 3.0 V at TT and 27 °C.

\[
\mathrm{Line\ Regulation}
=
\frac{V_{\mathrm{OUT,max}}-V_{\mathrm{OUT,min}}}
{V_{\mathrm{DD,max}}-V_{\mathrm{DD,min}}}
\]

The reported pre-layout line regulation is:

\[
\mathrm{LineReg}_{\mathrm{pre}}
\approx 15.51\ \mathrm{mV/V}
\]

### 5.5 Startup Verification

The startup circuit was tested using multiple supply-ramp durations to verify convergence to the intended bandgap operating point.

#### 1 µs Supply Ramp

<p align="center">
  <img src="img/prelayout/startup_1us.png" width="90%">
</p>

#### 100 µs Supply Ramp

<p align="center">
  <img src="img/prelayout/startup_100us.png" width="90%">
</p>

#### 1 ms Supply Ramp

<p align="center">
  <img src="img/prelayout/startup_1ms.png" width="90%">
</p>

The three runs verify startup under fast, intermediate, and slow supply ramps.

### 5.6 Power-Supply Rejection Ratio

<p align="center">
  <img src="img/prelayout/psrr_tt_27c.png" width="90%">
</p>

The reported low-frequency pre-layout PSRR is approximately:

\[
\mathrm{PSRR}_{\mathrm{pre}}
\approx -36.88\ \mathrm{dB}
\]

The response is dominated by the supply coupling through the biasing and amplifier paths, with frequency-dependent behavior caused by the amplifier and compensation network.

### 5.7 Monte Carlo Process Variation

<p align="center">
  <img src="img/prelayout/monte_carlo_process_27c.png" width="90%">
</p>

For 1000 process-variation samples at 27 °C:

\[
\mu_{\mathrm{process,pre}} \approx 1.20021\ \mathrm{V}
\]

\[
\sigma_{\mathrm{process,pre}} \approx 1.96\ \mathrm{mV}
\]

### 5.8 Monte Carlo Mismatch

<p align="center">
  <img src="img/prelayout/monte_carlo_mismatch_27c.png" width="90%">
</p>

For 1000 mismatch samples at 27 °C:

\[
\mu_{\mathrm{mismatch,pre}} \approx 1.20005\ \mathrm{V}
\]

\[
\sigma_{\mathrm{mismatch,pre}} \approx 445.86\ \mu\mathrm{V}
\]

The process distribution is wider than the mismatch-only distribution, showing that global process variation is the dominant simulated source of absolute reference-voltage spread.

---

## 6. Layout Implementation

### 6.1 Final Layout

<p align="center">
  <img src="img/postlayout/final_layout.png" width="95%">
</p>

The completed layout integrates:

- the large 24:1 bipolar device array,
- segmented high-value poly resistors,
- the bandgap current-mirror and output devices,
- the operational amplifier,
- compensation components,
- power and ground routing, and
- substrate/well connections and guard structures.

The bipolar array occupies a large portion of the layout because the 24:1 emitter-area ratio is implemented using repeated unit devices. Segmentation and symmetric placement were used where practical to improve matching and make routing more systematic.

### 6.2 Layout Considerations

Important implementation concerns include:

- preserving the 24:1 BJT unit-device ratio,
- maintaining equal routing environments for matched branches,
- keeping sensitive amplifier inputs away from noisy supply routing,
- segmenting long resistors into manageable physical structures,
- using robust VDD and VSS routing,
- placing well/substrate contacts close to active devices, and
- minimizing asymmetric parasitics around the two Banba core nodes.

---

## 7. Physical Verification

Physical verification was performed using Cadence Pegasus.

### 7.1 DRC Configuration

<details>
<summary><b>Show DRC run-data setup</b></summary>

<p align="center">
  <img src="img/postlayout/drc_setup_run_data.png" width="95%">
</p>

</details>

<details>
<summary><b>Show DRC rule setup</b></summary>

<p align="center">
  <img src="img/postlayout/drc_setup_rules.png" width="95%">
</p>

</details>

### 7.2 DRC Result

<p align="center">
  <img src="img/postlayout/drc_result.png" width="95%">
</p>

The captured DRC result shows one density-related rule reminder. This item should be interpreted as a density/check-deck result rather than a schematic-connectivity mismatch. The result was reviewed before continuing to LVS and PEX.

### 7.3 LVS Configuration

<details>
<summary><b>Show complete LVS setup</b></summary>

#### Run Data

<p align="center">
  <img src="img/postlayout/lvs_setup_run_data.png" width="95%">
</p>

#### Rules

<p align="center">
  <img src="img/postlayout/lvs_setup_rules.png" width="95%">
</p>

#### Layout and Schematic Input

<p align="center">
  <img src="img/postlayout/lvs_setup_input.png" width="95%">
</p>

#### Output Configuration

<p align="center">
  <img src="img/postlayout/lvs_setup_output.png" width="95%">
</p>

#### LVS Options

<p align="center">
  <img src="img/postlayout/lvs_setup_lvs_options.png" width="95%">
</p>

</details>

### 7.4 LVS Result

<p align="center">
  <img src="img/postlayout/lvs_result.png" width="95%">
</p>

**LVS status: MATCH**

The extracted layout devices and connectivity match the schematic reference netlist. The Pegasus flow also generated Quantus input data for parasitic extraction.

### 7.5 ERC Review

<p align="center">
  <img src="img/postlayout/erc_warning.png" width="95%">
</p>

The LVS run reports a non-empty ERC summary caused by a reviewed floating-well warning associated with the implemented device structure. The warning is documented explicitly rather than hidden, while the main LVS comparison remains matched.

---

## 8. Parasitic Extraction Flow

The post-layout netlist was generated using **Cadence Quantus with Pegasus query data**.

### 8.1 Extraction Workflow

\[
\text{Layout}
\rightarrow
\text{Pegasus LVS / SVDB}
\rightarrow
\text{Quantus RC Extraction}
\rightarrow
\text{Extracted SPICE Netlist}
\rightarrow
\text{Cadence ADE Post-Layout Simulation}
\]

### 8.2 Quantus Pre-Setup

<details>
<summary><b>Show Quantus pre-setup</b></summary>

<p align="center">
  <img src="img/postlayout/pex_setup_1.png" width="95%">
</p>

<p align="center">
  <img src="img/postlayout/pex_setup_2.png" width="95%">
</p>

</details>

### 8.3 Technology Setup Directory

<p align="center">
  <img src="img/postlayout/pex_setup_directory.png" width="95%">
</p>

The Quantus setup directory points to the PDK extraction directory containing the technology data used for resistance and capacitance extraction.

### 8.4 RC Extraction Configuration

<p align="center">
  <img src="img/postlayout/pex_extraction.png" width="95%">
</p>

The final verification uses RC extraction. C-only and no-parasitic extracted-device runs were also useful during debugging to separate interconnect parasitics from extracted-device netlisting issues.

### 8.5 Extracted SPICE Integration

<p align="center">
  <img src="img/postlayout/post_layout_spice_format.png" width="95%">
</p>

The Quantus output is included in the ADE testbench as the extracted implementation of the BGR block.

### 8.6 Post-Layout Testbench and Maestro Setup

<details>
<summary><b>Show post-layout testbench and ADE setup</b></summary>

<p align="center">
  <img src="img/postlayout/testbench_setup.png" width="95%">
</p>

<p align="center">
  <img src="img/postlayout/maestro_setup.png" width="95%">
</p>

</details>

---

## 9. Post-Layout Verification

### 9.1 Extracted Schematics

#### Bandgap Reference

<p align="center">
  <img src="img/postlayout/bgr_schematic.png" width="95%">
</p>

#### Operational Amplifier

<p align="center">
  <img src="img/postlayout/opamp_schematic.png" width="95%">
</p>

The op-amp, resistor devices, BJT array, and complete BGR were individually checked during extraction debugging. This helped isolate a PNP netlist-parameter compatibility issue described in [Section 11](#11-known-tool-and-pdk-compatibility-notes).

### 9.2 Temperature Sweep Across Process Corners

<p align="center">
  <img src="img/postlayout/temperature_sweep_all_corners.png" width="95%">
</p>

At TT and 27 °C, the extracted result is approximately:

\[
V_{\mathrm{REF,post}} \approx 1.2001\ \mathrm{V}
\]

The corner curves remain centered near 1.2 V, while the largest variation occurs at the fast/slow extremes over temperature.

### 9.3 Temperature Coefficient

<p align="center">
  <img src="img/postlayout/temperature_coefficient_all_corners.png" width="90%">
</p>

The current repository result shows a nominal TT temperature coefficient of approximately:

\[
TC_{\mathrm{post,TT}} \approx 2.55\ \mathrm{ppm}/^\circ\mathrm{C}
\]

This is close to, and slightly better than, the 2.66 ppm/°C pre-layout result.

The core PTAT resistor was retuned after extraction to recover the optimum temperature slope. The output resistor was then adjusted separately to center the 27 °C output near 1.2 V.

### 9.4 Line Regulation

<p align="center">
  <img src="img/postlayout/line_regulation_tt_27c.png" width="90%">
</p>

From the displayed markers:

- \(V_{\mathrm{OUT}}\) at \(V_{\mathrm{DD}}=2.0\ \mathrm{V}\) is approximately 1.19108 V,
- \(V_{\mathrm{OUT}}\) at \(V_{\mathrm{DD}}=3.0\ \mathrm{V}\) is approximately 1.20649 V.

Therefore:

\[
\mathrm{LineReg}_{\mathrm{post}}
\approx 15.42\ \mathrm{mV/V}
\]

This result is effectively unchanged from the pre-layout line regulation.

### 9.5 Startup Verification

#### 1 µs Supply Ramp

<p align="center">
  <img src="img/postlayout/startup_1u.png" width="90%">
</p>

#### 100 µs Supply Ramp

<p align="center">
  <img src="img/postlayout/startup_100u.png" width="90%">
</p>

#### 1 ms Supply Ramp

<p align="center">
  <img src="img/postlayout/startup_1m.png" width="90%">
</p>

The extracted circuit reaches the intended reference state for all three tested supply-ramp durations.

### 9.6 Power-Supply Rejection Across Process Corners

<p align="center">
  <img src="img/postlayout/psrr_all_corners.png" width="90%">
</p>

The low-frequency PSRR varies across process corners, with the displayed responses lying approximately in the -32 dB to -37 dB range.

### 9.7 PSRR Across Temperature

<p align="center">
  <img src="img/postlayout/psrr_temperature_variation.png" width="90%">
</p>

The displayed low-frequency PSRR values are approximately:

| Temperature | Low-Frequency PSRR |
|---|---:|
| -40 °C | -38.88 dB |
| 27 °C | -36.97 dB |
| 125 °C | -34.51 dB |

PSRR degrades at high temperature in the shown result, while the low-temperature case provides the strongest supply rejection.

### 9.8 Total Current Consumption

<p align="center">
  <img src="img/postlayout/total_curent.png" width="90%">
</p>

At the nominal 2.5 V supply:

\[
I_{\mathrm{DD,post}} \approx 85.17\ \mu\mathrm{A}
\]

\[
P_{\mathrm{post}}
=
V_{\mathrm{DD}}I_{\mathrm{DD}}
\approx
2.5\ \mathrm{V}\times 85.17\ \mu\mathrm{A}
\approx 212.9\ \mu\mathrm{W}
\]

The current increases with supply voltage, reaching approximately 137 µA at 3.0 V in the displayed sweep.

### 9.9 Monte Carlo Process Variation

<p align="center">
  <img src="img/postlayout/monte_carlo_process_27c.png" width="90%">
</p>

For 1000 process-variation samples:

\[
\mu_{\mathrm{process,post}} \approx 1.20023\ \mathrm{V}
\]

\[
\sigma_{\mathrm{process,post}} \approx 1.935\ \mathrm{mV}
\]

### 9.10 Monte Carlo Mismatch

<p align="center">
  <img src="img/postlayout/monte_carlo_mismatch_27c.png" width="90%">
</p>

For 1000 mismatch samples:

\[
\mu_{\mathrm{mismatch,post}} \approx 1.20001\ \mathrm{V}
\]

\[
\sigma_{\mathrm{mismatch,post}} \approx 545.19\ \mu\mathrm{V}
\]

The process variation remains the dominant contributor to the simulated absolute reference-voltage spread.

---

## 10. Pre-Layout vs. Post-Layout Comparison

| Metric | Pre-Layout | Post-Layout | Observation |
|---|---:|---:|---|
| VREF at TT, 27 °C | approximately 1.200 V | approximately 1.2001 V | Nominal target preserved |
| TT temperature coefficient | 2.66 ppm/°C | approximately 2.55 ppm/°C | Recovered through post-layout resistor tuning |
| Line regulation | 15.51 mV/V | approximately 15.42 mV/V | Essentially unchanged |
| Total current at 2.5 V | approximately 87.16 µA | approximately 85.17 µA | Slight reduction |
| Power at 2.5 V | approximately 217.9 µW | approximately 212.9 µW | Slight reduction |
| Low-frequency PSRR at 27 °C | -36.88 dB | approximately -36.97 dB | Similar |
| MC process mean | approximately 1.20021 V | approximately 1.20023 V | Nearly identical |
| MC process sigma | approximately 1.96 mV | approximately 1.935 mV | Nearly identical |
| MC mismatch mean | approximately 1.20005 V | approximately 1.20001 V | Nearly identical |
| MC mismatch sigma | approximately 445.86 µV | approximately 545.19 µV | Moderate extracted-result increase |
| Startup | Pass | Pass | Verified for three ramp times |

The close agreement between pre-layout and post-layout results indicates that the physical implementation successfully preserves the intended electrical behavior after parasitic extraction.

---

## 11. Known Tool and PDK Compatibility Notes

### 11.1 Quantus PNP `AREA` Parameter

During extracted-SPICE simulation, Quantus generated unit PNP instances in the form:

```spice
Q... VSS VSS emitter_node pnp5 AREA=2.5e-11
```

Using this extracted value directly caused the PNP devices to conduct incorrectly in the simulator, forcing the internal Banba nodes and output close to the supply rail.

The extracted netlist already represented the 24:1 BJT ratio using:

- one unit instance in the small branch, and
- 24 parallel unit instances in the large branch.

For simulation, the unit-device normalized area was therefore changed in a copy of the extracted netlist to:

```spice
AREA=1
```

The 24 parallel instances were retained, preserving the intended 24:1 ratio. After this correction, the extracted circuit returned to the expected approximately 1.2 V operating point.

This workaround is specific to the tested combination of:

- an older PDK integration,
- Pegasus-generated query data,
- Quantus 23.x extracted-SPICE output, and
- the simulator interpretation of the PNP `AREA` parameter.

The original PDK files and original extraction result should remain unchanged; the correction is applied only to a simulation copy of the generated netlist.

### 11.2 Extracted-View Compatibility

The available legacy `extview.trp` file was not directly compatible with the current Pegasus/Quantus extracted-view flow. Therefore, the project used extracted SPICE output for post-layout verification.

### 11.3 Physical-Verification Warnings

The repository intentionally records:

- a density-related DRC reminder, and
- a floating-well ERC warning.

These warnings are kept visible for traceability. LVS remains matched.

---

## 12. Design Limitations

The current project has the following limitations:

- The results are simulation-based and have not yet been validated in fabricated silicon.
- No post-fabrication trimming network is implemented.
- Absolute VREF spread is still dominated by global process variation.
- Noise characterization is not yet included in the current result set.
- The documented PNP netlist correction is a flow-specific workaround.
- PSRR is moderate and can be improved for noise-sensitive PMU applications.
- Package, pad, ESD, seal-ring, and top-level chip-integration effects are not included.
- The current repository does not report a final measured layout area.

---

## 13. Future Design Improvements

### 13.1 Resistor Trimming

Add a digitally controlled trimming network to the output-scaling resistor. A small trim range around the nominal value can correct post-fabrication VREF error caused by global process variation.

Possible implementation:

- 3- to 5-bit binary or thermometer-coded trim,
- metal-option or fuse-based trim,
- separate coarse and fine trim ranges.

### 13.2 Temperature-Coefficient Trimming

Add a trim option to the PTAT-path resistor so that the CTAT/PTAT balance can be adjusted independently from the absolute output voltage.

A two-dimensional trimming strategy can use:

- PTAT-path trim for temperature slope,
- output-resistor trim for nominal VREF.

### 13.3 Curvature Compensation

The first-order bandgap cancellation still leaves nonlinear \(V_{BE}\) curvature. Future work can investigate:

- second-order curvature compensation,
- piecewise nonlinear correction,
- temperature-dependent resistor/current shaping, or
- additional bipolar junction terms.

### 13.4 PSRR Improvement

Potential improvements include:

- a higher-PSRR op-amp topology,
- cascoding in supply-sensitive branches,
- RC filtering of internal bias nodes,
- a pre-regulator, or
- integration behind a low-noise LDO.

### 13.5 Noise Optimization

Future verification should include:

- output-noise spectral density,
- integrated RMS noise,
- dominant-noise-device identification, and
- low-frequency flicker-noise reduction.

### 13.6 Lower-Power Operation

The present design consumes approximately 213 µW post-layout at 2.5 V. Future work can reduce power by:

- lowering branch currents,
- optimizing op-amp biasing,
- increasing resistor values where headroom permits, and
- redesigning the startup path for negligible steady-state current.

### 13.7 Tapeout and Silicon Measurement

A tapeout-ready version should add:

- complete top-level pad integration,
- ESD protection,
- seal ring,
- density fill,
- antenna-rule verification,
- packaged test planning,
- temperature-chamber characterization,
- startup testing over supply-ramp conditions, and
- measured trimming and PSRR validation.

### 13.8 PMU Integration

The BGR can be integrated as the reference source for:

- a low-dropout regulator,
- current and voltage bias generators,
- a phase-locked loop bias block, or
- a larger power-management unit.

---

## 14. Repository Structure

```text
.
├── README.md
└── img
    ├── prelayout
    │   ├── bgr_schematic.png
    │   ├── opamp_schematic.png
    │   ├── temperature_sweep_all_corners.png
    │   ├── temperature_coefficient_all_corners.png
    │   ├── line_regulation_tt_27c.png
    │   ├── startup_1us.png
    │   ├── startup_100us.png
    │   ├── startup_1ms.png
    │   ├── psrr_tt_27c.png
    │   ├── monte_carlo_process_27c.png
    │   └── monte_carlo_mismatch_27c.png
    └── postlayout
        ├── bgr_schematic.png
        ├── opamp_schematic.png
        ├── final_layout.png
        ├── drc_setup_run_data.png
        ├── drc_setup_rules.png
        ├── drc_result.png
        ├── lvs_setup_run_data.png
        ├── lvs_setup_rules.png
        ├── lvs_setup_input.png
        ├── lvs_setup_output.png
        ├── lvs_setup_lvs_options.png
        ├── lvs_result.png
        ├── erc_warning.png
        ├── pex_setup_1.png
        ├── pex_setup_2.png
        ├── pex_setup_directory.png
        ├── pex_extraction.png
        ├── post_layout_spice_format.png
        ├── testbench_setup.png
        ├── maestro_setup.png
        ├── temperature_sweep_all_corners.png
        ├── temperature_coefficient_all_corners.png
        ├── line_regulation_tt_27c.png
        ├── startup_1u.png
        ├── startup_100u.png
        ├── startup_1m.png
        ├── psrr_all_corners.png
        ├── psrr_temperature_variation.png
        ├── total_curent.png
        ├── monte_carlo_process_27c.png
        └── monte_carlo_mismatch_27c.png
```

---

## 15. Tools and Technology

| Task | Tool |
|---|---|
| Schematic capture | Cadence Virtuoso Schematic Editor |
| Circuit simulation | Cadence Spectre / ADE Explorer |
| Layout | Cadence Virtuoso Layout XL |
| DRC | Cadence Pegasus |
| LVS and ERC | Cadence Pegasus |
| Parasitic extraction | Cadence Quantus |
| Process | TSMC 65 nm CMOS PDK |

---

## 16. References

1. H. Banba et al., “A CMOS Bandgap Reference Circuit with Sub-1-V Operation,” *IEEE Journal of Solid-State Circuits*, vol. 34, no. 5, 1999.
2. B. Razavi, *Design of Analog CMOS Integrated Circuits*, McGraw-Hill.
3. P. R. Gray, P. J. Hurst, S. H. Lewis, and R. G. Meyer, *Analysis and Design of Analog Integrated Circuits*, Wiley.
4. TSMC 65 nm process design-kit documentation and device-model documentation.
5. Cadence Pegasus documentation for DRC, LVS, ERC, and Quantus query-data generation.
6. Cadence Quantus documentation for transistor-level RC extraction and extracted-SPICE generation.

---

## 17. Author

**Reynaldo Nicholas Sianturi**

Electrical Engineering, Institut Teknologi Bandung  
Focus: Analog and Mixed-Signal Integrated-Circuit Design
