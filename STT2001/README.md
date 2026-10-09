# STT2001: Streamline Tracing Tool

Part of the [Three-Dimensional Nozzle Design Code](../README.md) (modified edition of NASA LEW-20180).

STT2001 builds **non-axisymmetric (3D) nozzle shapes** by streamline tracing through a known flowfield, usually the axisymmetric solution produced by [MOC_Grid_BDE](../MOC_Grid_BDE/README.md). The tool works as follows:

1. You define an inlet or exit cross-section shape: circles or arcs, up to five segments.
2. It traces streamlines from every point of that shape through the solution grid. Because each streamline is a stream surface element of the original flow, the resulting surface is an inviscid wall.
3. It trims the traced streamlines to the chosen shape and length.
4. It integrates the surface to give the new nozzle's **thrust, Isp, Cfg, exit area, area ratio, surface area and friction force**.

The result is a 3D nozzle that keeps the flow quality of the parent design. The traced surface can then be analysed or re-meshed, and [3D_MOC](../3D_MOC/README.md) can be used to check modified geometries.

Full theory: Rice (2003), JHU/APL RTDC-TPS-481.

---

## Running the program

Run `STT2001.exe`. The **Streamline Tracing Tool 2001** dialog opens. Output files are written to the current working folder.

You can enter inputs by hand, or load a saved case with **File > Open** (`*.inp`). **File > Save / Save As** stores the current inputs.

### Inputs (dialog)

**File Input**

| Field | Meaning |
|---|---|
| Streamline Data File | Streamlines from MOC_Grid_BDE (`MOC_SL.plt`) |
| MOC Data File | Characteristic grid from MOC_Grid_BDE (`MOC_Grid.plt`) |
| MOC Summary File | `summary.out` from MOC_Grid_BDE |
| Friction Data File | Two-column table of friction data (see `friction_table.txt`). The first line is the number of rows. |
| Output Files Prefix | Prefix for every output file, e.g. `M3.5Perf` |

**Other Inputs**: Ambient pressure (psia), Throat area, Ideal Cfg Isp, Mass flow. These are usually taken from the MOC summary.

**Center of Known SL Flowfield**: the start, end and step of the streamline-field origin in *X, Y, Z* and *R*. Use this to position and scale the parent flowfield relative to the traced shape.

**Streamline Trimming Definition** (up to 5 rows, each enabled by its own checkbox). Each row defines one circular-arc segment of the trimming shape:

| Column | Meaning |
|---|---|
| Yc, Zc, Rc | Centre and radius of the arc in the cross-plane |
| Start / End Angle | Angular extent of the arc (deg) |
| # of SLs | Number of streamlines traced along this segment |
| X Start / X End | Axial range over which this segment trims |
| Throat? | Segment defines the throat (capture) shape |
| Surface | Trimming surface type (drop-down) |

Further controls:

- **Max X0** with a checkbox: maximum nozzle length.
- **Grid Scale Factor**.
- **Crop nozzle due to SL trimming**.
- **Symmetry Definition...** opens a dialog for the number of symmetric occurrences per 360°, the line of symmetry (Y0, Z0, radius along the X-axis), and the two streamlines matched for the rotation calculation.

Press **Execute** to run. Progress is shown under *Execution Status*.

### Outputs

The **Nozzle Parameter Output** panel shows throat stream thrust, ∫P dA, friction force, P<sub>exit</sub> force, calculated Isp, Cfg, surface area, axial projected area, exit area, area ratio, max length and min/max Y. **Plots** opens centreline plots.

Files written, where `<prefix>` is the output files prefix:

| File | Contents |
|---|---|
| `<prefix>.plt` | Tecplot: traced nozzle surface |
| `<prefix>_Engine.plt` | Tecplot: engine outline |
| `<prefix>_STT_summary.out` | Performance summary |
| `<prefix>_ThroatSL.out`, `<prefix>_ThroatSummary.out` | Streamlines and properties from the throat definition |
| `<prefix>_TrimSL.out` | Trimmed streamlines |
| `<prefix>_trimmed_P3D.xyz`, `<prefix>_trimmed_P3D.dat`, `<prefix>_end_P3D.xyz` | PLOT3D surface grid and data (for CFD meshing or plotting) |
| `<prefix>_AvsX.out`, `<prefix>_AvsSL.out` | Area distribution vs axial position and vs streamline |
| `<prefix>_cl.dat` | Centreline data |
| `<prefix>_all_runs.dat` | Appended one-line summary of each run (useful for parameter sweeps) |

---

## Sample case: `outputs_M3.5Perf/`

`M3.5Perf.inp` traces the Mach 3.5 perfect nozzle from `../MOC_Grid_BDE/outputs_M3.5Perf/`. To reproduce it:

1. Copy `MOC_SL.plt`, `MOC_Grid.plt` and `summary.out` (included in the folder), `friction_table.txt` and `M3.5Perf.inp` into a folder together with `STT2001.exe`.
2. Run `STT2001.exe`, choose **File > Open**, select `M3.5Perf.inp`, then press **Execute**.

These outputs come from the original NASA code and the original (uncorrected) MOC grid.

---

## Building from source

Requirements: Visual Studio 2022+ with *Desktop development with C++* and **C++ MFC for latest build tools (x86 & x64)**.

1. Open `STT2001_solution.sln`. Accept the retarget prompt if one appears.
2. Choose **Release | x64** and run **Build > Rebuild Solution**.
3. The output is `x64\Release\STT2001_solution.exe`, a standalone exe. Rename it to `STT2001.exe` if you want MOC_Grid_BDE's STT button to find it.

`STT2001.dsw` is the original Visual C++ 6 workspace, kept for reference. It is not needed.

---

## Source files

| File | Role |
|---|---|
| `STT2001Dlg.cpp/.h` | Main dialog, tracing, trimming, integration and all output |
| `Vector.cpp/.hpp`, `Matrix.cpp/.hpp` | 3D vector and matrix utilities |
| `Chart.cpp/.h`, `Chart3d.cpp/.h`, `PlotDialog.cpp/.h` | Built-in 2D/3D plots |
| `m_SymDef_DIALOG.cpp`, `m_symdef_dialog.h` | Symmetry definition dialog |
| `STT2001.cpp/.h` | MFC application |
| `friction_table.txt` | Example friction input table |

## Reference

Rice, T. (2003). *2D and 3D Method of Characteristic Tools for Complex Nozzle Development*. JHU/APL RTDC-TPS-481. [NTRS 20030067852](https://ntrs.nasa.gov/citations/20030067852)

License: NASA Open Source Agreement v1.3. See [../license.txt](../license.txt) and [../NOTICE.md](../NOTICE.md).
