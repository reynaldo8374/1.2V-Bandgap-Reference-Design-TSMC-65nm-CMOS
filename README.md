# 1.2 V Bandgap Reference in TSMC 65 nm CMOS

This repository documents the complete design of a **1.2 V Banba-style bandgap reference** in a TSMC 65 nm CMOS process, from transistor-level schematic verification to full-custom layout, physical verification, parasitic extraction, and post-layout simulation.

The circuit operates from a nominal 2.5 V supply and uses a 24:1 PNP emitter-area ratio, an operational amplifier, a startup circuit, and a resistor network that separately controls the temperature slope and the final output-voltage level.

---

## 1. Design Overview

### Design Conditions

| Parameter | Value |
|---|---:|
| Process | TSMC 65 nm CMOS |
| Nominal supply | 2.5 V |
| Target reference voltage | 1.2 V |
| Temperature range | -40 °C to 125 °C |
| Process corners | TT, SS, SF, FF, FS |
| Monte Carlo runs | 1000 samples |

### Final Electrical Performance

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

The post-layout reference voltage and temperature coefficient are **final retuned results**. After extraction, the PTAT-path resistor was adjusted to recover the optimum temperature slope. The output resistor was then adjusted independently to recenter the TT output near 1.2 V at 27 °C.

---

## 2. Circuit Principle and Implementation

### 2.1 Banba Current-Summing Principle

The design follows the current-summing low-voltage bandgap principle. The amplifier regulates the two internal sensing nodes so that:

```math
V_X \approx V_Y
```

The two PNP devices use an emitter-area ratio of `N = 24`. Their base-emitter-voltage difference is:

```math
\Delta V_{BE}
=
V_{BE1}-V_{BE2}
=
V_T \ln(N)
```

where:

```math
V_T=\frac{kT}{q}
```

Because thermal voltage is proportional to absolute temperature, `ΔVBE` is a PTAT quantity. In this implementation, it appears across the small core resistor `R4`, producing the PTAT current:

```math
I_{\mathrm{PTAT}}
=
\frac{\Delta V_{BE}}{R_4}
=
\frac{V_T\ln(N)}{R_4}
```

The PNP base-emitter voltage provides the CTAT component. With `R5` and `R18` matched, the mirrored core current can be approximated by:

```math
I_{\mathrm{CORE}}
\approx
\frac{V_{BE1}}{R_{18}}
+
\frac{V_T\ln(N)}{R_4}
```

The first term decreases with temperature, while the second increases with temperature. First-order cancellation occurs when their slopes approximately satisfy:

```math
\frac{\mathrm{d}V_{BE1}}{\mathrm{d}T}
+
\frac{R_{18}}{R_4}
\frac{k}{q}\ln(N)
\approx 0
```

The output branch mirrors the core current into `R7`, giving:

```math
V_{\mathrm{REF}}
\approx
R_7 I_{\mathrm{CORE}}
```

or:

```math
V_{\mathrm{REF}}
\approx
\frac{R_7}{R_{18}}
\left[
V_{BE1}
+
\frac{R_{18}}{R_4}V_T\ln(N)
\right]
```

This relation separates the two tuning functions clearly:

| Element | Primary role |
|---|---|
| `R4` | Sets PTAT weighting and temperature coefficient |
| `R5` and `R18` | Preserve the matched Banba core branches |
| `R7` | Scales and centers the final reference voltage |
| PNP ratio `24:1` | Generates the PTAT `ΔVBE` term |
| Operational amplifier | Forces `VX ≈ VY` |
| Startup circuit | Prevents the zero-current operating point |

The temperature coefficient used in this project is:

```math
TC
=
\frac{
V_{\mathrm{REF,max}}-V_{\mathrm{REF,min}}
}{
V_{\mathrm{REF}}(27^\circ\mathrm{C})
\left(T_{\max}-T_{\min}\right)
}
\times 10^6
```

with `Tmin = -40 °C` and `Tmax = 125 °C`.

### 2.2 Complete BGR Schematic

<p align="center">
  <img src="img/prelayout/bgr_schematic.png" width="95%">
</p>

The schematic contains the Banba core, a 24:1 PNP array, matched resistor branches, an output-current mirror, an operational amplifier, a compensation network, and a startup circuit.

The resistor network is approximately:

| Component | Function | Nominal value |
|---|---|---:|
| `R4` | PTAT-current generation | 12.4 kΩ |
| `R5` | Core branch resistor | 73.5 kΩ |
| `R18` | Matched core branch resistor | 73.5 kΩ |
| `R7` | Output-current-to-voltage conversion | 75.2 kΩ |

### 2.3 Operational Amplifier

<p align="center">
  <img src="img/prelayout/opamp_schematic.png" width="95%">
</p>

The operational amplifier must provide adequate gain over the input common-mode range established by the BGR core. Finite gain creates an error between `VX` and `VY`, while input offset is amplified by the resistor ratio and directly shifts the final reference voltage.

The following figure compares the pre-layout and post-layout amplifier responses. The extracted response is expected to preserve sufficient DC gain, unity-gain bandwidth, and phase margin for correct BGR regulation.

<p align="center">
  <img src="img/postlayout/opamp_pre_post_comparison.png" width="95%">
</p>

The exact gain, GBW, and phase-margin values are annotated in the committed comparison figure.

---

## 3. Pre-Layout Verification

### 3.1 Temperature and Process Corners

<p align="center">
  <img src="img/prelayout/temperature_sweep_all_corners.png" width="95%">
</p>

The reference was swept from -40 °C to 125 °C across TT, SS, SF, FF, and FS. The nominal TT curve achieves a temperature coefficient of 2.66 ppm/°C.

<p align="center">
  <img src="img/prelayout/temperature_coefficient_all_corners.png" width="90%">
</p>

### 3.2 Line Regulation

<p align="center">
  <img src="img/prelayout/line_regulation_tt_27c.png" width="90%">
</p>

The supply was swept from 2.0 V to 3.0 V at TT and 27 °C. Line regulation was calculated as:

```math
\mathrm{LineReg}
=
\frac{
V_{\mathrm{OUT,max}}-V_{\mathrm{OUT,min}}
}{
V_{\mathrm{DD,max}}-V_{\mathrm{DD,min}}
}
```

The resulting pre-layout line regulation is 15.51 mV/V.

### 3.3 Startup

Startup was verified using fast, medium, and slow supply ramps.

<p align="center">
  <img src="img/prelayout/startup_1us.png" width="90%">
</p>

<p align="center">
  <img src="img/prelayout/startup_100us.png" width="90%">
</p>

<p align="center">
  <img src="img/prelayout/startup_1ms.png" width="90%">
</p>

All three cases converge to the intended bandgap operating point.

### 3.4 PSRR

<p align="center">
  <img src="img/prelayout/psrr_tt_27c.png" width="90%">
</p>

The plotted supply-to-output transfer is:

```math
\mathrm{PSRR}_{\mathrm{plot}}(f)
=
20\log_{10}
\left|
\frac{V_{\mathrm{OUT}}(f)}
{V_{\mathrm{DD}}(f)}
\right|
```

The low-frequency result at TT and 27 °C is -36.88 dB. More-negative values indicate stronger supply rejection in the displayed convention.

### 3.5 Monte Carlo Variation

#### Process Variation

<p align="center">
  <img src="img/prelayout/monte_carlo_process_27c.png" width="90%">
</p>

The 1000-sample process run gives a mean of 1.20021 V and a standard deviation of 1.96 mV.

#### Device Mismatch

<p align="center">
  <img src="img/prelayout/monte_carlo_mismatch_27c.png" width="90%">
</p>

The mismatch-only run gives a mean of 1.20005 V and a standard deviation of 445.86 µV. The wider process distribution shows that global process variation dominates the simulated absolute-output spread.

---

## 4. Layout and Physical Verification

### 4.1 Final Layout

<p align="center">
  <img src="img/postlayout/final_layout.png" width="95%">
</p>

The layout integrates the 24-unit PNP array, segmented poly resistors, the BGR mirrors, the operational amplifier, compensation components, power routing, and substrate/well contacts. Repeated unit devices and symmetric routing were used where practical to preserve matching.

### 4.2 DRC

<table>
  <tr>
    <td width="50%"><img src="img/postlayout/drc_setup_run_data.png" width="100%"></td>
    <td width="50%"><img src="img/postlayout/drc_setup_rules.png" width="100%"></td>
  </tr>
</table>

<p align="center">
  <img src="img/postlayout/drc_result.png" width="95%">
</p>

The captured run contains a reviewed density-related reminder. The result is documented explicitly because density handling is part of the final manufacturability flow.

### 4.3 LVS and ERC

<table>
  <tr>
    <td width="50%"><img src="img/postlayout/lvs_setup_run_data.png" width="100%"></td>
    <td width="50%"><img src="img/postlayout/lvs_setup_rules.png" width="100%"></td>
  </tr>
  <tr>
    <td width="50%"><img src="img/postlayout/lvs_setup_input.png" width="100%"></td>
    <td width="50%"><img src="img/postlayout/lvs_setup_output.png" width="100%"></td>
  </tr>
</table>

<p align="center">
  <img src="img/postlayout/lvs_setup_lvs_options.png" width="90%">
</p>

<p align="center">
  <img src="img/postlayout/lvs_result.png" width="95%">
</p>

**LVS status: MATCH**

The schematic and extracted layout contain equivalent devices, dimensions, and connectivity. The LVS run also generated the Pegasus query database used by Quantus.

<p align="center">
  <img src="img/postlayout/erc_warning.png" width="95%">
</p>

The ERC report includes a reviewed floating-well warning associated with the implemented device structure. It is retained in the documentation for traceability; the main LVS comparison remains matched.

---

## 5. Parasitic Extraction and Simulation Setup

The extraction flow is:

```text
Layout → Pegasus LVS/SVDB → Quantus RC extraction → SPICE netlist → Cadence ADE
```

### 5.1 Quantus Configuration

<table>
  <tr>
    <td width="50%"><img src="img/postlayout/pex_setup_1.png" width="100%"></td>
    <td width="50%"><img src="img/postlayout/pex_setup_2.png" width="100%"></td>
  </tr>
</table>

<p align="center">
  <img src="img/postlayout/pex_setup_directory.png" width="95%">
</p>

<p align="center">
  <img src="img/postlayout/pex_extraction.png" width="95%">
</p>

RC extraction was used for the final post-layout verification. No-parasitic and C-only extracted-device runs were also used during debugging to separate device-netlisting behavior from interconnect parasitics.

### 5.2 Extracted SPICE Integration

<p align="center">
  <img src="img/postlayout/post_layout_spice_format.png" width="95%">
</p>

<table>
  <tr>
    <td width="50%"><img src="img/postlayout/testbench_setup.png" width="100%"></td>
    <td width="50%"><img src="img/postlayout/maestro_setup.png" width="100%"></td>
  </tr>
</table>

### 5.3 PNP `AREA` Compatibility Issue

Quantus initially generated each extracted unit PNP with:

```spice
AREA=2.5e-11
```

The extracted netlist already represented the 24:1 ratio as one unit device in the small branch and 24 parallel unit devices in the large branch. In this simulator flow, directly using the physical area as the SPICE normalized `AREA` multiplier caused the PNP currents to collapse and forced `VX`, `VY`, and `VOUT` close to VDD.

For simulation, the generated netlist was copied and the unit-device parameter was changed to:

```spice
AREA=1
```

The 24 parallel PNP instances were retained, so the intended 24:1 ratio was preserved. This correction restored the expected 1.2 V operating point. The original PDK and original extraction output were not modified.

---

## 6. Post-Layout Results

### 6.1 Temperature and Process Corners

<p align="center">
  <img src="img/postlayout/temperature_sweep_all_corners.png" width="95%">
</p>

After retuning `R4`, the TT post-layout temperature coefficient is 2.55 ppm/°C.

<p align="center">
  <img src="img/postlayout/temperature_coefficient_all_corners.png" width="90%">
</p>

After the temperature slope was restored, `R7` was adjusted to center the TT output at 27 °C near 1.2 V. The output resistor was not used to correct the temperature slope.

### 6.2 Line Regulation and Current

<p align="center">
  <img src="img/postlayout/line_regulation_tt_27c.png" width="90%">
</p>

The post-layout line regulation is 15.42 mV/V, essentially unchanged from the pre-layout result.

<p align="center">
  <img src="img/postlayout/total_curent.png" width="90%">
</p>

At VDD = 2.5 V, the extracted circuit draws 85.17 µA. The corresponding nominal power is calculated directly as:

```math
P_{\mathrm{post}}
=
V_{\mathrm{DD}} I_{\mathrm{DD}}
=
(2.5\ \mathrm{V})(85.17\ \mu\mathrm{A})
=
212.9\ \mu\mathrm{W}
```

### 6.3 Startup

<p align="center">
  <img src="img/postlayout/startup_1u.png" width="90%">
</p>

<p align="center">
  <img src="img/postlayout/startup_100u.png" width="90%">
</p>

<p align="center">
  <img src="img/postlayout/startup_1m.png" width="90%">
</p>

The extracted circuit reaches the intended operating point for all three tested supply-ramp durations.

### 6.4 PSRR Robustness

#### Process Corners at 27 °C

<p align="center">
  <img src="img/postlayout/psrr_all_corners.png" width="90%">
</p>

| Corner | Low-frequency PSRR |
|---|---:|
| TT | -36.9861 dB |
| SS | -36.6821 dB |
| SF | -36.1072 dB |
| FF | -37.1939 dB |
| FS | -37.7213 dB |

#### Temperature Variation at TT

<p align="center">
  <img src="img/postlayout/psrr_temperature_variation.png" width="90%">
</p>

| Temperature | Low-frequency PSRR |
|---|---:|
| -40 °C | -38.8769 dB |
| 27 °C | -36.9861 dB |
| 125 °C | -34.5079 dB |

The displayed supply rejection is strongest at low temperature and degrades toward 125 °C.

### 6.5 Monte Carlo Variation

#### Process Variation

<p align="center">
  <img src="img/postlayout/monte_carlo_process_27c.png" width="90%">
</p>

The extracted 1000-sample process distribution has a mean of 1.20023 V and a standard deviation of 1.935 mV.

#### Device Mismatch

<p align="center">
  <img src="img/postlayout/monte_carlo_mismatch_27c.png" width="90%">
</p>

The mismatch-only distribution has a mean of 1.20001 V and a standard deviation of 545.19 µV.

### 6.6 Final Comparison

| Metric | Pre-Layout | Post-Layout | Result |
|---|---:|---:|---|
| VREF at TT, 27 °C | 1.2000 V | 1.2001 V | Recentered after extraction |
| TT temperature coefficient | 2.66 ppm/°C | 2.55 ppm/°C | Restored by `R4` retuning |
| Line regulation | 15.51 mV/V | 15.42 mV/V | Preserved |
| Supply current | 87.16 µA | 85.17 µA | Slightly reduced |
| Low-frequency PSRR | -36.88 dB | -36.9861 dB | Preserved |
| MC process sigma | 1.96 mV | 1.935 mV | Nearly unchanged |
| MC mismatch sigma | 445.86 µV | 545.19 µV | Increased after extraction |
| Startup | Pass | Pass | Preserved |

The final extracted circuit retains the pre-layout performance after a deliberate two-step retuning procedure: `R4` was used for temperature-coefficient recovery and `R7` for nominal-voltage centering.

---

## 7. Future Improvements and References

Future revisions can add independent trim networks for the PTAT path and output resistor, allowing post-silicon calibration of temperature coefficient and nominal VREF. Other useful extensions include curvature compensation, output-noise characterization, improved high-temperature PSRR, lower-power biasing, and full tapeout preparation with pads, ESD protection, density fill, seal ring, and silicon measurement planning.

### References

1. H. Banba et al., “A CMOS Bandgap Reference Circuit with Sub-1-V Operation,” *IEEE Journal of Solid-State Circuits*, vol. 34, no. 5, 1999.
2. B. Razavi, “The Design of a Low-Voltage Bandgap Reference,” *IEEE Solid-State Circuits Magazine*, Summer 2021.
3. B. Razavi, *Design of Analog CMOS Integrated Circuits*, McGraw-Hill.
4. P. R. Gray, P. J. Hurst, S. H. Lewis, and R. G. Meyer, *Analysis and Design of Analog Integrated Circuits*, Wiley.
