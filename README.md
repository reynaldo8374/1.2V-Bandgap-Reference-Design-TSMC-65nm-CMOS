# 1.2 V Bandgap Reference in TSMC 65 nm CMOS

<p align="">
  <b>Complete schematic-to-layout implementation of a Banba-style CMOS bandgap reference, including PVT, Monte Carlo, physical verification, parasitic extraction, and post-layout verification.</b>
</p>

> **Project status:** Schematic design, pre-layout verification, layout, DRC/LVS/ERC review, parasitic extraction, and post-layout verification completed.

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

This repository documents the design and physical implementation of a **1.2 V CMOS bandgap reference (BGR)** in a **TSMC 65 nm CMOS process**. The design is based on a Banba-style low-voltage bandgap architecture that combines:

- a complementary-to-absolute-temperature (CTAT) bipolar junction voltage,
- a proportional-to-absolute-temperature (PTAT) voltage/current,
- an operational amplifier that forces the two internal sensing nodes to approximately the same voltage,
- a resistor network that determines the PTAT/CTAT weighting and output scale, and
- a startup circuit that avoids the undesired zero-current operating point.

The project covers the complete custom IC workflow:

1. transistor-level schematic design,
2. nominal and PVT simulation,
3. Monte Carlo process and mismatch verification,
4. full-custom layout,
5. DRC, LVS, and ERC review,
6. Pegasus-to-Quantus parasitic extraction,
7. extracted-SPICE integration in Cadence ADE, and
8. post-layout verification and retuning.

The screenshots and reported numerical values below correspond to the result files currently stored in this repository.

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
| Main simulator | Cadence Spectre |
| Post-layout extractor | Cadence Quantus |
| DRC/LVS engine | Cadence Pegasus |

### 2.2 Performance Summary

| Metric | Pre-Layout | Post-Layout | Conditions |
|---|---:|---:|---|
| Reference voltage | approximately 1.200 V | approximately 1.2001 V | TT, 27 °C |
| TT temperature coefficient | 2.66 ppm/°C | approximately 2.55 ppm/°C | -40 °C to 125 °C |
| Line regulation | 15.51 mV/V | approximately 15.42 mV/V | VDD = 2.0 V to 3.0 V, TT, 27 °C |
| Total supply current | approximately 87.16 µA | approximately 85.17 µA | VDD = 2.5 V, TT, 27 °C |
| Power consumption | approximately 217.9 µW | approximately 212.9 µW | VDD = 2.5 V, TT, 27 °C |
| Low-frequency PSRR | -36.88 dB | approximately -36.97 dB | TT, 27 °C |
| MC process mean | approximately 1.20021 V | approximately 1.20023 V | 1000 samples, 27 °C |
| MC process standard deviation | approximately 1.96 mV | approximately 1.935 mV | 1000 samples, 27 °C |
| MC mismatch mean | approximately 1.20005 V | approximately 1.20001 V | 1000 samples, 27 °C |
| MC mismatch standard deviation | approximately 445.86 µV | approximately 545.19 µV | 1000 samples, 27 °C |
| DRC | — | Reviewed; one density-rule reminder shown | Pegasus |
| LVS | — | Match | Pegasus |
| ERC | — | One reviewed floating-well warning | Pegasus |
| Startup | Verified | Verified | 1 µs, 100 µs, and 1 ms supply ramps |

The post-layout values remain close to the pre-layout results, indicating that the layout and extracted interconnect parasitics did not significantly degrade the nominal voltage, line regulation, current consumption, or process-variation distribution.

---

## 3. Operating Principle

### 3.1 CTAT and PTAT Components

 The base-emitter voltage of a bipolar transistor, $V_{BE}$, has a negative temperature coefficient and therefore provides the **CTAT** component.

A pair of bipolar devices operated at different current densities produces:

$$
\Delta V_{BE} = V_T \ln(N)
$$

where:

- $V_T = kT/q$ is the thermal voltage,
- $N$ is the emitter-area or current-density ratio, and
- $\Delta V_{BE}$ is proportional to absolute temperature.

The implemented bipolar ratio is:

$$
Q_0:Q_1 = 24:1
$$

### 3.2 Banba-Style Current Summation

The resistor network converts the CTAT and PTAT voltages into currents. The output resistor converts the summed current back into the reference voltage:

$$
V_{\mathrm{REF}}
\approx
R_{\mathrm{OUT}}
\left(
\frac{V_{BE}}{R_{\mathrm{CTAT}}}
+
\frac{\Delta V_{BE}}{R_{\mathrm{PTAT}}}
\right)
$$

In this design:

- the small resistor in the $\Delta V_{BE}$ path primarily controls the PTAT contribution and temperature slope,
- the two larger branch resistors preserve the Banba core relationship,
- the output resistor primarily scales and recenters the final reference voltage, and
- the op-amp forces the two internal nodes $V_X$ and $V_Y$ to nearly equal voltages.

### 3.3 Temperature-Coefficient Definition

The reported temperature coefficient is calculated using:

$$
TC =
\frac{V_{\mathrm{REF,max}}-V_{\mathrm{REF,min}}}
{V_{\mathrm{REF}}(27^\circ\mathrm{C})\,(T_{\max}-T_{\min})}
	imes 10^6
\quad [\mathrm{ppm}/^\circ\mathrm{C}]
$$

with:

$$
T_{\min}=-40^\circ\mathrm{C},\qquad
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
| PTAT-path resistor | Sets $\Delta V_{BE}$-derived current | approximately 12.4 kΩ |
| Core branch resistor 1 | CTAT/core branch scaling | approximately 73.5 kΩ |
| Core branch resistor 2 | Matched core branch | approximately 73.5 kΩ |
| Output resistor | Converts summed current into VREF | approximately 75.2 kΩ |
| BJT ratio | Generates $\Delta V_{BE}$ | 24:1 |

The temperature coefficient was tuned using the core PTAT-path resistor. After the temperature slope was optimized, the output resistor was used to center the TT, 27 °C output near 1.2 V.

### 4.2 Operational Amplifier

<p align="center">
  <img src="img/prelayout/opamp_schematic.png" width="95%">
</p>

The operational amplifier regulates the Banba core by forcing:

$$
V_X \approx V_Y
$$

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

$$
TC_{\mathrm{pre,TT}} = 2.66\ \mathrm{ppm}/^\circ\mathrm{C}
$$

This result was used as the baseline during post-layout resistor retuning.

### 5.4 Line Regulation

<p align="center">
  <img src="img/prelayout/line_regulation_tt_27c.png" width="90%">
</p>

The supply voltage was swept from 2.0 V to 3.0 V at TT and 27 °C.

$$
\mathrm{Line\ Regulation}
=
\frac{V_{\mathrm{OUT,max}}-V_{\mathrm{OUT,min}}}
{V_{\mathrm{DD,max}}-V_{\mathrm{DD,min}}}
$$

The reported pre-layout line regulation is:

$$
\mathrm{LineReg}_{\mathrm{pre}}
\approx 15.51\ \mathrm{mV/V}
$$

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

$$
\mathrm{PSRR}_{\mathrm{pre}}
\approx -36.88\ \mathrm{dB}
$$

The response is dominated by the supply coupling through the biasing and amplifier paths, with frequency-dependent behavior caused by the amplifier and compensation network.

### 5.7 Monte Carlo Process Variation

<p align="center">
  <img src="img/prelayout/monte_carlo_process_27c.png" width="90%">
</p>

For 1000 process-variation samples at 27 °C:

$$
\mu_{\mathrm{process,pre}} \approx 1.20021\ \mathrm{V}
$$

$$
\sigma_{\mathrm{process,pre}} \approx 1.96\ \mathrm{mV}
$$

### 5.8 Monte Carlo Mismatch

<p align="center">
  <img src="img/prelayout/monte_carlo_mismatch_27c.png" width="90%">
</p>

For 1000 mismatch samples at 27 °C:

$$
\mu_{\mathrm{mismatch,pre}} \approx 1.20005\ \mathrm{V}
$$

$$
\sigma_{\mathrm{mismatch,pre}} \approx 445.86\ \mu\mathrm{V}
$$

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

$$
	ext{Layout}
\rightarrow
	ext{Pegasus LVS / SVDB}
\rightarrow
	ext{Quantus RC Extraction}
\rightarrow
	ext{Extracted SPICE Netlist}
\rightarrow
	ext{Cadence ADE Post-Layout Simulation}
$$

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

$$
V_{\mathrm{REF,post}} \approx 1.2001\ \mathrm{V}
$$

The corner curves remain centered near 1.2 V, while the largest variation occurs at the fast/slow extremes over temperature.

### 9.3 Temperature Coefficient

<p align="center">
  <img src="img/postlayout/temperature_coefficient_all_corners.png" width="90%">
</p>

The current repository result shows a nominal TT temperature coefficient of approximately:

$$
TC_{\mathrm{post,TT}} \approx 2.55\ \mathrm{ppm}/^\circ\mathrm{C}
$$

This is close to, and slightly better than, the 2.66 ppm/°C pre-layout result.

The core PTAT resistor was retuned after extraction to recover the optimum temperature slope. The output resistor was then adjusted separately to center the 27 °C output near 1.2 V.

### 9.4 Line Regulation

<p align="center">
  <img src="img/postlayout/line_regulation_tt_27c.png" width="90%">
</p>

From the displayed markers:

- $V_{\mathrm{OUT}}$ at $V_{\mathrm{DD}}=2.0\ \mathrm{V}$ is approximately 1.19108 V,
- $V_{\mathrm{OUT}}$ at $V_{\mathrm{DD}}=3.0\ \mathrm{V}$ is approximately 1.20649 V.

Therefore:

$$
\mathrm{LineReg}_{\mathrm{post}}
\approx 15.42\ \mathrm{mV/V}
$$

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

$$
I_{\mathrm{DD,post}} \approx 85.17\ \mu\mathrm{A}
$$

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

The first-order bandgap cancellation still leaves nonlinear $V_{BE}$ curvature. Future work can investigate:

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

