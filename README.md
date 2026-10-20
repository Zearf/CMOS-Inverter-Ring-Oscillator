# CMOS Inverter & Ring Oscillator — From Schematic to Layout

**Transistor-level CMOS design, physical verification, and post-layout performance analysis in Cadence Virtuoso.**

I built and characterized a CMOS inverter, implemented it as a reusable physical layout, and connected five inverter stages into a ring oscillator. The project explores how transistor sizing, input slew, loading, and layout parasitics influence switching behavior—and how reducing layout area can trade off against speed.

## Highlights

- **Transistor sizing:** Tuned the PMOS/NMOS width ratio for a switching threshold near half the 2.5 V supply.
- **Circuit characterization:** Measured VTCs, noise margins, and fanout-of-one (FO1) propagation delays; explored switching at 500 MHz, 5 GHz, and 50 GHz.
- **Physical design:** Created inverter and oscillator layouts, verified them with **DRC/LVS**, and extracted parasitic capacitances for post-layout simulation.
- **Ring oscillator:** Built a self-oscillating, five-stage feedback loop and measured the effect of extraction on its frequency.
- **Area optimization:** Redesigned the inverter's body-contact placement and supply routing to reduce its footprint, then evaluated the timing tradeoff.

## Results

| Measurement | Result |
|---|---:|
| Inverter switching threshold | **1.247 V** (2.5 V supply) |
| Inverter FO1 delay, schematic | **61.65 ps** |
| Inverter FO1 delay, extracted | **60.07 ps** |
| Five-stage oscillator, schematic | **2.026 GHz** |
| Five-stage oscillator, extracted | **1.829 GHz** (**9.72%** lower) |
| Inverter area, original → optimized | **29.664 → 21.600 µm²** (**27.18%** reduction) |
| Optimized inverter FO1 delay | **62.34 ps** (**3.78%** slower than original extracted layout) |

The extracted oscillator ran slower than its schematic counterpart, consistent with added layout capacitance increasing effective stage delay. The compact inverter layout achieved a meaningful area reduction while slightly increasing propagation delay—an example of the **area–performance tradeoff** in physical design.

## Tools & technology

- **EDA:** Cadence Virtuoso, Spectre, Diva (DRC, extraction, LVS)
- **Technology:** NCSU CDK / TSMC 0.25 µm teaching-process setup
- **Device models:** `tsmc25dN` and `tsmc25dP`
- **Operating voltage:** 2.5 V
- **Final device sizes:** NMOS `W/L = 0.72/0.24 µm`; PMOS `W/L = 1.74/0.24 µm`
- **Post-layout simulation:** `extracted_c` views containing extracted parasitic capacitances

## Repository guide

```text
.
├── design_source/                 # Cadence OpenAccess design library
│   ├── inverter/                  # CMOS inverter: schematic, symbol, layout, extraction
│   ├── inverter_opt/              # Area-optimized inverter and extracted views
│   ├── oscillator/                # Five-stage ring oscillator and extracted views
│   ├── inverter_tb*/              # DC, transient, FO1, and layout-comparison testbenches
│   └── oscillator_tb*/            # Schematic / extracted oscillator testbenches
├── ade_setting/                           # Saved Spectre ADE states, organized by testbench
└── README.md
```

The design cells include `schematic`, `symbol`, `layout`, `extracted`, and `extracted_c` views. Relevant testbenches also have `config` views to select schematic or extracted implementations. The separate **`119/` directory contains saved ADE settings** (analyses, model setup, variables, and outputs) to help reproduce the simulations without rebuilding each setup manually.

## Reproducing the simulations

1. **Set up the environment.** Launch Cadence Virtuoso with the NCSU CDK and compatible TSMC 0.25 µm models installed. The CDK/device models and Cadence licenses are **not included** in this repository.
2. **Register the library.** Point your Cadence `cds.lib` to the `design_source/` directory, using the library name **`119`** (used by the saved configuration views):
   ```text
   DEFINE 119 /absolute/path/to/design_source
   ```
   Preserve the existing CDK technology-library definitions.
3. **Open a testbench.** In Library Manager, browse the `inverter_tb*` cells for inverter analyses or `oscillator_tb*` for ring-oscillator simulations. Use the `config` views for extracted-circuit comparisons.
4. **Load saved simulation settings.** In ADE L, select **Session → Load State** and load the corresponding Spectre state supplied under `119/<testbench>/spectre/<state>/`. Verify the model paths, analysis settings, and selected `extracted_c` bindings—saved paths may reflect the original Cadence installation.
5. **Run and compare.** Simulate VTC/DC sweeps, FO1 transients, or the ring oscillator. The ring oscillator uses an initial condition on its output node (`n5/out = 0 V`) to initiate oscillation; measure the frequency after startup settles.

For layout verification, the NCSU CDK Diva rule files are normally located under `$CDK_DIR/techfile/` (`divaDRC.rul`, `divaEXT.rul`, `divaLVS.rul`). Verify DRC and LVS before using `extracted_c` for post-layout measurements.

## Takeaways

This project reinforced the complete **schematic → layout → DRC/LVS → parasitic extraction → simulation** workflow, as well as the importance of measuring real physical effects rather than relying on idealized gate delays alone.
