# High-Precision Bandgap Reference (BGR) in TSMC 65nm

## Overview
This repository contains the design, pre-layout, and post-layout simulations of a CMOS Bandgap Reference (BGR) circuit implemented in TSMC 65nm technology. The circuit is designed to provide a stable, temperature-independent 1.2V reference voltage across variations in temperature, supply voltage, and process corners.

> **Status:** Pre-layout simulations are complete. Layout design, physical verification, and post-layout simulations are currently in progress.

## Key Performance Specifications
| Parameter | Target / Pre-Layout | Post-Layout (PEX) | Conditions |
| :--- | :--- | :--- | :--- |
| **Technology** | TSMC 65nm | TSMC 65nm | - |
| **Supply Voltage (VDD)** | 2.5 V | 2.5 V | Nominal |
| **Output Voltage (Vout)** | 1.2 V | *TBA* | Nominal |
| **Temperature Coefficient**| 2.66 ppm/°C | *TBA* | -40°C to 125°C |
| **Line Regulation** | 15.51 mV/V | *TBA* | VDD = 2.0V to 3.0V |
| **Power Consumption** | ~217.9 µW | *TBA* | TT corner, 27°C |
| **PSRR** | -36.88 dB | *TBA* | @ 2.45 Hz |

## Tools Used
*   **Schematic & Simulation:** Cadence Virtuoso, Spectre
*   **Physical Verification:** Calibre (DRC, LVS, PEX)

## 1. Pre-Layout Simulation Results

### DC & Temperature Sweep
*   **Temp. Sweep:** The circuit achieves a nominal output of 1.2V with a highly stable curve over the -40°C to 125°C range, resulting in a minimum TC of 2.66 ppm/°C.
*   **Line Regulation:** Evaluated from VDD 2.0V to 3.0V, showing a maximum variation of 15.51 mV/V.

### Transient Analysis (Startup)
Simulations were performed with various supply ramp times to ensure proper and robust startup behavior without significant ringing (1 µs, 100 µs, and 1 ms supply ramps).

### Robustness Verification
*   **Corner Simulation (PVT):** Vout variations evaluated across typical (TT) and extreme (FF, SS, FS, SF) process corners over temperature sweeps and transient startup conditions.
*   **Monte Carlo Analysis (1000 Samples):**
    *   **Process Variation:** $\mu  pprox 1.20021V$, $\sigma  pprox 1.96mV$.
    *   **Mismatch:** $\mu  pprox 1.20005V$, $\sigma  pprox 445.86\mu V$.

## 2. Layout & Physical Verification (Work in Progress)
*   **Layout Area:** *TBA* $\mu m^2$
*   **DRC (Design Rule Check):** *Status TBA*
*   **LVS (Layout Versus Schematic):** *Status TBA*
*(Images of the final layout will be added here once completed).*

## 3. Post-Layout Simulation Results (Work in Progress)
*Parasitic Extraction (PEX) results will be documented here to compare the impact of routing resistances and capacitances on the temperature coefficient, startup behavior, and PSRR.*

## Future Work
*   **Trimming Circuitry:** Implement a resistor trimming network (e.g., using a multi-bit digital-to-analog interface) to calibrate the absolute output voltage and mitigate process variations post-fabrication.
*   **Tapeout Preparation:** Incorporate dummy devices, optimize matching techniques (e.g., common-centroid layout for current mirrors), and add seal rings for manufacturability.
*   **PSRR Enhancement:** Investigate advanced Op-Amp topologies or integrate a pre-regulator/LDO to improve the Power Supply Rejection Ratio.

## Author
*   **Reynaldo Nicholas Sianturi**

## References
1. Prelayout Simulation Bandgap Reference TSMC65NM.
2. B. Razavi, *Design of Analog CMOS Integrated Circuits*, McGraw-Hill Education.
3. P. R. Gray, P. J. Hurst, S. H. Lewis, and R. G. Meyer, *Analysis and Design of Analog Integrated Circuits*, John Wiley & Sons.
